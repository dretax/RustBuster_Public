### Class
`RustBuster2016.API.WaterSystem`

### Description
Optional, **disabled by default** client-side swimming/buoyancy simulation for Rust Legacy, which has no
built-in water interaction. Detects water purely by testing points against the bounds of every
`WaterBase` object on the level (no colliders, no physics queries), moves the player by writing
`CCMotor.velocity` (the same technique [`LocalAttachment`](LocalAttachment.md) uses for seats), and
drives an underwater visual/oxygen package (tint, fog, camera far-clip, oxygen bar).

The driver only exists while the local player is alive and spawned; it starts/stops itself based on an
internal aliveness heartbeat and shuts down automatically after `MaxConsecutiveFailures` to avoid a
broken feature spamming the log every frame.

### `WaterState` enum
- `None = 0` - dry.
- `Wading = 1` - feet in the water, head above it; the player floats at the surface.
- `Submerged = 2` - eyes below the surface; free vertical movement, air drains.

### Enabling it
```csharp
public override void Initialize()
{
    WaterSystem.Enabled = true;
}
```
Everything else below is optional tuning; the defaults are reasonable for a stock Rust Legacy map.

### Tunables (all `public static`, safe to set from `Initialize()`)

**Movement & detection**
- `const ushort SwimFlag = 32768` - `stateFlags` bit published to mark "this player is swimming".
- `float SwimSpeed` (6) - vertical swim speed, m/s.
- `float FloatEyeOffset` (0.3) - how far the eyes sit above the waterline when floating. Must stay
  positive.
- `float SurfaceHeadroom` (3) - how far above the waterline a swimmer can climb before gravity resumes.
- `float SurfaceExitMargin` (0.2) - how far the eyes must rise above the surface before counting as
  surfaced (avoids state chatter on the boundary).
- `float SubmergeTriggerOffset` (-0.15) - raises/lowers the point the underwater effect switches on;
  positive triggers earlier/shallower.
- `bool UseCameraAsFace` (true) - samples the player camera as the "face" position (exact); only
  disabled automatically if no camera can be found, falling back to `HeadHeight`.
- `float HeadHeight` (1.3) / `float FaceSampleOffset` (0) - face-height fallback and fine offset.
- `float SurfaceThrust` (20) / `float SurfaceThrustCooldown` (1.2) - upward speed boost (and its
  cooldown) given when pushing off the surface, so swimmers can climb onto shore.
- `float FloatSpring` (5) - how hard the float spring pulls a wading player toward the surface.
- `float SurfaceBias` (-0.1) - offset applied to a water volume's top when computing surface height
  (water meshes have thickness).
- `bool ClampToWaterLine` (true) - uses the server's single global `WaterLine.Height` as the
  authoritative surface instead of each volume's own renderer bounds, so client and server agree on
  depth (turn off only if the server also knows per-volume heights).
- `float RebuildInterval` (20) - seconds between water rescans while none has been found yet.
- `int MaxConsecutiveFailures` (15) - consecutive `Tick` exceptions before the whole system disables
  itself for the session.
- `bool DebugLog` (false) - logs swim state once a second while in/near water.
- `float AliveTimeout` (3) - seconds without an aliveness heartbeat before the driver shuts down.
- `KeyCode AscendKey` (`Space`) / `KeyCode DescendKey` (`LeftControl`).

**Oxygen**
- `float OxygenSeconds` (60) - total seconds of air.
- `float OxygenRefillMultiplier` (3) - refill rate relative to drain rate.
- `float OxygenEventStep` (0.01) - minimum `Oxygen01` change before `Hooks.OnRustBusterOxygenChanged`
  fires again.
- `bool ServerAuthoritativeOxygen { get; }` - `true` once the server has sent at least one oxygen value
  recently (see `ApplyServerOxygen`); while `false` the client predicts on its own.
- `float ServerOxygenTimeout` (6) - seconds without a server update before falling back to prediction.
- `float OxygenCorrectionThreshold` (0.08) - kept for compatibility, no longer used.
- `static void ApplyServerOxygen(float oxygen)` - feeds a server-authoritative `0..1` oxygen value in
  (e.g. from your own RPC); the client keeps predicting locally between updates and adopts the server
  value outright on the very first call.

**Underwater visuals**
- `bool DrawUnderwaterTint` (true) / `Color UnderwaterTint`.
- `bool DrawOxygenBar` (true), `float OxygenBarWidth` (220) / `OxygenBarHeight` (14) /
  `OxygenBarBottomMargin` (90) - turn `DrawOxygenBar` off if your own plugin draws a custom bar from
  `Hooks.OnRustBusterOxygenChanged`.
- `bool UnderwaterFog` (true), `FogMode UnderwaterFogMode` (`ExponentialSquared`),
  `float UnderwaterFogDensity` (0.015), `Color UnderwaterFogColor`, `float UnderwaterFogStart` (0),
  `float UnderwaterFogEnd` (60, fixed visibility distance at every depth).
- `bool HideSkyDomesUnderwater` (true) - hides TOD sky domes while submerged (removes the white glow at
  the world edge).
- `bool UseUnderwaterShell` (true) / `float ShellRadiusFactor` (0.8, must stay below 1.0) - an
  inside-out cube around the camera so there's nothing to see past the fog.
- `bool ClampFarClipUnderwater` (false) / `float FarClipFogMultiplier` (1) - alternative/complementary
  fix that pulls the camera's far clip plane in while submerged.
- `bool PaintCameraBackground` (true) - paints the camera's clear colour with the fog colour while
  submerged.

### Read-only state
- `static WaterState State { get; }`, `static bool IsSubmerged { get; }`, `static bool IsInWater { get; }`,
  `static bool IsActive { get; }` (driver running, i.e. alive and spawned).
- `static float Oxygen01 { get; }` (0 to 1) / `static float OxygenSecondsRemaining { get; }`.
- `static int VolumeCount { get; }` - water volumes found on the current level.
- `static float WaterLineHeight { get; }` - the server's global waterline, or `float.MinValue` if the
  level has none.

### Querying water without swimming
#### `static void RebuildVolumes()`
Forces an immediate rescan for `WaterBase` objects on the level (normally automatic).

#### `static bool IsPointInWater(Vector3 point)`
`true` if `point` is below the detected surface and not inside a registered dry region.

#### `static float GetSurfaceY(Vector3 point)`
The water surface height above `point`, or `float.MinValue` if `point` isn't over any water.

#### `static void AddExclusion(Bounds bounds)` / `static bool RemoveExclusion(Bounds bounds)` / `static void ClearExclusions()` / `static int ExclusionCount { get; }` / `static bool IsPointExcluded(Vector3 point)`
Marks a world-space region as dry regardless of the waterline - useful for a plugin that lets players
build sealed, underwater structures: check the structure is properly enclosed, then register its
interior so occupants don't drown or get the underwater fog.

### Related hooks
`Hooks.OnRustBusterWaterStateChanged`, `Hooks.OnRustBusterOxygenChanged`, `Hooks.OnRustBusterOutOfAir` -
see the [Hooks reference](../Hooks/README.md).
