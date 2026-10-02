### Class
`RustBuster2016.Loom` (note: not under the `RustBuster2016.API` namespace)

### Description
A `MonoBehaviour` that lets plugin code running on a background thread safely hop back onto the Unity
main thread (or run heavier work on a background thread with a bigger stack). Used internally by
[`AssetBundleLoader`](AssetBundleLoader.md), `Web`, `WinHttpClient`, and `ScriptWebSocket`, and directly
useful from any plugin callback that fires off the main thread.

### Members

#### `static Loom Current { get; }`
The singleton instance. Accessing it for the first time creates (and `DontDestroyOnLoad`s) the backing
`GameObject` if needed.

#### `int AmountOfThreads { get; }`
Instance property; number of currently queued sub-threads (`Loom.Current.AmountOfThreads`).

#### `const int MaxThreads = 30`
Maximum number of background threads that can be queued via `ExecuteInBiggerStackThread` at once -
do not raise this carelessly.

#### `static void QueueOnMainThread(Action action)`
#### `static void QueueOnMainThread(Action action, float time)`
Runs `action` on the main thread. If the calling thread is already the main thread, it runs immediately
and inline; otherwise it is queued and runs on the next `Update()`. The `time` overload additionally
delays execution by `time` seconds.

```csharp
WinHttpClient.GetInstance().MakeRequest(url, (status, body) =>
{
    // This callback runs on a ThreadPool thread.
    Loom.QueueOnMainThread(() =>
    {
        // Safe to touch GameObjects/UnityEngine APIs here.
        Hooks.LogData(Name, "Got: " + body);
    });
});
```

#### `static void ExecuteInBiggerStackThread(Action action)`
Runs `action` on a new background thread with a bigger stack than the default `ThreadPool` threads
provide. Useful for deep recursion / heavy parsing work that must not run on the main thread.

### See also
- [`Util.md`](Util.md) - `Util.IsMainThread`/`Util.MainThreadID` for checking which thread you're on
  before deciding whether to marshal through `Loom`.
