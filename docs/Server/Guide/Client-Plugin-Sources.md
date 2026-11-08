### Guide
Compiling client plugins from source (`ClientPluginSources`)

### Description
Besides serving pre-compiled client plugin DLLs from `ClientPlugins`, `RustBuster2016Server` can compile
client plugin **`.cs` source files** itself. Drop the sources in a folder, and the server builds them
into `ClientPlugins\<PluginName>.dll` and recompiles automatically whenever a source file changes. The
result is served to clients exactly like a hand-placed DLL.

All paths below are relative to the RustBuster module folder, the same folder that holds `ClientPlugins`
(usually `rust_dedicated\Save\RustBuster2016Server\`).

### Two ways to provide a client plugin

Both end up in `ClientPlugins` and are served to clients the same way:

1. **From sources (automatic):** put the `.cs` files in `ClientPluginSources\<PluginName>\`. The server
   compiles them into `ClientPlugins\<PluginName>.dll` and recompiles whenever they change. Do not edit
   or place files by hand for these; the generated DLL is overwritten and deleted if the source folder
   is removed.
2. **Self-compiled (manual):** build the DLL yourself (Visual Studio, `csc`, ...) against the client's
   DLLs and drop it straight into `ClientPlugins`. The server never compiles, changes or deletes these.

Plugins from sources can `// #require` self-compiled DLLs (see below).

### One plugin folder = many `.cs` files

**Important:** a plugin is not limited to a single file. Every `.cs` file inside a plugin folder
(subfolders included) is compiled together into one DLL, named after the folder. Split your plugin
into as many files as you like, for example:

```
ClientPluginSources\
  MyPlugin\                  <- becomes ClientPlugins\MyPlugin.dll
    MyPlugin.cs              <- the class deriving from RustBusterPlugin
    Helpers.cs
    Config\
      Settings.cs
    Hooks\
      ChatHooks.cs
```

- Files share one compilation, so types in one file can be used from the others without any `#require`.
- Only one class per plugin folder should derive from `RustBusterPlugin`.
- Editing, adding or deleting any of the files triggers a recompile of that plugin.
- `// #require` and the `Order` property are read from all files of the folder.
- Use `#require` only to depend on a different plugin folder or DLL, not to include your own files.

### Folder layout

The folders are created automatically when the server loads (or reloads) its config.

```
ClientPlugins\                         <- the DLLs served to clients: plugins compiled by the server from
                                          ClientPluginSources AND plugins you compiled yourself (hand-placed)
ClientPluginSources\                   <- created automatically
  References\                          <- shared reference DLLs for every plugin
  ClientSideReferences\                <- client side DLLs (see below)
  .cache\                              <- compile logs and build hashes, managed by the server
  HelloPlugin\                         <- one folder per plugin; the folder name is the DLL name
    HelloPlugin.cs                     <- any number of .cs files, subfolders allowed
    References\                        <- optional, DLLs used only by this plugin
```

- The plugin folder name becomes the output name: `HelloPlugin` produces `ClientPlugins\HelloPlugin.dll`.
- Folders named `bin`, `obj`, `References`, `ClientSideReferences` or starting with `.` are not treated as plugins.
- Every `.cs` file under the plugin folder (subfolders included) is compiled into that plugin's DLL.

### References

References are the DLLs your plugin code uses (`RustBusterPlugin`, `Debug`, game classes, ...). The
server passes every DLL found in the reference folders to the compiler. They are only needed to compile;
they are not sent to clients.

- **`ClientPluginSources\ClientSideReferences`**: copy **all DLLs from the client's `Managed` folder
  (`Rust_Data\Managed`) except `mscorlib.dll` and `mscorwks.dll`**. Copying everything is the
  recommended setup, do not cherry-pick. The release packages ship this set of DLLs and it is refreshed
  from time to time with new releases, so replace the folder contents when you update. `mscorlib` and
  `mscorwks` are left out because the compiler already supplies its own core library, and adding the
  client's would clash with it. Only the `.dll` files directly in the folder are used; subfolders
  (such as `RefDLL` and `Update`) and non-DLL files (such as `RustBuster2016.xml`) are not needed.
  The client `Managed` folder currently contains:
  `Assembly-CSharp`, `Assembly-CSharp-firstpass`, `Assembly-UnityScript`, `Assembly-UnityScript-firstpass`,
  `Boo.Lang`, `dfScriptLite`, `EasyRoads3D`, `Facepunch.Actor`, `Facepunch.Cursor`, `Facepunch.Geometry`,
  `Facepunch.HitBox`, `Facepunch.ID`, `Facepunch.MeshBatch`, `Facepunch.Movement`, `Facepunch.Prefetch`,
  `Facepunch.Utility`, `Google.ProtocolBuffers`, `Google.ProtocolBuffers.Serialization`, `I18N`,
  `I18N.West`, `LitJSON`, `LZ4`, `Mono.Data.Tds`, `Mono.Posix`, `Mono.Security`, `Newtonsoft.Json`,
  `Newtonsoft.Json.Schema`, `RustBuster2016` (for `RustBusterPlugin`), `System`, `System.Configuration`,
  `System.Configuration.Install`, `System.Core`, `System.Data`, `System.Data.SQLite`, `System.Drawing`,
  `System.Management`, `System.Runtime.Serialization`, `System.Security`, `System.Transactions`,
  `System.Xml`, `System.Xml.Linq`, `uLink`, `UnityEngine`, `UnityEngine.Cloud.Analytics`,
  `UnityEngine.Cloud.Analytics.Util` and `UnityScript.Lang` (all `.dll`), plus `mscorlib.dll` and
  `mscorwks.dll`, which you skip.
- **`ClientPluginSources\References`**: any other DLLs shared by several plugins.
- **`ClientPluginSources\<PluginName>\References`**: DLLs used by a single plugin.

**Server-side DLLs are not references.** Do not copy anything from the server's own `Managed` folder
(`rust_server_Data\Managed`) into any of these folders. Client plugins run on the client, so they are
compiled only against the client's DLLs. The server never adds its own DLLs (Fougerite, the server's
`Assembly-CSharp`, `RustBuster2016Server`, ...) to the compiler. The compiler itself supplies the core
framework library (`mscorlib`).

