### Class
`RustBuster2016Server.Caches.RustBusterUserCache` (+ `CachedRBUser`)

### Description
A persistent, per-HWID history cache maintained by RustBuster itself across server restarts: aliases,
IP addresses, SteamIDs, and login/logout timestamps for every machine that has ever connected. Useful
for alt-detection, ban-evasion checks, or just looking up a player's history. Loading/saving and login/
logout tracking are managed internally by RustBuster - plugins only consume the read-only query API and
`SaveToDisk`.

### `CachedRBUser` class
A single machine's (HWID's) cached profile.
- `string Name { get; set; }` - current/most recent display name.
- `List<string> Aliases { get; set; }` - every other name this machine has used in the past.
- `List<string> IPAddresses { get; set; }` - every IP this machine has connected from.
- `List<ulong> SteamIDs { get; set; }` - every SteamID that has logged in from this HWID.
- `DateTime? LastLogin { get; set; }` / `DateTime? LastLogout { get; set; }` - UTC, can be `null`.

### `RustBusterUserCache` members

#### `static RustBusterUserCache Instance { get; }`
The singleton instance.

#### `ConcurrentDictionary<string, CachedRBUser> CachedUsers { get; }`
Every cached profile, keyed by HWID.

#### `CachedRBUser GetUserByHWID(string hwid)`
O(1) lookup by HWID. Returns `null` if not found.

#### `CachedRBUser GetUserBySteamId(ulong steamId)` / `CachedRBUser GetUserBySteamId(string steamIdStr)`
Finds the first profile that has ever logged in with the given SteamID.

#### `CachedRBUser GetUserByName(string name)`
Finds the first profile whose **current** name matches exactly (case-insensitive).

#### `CachedRBUser GetUserByIP(string ip)`
Finds the first profile that has ever connected from the given IP.

#### `List<CachedRBUser> GetUsersByHWID(string hwid)`
Same as `GetUserByHWID`, wrapped in a list (API-compatibility convenience; there's only ever one profile
per HWID).

#### `List<CachedRBUser> GetUsersByIP(string ip)`
Every profile that has ever connected from the given IP - useful for spotting multiple accounts/machines
sharing a connection.

#### `List<CachedRBUser> GetUsersByNameContains(string namePart)`
Every profile whose current name contains `namePart` (case-insensitive substring match).

#### `void SaveToDisk()`
Forces an immediate save of the current cache state to disk. RustBuster calls this automatically on
shutdown/logout; you normally don't need to call it yourself.

### Example

```csharp
public override void Initialize()
{
    API.OnRustBusterLogin += OnUserLogin;
}

public override void DeInitialize()
{
    API.OnRustBusterLogin -= OnUserLogin;
}

private void OnUserLogin(API.RustBusterUserAPI user)
{
    CachedRBUser cached = RustBusterUserCache.Instance.GetUserByHWID(user.HardwareID);
    if (cached == null) return;

    if (cached.SteamIDs.Count > 1)
    {
        Fougerite.Logger.Log($"{user.Name}'s machine has {cached.SteamIDs.Count} known SteamIDs.");
    }

    List<CachedRBUser> sharingThisIP = RustBusterUserCache.Instance.GetUsersByIP(cached.IPAddresses.Last());
    if (sharingThisIP.Count > 1)
    {
        Fougerite.Logger.Log($"{sharingThisIP.Count} known machines share an IP with {user.Name}.");
    }
}
```
