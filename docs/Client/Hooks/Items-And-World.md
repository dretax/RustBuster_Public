### Category
Items-And-World hooks

### Description
Crafting, inventory cell clicks, item add/remove, research, loot windows, datablocks, and networked
object spawn/despawn. See [Hooks/README.md](README.md) for the full table and the general cancellation
rules.

---

### `OnRustBusterClientItemUse`
`public delegate void RustBusterItemUseDelegate(ItemUseEvent ie)`

Runs when an item's server-side use action (RPC) is triggered. **Cancellable.**
- `ie.ItemRepresentation` - the item.
- `ie.Number` - the action number (e.g. `3` for a melee swing server action).
- `ie.BitStream` / `ie.NetworkMessageInfo` - the raw network payload/sender info.

### `OnRustBusterClientCraft`
`public delegate void RustBusterCraftDelegate(CraftingEvent ce)`

Runs when the local player starts crafting. **Cancellable.**
- `ce.CraftingInventory`, `ce.BlueprintDataBlock`, `ce.Amount`.

```csharp
public void HandleCraft(CraftingEvent e)
{
    if (e.BlueprintDataBlock.name == "Explosive Charge")
    {
        Hooks.LogData(Name, "Blocked crafting of an Explosive Charge.");
        e.Cancel();
    }
}
```

### `OnRustBusterCellClickEvent`
`public delegate void RustBusterItemCellClickDelegate(ItemCellClickedEvent ce)`

Runs when the player clicks/moves an item in the RPOS inventory UI. **Cancellable.**
- `ce.RPOS`, `ce.RPOSInventoryCell`.
- `ce.ClickType` - `SelectedHoldable`, `ItemCombined`, `ItemMerged`, or `ItemMoved`.

### `OnRustBusterCraftWindowClick`
`public delegate void RustBusterCraftWindowClickDelegate(CraftWindowClickEvent ce)`

Runs when the player selects an item from the crafting window list. **Cancellable.**
- `ce.GameObject` - the clicked craft-window entry's `GameObject`.
- `ce.RPOSCraftWindow` - the craft window instance.

### `OnRustBusterItemAdded`
`public delegate void RustBusterItemAddedDelegate(ItemAddedEvent ie)`

Runs when an item is being added/assigned to an inventory slot. **Cancellable** (blocks the native add).
- `ie.Assignment` - the raw `Inventory.Payload.Assignment` struct.
- `ie.Inventory`, `ie.Slot`.

### `OnRustBusterItemRemoved`
`public delegate void RustBusterItemRemovedDelegate(ItemRemovedEvent ie)`

Runs when an item is being removed from an inventory slot. **Cancellable.**
- `ie.Inventory`, `ie.Slot`, `ie.Match` (the specific `InventoryItem` being matched),
  `ie.MustMatch` (whether removal requires a strict reference match to succeed natively).

### `OnRustBusterResearch`
`public delegate void RustBusterResearchEventDelegate(ResearchEvent researchEvent)`

Runs when the local player uses a research kit on an item. **Cancellable.**
- `researchEvent.Item` - the research tool item (typed `InventoryItem`; cast to the concrete
  `ResearchToolItem<T>` yourself if you need tool-specific data - see the XML docs on the property for
  why it can't be typed more specifically).
- `researchEvent.Inventory`, `researchEvent.OtherItem` (item being researched),
  `researchEvent.DataBlock` (its datablock).

### `OnRustBusterLootWindowOpen`
`public delegate void RustBusterLootWindowOpenDelegate(LootWindowOpen lootWindowOpen)`

Runs when a loot window opens for the local player. **Cancellable** - `Cancel()` sends a `StopLooting`
RPC and clears the looter state so the window never fully opens.
- `lootWindowOpen.LootableObject`, `lootWindowOpen.NetworkPlayer`.

### `OnRustBusterLootWindowClose`
`public delegate void RustBusterLootWindowCloseDelegate(LootWindowClose lootWindowClose)`

Runs when a loot window is about to close. **Cancellable** (keeps it open).
- `lootWindowClose.LootableObject`.

### `OnRustBusterDatablockDictionaryInitialized`
`public delegate void RustBusterDatablockDictionaryInitializedDelegate(DatablockDictionaryInitializedEvent e)`

Fired once, after the game's `DatablockDictionary` has been fully populated. Not cancellable. The
reliable place to read all datablocks/loot spawn lists at startup, and the recommended time to call
[`DatablockManager.RegisterItem`](../Classes/DatablockManager.md).
- `e.All` - every registered `ItemDataBlock`, in load order.
- `e.LootSpawnLists` - every registered `LootSpawnList`, keyed by name.

### `OnRustBusterNGCViewCreated`
`public delegate void RustBusterNGCViewCreatedDelegate(NGCViewCreatedEvent e)`

Fired after the server instantiates a networked object (NGC "A" RPC) and `PostInstantiate` has
completed; the view is fully initialised. Not cancellable.
- `e.NGC`, `e.View` (the new `NGCView`), `e.Info`.

### `OnRustBusterNGCViewDestroyed`
`public delegate void RustBusterNGCViewDestroyedDelegate(NGCViewDestroyedEvent e)`

Fired just before the server destroys a networked object (NGC "D" RPC); the view/`GameObject` are still
alive when subscribers run. Not cancellable.
- `e.NGC`, `e.View` (about to be destroyed - don't store a long-lived reference), `e.Info`.
