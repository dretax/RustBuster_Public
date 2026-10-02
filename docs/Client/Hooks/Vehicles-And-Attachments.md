### Category
Vehicles-And-Attachments hooks

### Description
[`LocalAttachment`](../Classes/LocalAttachment.md) seat mount/dismount, [`WaterSystem`](../Classes/WaterSystem.md)
swimming/oxygen, and [`LadderSystem`](../Classes/LadderSystem.md) climbing. See
[Hooks/README.md](README.md) for the full table and the general cancellation rules.

---

### `OnRustBusterLocalPlayerAttach`
`public delegate void RustBusterLocalPlayerAttachDelegate(LocalPlayerAttachEvent ae)`

Runs right before the local player is attached to an anchor (vehicle seat, mounted turret, chair).
**Cancellable** (blocks the mount).
- `ae.Owner` - the plugin name that requested the attachment.
- `ae.Anchor` - the `Transform` about to be attached to.
- `ae.UserData` - whatever the requester put in `AttachRequest.UserData`.

### `OnRustBusterLocalPlayerAttached`
`public delegate void RustBusterLocalPlayerAttachedDelegate(LocalPlayerAttachedEvent ae)`

Runs after the local player has been attached and every request flag applied. Not cancellable - call
`LocalAttachment.ForceDetach` if you need to undo it.
- `ae.AttachmentHandle` - the live handle; its `Flags`/`LocalOffset`/look limits can still be mutated.
- `ae.Owner`, `ae.UserData`.

### `OnRustBusterLocalPlayerDetach`
`public delegate void RustBusterLocalPlayerDetachDelegate(LocalPlayerDetachEvent de)`

Runs right before the local player is detached from an anchor. **Only cancellable when
`de.DetachReason == DetachReason.Requested`** - `Cancel()` silently does nothing for every other reason
(the detach already happened or cannot be refused without leaving the player stuck).
- `de.AttachmentHandle` - the handle about to be invalidated.
- `de.DetachReason` - **check this before calling `Cancel()`.** See `DetachReason` in
  [`LocalAttachment.md`](../Classes/LocalAttachment.md#detachreason-enum-rustbuster2016apievents) for
  every value.
- `de.Owner`, `de.UserData`.

```csharp
public void HandleDetach(LocalPlayerDetachEvent e)
{
    if (e.DetachReason == DetachReason.Requested && _playerIsTradingInSeat)
    {
        e.Cancel(); // Keep the player seated until the trade finishes.
    }
}
```

### `OnRustBusterLocalPlayerDetached`
`public delegate void RustBusterLocalPlayerDetachedDelegate(LocalPlayerDetachedEvent de)`

Runs after the local player has been detached and every saved value restored. Not cancellable - the
handle is already invalid, only the owner name/user data/reason are handed back.
- `de.Owner`, `de.UserData`, `de.DetachReason`.

### `OnRustBusterWaterStateChanged`
`public delegate void RustBusterWaterStateChangedDelegate(WaterStateChangedEvent we)`

Runs when the local player moves between dry land, the water surface and being fully submerged
(requires [`WaterSystem.Enabled`](../Classes/WaterSystem.md) to be `true`). Not cancellable.
- `we.Previous`, `we.Current` - `WaterState` values (`None`/`Wading`/`Submerged`).
- `we.Entered` - `true` if the player just entered the water from dry land.
- `we.Exited` - `true` if the player just left the water entirely.

### `OnRustBusterOxygenChanged`
`public delegate void RustBusterOxygenChangedDelegate(OxygenChangedEvent oe)`

Runs as the local player's air changes while swimming, throttled by `WaterSystem.OxygenEventStep` rather
than every frame. Not cancellable.
- `oe.Oxygen01` - remaining air, 0 to 1 (drive a UI bar with this).
- `oe.SecondsRemaining` - roughly how many seconds of air are left at the current drain rate.
- `oe.WaterState` - the state at the moment the air was sampled; air only drains while `Submerged`.

### `OnRustBusterOutOfAir`
`public delegate void RustBusterOutOfAirDelegate(OutOfAirEvent oe)`

Runs once when the local player's air hits zero, and not again until it refills. The client cannot apply
damage (the server owns health) - use this to send your own drown message to your server-side plugin
half. Not cancellable.
- `oe.SubmergedSeconds` - how long the player had been continuously submerged when air ran out.

### `OnRustBusterLadderStateChanged`
`public delegate void RustBusterLadderStateChangedDelegate(LadderState previous, LadderState current)`

Runs when the local player grabs or releases a ladder (requires
[`LadderSystem.Enabled`](../Classes/LadderSystem.md) to be `true`). Not cancellable. Both parameters are
`LadderState` values (`None`/`OnLadder`) passed directly, there is no event-argument class.
