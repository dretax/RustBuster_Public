### Class
`RustBuster2016Server.MessageResponse`

### Description
Event argument for [`API.OnRustBusterClientResponse`](API.md#static-event-rustbusterclientresponsedelegate-onrustbusterclientresponse),
fired when an RB client responds to a message **you** sent it via
`API.RustBusterUserAPI.SendMessage`/`SendMessageAsync`. Not cancellable, purely informational.

### Members

#### `string Response { get; }`
The client plugin's response message.

#### `string Pluginsender { get; }`
The name of the **client-side** plugin that responded.

#### `string SenderIP { get; }`
The sender's IP address.

### Example

```csharp
public override void Initialize()
{
    API.OnRustBusterClientResponse += OnClientResponse;
}

public override void DeInitialize()
{
    API.OnRustBusterClientResponse -= OnClientResponse;
}

private void OnClientResponse(API.RustBusterUserAPI user, MessageResponse response)
{
    if (response.Pluginsender != "MyFirstPlugin") return;

    Fougerite.Logger.Log($"{user.Name} replied: {response.Response}");
}
```

### See also
- [`Message.md`](Message.md) - the opposite direction: a custom message sent *by* a client plugin to
  your server plugin.
