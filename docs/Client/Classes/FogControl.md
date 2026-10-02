### Class
`RustBuster2016.API.FogControl` (+ `FogRequest`, `FogHandle`, `FogPriority`)

### Description
The single owner of `RenderSettings` fog. Time of Day writes `RenderSettings.fogColor` every frame,
RustBuster's own [`WaterSystem`](WaterSystem.md) takes fog over while the player is underwater, and
plugins may want fog for weather or events - whoever writes `RenderSettings` last wins, and nobody can
restore a value they never saw. Instead of a fragile snapshot/restore approach, fog is **composed from a
stack of priority-ordered layers**: releasing a layer doesn't restore a stale snapshot, it simply
recomposes from whatever layers remain, so effects always resolve correctly regardless of ordering.

Layers are sorted by `Priority` (low to high) and applied in order; every field on a layer is nullable,
so a layer only overrides what it actually sets and inherits the rest from the layers below it (or from
the real pre-RustBuster fog settings, captured once as the baseline). The underwater override used by
`WaterSystem` is just another layer, pushed at `FogPriority.Underwater`.

Time of Day only ever writes `fogColor`, and only while `TOD_Sky.World.SetFogColor` is true - whenever
any active layer specifies a `Color`, that flag is cleared for the duration (no IL patching needed) and
restored once no layer does.

### `FogPriority` (suggested constants - any `int` works)
- `Plugin = 0` - ambient/decorative fog, the default.
- `Weather = 100` - weather-driven fog, should beat decorative fog.
- `Event = 500` - an event/cutscene, should beat weather.
- `Underwater = 1000` - reserved for the underwater override; beats everything.

### `FogRequest` class
Describes a layer to push. Every field besides `Owner`/`Priority` is optional (nullable) - leave it
`null` to inherit whatever is underneath.
- `string Owner` (default `"Unknown"`) - your plugin name; used for logging and for automatically
  dropping the layer when your plugin unloads.
- `int Priority` (default `FogPriority.Plugin`).
- `bool? Enabled`, `FogMode? Mode`, `Color? Color` (setting this suppresses Time of Day's own colour
  writes while the layer is active), `float? StartDistance`, `float? EndDistance`, `float? Density`.
- `object UserData`.

### `FogHandle` class
Live handle to a pushed layer; mutate its fields directly to change it, no need to release and re-push.
- `string Owner`, `object UserData`, `bool IsValid` (`false` once released).
- `int Priority`, `bool? Enabled`, `FogMode? Mode`, `Color? Color`, `float? StartDistance`,
  `float? EndDistance`, `float? Density` - all public fields, mutable at any time.
- `bool Release()` - drops this layer. Idempotent, never throws.

### `FogControl` static members

#### `static FogHandle Push(FogRequest request)`
Adds a layer and returns a live handle. Returns `null` only if `request` is `null`.

#### `static bool Release(FogHandle handle)`
Drops a layer and recomposes. Returns `false` if the handle was already released or isn't a valid
handle. Never throws.

#### `static int ReleaseAllFor(string owner)`
Drops every layer belonging to `owner`. Called automatically when a plugin unloads, so a plugin that
forgets to clean up never leaves the world foggy. Returns the number of layers removed.

#### `static void ReleaseAll()`
Drops every layer and restores the fog exactly as it was before anything was pushed this session.

#### `static int LayerCount { get; }`
How many layers are currently active.

#### `static bool UnderwaterActive { get; }`
`true` while `WaterSystem`'s own underwater fog layer is active.

#### `static WaterState WaterState { get; }`
Mirrors `WaterSystem.State`, so a fog plugin can react to water without taking a dependency on
[`WaterSystem`](WaterSystem.md) directly.

### Example

```csharp
private FogHandle _fog;

public override void Initialize()
{
    _fog = FogControl.Push(new FogRequest
    {
        Owner = Name,
        Priority = FogPriority.Weather,
        Enabled = true,
        Mode = FogMode.Linear,
        Color = new Color(0.6f, 0.6f, 0.65f),
        StartDistance = 10f,
        EndDistance = 200f
    });
}

public void ThickenFog()
{
    // Mutate the handle directly, takes effect on the next frame.
    _fog.EndDistance = 120f;
}

public override void DeInitialize()
{
    _fog.Release();
}
```
