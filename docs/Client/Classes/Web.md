### Class
`RustBuster2016.API.Web`

### Description
Helps plugins make simple web requests. Some certificates are not supported by Mono's own TLS stack, so
HTTPS URLs are automatically routed through [`WinHttpClient`](WinHttpClient.md) (native WinHTTP) while
plain HTTP URLs use `System.Net.WebClient`/`HttpWebRequest`.

### Members
`Web` is a singleton with a private constructor: get the instance via `GetInstance()`, then call the
instance methods below on it.

#### `static Web GetInstance()`
Returns the singleton instance.

#### `string GET(string url)`
#### `string POST(string url, string data, string contentType = "application/x-www-form-urlencoded")`
#### `string GETWithSSL(string url)`
#### `string POSTWithSSL(string url, string data)`
Synchronous HTTP GET/POST. **Blocks the calling thread until the request completes - calling these from
the main thread will freeze the game.** Prefer `CreateAsyncHTTPRequest` below. Returns the response body,
or an error message string on failure.

#### `void CreateAsyncHTTPRequest(string url, Action<int, string> callback, string method = "GET", string inputBody = null, Dictionary<string, string> additionalHeaders = null, string contentType = "application/x-www-form-urlencoded", float timeout = 0f, bool allowDecompression = false)`
The recommended way to make requests. HTTPS URLs use WinHTTP, HTTP URLs use `HttpWebRequest`. The
`callback(statusCode, responseBody)` runs **on a background thread** - use
[`Loom.QueueOnMainThread`](Loom.md) inside it if you need to touch Unity objects.

```csharp
Web.GetInstance().CreateAsyncHTTPRequest(
    "https://example.com/api",
    (statusCode, body) =>
    {
        if (statusCode != 200)
        {
            Hooks.LogData(Name, "Request failed: " + statusCode);
            return;
        }

        Loom.QueueOnMainThread(() => Hooks.LogData(Name, "Response: " + body));
    },
    method: "POST",
    inputBody: "{\"name\":\"test\"}",
    contentType: "application/json");
```

#### `void DoWithResponse(HttpWebRequest request, Action<HttpWebResponse> responseAction)`
Lower-level helper backing `CreateAsyncHTTPRequest`; use it directly only if you are constructing your
own `HttpWebRequest`.

### See also
- [`WinHttpClient.md`](WinHttpClient.md) - the native WinHTTP client used for HTTPS requests and
  file upload/download, usable directly as well.
