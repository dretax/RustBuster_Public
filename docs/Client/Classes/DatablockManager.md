### Class
`RustBuster2016.DatablockManager` (note: not under the `RustBuster2016.API` namespace)

### Description
Lets a plugin register custom `ItemDataBlock` instances into the game's own `DatablockDictionary` at
runtime, without modifying the game's default item definitions. Useful for adding new item types that
your own asset bundle content refers to.

### Members

#### `static T CloneDatablock<T>(T template, string name, int uniqueID) where T : ItemDataBlock`
Clones an existing `ItemDataBlock` `ScriptableObject` and assigns it a new `name` and `uniqueID`. The
clone is **not** registered yet - pass it to `RegisterItem` afterwards. Returns `null` (and logs) if
`template` is `null`.

#### `static void RegisterItem(ItemDataBlock item)`
Adds `item` to RustBuster's custom-datablock collection and, if the game's `DatablockDictionary` has
already been initialized, also injects it directly into the dictionary's lookup tables (`_dataBlocks`,
`_dataBlocksByUniqueID`) and `_all` array so `DatablockDictionary.GetByName`/`GetByUniqueID`/`All` see it
immediately. No-op if `item` is `null`.

#### `static bool UnregisterItem(ItemDataBlock item)`
Removes `item` from the custom collection and, if present in `DatablockDictionary`, rebuilds its lookup
tables to keep indices consistent. Returns `true` if the item was found and removed.

#### `static void Clear()`
Unregisters every custom datablock that was registered through this manager. RustBuster itself calls
this automatically on plugin reload/shutdown; you normally don't need to call it yourself.

#### `static List<ItemDataBlock> GetAllDatablocks()`
A shallow copy of every datablock currently registered through this manager.

### Example

```csharp
private ItemDataBlock _customItem;

public override void Initialize()
{
    ItemDataBlock template = Util.ConvertNameToData("Wood");
    _customItem = DatablockManager.CloneDatablock(template, "MyFirstPlugin_SuperWood", 90001);
    DatablockManager.RegisterItem(_customItem);
}

public override void DeInitialize()
{
    DatablockManager.UnregisterItem(_customItem);
}
```

### Notes
- Only safe to call once the game's own `DatablockDictionary` initialization has happened (or is about
  to happen) - see `Hooks.OnRustBusterDatablockDictionaryInitialized` for a reliable point to do this.
- `uniqueID` must not collide with an existing item's ID.
