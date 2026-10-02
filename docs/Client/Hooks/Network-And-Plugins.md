### Category
Network-And-Plugins hooks

### Description
Downloadable files, plugin load/unload, inter-plugin messaging ([`PluginMessaging`](../Classes/PluginMessaging.md)),
and client-ready lifecycle. See [Hooks/README.md](README.md) for the full table and the general
cancellation rules.

---

### `OnRustBusterClientFilesDownloaded` *(obsolete)*
`public delegate void RustBusterClientFilesDownloadedDelegate(RBDownloadableState en)`

Runs when all client RustBuster-downloadables were downloaded. **Obsolete** - use
`OnRustBusterClientFilesDownloaded2` instead, which provides a per-file status list and correctly
handles the "no downloadables" case. Not cancellable.

### `OnRustBusterClientFilesDownloaded2`
`public delegate void RustBusterClientFilesDownloaded2Delegate(ClientFilesDownloadedEvent e)`

Runs when all client RustBuster-downloadables were downloaded, or when the server declared there were
none. Not cancellable.
- `e.OverallState` - an `Hooks.RBDownloadableState`: `AllDownloaded`, `ThereWereNoDownloadables` (in
  which case `e.Files` is empty), or `DownloadFailed` (the download request to the server failed
  entirely; `e.Files` contains whatever partial records were available).
- `e.Files` - a `List<ClientFileStatus>`, one entry per file the server declared:
  `FileName`, `ClientPath` (relative sub-directory under `RB_Data`), `IsUrlDownload`, `State`.

```csharp
public void HandleFilesDownloaded(ClientFilesDownloadedEvent e)
{
    if (e.OverallState == Hooks.RBDownloadableState.DownloadFailed)
    {
        Hooks.LogData(Name, "Download phase failed, my content may be missing.");
        return;
    }

    foreach (ClientFileStatus file in e.Files)
    {
        Hooks.LogData(Name, "Got: " + file.FileName + " at " + file.ClientPath);
    }
}
```

### `OnRustBusterClientPluginsLoaded`
`public delegate void RustBusterClientPluginsLoadedDelegate()`

Runs once, after **all** client-side plugins have been loaded. No arguments. Not cancellable. Useful if
your plugin needs to call into another plugin and must be sure it has already initialized.

### `OnRustBusterClientReady`
`public delegate void RustBusterClientLoadedAndReadyDelegate()`

Runs once the client has loaded and is ready/spawned in on the server. No arguments. Not cancellable.

### `OnRustBusterClientPluginLoaded`
`public delegate void RustBusterClientPluginInitializedDelegate(string name, PluginState state)`

Runs when a client-side plugin is loaded. Not cancellable.
- `name` - the plugin's name.
- `state` - a `Hooks.PluginState`: `UnLoading`, `Unloaded`, `Loading`, or `Loaded`.

### `OnRustBusterClientPluginUnLoaded`
`public delegate void RustBusterClientPluginDeInitializedDelegate(string name, PluginState state)`

Runs when a client-side plugin is unloaded. Same parameters/enum as above. Not cancellable.

### `OnRustBusterMessageReceived`
`public delegate void RustBusterMessageReceivedDelegate(MessageReceivedEvent mr)`

Runs when a message is received from the server (the counterpart of
[`RustBusterPlugin.SendMessageToServer`](../Classes/RustBusterPlugin.md), but in the server→client
direction for your own custom server messages). Not cancellable - "responding" is done by setting
properties, not calling `Cancel()`.
- `mr.MessageByServer` - the message text sent by the server.
- `mr.PluginSender` - the name of the plugin on the server side that sent it.
- `mr.RespondingPluginName` - **set this** to your plugin's name to identify yourself as the handler.
- `mr.ReturnMessage` - **set this** to the text you want sent back as the response.
- `~` and `@` characters are stripped automatically from both `RespondingPluginName` and `ReturnMessage`
  (they are protocol separators) and a warning is logged if `ReturnMessage` contained any.

```csharp
public void HandleServerMessage(MessageReceivedEvent e)
{
    if (e.MessageByServer != "ping") return;

    e.RespondingPluginName = Name;
    e.ReturnMessage = "pong";
}
```

### `OnRustBusterPluginMessage`
`public delegate void RustBusterPluginMessageEventDelegate(PluginMessageEvent e)`

Runs when a plugin sends a message to another plugin via
[`PluginMessaging`](../Classes/PluginMessaging.md). **Cancellable** (rejects the message, the sender
gets `PluginMessageResponse.Rejected`). See [`PluginMessaging.md`](../Classes/PluginMessaging.md) for
the full `PluginMessageEvent`/`PluginMessageResult` reference and a complete example.
- `e.SenderName`, `e.ReceiverName`, `e.Message`.
- `e.Response` - set this to hand data back to the sender.
