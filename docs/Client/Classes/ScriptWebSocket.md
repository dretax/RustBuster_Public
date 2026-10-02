### Class
`RustBuster2016.API.Tools.ScriptWebSocket` (+ `RustBuster2016.API.Events.WebSocketEvent`)

### Description
A WebSocket client (`ws://`/`wss://`) built on native WinHTTP (same underlying API as
[`WinHttpClient`](WinHttpClient.md)), for plugins that need a persistent, bidirectional connection
instead of request/response HTTP. Connects and receives on background threads and dispatches every
event back to the main thread via [`Loom`](Loom.md).

**Thread safety:** this class is **not** thread safe as a whole. `Connect()` and `Close()` should only
be called from the main thread, ideally always by the same plugin. `Send()` is safe to call from any
thread.

### `WebSocketEvent` class
Passed to every `Hooks` websocket event below.
- `string PluginName`, `string SocketId` - as passed to the `ScriptWebSocket` constructor.
- `string Message` - the message payload (empty for connect/close/error events).
- `string ErrorMessage` - set on error/close events, otherwise `null`.

### Members

#### `ScriptWebSocket(string pluginName, string socketId, string url, int bufferSize = 32768)`
Creates (but does not connect) a socket. `socketId` is any string you pick to tell multiple sockets
apart in the shared `Hooks` events (see below) - it is not required to be globally unique. `url` must be
`ws://` or `wss://`. `bufferSize` is the chunk size (bytes) used when reading incoming messages.

#### `string SocketId { get; }` / `string PluginName { get; }` / `string Url { get; }` / `int BufferSize { get; }`
Read back the constructor arguments.

#### `bool IsConnected { get; }`
`true` while the socket is connected and open.

#### `void Connect()`
Starts connecting asynchronously on a background thread. No-op (dispatches an error event) if the
object has already been disposed.

#### `bool Send(string message)`
Queues `message` to be sent as a UTF-8 text frame on a `ThreadPool` thread. Returns `false` immediately
(and dispatches an error event) if not currently connected, without throwing.

#### `void Close(string errorMessage)` / `void Close()`
Closes the connection and releases the native WinHTTP handles, firing `Hooks.SocketClosed`. Does **not**
dispose the object - you can call `Connect()` again afterwards. The parameterless overload closes with
`"Disconnected by plugin"`.

#### `void Dispose()`
Closes (if still connected) and permanently releases all native resources; the object cannot be reused
afterwards. Also runs automatically from the finalizer if you forget to call it, though you should still
call it explicitly from `DeInitialize()`.

### Related hooks
All fire on the main thread with a `WebSocketEvent` - see the [Hooks reference](../Hooks/README.md):
- `Hooks.OnWebSocketConnected` - fired once the handshake completes.
- `Hooks.OnWebSocketMessage` - fired for every received text/binary message (`WebSocketEvent.Message`).
- `Hooks.OnWebSocketClosed` - fired after `Close()`/`Dispose()`, or if the server closes the connection.
- `Hooks.OnWebSocketError` - fired on send/connect/receive failures (`WebSocketEvent.ErrorMessage`).

> These `Hooks` events are shared across every `ScriptWebSocket` instance in the process - filter by
> `WebSocketEvent.PluginName`/`SocketId` in your handler if you (or other plugins) have more than one
> socket open.

### Example

```csharp
private ScriptWebSocket _socket;

public override void Initialize()
{
    Hooks.OnWebSocketConnected += OnConnected;
    Hooks.OnWebSocketMessage += OnMessage;
    Hooks.OnWebSocketClosed += OnClosed;
    Hooks.OnWebSocketError += OnError;

    _socket = new ScriptWebSocket(Name, "main", "wss://example.com/socket");
    _socket.Connect();
}

public override void DeInitialize()
{
    Hooks.OnWebSocketConnected -= OnConnected;
    Hooks.OnWebSocketMessage -= OnMessage;
    Hooks.OnWebSocketClosed -= OnClosed;
    Hooks.OnWebSocketError -= OnError;

    _socket.Dispose();
}

private bool IsMine(WebSocketEvent e)
{
    return e.PluginName == Name && e.SocketId == "main";
}

private void OnConnected(WebSocketEvent e)
{
    if (!IsMine(e)) return;
    _socket.Send("{\"type\":\"hello\"}");
}

private void OnMessage(WebSocketEvent e)
{
    if (!IsMine(e)) return;
    Hooks.LogData(Name, "Received: " + e.Message);
}

private void OnClosed(WebSocketEvent e)
{
    if (!IsMine(e)) return;
    Hooks.LogData(Name, "Socket closed: " + e.ErrorMessage);
}

private void OnError(WebSocketEvent e)
{
    if (!IsMine(e)) return;
    Hooks.LogData(Name, "Socket error: " + e.ErrorMessage);
}
```
