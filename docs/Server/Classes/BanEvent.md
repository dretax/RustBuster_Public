### Class
`RustBuster2016Server.BanEvent`

### Description
Event argument for [`API.OnRustBusterUserBan`](API.md#static-event-rustbusteruserbandelegate-onrustbusteruserban),
fired when a user receives a RustBuster ban. **Cancellable.**

### Members

#### `string BanReason { get; }`
The reason RustBuster gave for the ban.

#### `API.RustBusterUserAPI RBUser { get; }`
The user being banned - see [`API.md`](API.md#rustbusteruserapi-nested-class).

#### `bool Cancelled { get; }`
`true` if a subscriber already called `Cancel()`.

#### `void Cancel()`
Cancels the ban.

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
    if (IsWhitelisted(e.RBUser.SteamID))
    {
        Fougerite.Logger.Log($"Blocked a ban against whitelisted user {e.RBUser.Name}: {e.BanReason}");
        e.Cancel();
    }
}
```
