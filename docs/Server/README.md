### RustBuster2016 Server Plugin Documentation

This folder documents the **public server-side API** exposed by `RustBuster2016Server.dll` for plugin
developers.

### Relationship to Fougerite

RustBuster's server component runs **inside the Fougerite-modded Rust Legacy server process**, as a
Fougerite `Module` itself. A RustBuster server plugin is therefore just an ordinary **Fougerite C#
module** (see Fougerite's own "Writing your first C# plugin" guide for the base `Module`
skeleton/lifecycle) that additionally references `RustBuster2016Server.dll` and talks to the static
[`RustBuster2016Server.API`](Classes/API.md) hub class - the server-side equivalent of the client's
[`RustBuster2016.API.Hooks`](../Client/Classes/Hooks.md).

```csharp
using System;
using Fougerite;
using RustBuster2016Server;

namespace MyFirstServerPlugin
{
    public class MyFirstServerPlugin : Module
    {
        public override string Name { get { return "MyFirstServerPlugin"; } }
        public override string Author { get { return "YourName"; } }
        public override string Description { get { return "My first RustBuster server plugin."; } }
        public override Version Version { get { return new Version(1, 0); } }

        public override void Initialize()
        {
            API.OnRustBusterSpawned += OnRustBusterSpawned;
        }

        public override void DeInitialize()
        {
            API.OnRustBusterSpawned -= OnRustBusterSpawned;
        }

        private void OnRustBusterSpawned(API.RustBusterUserAPI user)
        {
            Logger.Log("[MyFirstServerPlugin] " + user.Name + " is running RustBuster.");
        }
    }
}
```

Add a reference to `Fougerite.dll` (always, for the `Module` base class) and `RustBuster2016Server.dll`
(for `API` and everything documented here), then install/register the module exactly like any other
Fougerite module (a folder under `Modules`, an entry under `[Modules]` in `Fougerite.cfg`).

### Where to start

- **Writing a third-party module that hooks into RustBuster?** See
  [`Guide/Hooking-Into-RustBusterServer.md`](Guide/Hooking-Into-RustBusterServer.md) for a full
  walkthrough (detecting RustBuster safely, exchanging custom messages with a client plugin).
- **Looking for the main hub class?** See [`Classes/API.md`](Classes/API.md) - every server-side event
  and most helper methods live there.
- **Looking for a specific class?** See the [Classes](Classes/README.md) reference.

### Layout

```
docs/Server/
  README.md                  <- you are here
  Guide/
    Hooking-Into-RustBusterServer.md  <- step by step example for third-party modules
  Classes/
    README.md                <- index of every documented class
    API.md                   <- the hub class: events, RustBusterUserAPI, downloadables
    BanEvent.md
    Message.md
    MessageResponse.md
    RBDownloadable.md
    BannedHWsData.md
    RustBusterUserCache.md
```

### Conventions used throughout this documentation

- Every hook is a `public static event` on the sealed [`API`](Classes/API.md) class. Subscribe with `+=`
  in `Initialize()` and unsubscribe with `-=` in `DeInitialize()`, exactly like on the client - see the
  example above.
- Event classes that expose a `Cancelled` property and a `Cancel()` method are **cancellable**.
- Event classes without `Cancel()` are purely informational/notification hooks.
