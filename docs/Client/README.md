### RustBuster2016 Client Plugin Documentation

This folder documents the **public client-side API** exposed by RustBuster2016 (RB) for plugin
developers. RustBuster is an anti-cheat / enhancement client for Rust Legacy (old Rust), in the same
spirit as the Fougerite server-side mod.

A RustBuster **client plugin** is a compiled C# `.dll` that contains one or more classes deriving from
[`RustBusterPlugin`](Classes/RustBusterPlugin.md). The RustBuster-enabled server you connect to decides
which plugins are delivered to your client; once delivered, RustBuster loads them automatically and
calls `Initialize()`/`DeInitialize()` on them, exactly like the server-side `Module` lifecycle in
Fougerite.

> This covers the **client-side** public API only. For the server-side public API, see
> [`../Server/README.md`](../Server/README.md).

### Where to start

- **New to RustBuster plugin development?** Start with the
  [Writing your first C# client plugin](Guide/Writing-Your-First-Plugin.md) guide.
- **Looking for a specific class?** See the [Classes](Classes/README.md) reference.
- **Looking for a specific event/hook?** See the [Hooks](Hooks/README.md) reference.

### Layout

```
docs/Client/
  README.md                          <- you are here
  Guide/
    Writing-Your-First-Plugin.md     <- step by step beginner tutorial
  Classes/
    README.md                        <- index of every documented class
    RustBusterPlugin.md              <- the base class every plugin inherits from
    Hooks.md                         <- static helpers/properties on Hooks (not the events themselves)
    AssetBundleLoader.md
    DataStore.md
    Util.md
    Timers.md                        <- TimedEvent / SystemTimerEvent / Util timer helpers
    Web.md
    WinHttpClient.md
    Loom.md
    IniParser.md
    KeyBindingRegistry.md
    KeyboardAPI.md
    LocalAttachment.md
    CustomMap.md
    DatablockManager.md
    WaterSystem.md
    LadderSystem.md
    FogControl.md
    Zone3D.md
    Extensions.md
    Stopper.md
    PluginMessaging.md
    ScriptWebSocket.md
  Hooks/
    README.md                        <- full table of every hook, grouped by category
    Player.md
    Combat.md
    Items-And-World.md
    Vehicles-And-Attachments.md
    Network-And-Plugins.md
    WebSocket.md
```

### Conventions used throughout this documentation

- Every hook is a `public static event` on the sealed [`Hooks`](Classes/Hooks.md) class. Subscribe with
  `+=` in `Initialize()` and unsubscribe with `-=` in `DeInitialize()` - see the guide for why this
  matters.
- Event classes that expose a `Cancelled` property and a `Cancel()` method are **cancellable**: calling
  `Cancel()` from any subscriber stops the native action the event describes (sending a chat message,
  crafting an item, exploding a C4, etc.).
- Event classes without `Cancel()` are purely informational/notification hooks.
- Code samples assume a plugin class that inherits from `RustBusterPlugin`, as shown in the
  [first plugin guide](Guide/Writing-Your-First-Plugin.md).
