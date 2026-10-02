### Class
`RustBuster2016.API.WinHttpClient`

### Description
Since UnityEngine runs on an old Mono runtime and updating its security DLLs would take a lot of work,
RustBuster talks HTTPS through native **WinHTTP** via P/Invoke instead, which supports modern TLS so
sites enforcing TLS 1.3 don't reject the client. Should also work under Wine on Linux.

`const int BUFFER_SIZE` = 4 KB chunk size for reading responses. `const int MAX_SIZE` = 10 MB maximum
response body size (change the class field if you need more).

### Members
`WinHttpClient` is a singleton with a private constructor: get the instance via `GetInstance()`, then
call the instance methods below on it.

#### `static WinHttpClient GetInstance()`

#### `void MakeRequest(string url, Action<int, string> callback, string method = "GET", string inputBody = null, Dictionary<string, string> additionalHeaders = null, string contentType = "application/x-www-form-urlencoded", float timeout = 0f)`
Queues a request to run asynchronously on a `ThreadPool` thread. Non-blocking for the caller;
`callback(statusCode, responseBody)` runs on that pool thread - use
[`Loom.QueueOnMainThread`](Loom.md) if you need to touch Unity objects.

```csharp
WinHttpClient.GetInstance().MakeRequest("https://example.com/api", (status, body) =>
{
    Loom.QueueOnMainThread(() => Hooks.LogData(Name, status + ": " + body));
});
```

#### `string GetBlocking(string url, float timeout = 5f)`
#### `string PostBlocking(string url, string inputBody, string contentType = "application/x-www-form-urlencoded", float timeout = 5f)`
Synchronous variants. **Calling from the main thread may freeze the game** for up to `timeout + 1`
seconds. Returns the response body on 2xx, `"HTTP {statusCode}"` on other status codes, `"Timeout"` on
timeout, or `"Error {exception}"` on failure.

#### `bool DownloadFileBlocking(string url, string destinationPath, float timeout = 30f)`
#### `bool DownloadFileBlocking(string url, string destinationPath, out string error, float timeout = 30f)`
Synchronously downloads binary content directly into a file. Returns `true` on success; the `out error`
overload also reports the failure reason.

#### `void DownloadFileAsync(string url, string destinationPath, Action<bool, string> onComplete, float timeout = 30f)`
Asynchronous download; `onComplete(success, error)` runs on the main thread.

#### `bool UploadFileBlocking(string url, string filePath, float timeout = 30f)`
#### `bool UploadFileBlocking(string url, string filePath, out string error, float timeout = 30f)`
Synchronously POSTs the binary content of a file.

#### `void UploadFileAsync(string url, string filePath, Action<bool, string> onComplete, float timeout = 30f)`
Asynchronous upload; `onComplete(success, error)` runs on the main thread.

> `InitSession()`/`CloseSession()`/`DoWinHttpRequest(...)` exist but are plumbing used internally by the
> methods above - plugins do not need to call them directly.

### Notes
- SSL certificate validation is disabled.
- Response size for `MakeRequest`/blocking calls is capped at `MAX_SIZE` (10 MB by default).
