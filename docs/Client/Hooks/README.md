### Hooks reference

Every hook is a `public static event` on the sealed [`RustBuster2016.API.Hooks`](../Classes/Hooks.md)
class. Subscribe with `+=` in `Initialize()` and unsubscribe with `-=` in `DeInitialize()` - see the
[first plugin guide](../Guide/Writing-Your-First-Plugin.md) for why this matters.

Hooks whose event-argument class exposes `bool Cancelled { get; }` + `void Cancel()` (or `Cancel(bool)`)
are **cancellable**: calling `Cancel()` from any subscriber stops the native action the event describes.
Everything else is a pure notification - the action already happened (or cannot be stopped) by the time
your handler runs.

### Categories

| Category | Covers |
|---|---|
| [Player](Player.md) | Chat, movement, death/respawn, inventory UI toggle, interaction ("Use"), console. |
| [Combat](Combat.md) | Weapon fire (bullet/shotgun/bow/melee), explosions, damage. |
| [Items-And-World](Items-And-World.md) | Crafting, inventory cell clicks, item add/remove, research, loot windows, datablocks, networked object spawn/despawn. |
| [Vehicles-And-Attachments](Vehicles-And-Attachments.md) | [`LocalAttachment`](../Classes/LocalAttachment.md) seat mount/dismount, [`WaterSystem`](../Classes/WaterSystem.md) swimming/oxygen, [`LadderSystem`](../Classes/LadderSystem.md) climbing. |
| [Network-And-Plugins](Network-And-Plugins.md) | Downloadable files, plugin load/unload, inter-plugin messaging, client-ready lifecycle. |
| [WebSocket](WebSocket.md) | [`ScriptWebSocket`](../Classes/ScriptWebSocket.md) connect/message/close/error. |

### Full event table

| Hook | Argument type | Cancellable | Category |
|---|---|---|---|
| `OnRustBusterClientConsole` | `string` | No | [Player](Player.md) |
| `OnRustBusterClientMove` | `HumanController, Character, int` | No | [Player](Player.md) |
| `OnRustBusterClientChat` | `ChatEvent` | Yes | [Player](Player.md) |
| `OnRustBusterClientDeathScreen` | `DeathScreenEvent` | Yes | [Player](Player.md) |
| `OnRustBusterClientDeathScreenDisAppear` | `DeathScreenDisAppearEvent` | Yes | [Player](Player.md) |
| `OnRustBusterPlayerRespawn` | `PlayerRespawnEvent` | No | [Player](Player.md) |
| `OnRustBusterClientInventoryToggle` | `InventoryToggleEvent` | Yes | [Player](Player.md) |
| `OnRustBusterGuiReady` | `GuiReadyEvent` | No | [Player](Player.md) |
| `OnRustBusterPlayerUse` | `PlayerUseEvent` | Yes | [Player](Player.md) |
| `OnRustBusterPlayerInventoryRefreshed` | `PlayerInventoryRefreshedEvent` | No | [Player](Player.md) |
| `OnRustBusterMetabolismDamage` | `DamageEvent` | No | [Combat](Combat.md) |
| `OnRustBusterStructureDamage` | `DamageEvent` | No | [Combat](Combat.md) |
| `OnRustBusterC4Explosion` | `C4ExplosionEvent` | No | [Combat](Combat.md) |
| `OnRustBusterGrenadeExplode` | `GrenadeExplosionEvent` | No | [Combat](Combat.md) |
| `OnRustBusterSupplySignalExplode` | `SupplySignalExplosionEvent` | No | [Combat](Combat.md) |
| `OnRustBusterWeaponFire` | `BulletWeaponFireEvent` | Partial (`Cancel*` flags) | [Combat](Combat.md) |
| `OnRustBusterShotgunFire` | `ShotgunWeaponFireEvent` | Partial (`Cancel*` flags) | [Combat](Combat.md) |
| `OnRustBusterBowFire` | `BowFireEvent` | No | [Combat](Combat.md) |
| `OnRustBusterMeleeFire` | `MeleeWeaponFireEvent` | Partial (`Cancel*` flags) | [Combat](Combat.md) |
| `OnRustBusterClientItemUse` | `ItemUseEvent` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterClientCraft` | `CraftingEvent` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterCellClickEvent` | `ItemCellClickedEvent` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterCraftWindowClick` | `CraftWindowClickEvent` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterItemAdded` | `ItemAddedEvent` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterItemRemoved` | `ItemRemovedEvent` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterResearch` | `ResearchEvent` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterLootWindowOpen` | `LootWindowOpen` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterLootWindowClose` | `LootWindowClose` | Yes | [Items-And-World](Items-And-World.md) |
| `OnRustBusterDatablockDictionaryInitialized` | `DatablockDictionaryInitializedEvent` | No | [Items-And-World](Items-And-World.md) |
| `OnRustBusterNGCViewCreated` | `NGCViewCreatedEvent` | No | [Items-And-World](Items-And-World.md) |
| `OnRustBusterNGCViewDestroyed` | `NGCViewDestroyedEvent` | No | [Items-And-World](Items-And-World.md) |
| `OnRustBusterLocalPlayerAttach` | `LocalPlayerAttachEvent` | Yes | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterLocalPlayerAttached` | `LocalPlayerAttachedEvent` | No | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterLocalPlayerDetach` | `LocalPlayerDetachEvent` | Partial (only `DetachReason.Requested`) | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterLocalPlayerDetached` | `LocalPlayerDetachedEvent` | No | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterWaterStateChanged` | `WaterStateChangedEvent` | No | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterOxygenChanged` | `OxygenChangedEvent` | No | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterOutOfAir` | `OutOfAirEvent` | No | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterLadderStateChanged` | `LadderState, LadderState` | No | [Vehicles-And-Attachments](Vehicles-And-Attachments.md) |
| `OnRustBusterClientFilesDownloaded` *(obsolete)* | `RBDownloadableState` | No | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnRustBusterClientFilesDownloaded2` | `ClientFilesDownloadedEvent` | No | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnRustBusterClientPluginsLoaded` | *(none)* | No | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnRustBusterClientReady` | *(none)* | No | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnRustBusterClientPluginLoaded` | `string, PluginState` | No | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnRustBusterClientPluginUnLoaded` | `string, PluginState` | No | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnRustBusterMessageReceived` | `MessageReceivedEvent` | No (responds via properties) | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnRustBusterPluginMessage` | `PluginMessageEvent` | Yes | [Network-And-Plugins](Network-And-Plugins.md) |
| `OnWebSocketConnected` | `WebSocketEvent` | No | [WebSocket](WebSocket.md) |
| `OnWebSocketMessage` | `WebSocketEvent` | No | [WebSocket](WebSocket.md) |
| `OnWebSocketClosed` | `WebSocketEvent` | No | [WebSocket](WebSocket.md) |
| `OnWebSocketError` | `WebSocketEvent` | No | [WebSocket](WebSocket.md) |
