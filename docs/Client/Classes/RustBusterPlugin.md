### Class
`RustBuster2016.API.RustBusterPlugin`

### Description
Abstract base class for every RustBuster2016 client plugin. Implements `IDisposable`. Every plugin
class must derive from this and implement `Initialize()` and `DeInitialize()`; see the
[first plugin guide](../Guide/Writing-Your-First-Plugin.md) for the full walkthrough.

### Members

#### `string Name { get; }`
Virtual, defaults to `"None"`. Override to give your plugin a unique, stable name - it is used for
logging, for `SendMessageToServer`, and as the key the loader reports in its startup/shutdown log lines.

#### `Version Version { get; }`
Virtual, defaults to `new Version(1, 0)`. Override to report your plugin's version.

#### `string Author { get; }`
Virtual, defaults to `"None"`. Override to report your name/handle.

#### `uint Order { get; }`
Virtual, defaults to `uint.MaxValue`. Plugins with a lower `Order` are initialized before plugins with a
higher one. Leave the default unless your plugin has a real ordering dependency on another plugin.

#### `bool IsConnectedToAServer { get; }`
`true` while the client is connected to a server. Shorthand so you don't have to reach into game state
yourself.

#### `string RustBusterVersion { get; }`
The running RustBuster version string (same value as `Hooks.RBVersion`).

#### `string SendMessageToServer(string msg)`
*Obsolete* - synchronous variant, kept for backwards compatibility. Blocks the calling thread while it
waits for the server's response. Prefer the asynchronous overload below. Returns `null` immediately
(without contacting the server) if not connected or if `Name` was never overridden from `"None"`.

```csharp
#pragma warning disable CS0618
string status = SendMessageToServer("ping");
#pragma warning restore CS0618
```

#### `void SendMessageToServer(string msg, Action<bool, string> callback)`
Asynchronous variant. Sends `msg` to your plugin's server-side counterpart and invokes `callback` with
`(isError, responsePayload)` once the server replies. Does nothing if not connected or `Name` is
`"None"`.

```csharp
SendMessageToServer("getStatus", (isError, response) =>
{
    if (isError)
    {
        Hooks.LogData(Name, "Server call failed.");
        return;
    }

    Hooks.LogData(Name, "Server replied: " + response);
});
```

> Both overloads are a thin wrapper that talks to **your own plugin's server-side half** through
> RustBuster's internal transport - they are not a general-purpose way to call arbitrary server
> endpoints.

#### `void LoadBundle(string path, int version, Action<AssetBundle> done)`
Convenience wrapper over the shared, throttled
[`AssetBundleLoader.LoadBundle`](AssetBundleLoader.md#public-static-void-loadbundlestring-path-int-version-actionassetbundle-done)
queue. See that page for the full parameter documentation.

#### `void Dispose()`
Standard `IDisposable` implementation; override `protected virtual void Dispose(bool disposing)` if your
plugin needs custom cleanup beyond `DeInitialize()`.

#### `abstract void Initialize()`
Entry point. Called once when your plugin is loaded. Subscribe to `Hooks` events, start timers, open
files, etc. here.

#### `abstract void DeInitialize()`
Exit point. Called once when your plugin is unloaded (e.g. on disconnect). **Unsubscribe every `Hooks`
event you subscribed to in `Initialize()` here** - `Hooks` events are static, so a handler left behind
survives past your plugin instance's lifetime until explicitly cleared.

### Example

```csharp
using System;
using RustBuster2016.API;
using RustBuster2016.API.Events;

namespace MyFirstPlugin
{
    public class MyFirstPlugin : RustBusterPlugin
    {
        public override string Name { get { return "MyFirstPlugin"; } }
        public override string Author { get { return "YourName"; } }
        public override Version Version { get { return new Version(1, 0); } }

        public override void Initialize()
        {
            Hooks.OnRustBusterClientChat += OnChat;
        }

        public override void DeInitialize()
        {
            Hooks.OnRustBusterClientChat -= OnChat;
        }

        private void OnChat(ChatEvent e)
        {
            Hooks.LogData(Name, "Chat event fired.");
        }
    }
}
```
