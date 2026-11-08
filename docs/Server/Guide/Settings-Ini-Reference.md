### Guide
Settings.ini reference

### Description
This page lists the general settings written into `Settings.ini` by `RustBusterServer.ReloadConfig()`, what It
controls, and what happens when you change It. Settings are written with `IniParser.AddDefault`, so
existing installs automatically pick up new settings (with their default value) the next time the server
starts, without overwriting values you've already customized.

### `[Settings]` section

| Setting | Default | What it does |
|---|---|---|
| `NotifyMSG` | *(built-in message)* | The chat message sent to players who join without RustBuster installed, telling them where to download It. |
| `NotifyMSG2` | *(built-in message)* | A secondary message/link shown to players who are missing RustBuster, pointing them to the download page. |
| `NotifyMSG3` | *(built-in message)* | The reason shown to a player when they get restricted/kicked for not using RustBuster. |
| `AllowNewVersions` | `true` | If `true`, clients running a newer RustBuster version than the server expects will still be allowed to connect, instead of being rejected as a version mismatch. |
| `AnnounceN` | `false` | If `true`, the server will broadcast a chat announcement to everyone whenever RustBuster bans a cheater. |
| `Respond` | `false` | If `true`, RustBuster will reply in chat/console when It receives certain commands/requests, mostly useful for debugging. |
| `MaxServerChannels` | `2` | The maximum number of simultaneous communication channels/threads the server will use to talk to clients. |
| `KickNonRustBuster` | `false` | If `true`, players who are not using RustBuster will be kicked from the server instead of just being chat-restricted. |
| `AllowRestrictedChat` | `true` | If `true`, players who are not using RustBuster can still use chat (restricted), instead of being fully silenced. |
| `ClientPluginCompiler` | *(empty)* | Optional full path to a C# compiler (`csc.exe` or `mcs.exe`) used to build client plugin sources from `ClientPluginSources\<PluginName>\*.cs` into `ClientPlugins`. Leave empty to auto-detect (MSBuild Roslyn first). See [`Client-Plugin-Sources.md`](Client-Plugin-Sources.md). |

Options related to anti-cheat and abuse protection are intentionally not documented here.

### Where to go next
- [`Client-Plugin-Sources.md`](Client-Plugin-Sources.md) - compiling client plugins from `.cs` sources.
- [`Classes/API.md`](../Classes/API.md) - every server-side event and helper method.
- [`../README.md`](../README.md) - server plugin documentation index.
