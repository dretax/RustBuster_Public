### Category
WebSocket hooks

### Description
Connect/message/close/error notifications for every [`ScriptWebSocket`](../Classes/ScriptWebSocket.md)
instance in the process. All four hooks share the same `WebSocketEvent` argument type and are **not**
cancellable - they are pure notifications fired after the fact. See
[`ScriptWebSocket.md`](../Classes/ScriptWebSocket.md) for the full class reference and a complete
connect/send/receive example.

Since these hooks are shared across every socket (yours and any other plugin's), always filter by
`WebSocketEvent.PluginName`/`SocketId` in your handler.

---

### `OnWebSocketConnected`
`public delegate void WebSocketEventHandlerDelegate(WebSocketEvent e)`

Fired once a socket's handshake completes successfully.

### `OnWebSocketMessage`
`public delegate void WebSocketEventHandlerDelegate(WebSocketEvent e)`

Fired for every text/binary message received on a socket. `e.Message` contains the payload.

### `OnWebSocketClosed`
`public delegate void WebSocketEventHandlerDelegate(WebSocketEvent e)`

Fired after `Close()`/`Dispose()` is called, or if the server closes the connection.
`e.ErrorMessage` contains the close reason.

### `OnWebSocketError`
`public delegate void WebSocketEventHandlerDelegate(WebSocketEvent e)`

Fired on send/connect/receive failures. `e.ErrorMessage` contains the failure reason.

### `WebSocketEvent` properties
- `string PluginName`, `string SocketId` - identify which `ScriptWebSocket` instance raised the event.
- `string Message` - the message payload (empty for connect/close/error events).
- `string ErrorMessage` - set on error/close events, otherwise `null`.

```csharp
private void OnMessage(WebSocketEvent e)
{
    if (e.PluginName != Name || e.SocketId != "main") return; // Not our socket.

    Hooks.LogData(Name, "Received: " + e.Message);
}
```
