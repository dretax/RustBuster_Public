### Class
`RustBuster2016.API.Tools.PluginMessaging` (+ `PluginMessageResult`,
`RustBuster2016.API.Events.PluginMessageEvent`, `RustBuster2016.API.Events.PluginMessageResponse`)

### Description
API for inter-plugin communication: lets one client-side plugin send an arbitrary object payload to
another client-side plugin by name, and get a response/status back, without either plugin needing a
direct reference to the other. The receiving side handles it through
`Hooks.OnRustBusterPluginMessage` (see the [Hooks reference](../Hooks/README.md)).

### `PluginMessageResponse` enum
- `Success` - delivered successfully.
- `TargetNotFound` - no plugin with that name is loaded.
- `Error` - an internal exception occurred while dispatching.
- `Rejected` - delivered, but the target explicitly rejected it via `PluginMessageEvent.Cancel()`.

### `PluginMessageEvent` class
Received by the target plugin's `Hooks.OnRustBusterPluginMessage` handler.
- `string SenderName`, `string ReceiverName`, `object Message` - the payload as sent.
- `object Response { get; set; }` - set this to hand data back to the sender.
- `bool Cancelled { get; }` / `void Cancel()` - reject the message; the sender gets back
  `PluginMessageResponse.Rejected`.

### `PluginMessageResult` class
Returned to the sender.
- `PluginMessageResponse Status { get; }`
- `PluginMessageEvent Event { get; }` - the same event instance the receiver saw, so you can read
  `Event.Response`/`Event.Cancelled` after the call.

### `PluginMessaging` static members

#### `static PluginMessageResult Send(string sender, string targetName, object message)`
Sends synchronously and returns the result immediately. Runs on the calling thread - if called from the
main thread, the receiving handler also runs synchronously on the main thread.

#### `static void SendAsync(string sender, string targetName, object message, Action<PluginMessageResult> callback, bool runInThreadPool = true)`
Sends asynchronously; `callback` is always invoked on the main thread with the result. When
`runInThreadPool` is `true` (default), the hook dispatch itself happens on a `ThreadPool` thread -
pass `false` to dispatch on the calling thread instead (but the callback still always marshals back to
the main thread via `Loom`).

### Example

Receiving plugin:
```csharp
public override void Initialize()
{
    Hooks.OnRustBusterPluginMessage += OnPluginMessage;
}

public override void DeInitialize()
{
    Hooks.OnRustBusterPluginMessage -= OnPluginMessage;
}

private void OnPluginMessage(PluginMessageEvent e)
{
    if (e.ReceiverName != Name) return;

    string command = e.Message as string;
    if (command == "ping")
    {
        e.Response = "pong";
        return;
    }

    e.Cancel(); // Anything we don't understand gets rejected.
}
```

Sending plugin:
```csharp
PluginMessaging.SendAsync(Name, "OtherPlugin", "ping", result =>
{
    if (result.Status == PluginMessageResponse.Success)
    {
        Hooks.LogData(Name, "OtherPlugin replied: " + result.Event.Response);
    }
    else
    {
        Hooks.LogData(Name, "Message failed: " + result.Status);
    }
});
```
