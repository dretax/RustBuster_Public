### Class
`RustBuster2016.API.Stopper`

### Description
A tiny `IDisposable` stopwatch for finding slow code. Wrap a block of code in a `using (new Stopper(...))`
and a warning is logged automatically (through `Logger`) if the block took longer than the configured
threshold to run - nothing is logged if it finished quickly.

### Members

#### `Stopper(string type, string method, float warnSecs = 0.1f)`
Starts the stopwatch immediately. `type` and `method` are only used to label the log line (e.g. your
plugin name and the method being measured); `warnSecs` is the threshold in seconds (default `0.1`, i.e.
100ms).

#### `void Dispose()`
Stops the stopwatch. If the elapsed time exceeded `warnSecs`, logs
`[Stopper.{type}.{method}] Took: {seconds}s ({milliseconds}ms)`.

### Example

```csharp
public void HandleCraft(CraftingEvent e)
{
    using (new Stopper(Name, nameof(HandleCraft)))
    {
        // Anything slow in here (over 100ms by default) gets logged automatically.
        DoSomeExpensiveLookup();
    }
}
```
