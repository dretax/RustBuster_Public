### Guide
Writing your first C# client plugin for RustBuster2016

### Description
This is a beginner-friendly, step-by-step tutorial for writing a compiled C# client plugin for
RustBuster2016. A client plugin is a `.dll` that is delivered to your game client by the RustBuster-
enabled server you connect to, and is loaded directly into the game process - there is no script
interpreter involved. If you're looking for the full technical reference instead of a walkthrough, see
[`Classes/RustBusterPlugin.md`](../Classes/RustBusterPlugin.md) (the base class every plugin inherits
from) and the [Hooks reference](../Hooks/README.md).

### 1. Requirements
- **Microsoft Visual Studio** (2013 or newer - Community Edition works fine) **or JetBrains Rider** -
  either is fine for writing a RustBuster client plugin; use whichever you're more comfortable with.
- **`RustBuster2016.dll`** (referred to as the RustBuster API assembly) - required by every client
  plugin, contains `RustBusterPlugin`, `Hooks`, and every other type under the `RustBuster2016.API`
  namespace documented here.
- **`UnityEngine.dll`** and **`Assembly-CSharp.dll`** - found in your Rust Legacy installation's
  `rust_Data\Managed` folder. Needed for anything beyond the most trivial plugin: `UnityEngine.dll` gives
  you `Vector3`/`GameObject`/`Transform`/etc., and `Assembly-CSharp.dll` gives you the game's own classes
  (`Character`, `ItemDataBlock`, `HumanController`, ...) that most hook/event parameters are typed with.

### 2. Creating the project
1. In Visual Studio, create a new project of type **Class Library**.
2. In the **Solution Explorer**, right-click **References** -> **Add Reference...** and add:
   - `RustBuster2016.dll` (always).
   - `UnityEngine.dll` and `Assembly-CSharp.dll` (strongly recommended, see above).
3. Match the target framework RustBuster itself runs on (check the RustBuster release notes / server
   admin for the exact version in use) so your plugin's assembly loads without surprises.

### 3. The plugin skeleton
Every C# client plugin must inherit from the abstract
[`RustBuster2016.API.RustBusterPlugin`](../Classes/RustBusterPlugin.md) class and implement
`Initialize()` and `DeInitialize()`.

```csharp
using RustBuster2016.API;

namespace MyFirstPlugin
{
    public class MyFirstPlugin : RustBusterPlugin
    {
        public override string Name
        {
            get { return "MyFirstPlugin"; }
        }

        public override string Author
        {
            get { return "YourName"; }
        }

        public override System.Version Version
        {
            get { return new System.Version(1, 0); }
        }

        public override void Initialize()
        {
            // Code that runs once, when the plugin is loaded on your client.
            // This is the right place to subscribe to Hooks, create timers, etc.
        }

        public override void DeInitialize()
        {
            // Code that runs once, when the plugin is unloaded (e.g. on disconnect).
            // This is the right place to unsubscribe from every Hook you subscribed to
            // in Initialize(), otherwise a reconnect can leave "ghost" handlers behind.
        }
    }
}
```

`Name`, `Author` and `Version` are metadata read by the loader for logging; they don't need any further
explanation. `RustBusterPlugin` also gives you, for free (see
[`Classes/RustBusterPlugin.md`](../Classes/RustBusterPlugin.md) for the full list):
- `Order` - override to control load/initialize order relative to other plugins.
- `IsConnectedToAServer` - quick check without having to poke at game state yourself.
- `RustBusterVersion` - the RB version string, handy for compatibility checks/logging.
- `SendMessageToServer(...)` - a built-in channel to talk to your own server-side counterpart.
- `LoadBundle(...)` - a shared, throttled asset-bundle downloader (see
  [`Classes/AssetBundleLoader.md`](../Classes/AssetBundleLoader.md)).

### 4. Hooks: reacting to events
Hooks let your plugin run code whenever something happens on the client - a chat message, a weapon
being fired, an item being crafted, the local player attaching to a vehicle seat, etc. The full list
(with every parameter documented) lives in the [Hooks reference](../Hooks/README.md); this section only
covers the basics.

**The pattern is always the same:**
- In `Initialize()`, subscribe your method to the hook with `+=`.
- In `DeInitialize()`, unsubscribe the *same* method with `-=`.

```csharp
public override void Initialize()
{
    Hooks.OnRustBusterClientChat += HandleChat;
}

public override void DeInitialize()
{
    Hooks.OnRustBusterClientChat -= HandleChat;
}
```

