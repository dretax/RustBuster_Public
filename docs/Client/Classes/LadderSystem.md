### Class
`RustBuster2016.API.LadderSystem` (+ `RustBuster2016.API.LadderMonoBehaviour`)

### Description
Optional, **disabled by default** client-side ladder-climbing system. Mirrors
[`WaterSystem`](WaterSystem.md)'s lifecycle exactly: the driver only runs while the local player is
alive and spawned, and shuts itself down after too many consecutive failures.

**Discovery.** Plugins mark climbable objects by attaching a class derived from
`LadderMonoBehaviour` to the relevant `GameObject`. `LadderSystem` discovers every instance with
`FindObjectsOfType<LadderMonoBehaviour>()` (only currently-active objects in a loaded scene), additionally
requiring an active `Renderer` on the object or its children so invisible trigger-only objects are
excluded from the cached bounds.

**Movement.** CS:GO style, driven by where the camera is pointing:
- Camera forward Y > `+LookThreshold` → climb up at `ClimbSpeed`.
- Camera forward Y < `-LookThreshold` → slide down at `ClimbSpeed`.
- Otherwise → hang in place (vertical velocity zeroed).

Horizontal velocity is left untouched so the player can still push away with WASD; leaving a ladder lets
the `CCMotor` resume gravity on its own.

### `LadderState` enum
- `None = 0` - not touching any ladder.
- `OnLadder = 1` - character bounds overlap a ladder collider.

### `LadderMonoBehaviour` (abstract class)
Base class for ladder objects. Minimal usage:
```csharp
public class MyLadder : LadderMonoBehaviour { }
```
Attach `MyLadder` to any `GameObject` (with a `Renderer`, even a simple trigger mesh) that should be
climbable.
- `virtual void OnLadderEnter()` - called when the local player starts climbing this specific ladder.
  Override for sounds/particles/UI prompts.
- `virtual void OnLadderExit()` - called when the local player leaves this specific ladder.

### Enabling it
```csharp
public override void Initialize()
{
    LadderSystem.Enabled = true;
}
```

### Tunables (all `public static`)
- `bool Enabled` (false) - master switch.
- `float ClimbSpeed` (4) - climb/descend speed, m/s.
- `float LookThreshold` (0.25) - camera forward-Y threshold (0 to 1) that triggers movement; `0.25` ≈
  looking ~15° above/below the horizon.
- `float DetachMargin` (0.15) - how far the character's centre must be outside the ladder bounds before
  counting as off the ladder (prevents edge chatter).
- `float OverlapPadding` (0.3) - extra vertical padding added to the character capsule when testing
  overlap; increase if the player pops off before reaching the top.
- `float AliveTimeout` (3) - seconds without a heartbeat before the driver shuts down.
- `int MaxConsecutiveFailures` (15) - consecutive tick failures before the system disables itself for
  the session.
- `float RebuildInterval` (20) - seconds between rescans while the ladder list is empty.
- `bool DebugLog` (false) - logs state changes and rebuild results.

### Read-only state
- `static LadderState State { get; }`, `static bool IsActive { get; }` (driver running),
  `static bool IsOnLadder { get; }`, `static int LadderCount { get; }` (cached ladder volumes for the
  current level).

### Members
#### `static void RebuildLadders()`
Rescans the active scene for `LadderMonoBehaviour` instances and rebuilds the cached bounds list. Runs
automatically on start, on level change, and on a timer while empty - call it yourself after
programmatically spawning new ladder objects at runtime.

### Related hooks
`Hooks.OnRustBusterLadderStateChanged` - see the [Hooks reference](../Hooks/README.md).

### Example

```csharp
public class MyLadder : LadderMonoBehaviour
{
    public override void OnLadderEnter()
    {
        Hooks.LogData("MyFirstPlugin", "Player started climbing.");
    }
}

public override void Initialize()
{
    LadderSystem.Enabled = true;
    Hooks.OnRustBusterLadderStateChanged += OnLadderStateChanged;
}

public override void DeInitialize()
{
    Hooks.OnRustBusterLadderStateChanged -= OnLadderStateChanged;
}

private void OnLadderStateChanged(LadderState previous, LadderState current)
{
    Hooks.LogData(Name, "Ladder state: " + previous + " -> " + current);
}
```
