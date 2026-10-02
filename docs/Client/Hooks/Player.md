### Category
Player hooks

### Description
Chat, movement, death/respawn, inventory UI toggle, world interaction ("Use"), and console output. See
[Hooks/README.md](README.md) for the full table and the general cancellation rules.

---

### `OnRustBusterClientConsole`
`public delegate void RustBusterClientConsoleDelegate(string msg)`

Runs when a console message is sent or received. Not cancellable.

```csharp
Hooks.OnRustBusterClientConsole += msg => Hooks.LogData(Name, "Console: " + msg);
```

### `OnRustBusterClientMove`
`public delegate void RustBusterClientMoveDelegate(HumanController hc, Character ch, int num)`

Runs while the player is moving. Not cancellable.

### `OnRustBusterClientChat`
`public delegate void RustBusterClientChatDelegate(ChatEvent ce)`

Runs when the local player sends a chat message. **Cancellable.**
- `ce.ChatUI` - the `ChatUI` instance; read all chat-box state from here.
- `ce.Cancel(bool makeChatUIDisappear)` - prevents the message from being sent; optionally also closes
  the chat UI immediately.

```csharp
public void HandleChat(ChatEvent e)
{
    if (e.ChatUI.ToString().Contains("badword"))
    {
        e.Cancel(true);
    }
}
```

### `OnRustBusterClientDeathScreen`
`public delegate void RustBusterDeathScreenDelegate(DeathScreenEvent de)`

Runs when the death/respawn screen is shown. **Cancellable** (prevents the screen from appearing / the
respawn from processing).
- `de.DeathScreen` - the `DeathScreen` instance, modify its UI freely.
- `de.DeathScreenType` - `Died`, `RespawnRequest`, or `RespawnRequestHome`.

### `OnRustBusterClientDeathScreenDisAppear`
`public delegate void RustBusterDeathScreenDisAppearDelegate(DeathScreenDisAppearEvent de)`

Runs when the death screen is about to disappear (the player has respawned). **Cancellable** (keeps the
death screen visible, blocking the hide animation).
- `de.DeathScreen` - the `DeathScreen` instance.

### `OnRustBusterPlayerRespawn`
`public delegate void RustBusterPlayerRespawnDelegate(PlayerRespawnEvent pe)`

Runs once respawn setup fully completes. Not cancellable.
- `pe.HumanController` - the respawning player's `HumanController`.
- `pe.NetworkMessageInfo` - the respawn RPC's network info.

### `OnRustBusterClientInventoryToggle`
`public delegate void RustBusterInventoryToggleDelegate(InventoryToggleEvent te)`

Runs when the player's inventory (RPOS) is about to be toggled. **Cancellable.**
- `te.RPOS` - the `RPOS` instance.
- `te.Value` - `true`/`false`, the state the inventory wants to toggle to.

```csharp
public void HandleInventoryToggle(InventoryToggleEvent e)
{
    if (_tradingInProgress)
    {
        e.Cancel(); // Don't let the player close/open their inventory mid-trade.
    }
}
```

### `OnRustBusterGuiReady`
`public delegate void RustBusterGuiReadyDelegate(GuiReadyEvent ge)`

Runs once the RPOS (HUD/GUI) has awoken. Not cancellable. Good place to build your own custom UI that
needs to attach to the HUD.
- `ge.RPOS` - the `RPOS` instance that just woke up.

### `OnRustBusterPlayerUse`
`public delegate void RustBusterPlayerUseDelegate(PlayerUseEvent e)`

Fired when the player presses Use (`E`) on an interactable (door, loot box, etc.). **Cancellable**
(blocks the interaction entirely).
- `e.Contextual` - the interactable the player aimed at.
- `e.Implementor` - the `Facepunch.MonoBehaviour` implementing it (e.g. the door/crate component).
- `e.NetEntityID` - the resolved network entity ID.

```csharp
public void HandlePlayerUse(PlayerUseEvent e)
{
    if (IsLockedByMyPlugin(e.NetEntityID))
    {
        e.Cancel();
    }
}
```

### `OnRustBusterPlayerInventoryRefreshed`
`public delegate void RustBusterPlayerInventoryRefreshedDelegate(PlayerInventoryRefreshedEvent e)`

Runs once the local player's inventory has been refreshed by the server. Not cancellable. Good place to
react to item additions/removals/slot updates in bulk rather than per-item.
- `e.Inventory` - the refreshed `PlayerInventory`.
