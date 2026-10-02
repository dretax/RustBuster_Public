### Guide
Hooking into RustBusterServer from a Fougerite module

### Description
This guide shows how an ordinary **Fougerite server module** (one that does not ship as part of
RustBuster itself) can detect that `RustBuster2016Server` is loaded and talk to it - subscribing to its
events, sending the client a custom message, and reacting to the client's reply. For the base `Module`
skeleton/lifecycle and the general hook pattern (`+=` in `Initialize()`, `-=` in `DeInitialize()`), see
Fougerite's own "Writing your first C# plugin" guide; this page only covers the RustBuster-specific part.
For the full member reference of every type used below, see [`Classes/API.md`](../Classes/API.md) and
[`Classes/Message.md`](../Classes/Message.md).

### 1. Reference `RustBuster2016Server.dll`
Add a reference to `Fougerite.dll` (always) and `RustBuster2016Server.dll` (for `API` and the event
argument types), the same way you would for `RustBuster2016.dll` on the client - see
[`Server/README.md`](../README.md) for the full `Module` skeleton.

### 2. Don't assume RustBuster is loaded
Nothing stops a server admin from running your module without RustBuster installed. If you call
`API.OnRustBusterUserMessage += ...` straight from `Initialize()` and the `RustBuster2016Server` module
isn't loaded, that's harmless by itself (it's just a static event), but anything that actively **calls
into** `RustBuster2016Server.API` (e.g. reading `API.RustBusterUsersList`) could throw if the assembly
never finished initializing. The safe pattern is to detect RustBuster through Fougerite's own
`Hooks.OnModulesLoaded` hook (fired once every module has finished `Initialize()`), and only subscribe to
`API`'s events from there:

```csharp
using System;
using Fougerite;
using Fougerite.PluginLoaders;
using RustBuster2016Server;

namespace MyFirstServerPlugin
{
    public class MyFirstServerPlugin : Module
    {
        private bool _foundRustBuster;

        public override string Name { get { return "MyFirstServerPlugin"; } }
        public override string Author { get { return "YourName"; } }
        public override string Description { get { return "My first RustBuster server plugin."; } }
        public override Version Version { get { return new Version(1, 0); } }

        public override void Initialize()
        {
            // Hooks that don't touch RustBuster can be subscribed right away.
            Hooks.OnModulesLoaded += OnModulesLoaded;
        }

        public override void DeInitialize()
        {
            Hooks.OnModulesLoaded -= OnModulesLoaded;

            // Only unsubscribe from API's events if we actually subscribed to them.
            if (_foundRustBuster)
            {
                API.OnRustBusterUserMessage -= OnRustBusterUserMessage;
                API.OnRustBusterSpawned -= OnRustBusterSpawned;
            }
        }

        private void OnModulesLoaded()
        {
            foreach (var plugin in PluginLoader.GetInstance().Plugins.Values)
            {
                if (plugin.Name == "RustBusterServer")
                {
                    _foundRustBuster = true;
                    AddRustBusterHooks();
                    break;
                }
            }
        }

        private void AddRustBusterHooks()
        {
            API.OnRustBusterUserMessage += OnRustBusterUserMessage;
            API.OnRustBusterSpawned += OnRustBusterSpawned;
        }

        private void OnRustBusterSpawned(API.RustBusterUserAPI user)
        {
            Logger.Log("[MyFirstServerPlugin] " + user.Name + " is running RustBuster.");
        }

        private void OnRustBusterUserMessage(API.RustBusterUserAPI user, Message msg)
        {
            // See step 3 below.
        }
    }
}
```

`PluginLoader.GetInstance().Plugins.Values` lists every loaded Fougerite module by its `Name` property;
`RustBusterServer` is the `Name` of RustBuster's own server module (see `RustBuster2016Server.Server`).
Splitting the detection (`Hooks.OnModulesLoaded`) from the actual subscription (`AddRustBusterHooks`)
into two methods, like above, also avoids a `MissingAssemblyException`/`TypeLoadException` being thrown
from inside the loop itself if `RustBuster2016Server.dll` happens to be entirely absent on disk.

### 3. Talking to a specific client plugin
`API.OnRustBusterUserMessage` fires whenever **any** RustBuster client plugin calls
`RustBusterPlugin.SendMessageToServer` on the game side. Because every server plugin receives the same
event, always check `Message.PluginSender` first and ignore messages that aren't addressed to you -
otherwise you risk reacting to another plugin's traffic that happens to look similar. This mirrors a
simple login/registration protocol, inspired by a real plugin (`AuthMe`) that ships a matching client
plugin named `"AuthMe"`:

```csharp
private void OnRustBusterUserMessage(API.RustBusterUserAPI user, Message msg)
{
    if (msg.PluginSender != "AuthMe")
    {
        return;
    }

    // Example wire format: "AuthMeLogin-username-password"
    string[] parts = msg.MessageByClient.Split('-');
    if (parts.Length != 3)
    {
        return;
    }

    string action = parts[0];
    string username = parts[1];
    string password = parts[2];

    if (action == "AuthMeLogin" && IsValidLogin(user.UID, username, password))
    {
        msg.ReturnMessage = "Approved";
    }
    else
    {
        msg.ReturnMessage = "DisApproved";
    }
}
```

`msg.ReturnMessage` (default `"Done"` if you never set it) is what gets sent back to the exact client
plugin that called `SendMessageToServer`, so it can branch on the result (see
[`RustBusterPlugin.SendMessageToServer`](../../Client/Classes/RustBusterPlugin.md) on the client side for
the other end of this exchange). Avoid `~` and `@` in the strings you exchange - they are used as
protocol separators.

### 4. Pushing a message to the client first
Instead of waiting for the client to speak first, you can send a message of your own once the client's
communication channel is ready, then read its reply via `API.OnRustBusterClientResponse`:

```csharp
public override void AddRustBusterHooks()
{
    API.OnRustBusterServerChannelAuth += OnRustBusterServerChannelAuth;
    API.OnRustBusterClientResponse += OnRustBusterClientResponse;
}

private void OnRustBusterServerChannelAuth(API.RustBusterUserAPI user)
{
    user.SendMessageAsync("MyFirstPlugin", "ping");
}

private void OnRustBusterClientResponse(API.RustBusterUserAPI user, MessageResponse response)
{
    if (response.Pluginsender != "MyFirstPlugin")
    {
        return;
    }

    Logger.Log(user.Name + " replied: " + response.Response);
}
```

See [`Classes/MessageResponse.md`](../Classes/MessageResponse.md) for the full reference of `response`.

### Where to go next
- [`Classes/API.md`](../Classes/API.md) - every server-side event and helper method.
- [`Classes/Message.md`](../Classes/Message.md) / [`Classes/MessageResponse.md`](../Classes/MessageResponse.md) -
  the two message-exchange event arguments used above.
- [`Classes/BanEvent.md`](../Classes/BanEvent.md) - cancelling a RustBuster ban from your own plugin.
- [`../../Client/README.md`](../../Client/README.md) - the matching client-side API, if your plugin also
  ships a client component (like `AuthMe`'s client plugin that calls `SendMessageToServer`).
