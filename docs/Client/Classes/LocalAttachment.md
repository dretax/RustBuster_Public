### Class
`RustBuster2016.API.LocalAttachment` (+ `AttachRequest`, `AttachmentHandle`, `AttachFlags`,
`RustBuster2016.API.Events.DetachReason`)

### Description
Mounts the **local** player to a vehicle seat, turret, chair, or any other `Transform` ("anchor") - a
seat, basically. **Client side only**: it moves *your* player on *your* machine and tells nobody else.
The server half of your plugin owns who is allowed in the seat and must broadcast it - call `Attach`
when that message arrives, not straight out of your own "use" handler, or remote players will see the
driver standing in a field. The position RustBuster writes replicates normally, since
`HumanController.SendToServer` sends `Character.origin` on its own schedule without checking
`lockMovement`.

### `AttachFlags` enum (`[Flags]`)
Behaviour toggles for an attachment - every one is optional, anything you leave off you are free to
drive yourself from plugin code:
- `None = 0` - RustBuster only tracks the attachment and runs its watchdog.
- `FollowAnchor` - writes the character transform from the anchor every `FixedUpdate`/`LateUpdate`.
- `LockMovement` - sets `Character.lockMovement`; keep this on whenever `FollowAnchor` is on.
- `LockLook` - sets `Character.lockLook`, freezing the view completely.
- `ClampLook` - clamps yaw/pitch relative to the anchor using the request's limits.
- `FreezeMotor` - disables the `CCMotor`; re-enabled and teleported to the dismount point on detach.
- `IgnoreAnchorCollision` - ignores collision between the player capsule and the anchor's root colliders.
- `SuppressFallDamage` - zeroes `CCMotor.velocity` on detach and starts the fall-grace window.
- `PlaceOnDetach` - sweeps for a clear spot beside the anchor on detach instead of leaving the player
  inside the vehicle.
- `Default = FollowAnchor | LockMovement | ClampLook | FreezeMotor | IgnoreAnchorCollision | SuppressFallDamage | PlaceOnDetach`
  - everything a normal vehicle seat wants.

### `AttachRequest` class
Describes an attachment you want set up. Only `Anchor` is required; every other property has a sane
seat default.
- `string Owner` - your plugin name (same string as `Hooks.LogData`); used for logging and for
  auto-detaching when your plugin unloads.
- `Transform Anchor` - **required.** The seat transform on the vehicle.
- `AttachFlags Flags` - defaults to `AttachFlags.Default`.
- `Vector3 LocalOffset` - offset from the anchor, in anchor local space.
- `float YawLimit` (default `120`), `float PitchMin` (default `-70`), `float PitchMax` (default `70`) -
  degrees, used with `ClampLook`. A `YawLimit` of 180+ disables the yaw clamp.
- `int FallGraceMs` (default `1500`) - how long `LocalAttachment.IsInFallGrace` stays true after detach.
- `object UserData` - anything you want handed back on the handle and in detach events.

### `AttachmentHandle` class
Live handle to the current attachment. `Anchor`, `Flags`, `LocalOffset` and the look limits can all be
changed while it is valid - e.g. to unlock the view when a car stops, or slide the player to another
seat without a detach/reattach cycle.
- `string Owner`, `object UserData`, `float AttachedAt` (`Time.time` at attach).
- `bool IsValid` - `false` once torn down; a stale handle is inert.
- `Transform Anchor { get; set; }` - reassigning moves the player to the new anchor on the next tick
  (seat swapping). Collision ignores stay pointed at the *original* anchor's root.
- `AttachFlags Flags { get; set; }` - toggling `FreezeMotor`/`IgnoreAnchorCollision` is **not** applied
  retroactively; detach and reattach for those two.
- `Vector3 LocalOffset { get; set; }`, `float YawLimit/PitchMin/PitchMax { get; set; }`,
  `int FallGraceMs { get; set; }`.
- `bool Detach()` - detaches with `DetachReason.Requested`. Idempotent, never throws. Returns `false` if
  the handle is stale or a plugin cancelled the detach.

### `LocalAttachment` static members

#### `static int DismountMask` / `static float DismountSearchRadius` / `static float DismountCapsuleRadius` / `static float DismountCapsuleHeight`
Tunables for the `PlaceOnDetach` dismount sweep.

#### `static AttachmentHandle Current { get; }`
The current attachment, or `null`.

#### `static bool IsAttached { get; }`
`true` while the local player is attached to something.

#### `static bool IsInFallGrace { get; }`
`true` during the post-detach fall-grace window. Purely informational - RustBuster does not suppress
fall damage for you.

#### `static Character LocalCharacter { get; }`
The local `Character`, or `null` if not connected/spawned.

#### `static AttachmentHandle Attach(AttachRequest request)`
Attaches the local player and returns a live handle, or `null` if the attach failed or a plugin
cancelled it (via `Hooks.OnRustBusterLocalPlayerAttach`). If something is already attached, it is
detached first with `DetachReason.Replaced`. **Must be called on the main thread** - wrap in
`Loom.QueueOnMainThread` otherwise; logs and returns `null` if called off-thread. Returns `null` and
logs if `request` or `request.Anchor` is `null`.

#### `static bool Detach(AttachmentHandle handle)`
Detaches, same as `handle.Detach()`.

#### `static bool ForceDetach(DetachReason reason)`
Detaches the current attachment (if any) with an arbitrary `reason`. RustBuster itself never raises
`DetachReason.Forced` - seeing it means a plugin ejected the player for its own purposes.

### `DetachReason` enum (`RustBuster2016.API.Events`)
Only `Requested` is cancellable (via `Hooks.OnRustBusterLocalPlayerDetach`); every other reason has
already happened and refusing it would leave the player stuck in a dead seat.
- `Requested = 1` - a plugin called `Detach()`.
- `Replaced = 2` - another `Attach()` took over.
- `AnchorDestroyed = 3` - the anchor transform was destroyed/deactivated.
- `LocalPlayerDied = 4` / `LocalPlayerGone = 5` - character died / went away.
- `Disconnected = 6` - the client left the server.
- `PluginUnloaded = 7` - the owning plugin was unloaded.
- `Forced = 8` - via `ForceDetach`.

### Related hooks
`Hooks.OnRustBusterLocalPlayerAttach` (cancellable), `Hooks.OnRustBusterLocalPlayerAttached`,
`Hooks.OnRustBusterLocalPlayerDetach` (cancellable only for `Requested`),
`Hooks.OnRustBusterLocalPlayerDetached` - see the [Hooks reference](../Hooks/README.md).

### Example

```csharp
private AttachmentHandle _seat;

// Call this once your server confirms the local player is allowed to sit down.
public void TakeSeat(Transform seatTransform)
{
    AttachRequest request = new AttachRequest
    {
        Owner = Name,
        Anchor = seatTransform,
        UserData = "driverSeat"
    };

    _seat = LocalAttachment.Attach(request);
    if (_seat == null)
    {
        Hooks.LogData(Name, "Could not attach to the seat.");
    }
}

public override void DeInitialize()
{
    // Make sure the player isn't left stuck in a seat if the plugin unloads.
    if (_seat != null && _seat.IsValid)
    {
        _seat.Detach();
    }
}
```
