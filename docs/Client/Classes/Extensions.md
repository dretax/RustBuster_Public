### Class
`RustBuster2016.API.Extensions`

### Description
A small set of C# extension methods for `string` and `GameObject`.

### Members

#### `static string Combine(this string path1, string path2)`
`Path.Combine` that also handles the case where `path2` is itself rooted (e.g. starts with `\` or `/`):
the leading separator is trimmed first, so the result is always `path1` + `path2` rather than
`Path.Combine` discarding `path1` entirely.

```csharp
string full = Hooks.GameDirectory.Combine("MyFirstPlugin\\config.ini");
```

#### `static bool GetComponent<T>(this GameObject gameObject, out T component) where T : Component`
A `TryGetComponent`-style version of `GameObject.GetComponent<T>` (not available in this old Unity/Mono
runtime). Returns `false` (and sets `component` to `default(T)`) if `gameObject` is `null` or doesn't
have the component, instead of returning `null`/throwing.

```csharp
if (someGameObject.GetComponent(out Rigidbody rb))
{
    rb.velocity = Vector3.zero;
}
```

#### `static bool IsNullOrWhiteSpace(this string value)`
Equivalent of `string.IsNullOrWhiteSpace` (not available on this old .NET target) - `true` if `value` is
`null`, empty, or consists only of whitespace characters.

```csharp
if (someInput.IsNullOrWhiteSpace())
{
    Hooks.LogData(Name, "Input was empty.");
}
```