Forgetting the `-=` step means your handler can keep running (and firing twice, three times, ...) across
reconnects, since `Hooks` events are static and outlive a single plugin instance until explicitly
cleared.

#### Example: a cancellable hook - `OnRustBusterClientChat`
Fired when the local player sends a chat message. Full reference:
[`Hooks/Player.md`](../Hooks/Player.md#onrustbusterclientchat).

```csharp
public void HandleChat(RustBuster2016.API.Events.ChatEvent e)
{
    if (e.ChatUI.ToString().Contains("badword"))
    {
        // Cancel() prevents the message from being sent and optionally closes the chat box.
        e.Cancel(true);
    }
}
```

#### Example: a non-cancellable, informational hook - `OnRustBusterPlayerRespawn`
Fired after a player finishes respawning. Full reference:
[`Hooks/Player.md`](../Hooks/Player.md#onrustbusterplayerrespawn).

```csharp
public override void Initialize()
{
    Hooks.OnRustBusterPlayerRespawn += HandleRespawn;
}

public override void DeInitialize()
{
    Hooks.OnRustBusterPlayerRespawn -= HandleRespawn;
}

public void HandleRespawn(RustBuster2016.API.Events.PlayerRespawnEvent e)
{
    Hooks.LogData(Name, "Local player just respawned.");
}
```

#### Example: combat hook with mutable (not just cancellable) fields - `OnRustBusterWeaponFire`
Some events expose settable properties instead of a single `Cancel()`, letting you tweak a specific
side effect rather than block the whole action. Full reference:
[`Hooks/Combat.md`](../Hooks/Combat.md#onrustbusterweaponfire).

```csharp
public override void Initialize()
{
    Hooks.OnRustBusterWeaponFire += HandleWeaponFire;
}

public override void DeInitialize()
{
    Hooks.OnRustBusterWeaponFire -= HandleWeaponFire;
}

public void HandleWeaponFire(RustBuster2016.API.Events.BulletWeaponFireEvent e)
{
    // Halves the recoil applied after this shot, without blocking the shot itself.
    e.Pitch *= 0.5f;
    e.Yaw *= 0.5f;
}
```

> These three hooks are only a small sample. See the [full Hooks reference](../Hooks/README.md) for
> every available event, grouped by category, each with the exact delegate signature, its backing event
> class, and a working C# example.

### 5. Logging
Use `Hooks.LogData(Name, message)` to write to the RustBuster plugin log with your plugin's name
attached, rather than `UnityEngine.Debug.Log` directly - see
[`Classes/Hooks.md`](../Classes/Hooks.md#logdata).

### 6. Timers
RustBuster ships two timer flavours so you never have to touch `UnityEngine` timing primitives directly:
- A **main-thread ("Normal") timer**, backed by a `MonoBehaviour` (`TimedEvent`), safe to touch
  `GameObject`/Unity APIs from.
- A **background-thread ("System") timer**, backed by `System.Timers.Timer` (`SystemTimerEvent`), for
  work that must not block the main thread and must NOT touch Unity objects.

See [`Classes/Timers.md`](../Classes/Timers.md) for the full reference and the crucial difference
between the two.

```csharp
public override void Initialize()
{
    Util.CreateTimer("heartbeat", 5000, OnHeartbeat, autoReset: true, pluginName: Name);
}

public override void DeInitialize()
{
    Util.KillTimer("heartbeat");
}

private void OnHeartbeat(RustBuster2016.API.Events.TimedEvent timer)
{
    Hooks.LogData(Name, "Still alive.");
}
```

### 7. Where to go next
- [Hooks reference](../Hooks/README.md) - every hook RustBuster2016 exposes client-side, grouped by
  category.
- [`Classes/RustBusterPlugin.md`](../Classes/RustBusterPlugin.md) - everything your plugin inherits.
- [`Classes/DataStore.md`](../Classes/DataStore.md) - persisting simple key/value data to disk, scoped
  per-server or globally.
- [`Classes/Web.md`](../Classes/Web.md) / [`Classes/WinHttpClient.md`](../Classes/WinHttpClient.md) -
  making HTTP requests from a plugin.
- [`Classes/LocalAttachment.md`](../Classes/LocalAttachment.md) - mounting the local player to vehicle
  seats/anchors.
- [`Classes/CustomMap.md`](../Classes/CustomMap.md) - replacing the map of the current join with your
  own scene bundle.
- [`Classes/KeyBindingRegistry.md`](../Classes/KeyBindingRegistry.md) - registering your own
  rebindable hotkeys.
