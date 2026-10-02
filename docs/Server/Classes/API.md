### Class
`RustBuster2016Server.API` (sealed)

### Description
The hub class for the server-side RustBuster API: every event, the `RustBusterUserAPI` wrapper for a
connected RustBuster user, and downloadable-file management (the server-side counterpart of
[`RBDownloadable`](RBDownloadable.md) registration). See [`Server/README.md`](../README.md) for how a
Fougerite module references this class.

### Events

All events are `public static`. Subscribe with `+=` in `Initialize()`, unsubscribe with `-=` in
`DeInitialize()`.

#### `static event RustBusterUserLoginDelegate OnRustBusterLogin`
`public delegate void RustBusterUserLoginDelegate(API.RustBusterUserAPI user)`

Runs when an RB client starts connecting to the server. Not cancellable.

#### `static event RustBusterUserSpawnedDelegate OnRustBusterSpawned`
`public delegate void RustBusterUserSpawnedDelegate(API.RustBusterUserAPI user)`

Runs when an RB client has just logged into the server and spawned. Not cancellable.

#### `static event RustBusterUserDisconnectedDelegate OnRustBusterDisconnected`
`public delegate void RustBusterUserDisconnectedDelegate(API.RustBusterUserAPI user)`

Runs when an RB client disconnected. Not cancellable.

#### `static event RustBusterUserUploadedIMGDelegate OnRustBusterUserUploadedIMG`
`public delegate void RustBusterUserUploadedIMGDelegate(API.RustBusterUserAPI user, byte[] jpgBytes)`

Runs when the server receives a screenshot from the client (see
[`Hooks.TakeManualScreenshot`](../../Client/Classes/Hooks.md) on the client side). Not cancellable.

#### `static event RustBusterUserMessageDelegate OnRustBusterUserMessage`
`public delegate void RustBusterUserMessageDelegate(API.RustBusterUserAPI user, Message msgc)`

Runs when an RB client sends a custom message, mostly from a client plugin's
[`RustBusterPlugin.SendMessageToServer`](../../Client/Classes/RustBusterPlugin.md). Not cancellable -
respond by setting `msgc.ReturnMessage`. See [`Message.md`](Message.md) for the full reference and a
complete request/response example.

#### `static event RustBusterServerChannelAuthDelegate OnRustBusterServerChannelAuth`
`public delegate void RustBusterServerChannelAuthDelegate(API.RustBusterUserAPI user)`

Runs once the RB client has opened its server-communication channel, meaning you can start sending it
your own messages via [`RustBusterUserAPI.SendMessage`](#bool-sendmessagestring-pluginame-string-message)/
[`SendMessageAsync`](#bool-sendmessageasyncstring-pluginname-string-message). Not cancellable.

#### `static event RustBusterClientResponseDelegate OnRustBusterClientResponse`
`public delegate void RustBusterClientResponseDelegate(API.RustBusterUserAPI user, MessageResponse msgc)`

Runs when the RB client responds to a message you sent it via `RustBusterUserAPI.SendMessage`/
`SendMessageAsync`. Not cancellable. See [`MessageResponse.md`](MessageResponse.md).

#### `static event RustBusterUserBanDelegate OnRustBusterUserBan`
`public delegate void RustBusterUserBanDelegate(BanEvent be)`

Runs when a user receives a RustBuster ban. **Cancellable** - see [`BanEvent.md`](BanEvent.md).

#### `static event RustBusterConfigReloadDelegate OnRustBusterConfigReload`
`public delegate void RustBusterConfigReloadDelegate()`

Runs when [`API.ReloadConfig`](#static-void-reloadconfig) is called. No arguments. Not cancellable.

### Properties

#### `static List<RustBusterUserAPI> RustBusterUsersList { get; }`
The currently online RustBuster users, as a fresh `List<T>` snapshot.

#### `static Dictionary<ulong, RustBusterUserAPI> RBUsers { get; }`
A shallow copy of the currently stored users dictionary, keyed by SteamID (`ulong`).

#### `static string RustBusterVersion { get; }`
The server's configured/expected RustBuster version string.

#### `static BannedHWsData BannedHWsData { get; }`
The persistent banned-HWID store - see [`BannedHWsData.md`](BannedHWsData.md).

#### `static List<RBDownloadable> RBDownloadables { get; }`
A fresh copy of the currently registered downloadable file list.

### Methods

#### `static void ReloadConfig()`
Fires `OnRustBusterConfigReload` for every subscriber. Call this from your own in-game "reload" command
handler if your plugin's config affects RustBuster behavior.

#### `static byte[] CompressByte(byte[] array)` / `static byte[] DeCompressByte(byte[] array)`
LZ4 compress/decompress helpers, mirroring the client's `Hooks.CompressByte`/`DeCompressByte`.

#### `static RustBusterUserAPI FindRustBusterUserBySteamID(string steamid)`
#### `static RustBusterUserAPI FindRustBusterUserBySteamID(ulong steamid)`
Finds a currently online RustBuster user by SteamID (string or `ulong` overload). Returns `null` if not
found/not online.

#### `static string GetHWID(Fougerite.Player player)`
Finds the HWID of a currently online RustBuster user matching the given Fougerite `Player`. Returns
`null` if the player isn't running RustBuster (or isn't online).

#### `static Fougerite.Player FindPlayerByHWID(string hwid)`
The inverse of `GetHWID`: finds the `Fougerite.Player` behind a given HWID. Returns `null` if not found.

```csharp
public void HandleLogin(API.RustBusterUserAPI user)
{
    Fougerite.Logger.Log($"{user.Name} ({user.SteamID}) connected with HWID {user.HardwareID}.");
}
```

#### `static bool AddFileToDownload(RBDownloadable rbd)`
Registers a file to be downloaded by every RustBuster client, if not already registered. Returns `false`
if it was already present. See [`RBDownloadable.md`](RBDownloadable.md) for how to construct `rbd` and a
full example (including custom maps).

#### `static bool DeleteFileFromDownloadByPath(string filepath)` / `static bool DeleteFileFromDownloadByName(string name)` / `static bool DeleteFileFromDownloadByClass(RBDownloadable rbd)`
Unregisters a previously added downloadable, by its server file path, by its file name, or by the exact
`RBDownloadable` instance. Returns `false` if no match was found.

### `RustBusterUserAPI` (nested class)
Represents a RustBuster user (online or recently seen).
- `Fougerite.Player Player { get; }` - performs a fresh lookup every time (handles cases where the
  player wasn't yet registered in Fougerite when this object was constructed). Can be `null`.
- `string SteamID { get; }` - as a string.
- `ulong UID { get; }` - as a `ulong`.
- `string HardwareID { get; }`.
- `string Name { get; }`.
- `bool SendMessage(string pluginame, string message)` - sends a message to the client synchronously.
  Returns `false` if the player is offline.
- `bool SendMessageAsync(string pluginName, string message)` - asynchronous variant; returns `true` if
  the player is online (does not wait for delivery/response).

```csharp
public void HandleServerChannelAuth(API.RustBusterUserAPI user)
{
    // Now that the channel is open, we can push a message to this specific client plugin.
    user.SendMessageAsync("MyFirstPlugin", "hello from the server");
}
```
