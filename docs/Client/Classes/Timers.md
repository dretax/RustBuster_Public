### Classes
`RustBuster2016.API.Events.TimedEvent` (main thread) and
`RustBuster2016.API.Events.SystemTimerEvent` (background thread)

### Description
RustBuster ships two timer flavours. Picking the wrong one is a common source of crashes, so the
difference matters:

| | `TimedEvent` ("Normal Timer") | `SystemTimerEvent` ("System Timer") |
|---|---|---|
| Backed by | A `MonoBehaviour` + Unity coroutine | `System.Timers.Timer` |
| Fires on | The Unity **main thread** | A **background thread pool thread** |
| Safe to touch `GameObject`/Unity APIs? | Yes | **No** - will crash the client |
| Created via | `Util.CreateTimer` / `Util.CreateParallelTimer` | `Util.CreateSystemTimer` / `Util.CreateParallelSystemTimer` |

Use `TimedEvent` for anything that touches game state. Use `SystemTimerEvent` only for pure
background work (e.g. computing something, then hopping back to the main thread via
[`Loom.QueueOnMainThread`](Loom.md#static-void-queueonmainthreadaction-action) before touching Unity
objects).

### `TimedEvent` members
- `event TimedEventFireDelegate OnFire` - fired every time the timer elapses.
- `event Action<string> OnKilled` - fired once, when the timer is killed/disposed.
- `string Name`, `string PluginName`, `double Interval` (ms), `bool AutoReset`, `int MaxElapsedCount`
  (`0` = infinite), `Dictionary<string, object> Args`.
- `void Start()` / `void Stop()` / `void Kill()` - `Kill()` stops and destroys the underlying
  `GameObject`.
- `bool IsRunning`, `bool IsKilled`, `int ElapsedCount`, `double TimeLeft`, `long LastTick`,
  `DateTime LastTickDate`.

### `SystemTimerEvent` members
Same shape as `TimedEvent`: `event SystemTimerFireDelegate OnFire`, `event Action<string> OnKilled`,
`Name`/`PluginName`/`Interval`/`AutoReset`/`MaxElapsedCount`/`Args`, and `Start()`/`Stop()`/`Kill()`.
Additionally implements `IDisposable`.

### Creating timers
Prefer going through [`Util`](Util.md) rather than constructing `TimedEvent`/`SystemTimerEvent`
directly - it also de-duplicates named timers and tracks them for you.

```csharp
public override void Initialize()
{
    // Main-thread timer: fires every 10 seconds, safe to touch GameObjects.
    Util.CreateTimer("announce", 10000, OnAnnounce, autoReset: true, pluginName: Name);

    // Background timer: fires once after 2 seconds, do NOT touch Unity objects in the callback.
    Util.CreateSystemTimer("delayedCompute", 2000, OnComputeDone, pluginName: Name);
}

public override void DeInitialize()
{
    Util.KillTimer("announce");
    Util.KillSystemTimer("delayedCompute");
}

private void OnAnnounce(RustBuster2016.API.Events.TimedEvent timer)
{
    Hooks.LogData(Name, "Still running. Elapsed count: " + timer.ElapsedCount);
}

private void OnComputeDone(RustBuster2016.API.Events.SystemTimerEvent timer)
{
    int result = DoExpensiveComputation();

    // Hop back to the main thread before touching anything Unity-related.
    Loom.QueueOnMainThread(() => Hooks.LogData(Name, "Result: " + result));
}
```

### See also
- [`Util.md`](Util.md) - `CreateTimer`/`CreateParallelTimer`/`CreateSystemTimer`/`CreateParallelSystemTimer`
  and their lookup/kill counterparts.
- [`Loom.md`](Loom.md) - jumping back to the main thread from a `SystemTimerEvent` callback.
