### Class
`RustBuster2016.API.Zone3D`

### Description
A 3D volumetric area defined by a 2D polygon floor (ray-casting point-in-polygon test) plus a vertical
height range, with an AABB pre-check for fast rejection. Every instance auto-registers itself in a
global, thread-safe, name-keyed registry on construction, so plugins don't need to manage their own zone
collections. Intended for things like safe zones, PvP/building restriction areas, or any other
"is this point inside my area" check - you implement the actual gameplay effect (e.g. in your own hurt
event hook), `Zone3D` only answers the containment question.

### Instance members

#### `Zone3D(string name)`
Creates a new zone with the given default `PVP = true`, `Protected = false`, and registers it globally
under `name` (overwriting any existing zone with the same name).

#### `float MinY { get; set; }` (default `-1000`) / `float MaxY { get; set; }` (default `5000`)
Vertical bounds of the zone.

#### `List<Vector2> Points { get; set; }`
The polygon's vertices, in X/Z world space (`Vector2.y` maps to world Z).

#### `void Mark(Vector2 v)` / `void Mark(float x, float y)`
Appends a corner point to the polygon (`y` here is world Z) and updates the cached bounding box.

#### `bool Contains(Vector3 v)`
`true` if `v` is inside the zone: first an elevation check against `MinY`/`MaxY`, then an AABB check for
fast rejection, then a precise ray-casting point-in-polygon test.

#### `bool Protected { get; set; }`
Whether the zone is protected from damage/building. Purely a flag - enforce it yourself in your own
hurt/build hook.

#### `bool PVP { get; set; }`
Whether PvP is allowed in the zone. Also purely a flag - enforce it yourself.

### Static (registry) members

#### `static ConcurrentDictionary<string, Zone3D> GetZones()`
The full, thread-safe registry of every currently registered zone.

#### `static bool Exists(string name)` / `static Zone3D Get(string name)`
Lookup by name; `Get` returns `null` if not found.

#### `static bool Remove(string name)`
Removes a zone from the registry. Returns `true` if it existed.

#### `static void Clear()`
Removes every registered zone.

#### `static List<Zone3D> GetZonesAt(Vector3 v)`
Every zone that contains `v` - useful for overlapping protection layers.

#### `static Zone3D GetFirstZoneAt(Vector3 v)`
The first zone that contains `v`, or `null`. Optimized for high-frequency checks where only the first
match matters.

### Example

```csharp
public override void Initialize()
{
    Zone3D spawn = new Zone3D("SpawnSafeZone")
    {
        PVP = false,
        Protected = true,
        MinY = -10f,
        MaxY = 200f
    };
    spawn.Mark(-50f, -50f);
    spawn.Mark(-50f, 50f);
    spawn.Mark(50f, 50f);
    spawn.Mark(50f, -50f);
}

public override void DeInitialize()
{
    Zone3D.Remove("SpawnSafeZone");
}

// Example usage elsewhere in the plugin, e.g. your own damage hook:
private bool IsInSafeZone(Vector3 position)
{
    Zone3D zone = Zone3D.Get("SpawnSafeZone");
    return zone != null && zone.Contains(position);
}
```
