### Class
`RustBuster2016.API.AssetBundleLoader`

### Description
Shared, serialised asset-bundle loader for RustBuster plugins.

**Problem it solves.** When several plugins each call `WWW.LoadFromCacheOrDownload` independently they
all start at the same moment (end of the loading screen), producing a single large managed-heap spike
that coincides with Mono JIT-compiling player code.

**How to use.** Instead of calling `WWW.LoadFromCacheOrDownload` directly, a plugin calls
`LoadBundle(path, version, callback)` (or the convenience wrapper
[`RustBusterPlugin.LoadBundle`](RustBusterPlugin.md#void-loadbundlestring-path-int-version-actionassetbundle-done)).
RustBuster owns the queue and processes entries one at a time; the callback receives the loaded
`AssetBundle` (or `null` on error) when it is the plugin's turn.

The queue is started automatically on first use and runs on the same [`Loom`](Loom.md) MonoBehaviour
that the rest of the codebase uses for main-thread coroutines. No plugin needs to manage the runner
lifecycle.

### Members

#### `static void LoadBundle(string path, int version, Action<AssetBundle> done)`
Enqueues an asset-bundle load request. The bundle will be loaded via `WWW.LoadFromCacheOrDownload`
after all previously queued requests have completed, so at most one WWW download is in flight at any
given moment.

- `path` - the URL or local path passed to `WWW.LoadFromCacheOrDownload`. Local paths must begin with
  `file://`.
- `version` - cache version number. Pass `0` to disable caching (equivalent to constructing a plain
  `WWW`).
- `done` - callback invoked on the main Unity thread once the bundle has been loaded. The `AssetBundle`
  argument is `null` when loading failed. The callback is always invoked, even on failure, so callers
  can release waiting state unconditionally.

Throws `ArgumentNullException` if `path` or `done` is `null`.

```csharp
AssetBundleLoader.LoadBundle("file://" + path, 0, bundle =>
{
    if (bundle == null)
    {
        Hooks.LogData(Name, "Failed to load my bundle.");
        return;
    }

    GameObject prefab = bundle.Load("MyPrefab", typeof(GameObject)) as GameObject;
    // ... use prefab ...
});
```

#### `static int PendingCount { get; }`
Number of bundle requests currently waiting in the queue (not counting the one actively loading, if
any).

### Relationship to `Facepunch.Bundling`
This loader only deals with *downloading* bundles over `WWW`; by itself it has no connection to the
game's own asset registry. The game itself keeps every asset it has ever loaded from a bundle inside its
own internal, static `Facepunch.Bundling` class, which is populated during the game's own bundle-loading
boot sequence (`gameobject.00X`, `texture.00X`, etc. streamed through `Facepunch.Load.Loader`).

#### `static bool InjectIntoFacepunchBundling(AssetBundle bundle, Type assetType)`
Merges an already-loaded `AssetBundle` (e.g. one obtained through `LoadBundle`) into the game's own
`Facepunch.Bundling` registry for the given `assetType`, so that `Facepunch.Bundling.Load`/`LoadAll` (and
therefore any game system built on top of them) can resolve assets coming from your bundle. Intended for
content you legitimately own the rights to ship as part of your own server/plugin content pipeline.

- Returns `false` if `Facepunch.Bundling` has not finished its own startup yet, or if there is no
  existing bundle of `assetType` already registered to merge into (brand-new asset types with zero
  existing bundles of that type are not supported).
- The injected entries have no `Facepunch.Load.Item` metadata (it is left `null`); only
  `Load`/`LoadAll`/`Contains` lookups are guaranteed to work afterwards.
- Lookups are cached per path; a path that was already looked up and not found before injection is
  retried after a successful injection, but a path that collides with one already shipped by the game
  still resolves to the game's original asset.

```csharp
AssetBundleLoader.LoadBundle("file://" + path, 0, bundle =>
{
    if (bundle == null) return;

    if (!AssetBundleLoader.InjectIntoFacepunchBundling(bundle, typeof(GameObject)))
    {
        Hooks.LogData(Name, "Could not register my bundle's GameObjects with Facepunch.Bundling.");
    }
});
```
