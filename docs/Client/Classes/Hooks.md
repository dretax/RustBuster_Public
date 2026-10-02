### Class
`RustBuster2016.API.Hooks` (sealed)

### Description
`Hooks` is the hub class that exposes every client-side event (documented separately in the
[Hooks reference](../Hooks/README.md)) plus a handful of static helper members. This page only covers
those helper members - properties and methods, not the `public static event` fields.

### Members

#### `const string RBVersion`
The running RustBuster version string, e.g. `"3.2.2"`.

#### `static TOD_Components TODComps`
The Time-Of-Day/weather component assigned by RustBuster. Read-only in practice; set internally.

#### `static string GameDirectory { get; }`
Absolute path to the folder `rust.exe` is located in.

#### `static string PlayerName { get; }`
The local player's name, as reported by RustBuster at login.

#### `static string SteamID { get; }`
The local player's SteamID.

#### `static string HWID { get; }`
The local player's hardware ID, as computed by RustBuster.

#### `static PlayerClient LocalPlayer { get; }`
The local `PlayerClient`, or `null` if not connected / not spawned yet.

#### `static List<string> ClientSidePlugins { get; }`
Names of every client-side plugin currently loaded.

#### `static void LogData(string yourPluginName, string message)`
Appends a line to the RustBuster plugin log, prefixed with `yourPluginName`. Prefer this over
`UnityEngine.Debug.Log` for plugin diagnostics.

```csharp
Hooks.LogData(Name, "Plugin initialized.");
```

#### `static void TakeManualScreenshot(Action<byte[]> methodToCall)`
Takes a screenshot and invokes `methodToCall` with the resulting PNG bytes once it's ready (dispatched
onto the main thread internally, safe to call from any thread). Invokes the callback with `null` if
RustBuster couldn't create the screenshot (e.g. not connected, or the local character hasn't loaded
yet).

```csharp
Hooks.TakeManualScreenshot(bytes =>
{
    if (bytes == null)
    {
        Hooks.LogData(Name, "Screenshot failed.");
        return;
    }

    System.IO.File.WriteAllBytes("myplugin_shot.png", bytes);
});
```

#### `static byte[] CompressByte(byte[] array)` / `static byte[] DeCompressByte(byte[] array)`
LZ4 compress/decompress helpers for arbitrary byte arrays - handy for shrinking payloads before sending
them with `RustBusterPlugin.SendMessageToServer` or persisting them via `DataStore`.

#### `static List<Player> GetPlayerList()`
Returns every currently alive, loaded online player except the local one. **Must be called from the main
thread** - calling it from a background thread logs a warning and returns an empty list instead of
crashing.

```csharp
foreach (Player p in Hooks.GetPlayerList())
{
    Hooks.LogData(Name, "Nearby player: " + p.name);
}
```

### See also
- [Hooks reference](../Hooks/README.md) - every event exposed through this class.
