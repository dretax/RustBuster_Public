### Class
`RustBuster2016.API.CustomMap` (+ `CustomMapHandle`, `CustomMapState`, `RustBuster2016.API.Events.CustomMapLoadedEvent`)

### Description
Lets **one** plugin replace the map of the current join with a scene from its own asset bundle, instead
of the server's own level.

### What you need to prepare
1. **The scene.** Build it in Unity 4.5.5f1, the editor version Rust Legacy runs on - bundles from any
   other version will not load. Put the game's `Assembly-CSharp.dll` and other managed DLLs in
   `Assets/Plugins` if your scene uses Rust components. The scene only needs to contain the world
   (terrain, water, props, spawners); RPOS and `LevelShared` are still loaded on top, exactly as for the
   stock map. If a scene named `"yourscene-TREES"` exists it is loaded automatically too.
2. **The bundle.** Build it as a streamed scene bundle for `StandaloneWindows`:
   ```csharp
   BuildPipeline.BuildStreamedSceneAssetBundle(
       new[] { "Assets/Scenes/hapis_island.unity" }, "hapis_island.unity3d",
       BuildTarget.StandaloneWindows, BuildOptions.UncompressedAssetBundle);
   ```
   The scene name you pass to `Commit` is the `.unity` file name without extension
   (`"hapis_island"`), not the bundle's file name. Uncompressed + `AssetBundle.CreateFromFile` is
   simplest (loads straight from disk); a compressed bundle is smaller but needs
   `File.ReadAllBytes` + `AssetBundle.CreateFromMemory`.
3. **Delivery.** Add the `.unity3d` file to the server's RustBuster downloadables (use a URL download
   for big maps). Every downloadable is already on disk by the time any plugin's `Initialize()` runs.
4. **The server.** The server must run the same map - collision, building placement, resource/loot
   spawns, animal and player spawn points are all simulated server-side.

### How the load works
The level load is held right before `Application.LoadLevelAsync` until every downloadable/plugin has
finished loading **and** (if a plugin claimed the map) that plugin has committed, abandoned, or failed.
Only then is the scene loaded - nothing of the stock map is ever loaded before that point.

### `CustomMapState` enum
`None` → `Pending` → (`Committed` → `Loading` → `Loaded`) | `Abandoned` | `Failed`.

### `CustomMap` static members
- `const float DefaultTimeoutSeconds = 120f`, `const float MaxTimeoutSeconds = 600f`.
- `static event CustomMapLoadedDelegate OnMapLoaded` - fires once the join's level has finished loading,
  for the stock map too (check `CustomMapLoadedEvent.IsCustomMap`). Subscribers are dropped at the end
  of every join; re-subscribe in each `Initialize()`.
- `static CustomMapState State`, `static bool IsClaimed`, `static bool CanClaim`, `static string Owner`,
  `static string ServerLevelName`, `static string SceneName`.
- `static CustomMapHandle Claim(string owner, float timeoutSeconds = DefaultTimeoutSeconds)` - claims the
  map for the current join. Returns `null` if another plugin already owns it or the level was already
  chosen. **Call from `Initialize()`.**

### `CustomMapHandle` members
- `string Owner`, `string AssemblyName`, `bool IsValid`, `CustomMapState State`, `string ServerLevelName`.
- `float Progress { get; set; }` - 0 to 1, drives the loading-screen bar while pending.
- `bool SetStatus(string text)` - loading-screen text while pending.
- `bool ExtendTimeout(float seconds)` - gives more time, counted from now, capped at `MaxTimeoutSeconds`.
- `bool Commit(AssetBundle bundle, string sceneName)` - loads `sceneName` from `bundle` instead of the
  server's level. **Main thread only.** RustBuster takes ownership of `bundle` - do not `Unload` it
  yourself.
- `bool Abandon(string reason)` - gives up, lets the server's level load.
- `bool Fail(string reason)` - the map cannot be loaded and the player must not join without it;
  disconnects the client.

### `CustomMapLoadedEvent` members
`string SceneName`, `string ServerLevelName` (differs from `SceneName` when replaced), `string Owner`
(`null` for the stock map), `AssetBundle Bundle` (`null` for the stock map, do not `Unload`),
`bool IsCustomMap`.

### Rules
- One map per join - the first `Claim` wins, every later one returns `null`.
- `Commit`, `Abandon` and `Fail` are final; `Pending` is the only state they can leave.
- `Commit` must be called on the main thread (it touches the `AssetBundle`).
- If none of `Commit`/`Abandon`/`Fail` is called within the timeout, the client disconnects.

### Example

```csharp
private CustomMapHandle _map;
private const string MapPath = "file:///C:/path/to/hapis_island.unity3d";

public override void Initialize()
{
    _map = CustomMap.Claim(Name, 180f);
    if (_map == null) return; // Another plugin owns the map, or it is too late.

    if (!File.Exists(MapPath))
    {
        _map.Fail("hapis_island.unity3d is missing");
        return;
    }

    _map.SetStatus("Loading Hapis Island");
    CustomMap.OnMapLoaded += OnMapLoaded;

    AssetBundle bundle = AssetBundle.CreateFromFile(MapPath);
    if (bundle == null)
    {
        _map.Fail("hapis_island.unity3d is corrupt or built with the wrong Unity version");
        return;
    }

    _map.Commit(bundle, "hapis_island");
}

private void OnMapLoaded(CustomMapLoadedEvent ev)
{
    if (!ev.IsCustomMap) return;

    // The scene is in. Find your objects, set up weather, and so on.
}
```
