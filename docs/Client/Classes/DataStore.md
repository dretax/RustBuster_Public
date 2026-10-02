### Class
`RustBuster2016.API.DataStore` / `RustBuster2016.API.DataStoreContext`

### Description
A thread-safe, JSON-backed (via Newtonsoft.Json) key/value storage system, organised into named tables.
There is exactly one instance, backed by a single `Datastore.ds` file on disk; global and per-server
data both live in that file, isolated internally by namespacing table names rather than by separate
files.

### `DataStoreContext` enum
Selects which partition an operation targets:
- `Global` - shared and accessible across all servers.
- `Server` - isolated to the server the client is currently connected to. Throws
  `InvalidOperationException` if there is no active server connection when used.

### Members
All instance methods take an optional trailing `DataStoreContext context = DataStoreContext.Global`
parameter.

#### `static DataStore GetInstance()`
Returns the singleton instance.

#### `void Add(string tablename, object key, object val, DataStoreContext context = Global)`
Adds/overwrites a key-value pair in the given table. Creates the table if it doesn't exist. No-op if
`key` is `null`.

#### `bool ContainsKey(string tablename, object key, DataStoreContext context = Global)`
#### `bool ContainsValue(string tablename, object val, DataStoreContext context = Global)`
Existence checks.

#### `object Get(string tablename, object key, DataStoreContext context = Global)`
Retrieves the value for `key`, or `null` if not found.

#### `void Remove(string tablename, object key, DataStoreContext context = Global)`
Removes a key from a table.

#### `int Count(string tablename, DataStoreContext context = Global)`
Number of entries in a table.

#### `object[] Keys(string tablename, DataStoreContext context = Global)` / `object[] Values(string tablename, DataStoreContext context = Global)`
All keys / all values currently in a table.

#### `Hashtable GetTable(string tablename, DataStoreContext context = Global)`
Direct access to the underlying `Hashtable` for a table.

#### `string[] GetTableNames(DataStoreContext context = Global)`
Names of every table currently known in the requested partition.

#### `void Flush(string tablename, DataStoreContext context = Global)`
Removes every entry from a table (but keeps the empty table around).

#### `void Load()` / `void Save()`
Loads from / persists to `Datastore.ds` on disk. `Vector3` values are automatically stringified/parsed
on the way in/out via the internal `UnityTypeConverter`.

### Example

```csharp
private DataStore _store;

public override void Initialize()
{
    _store = DataStore.GetInstance();
    _store.Load();

    if (!_store.ContainsKey("MyFirstPlugin_Settings", "welcomeMessage"))
    {
        _store.Add("MyFirstPlugin_Settings", "welcomeMessage", "Welcome back!");
        _store.Save();
    }
}

public void OnClientReadyHandler()
{
    string message = (string)_store.Get("MyFirstPlugin_Settings", "welcomeMessage");
    Hooks.LogData(Name, message);
}
```

Using the `Server` context keeps data isolated per-server automatically:

```csharp
_store.Add("Stats", "kills", 0, DataStoreContext.Server);
```