Replacing a DLL in any of these folders triggers a recompile.

Reference order (the first DLL wins when names are duplicated):

1. The plugin's own `References` folder
2. `ClientPluginSources\References`
3. `ClientPluginSources\ClientSideReferences`
4. The compiled DLLs of the plugins it `#require`s

### Example plugin

`ClientPluginSources\HelloPlugin\HelloPlugin.cs`:

```csharp
using System;
using RustBuster2016.API;
using UnityEngine;

public class HelloPlugin : RustBusterPlugin
{
    public override string Name { get { return "HelloPlugin"; } }
    public override Version Version { get { return new Version(1, 0); } }
    public override string Author { get { return "You"; } }
    public override uint Order { get { return 10; } }   // lower loads first, default is uint.MaxValue

    public override void Initialize()
    {
        Debug.Log("[HelloPlugin] Initialized");
    }

    public override void DeInitialize()
    {
        Debug.Log("[HelloPlugin] DeInitialized");
    }
}
```

### Load order

Clients download and load plugins in the order the server sends them. The server decides it like this:

1. **The `Order` property.** The server reads the constant returned by `public override uint Order` of
   the `RustBusterPlugin` class straight from the source (no compile needed). Comments and strings are
   ignored. Plain numbers, hex, `uint.MaxValue`/`uint.MinValue` and casts are understood. A plugin
   without an override, or with a computed value, gets `uint.MaxValue`. Lower values load first, ties
   are broken by name.
2. **`// #require`.** A plugin never loads before something it requires. A required plugin takes the
   lowest `Order` of the plugins that need it.
3. Hand-placed DLLs in `ClientPlugins` have no source to read, so they load first, alphabetically.

If any plugin declares an `Order`, the server logs the final order.

### `// #require`

Put the directive on its own line in any `.cs` file of the plugin:

```csharp
// #require HelloPlugin
```

- The name is another source plugin folder, or a hand-placed DLL in `ClientPlugins`.
- The required plugin is compiled first and its DLL is added as a reference, so you can use its types.
- Circular requirements are detected and logged. A plugin whose requirement failed to build is skipped.

### Automatic recompilation

- A watcher on `ClientPluginSources` waits 1 second after the last edit, then rebuilds what changed
  (and everything that requires it).
- A plugin is only rebuilt when the hash of its sources, references or compiler changed.
- After a rebuild the served plugin cache is refreshed, so newly connecting clients get the new DLL.
- Compile output and errors are written to `ClientPluginSources\.cache\<PluginName>.log`.
- If a plugin's source folder is removed, the DLL the server generated for it is deleted.
  Hand-placed DLLs are never touched.

### Which compiler is used

1. The full path in the `ClientPluginCompiler` setting of `Settings.ini` (empty by default), if the file
   exists.
2. The Roslyn `csc.exe` from the Visual Studio MSBuild install, found through `vswhere` (the same
   compiler Fougerite uses). The log shows `Using the MSBuild Roslyn compiler at ...`.
3. The legacy .NET Framework `csc.exe` (only supports old C# versions).
4. Mono `mcs.exe`.

### Quick test

1. Start the server once so the folders are created.
2. Copy all DLLs from the client's `Managed` folder except `mscorlib.dll` and `mscorwks.dll` into
   `ClientPluginSources\ClientSideReferences`.
3. Create `ClientPluginSources\HelloPlugin\HelloPlugin.cs` with the example above.
4. `ClientPlugins\HelloPlugin.dll` should appear. If not, read `ClientPluginSources\.cache\HelloPlugin.log`.

### Where to go next
- [`Settings-Ini-Reference.md`](Settings-Ini-Reference.md) - the `ClientPluginCompiler` setting.
- [`../README.md`](../README.md) - server plugin documentation index.
