### Class
`RustBuster2016Server.Message`

### Description
Event argument for [`API.OnRustBusterUserMessage`](API.md#static-event-rustbusterusermessagedelegate-onrustbusterusermessage),
fired when an RB client sends a custom message - typically via a client plugin's
[`RustBusterPlugin.SendMessageToServer`](../../Client/Classes/RustBusterPlugin.md). Not cancellable -
"responding" is done by setting `ReturnMessage`, not by blocking the message.

### Members

#### `string MessageByClient { get; }` (public readonly field)
The message text sent by the client.

#### `string PluginSender { get; }`
The name of the **client-side** plugin that sent the message (matches its
`RustBusterPlugin.Name`).

#### `string ReturnMessage { get; set; }`
The text that will be sent back to the client as the response. Defaults to `"Done"` if never set.
`~` and `@` characters are not allowed (they are protocol separators).

#### `string SenderIP { get; }`
The sender's IP address.

### Example

```csharp
public override void Initialize()
{
    API.OnRustBusterUserMessage += OnUserMessage;
}

public override void DeInitialize()
{
    API.OnRustBusterUserMessage -= OnUserMessage;
}

private void OnUserMessage(API.RustBusterUserAPI user, Message msg)
{
    if (msg.PluginSender != "MyFirstPlugin") return;

    if (msg.MessageByClient == "getStatus")
    {
        msg.ReturnMessage = IsPremium(user.SteamID) ? "premium" : "free";
    }
}
```

### See also
- [`MessageResponse.md`](MessageResponse.md) - the opposite direction: a client's reply to a message
  *you* sent it via `API.RustBusterUserAPI.SendMessage`/`SendMessageAsync`.
