### Classes reference

Index of every documented public client-side class under `RustBuster2016.API` (unless noted otherwise).

| Class | Summary |
|---|---|
| [`RustBusterPlugin`](RustBusterPlugin.md) | Abstract base class every client plugin inherits from. |
| [`Hooks`](Hooks.md) | Static helpers/properties on the hub class (the events themselves are in the [Hooks reference](../Hooks/README.md)). |
| [`AssetBundleLoader`](AssetBundleLoader.md) | Shared, serialised asset-bundle download queue. |
| [`DataStore`](DataStore.md) | Thread-safe, JSON-backed key/value store, global or per-server. |
| [`Util`](Util.md) | Grab-bag of raycast/terrain helpers, reflection helpers, and the Normal-timer API. |
| [`Timers`](Timers.md) | `TimedEvent` (main-thread) and `SystemTimerEvent` (background-thread) timers. |
| [`Web`](Web.md) | `System.Net`-based HTTP GET/POST (sync, async, SSL). |
| [`WinHttpClient`](WinHttpClient.md) | Native WinHTTP-based HTTP/WebSocket client, blocking and async. |
| [`Loom`](Loom.md) | Dispatch work onto the Unity main thread from any thread. |
| [`IniParser`](IniParser.md) | Minimal classic `.ini` file reader/writer. |
| [`KeyBindingRegistry`](KeyBindingRegistry.md) | Register rebindable, conflict-aware plugin hotkeys. |
| [`KeyboardAPI`](KeyboardAPI.md) | Low-level `SendInput`-based synthetic key/mouse injection. |
| [`LocalAttachment`](LocalAttachment.md) | Mount the local player onto a vehicle seat / anchor transform. |
| [`CustomMap`](CustomMap.md) | Replace the map of the current join with your own scene bundle. |
| [`DatablockManager`](DatablockManager.md) | Clone/register/unregister custom `ItemDataBlock`s at runtime. |
| [`WaterSystem`](WaterSystem.md) | Optional client-side swimming/buoyancy simulation. |
| [`LadderSystem`](LadderSystem.md) | Optional client-side ladder-climbing simulation. |
| [`FogControl`](FogControl.md) | Priority-stacked fog layer composition. |
| [`Zone3D`](Zone3D.md) | Named polygon + height-range 3D zone registry. |
| [`Extensions`](Extensions.md) | Small `string`/`GameObject` extension methods. |
| [`Stopper`](Stopper.md) | `IDisposable` stopwatch that logs a warning if a block runs too long. |
| [`PluginMessaging`](PluginMessaging.md) | Inter-plugin communication, send an object payload to another plugin by name. |
| [`ScriptWebSocket`](ScriptWebSocket.md) | WinHTTP-based `ws://`/`wss://` WebSocket client. |

Not documented separately (part of the game's own code, only referenced from RustBuster's API):
`DatablockDictionary`, `ItemDataBlock`, `Character`, `HumanController`, and other `Assembly-CSharp`/
`UnityEngine` types that appear as parameters throughout the event classes.
