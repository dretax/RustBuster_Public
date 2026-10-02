### Class
`RustBuster2016.API.IniParser`

### Description
A minimal, classic `[section]` / `key=value` `.ini` file reader/writer, fully loaded into memory on
construction. Lines starting with `;` are treated as comments and preserved on save.

### Members

#### `IniParser(string iniPath)`
Loads and parses the file at `iniPath`. Throws `FileNotFoundException` if the file does not exist.
Keys outside any `[section]` header are placed under an implicit `"ROOT"` section.

#### `string[] Sections { get; }`
All section names currently known, in file order.

#### `int Count()`
Number of sections (`Sections.Length`).

#### `string[] EnumSection(string sectionName)`
All (non-comment) key names within `sectionName`.

#### `string GetSetting(string sectionName, string settingName)`
Returns the raw string value, or `null` if not found.

#### `bool GetBoolSetting(string sectionName, string settingName)`
Parses the value as a `bool`; returns `false` if missing or unparsable.

#### `bool isCommandOn(string cmdName)`
Shorthand for `GetBoolSetting("Commands", cmdName)`.

#### `void AddSetting(string sectionName, string settingName)` / `void AddSetting(string sectionName, string settingName, string settingValue)`
Adds a setting, overwriting any existing entry with the same section/key.

#### `void SetSetting(string sectionName, string settingName, string value)`
Updates an **existing** setting's value; silently does nothing if the key is not already present (use
`AddSetting` to create new keys).

#### `void DeleteSetting(string sectionName, string settingName)`
Removes a setting.

#### `bool ContainsSetting(string sectionName, string settingName)` / `bool ContainsValue(string valueName)`
Existence checks.

#### `void Save()`
Writes the current in-memory state back to the original file path.

#### `void SaveSettings(string newFilePath)`
Writes the current in-memory state to an arbitrary path instead.

### Example

```csharp
private IniParser _config;

public override void Initialize()
{
    string path = Path.Combine(Hooks.GameDirectory, "MyFirstPlugin.ini");
    if (!File.Exists(path))
    {
        File.WriteAllText(path, "[Settings]\r\nWelcomeMessage=Hello!\r\n");
    }

    _config = new IniParser(path);
    string message = _config.GetSetting("Settings", "WelcomeMessage");
    Hooks.LogData(Name, message);
}
```
