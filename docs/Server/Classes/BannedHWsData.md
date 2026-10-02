### Class
`RustBuster2016Server.Caches.BannedHWsData` (+ `BannedUser`)

### Description
Persistent, JSON-backed storage of RustBuster-banned hardware IDs. The live instance RustBuster itself
maintains is exposed read/write via [`API.BannedHWsData`](API.md#static-bannedhwsdata-bannedhwsdata-get)
- you don't normally construct your own instance, except to load a standalone file.

### `BannedUser` class
A single ban record.
- `ulong SteamID { get; set; }` - the Steam ID of the banned user.
- `string Name { get; set; }` - their last known display name.
- `DateTime BannedAt { get; set; }` - UTC timestamp of the ban (defaults to `DateTime.UtcNow` at creation).

### `BannedHWsData` members

#### `ConcurrentDictionary<string, BannedUser> Bans { get; }`
Every ban, keyed by HWID.

#### `bool AddBan(string hwid, ulong steamId, string name, DateTime bannedAt)`
Adds/overwrites a ban for `hwid` and immediately persists to disk (`Save()`). Always returns `true`.

#### `bool RemoveBan(string hwid)`
Removes a ban and persists to disk if it existed. Returns `false` if `hwid` wasn't banned.

#### `bool Contains(string hwid)`
Checks if `hwid` is currently banned.

#### `void Save()`
Serializes the current ban list to disk as JSON.

#### `static BannedHWsData Load(string filePath)`
Loads (or creates, if missing) a `BannedHWsData` instance from a JSON file at `filePath`. Transparently
migrates the legacy array-based file format if detected.

### Example

```csharp
public override void Initialize()
{
    API.OnRustBusterUserBan += OnUserBan;
}

public override void DeInitialize()
{
    API.OnRustBusterUserBan -= OnUserBan;
}

private void OnUserBan(BanEvent e)
{
    if (!API.BannedHWsData.Contains(e.RBUser.HardwareID))
    {
        Fougerite.Logger.Log("New HWID ban recorded: " + e.RBUser.HardwareID);
    }
}

public void ManuallyBanHWID(string hwid, ulong steamId, string name)
{
    API.BannedHWsData.AddBan(hwid, steamId, name, DateTime.UtcNow);
}
```
