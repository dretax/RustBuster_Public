### Class
`RustBuster2016.API.Util`

### Description
A grab-bag of static helpers: raycasting/line-of-sight, terrain queries, reflection shortcuts, item/data
block lookups, and the main-thread ("Normal") timer API (the background-thread equivalent lives on
`Util` too - see [`Timers.md`](Timers.md) for the full picture).

### Raycasting / look helpers

#### `static GameObject GetLineObject(Vector3 start, Vector3 end, out Vector3 point, int layerMask = -1)`
Returns the object hit by a line cast between two points, and the exact hit point.

#### `static GameObject GetLookObject(Character character, int layerMask = -1)`
Returns the object the given character is currently looking at (eye position + look direction).

#### `static GameObject GetLookObject(Ray ray, float distance = 300f, int layerMask = -1)`
#### `static GameObject GetLookObject(Ray ray, out Vector3 point, float distance = 300f, int layerMask = -1)`
Lower-level overloads operating directly on a `Ray`.

#### `static Ray GetLookRay(Character character)`
The look ray (eye position + eye direction) for a character.

```csharp
GameObject looked = Util.GetLookObject(LocalAttachment.LocalCharacter);
```

### Terrain helpers

#### `static float GetGround(float x, float z)` / `static float GetGround(Vector3 target)`
Terrain height at the given X/Z coordinates.

#### `static float GetTerrainHeight(Vector3 target)` / `static float GetTerrainHeight(float x, float y, float z)`
#### `static float GetTerrainSteepness(Vector3 target)` / `static float GetTerrainSteepness(float x, float z)`
#### `static float GetGroundDist(float x, float y, float z)` / `static float GetGroundDist(Vector3 target)`
Distance from a point down to the terrain.

#### `static List<TreeInstance> GetAllTreeInstances()`
Every tree instance on the currently loaded terrain.

### Datablock helpers

#### `static ItemDataBlock ConvertNameToData(string name)`
Looks up an `ItemDataBlock` by its display name (via the game's `DatablockDictionary`). Returns `null`
if not found.

#### `static BlueprintDataBlock BlueprintOfItem(ItemDataBlock item)`
Finds the blueprint whose `resultItem` is `item`, or `null`.

#### `static readonly string[] UStackable`
All item names in Rust Legacy that are *unstackable* (i.e. max stack size 1) - weapons, tools, clothing,
and a handful of deployables.

### Reflection helpers
Thin, exception-swallowing wrappers over `System.Reflection`, useful for reaching into the game's own
non-public members. Works for static members too (pass `null` as `instance`).

#### `static object GetInstanceField(Type type, object instance, string fieldName)`
#### `static void SetInstanceField(Type type, object instance, string fieldName, object val)`
#### `static object GetInstanceProperty(Type type, object instance, string propertyName)`
#### `static bool SetInstanceProperty(Type type, object instance, string propertyName, object val)`

```csharp
object privateValue = Util.GetInstanceField(typeof(SomeGameClass), someInstance, "_privateField");
```

### Normal (main-thread) timers
See [`Timers.md`](Timers.md) for the full reference, including the background-thread System Timer
counterpart.

#### `static TimedEvent CreateTimer(string name, int timeoutDelay, Action<TimedEvent> callback, bool autoReset = false, string pluginName = "", int maxElapsedCount = 0)`
Creates (or returns the existing) named main-thread timer. `timeoutDelay` is in milliseconds.

#### `static TimedEvent CreateParallelTimer(string name, int timeoutDelay, Dictionary<string, object> args, Action<TimedEvent> callback, bool autoReset = false, string pluginName = "", int maxElapsedCount = 0)`
Like `CreateTimer`, but multiple timers can share the same `name`; `args` lets you pass per-instance data
read back via `TimedEvent.Args`.

#### `static TimedEvent GetTimer(string name)` / `static void KillTimer(string name)`
#### `static List<TimedEvent> GetParallelTimer(string name)` / `static void KillParallelTimer(string name)`
#### `static void KillTimers()`
Lookup/teardown helpers. `KillTimers()` stops everything (both named and parallel timers).

### Misc
#### `static ulong GetTickCount64()`
P/Invoke wrapper over `kernel32!GetTickCount64`.
