### Classes reference

Index of every documented public server-side class under `RustBuster2016Server` (unless noted
otherwise).

| Class | Summary |
|---|---|
| [`API`](API.md) | The hub class: every server-side event, `RustBusterUserAPI`, downloadable-file management, user lookup helpers. |
| [`BanEvent`](BanEvent.md) | Cancellable event argument for `API.OnRustBusterUserBan`. |
| [`Message`](Message.md) | Event argument for `API.OnRustBusterUserMessage` (a custom message sent by a client plugin). |
| [`MessageResponse`](MessageResponse.md) | Event argument for `API.OnRustBusterClientResponse` (a client's reply to your own server message). |
| [`RBDownloadable`](RBDownloadable.md) | Describes a file to be pushed to RustBuster clients, registered via `API.AddFileToDownload`. |
| [`BannedHWsData`](BannedHWsData.md) (+ `BannedUser`) | Persistent storage of RustBuster-banned hardware IDs, exposed via `API.BannedHWsData`. |
| [`RustBusterUserCache`](RustBusterUserCache.md) (+ `CachedRBUser`) | Persistent per-HWID history (aliases, IPs, SteamIDs, login times) across sessions. |

Not documented separately (part of Fougerite, only referenced from RustBuster's server API):
`Fougerite.Player`, `Fougerite.Server`, `Fougerite.Module`, `Fougerite.Logger`, and other Fougerite types
that appear as parameters/return values throughout this API.
