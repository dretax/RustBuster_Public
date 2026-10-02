### Class
`RustBuster2016.API.KeyBindingRegistry` (static)

### Description
Central registry for named key-binding actions declared by plugins. Instead of hardcoding a `KeyCode`
inside each plugin, a plugin registers a named action once (with a preferred default key). RustBuster
owns the rebinding table; the plugin just polls `IsPressed`/`IsHeld`/`IsReleased` each frame. When two
plugins request the same key, `GetConflicts` exposes the collision so a UI can warn the user.

Thread safety: all public methods lock internally. The registry is designed to be written once at plugin
load time and read many times on the main Unity thread every frame.

### `ActionBinding` (nested class)
Describes a single registered action. Instances are only ever obtained from the registry (no public
constructor).
- `string PluginName`, `string ActionName`, `string Description` - as passed to `Register`.
- `KeyCode DefaultKey` - the key the plugin suggested as its default.
- `KeyCode BoundKey` - the key currently assigned to this action; changed by `Rebind`.

### `BindingConflict` (nested class)
Describes a conflict between two or more actions that share the same `KeyCode`.
- `KeyCode Key` - the key bound to more than one action.
- `ActionBinding[] Actions` - all actions currently mapped to `Key`.

### Members

#### `static ActionBinding Register(string pluginName, string actionName, KeyCode defaultKey, string description = null)`
Registers a named action for `pluginName` with the given `defaultKey`. Idempotent: calling this a second
time for the same plugin+action pair is a no-op, and the **existing** binding is returned so player
customisations survive plugin reloads. Throws `ArgumentException` if `pluginName`/`actionName` is
null/empty.

#### `static bool Rebind(string pluginName, string actionName, KeyCode newKey)`
Reassigns the key bound to the specified action. Returns `false` if the action was never registered.

#### `static void Unregister(string pluginName)`
Removes every action registered by `pluginName`. **Call this from your plugin's `DeInitialize()`.**

#### `static ActionBinding Get(string pluginName, string actionName)`
Returns the binding, or `null` if not registered.

#### `static ActionBinding[] GetAll()`
A snapshot copy of every currently registered binding.

#### `static BindingConflict[] GetConflicts()`
Every group of two or more actions that currently share the same bound key. An empty array means no
conflicts.

#### `static bool IsHeld(string pluginName, string actionName)`
`true` while the bound key is held down this frame (`Input.GetKey`). `false` if not registered.

#### `static bool IsPressed(string pluginName, string actionName)`
`true` on the first frame the bound key is pressed (`Input.GetKeyDown`). `false` if not registered.

#### `static bool IsReleased(string pluginName, string actionName)`
`true` on the first frame the bound key is released (`Input.GetKeyUp`). `false` if not registered.

### Example

```csharp
public override void Initialize()
{
    KeyBindingRegistry.Register(Name, "ToggleMenu", KeyCode.F1, "Toggles my plugin's menu");

    // RustBusterPlugin has no built-in per-frame callback, so poll with a fast,
    // main-thread Normal Timer instead.
    Util.CreateTimer("MyFirstPlugin_KeyPoll", 50, OnKeyPoll, autoReset: true, pluginName: Name);
}

public override void DeInitialize()
{
    KeyBindingRegistry.Unregister(Name);
    Util.KillTimer("MyFirstPlugin_KeyPoll");
}

private void OnKeyPoll(RustBuster2016.API.Events.TimedEvent timer)
{
    if (KeyBindingRegistry.IsPressed(Name, "ToggleMenu"))
    {
        Hooks.LogData(Name, "Menu toggled!");
    }
}
```
