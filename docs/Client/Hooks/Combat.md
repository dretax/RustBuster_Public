### Category
Combat hooks

### Description
Weapon fire (bullet/shotgun/bow/melee), explosions, and damage. See [Hooks/README.md](README.md) for
the full table and the general cancellation rules.

---

### `OnRustBusterMetabolismDamage`
`public delegate void RustBusterClientMetabolismDelegate(DamageEvent de)`

Runs when a metabolism (mostly a player or animal) is damaged. Not cancellable (the client cannot
override server-authoritative damage). `DamageEvent` is the game's own class (`Assembly-CSharp`), not
documented separately here - inspect it with your IDE's "Go to Definition"/decompiler for the exact
fields available (amount, damage type, attacker, etc.).

### `OnRustBusterStructureDamage`
`public delegate void RustBusterClientStructureDelegate(DamageEvent de)`

Runs when a structure is damaged. Not cancellable, same `DamageEvent` type as above.

### `OnRustBusterC4Explosion`
`public delegate void RustBusterC4ExplosionDelegate(C4ExplosionEvent ce)`

Runs **twice** per explosion: once before, once after. Not cancellable.
- `ce.ExplosionType` - `BeforeExplosion` or `AfterExplosion`. **Always check this**, or your handler logic
  will run twice per explosion.
- `ce.TimedExplosive` - the `TimedExplosive` component.
- `ce.ExplosionObject` - the explosion's spawned object; only populated on `AfterExplosion`, `null`
  otherwise.

```csharp
public void HandleC4Explosion(C4ExplosionEvent e)
{
    if (e.ExplosionType != ExplosionType.AfterExplosion) return;

    Hooks.LogData(Name, "C4 exploded at " + e.ExplosionObject.name);
}
```

### `OnRustBusterGrenadeExplode`
`public delegate void RustBusterGrenadeExplodeDelegate(GrenadeExplosionEvent ce)`

Same shape as `OnRustBusterC4Explosion`, for frag grenades: `ce.ExplosionType`, `ce.TimedGrenade`,
`ce.ExplosionObject`. Fires twice. Not cancellable.

### `OnRustBusterSupplySignalExplode`
`public delegate void RustBusterSupplySignalExplodeDelegate(SupplySignalExplosionEvent ce)`

Same shape, for supply signal (smoke) grenades: `ce.ExplosionType`, `ce.SignalGrenade`,
`ce.ExplosionObject`. Fires twice. Not cancellable.
- `ce.SmokeEffectTime` - gets/sets the smoke effect duration in seconds (default `60`). **Only
  meaningful at `AfterExplosion`.**

```csharp
public void HandleSmoke(SupplySignalExplosionEvent e)
{
    if (e.ExplosionType != ExplosionType.AfterExplosion) return;
    e.SmokeEffectTime = 15f; // Make smoke signals disperse faster.
}
```

### `OnRustBusterWeaponFire`
`public delegate void RustBusterWeaponFireDelegate(BulletWeaponFireEvent be)`

Runs when a bullet-based weapon is discharged by the local player. **Partially cancellable** - there's no
single `Cancel()`, but individual side effects can be suppressed:
- `be.BulletWeaponDataBlock`, `be.ItemRepresentation`, `be.ViewModel`, `be.IBulletWeaponItem`,
  `be.HumanController` (the `InputSample` at fire time) - read-only context.
- `be.Pitch` / `be.Yaw` - mutable recoil amounts applied after firing.
- `be.CancelFireAnimation`, `be.CancelFireEffect` (muzzle flash etc.), `be.CancelHeadBob`,
  `be.CancelRecoil` - independent `bool` switches.

```csharp
public void HandleWeaponFire(BulletWeaponFireEvent e)
{
    e.Pitch *= 0.5f; // Halve vertical recoil.
    e.Yaw *= 0.5f;   // Halve horizontal recoil.
}
```

### `OnRustBusterShotgunFire`
`public delegate void RustBusterShotgunFireDelegate(ShotgunWeaponFireEvent se)`

Same shape as `OnRustBusterWeaponFire` for shotguns: `se.ShotgunDataBlock`, `se.ItemRepresentation`,
`se.ViewModel`, `se.IBulletWeaponItem`, `se.HumanController`, mutable `se.Pitch`/`se.Yaw`, and the same
four `Cancel*` bool switches.

### `OnRustBusterBowFire`
`public delegate void RustBusterBowFireDelegate(BowFireEvent be)`

Runs when an arrow is shot from a bow. Not cancellable, read-only context only:
`be.BowWeaponDataBlock`, `be.ItemRepresentation`, `be.ViewModel`, `be.HumanController`,
`be.IBowWeaponItem`, `be.Ray`/`be.Vector3`/`be.Rotation` (the character's eye ray/origin/rotation at fire
time, useful for custom projectile logic).

### `OnRustBusterMeleeFire`
`public delegate void RustBusterMeleeFireDelegate(MeleeWeaponFireEvent me)`

Runs when a melee weapon is swung. **Partially cancellable:**
- `me.MeleeWeaponDataBlock`, `me.ViewModel`, `me.ItemRepresentation`, `me.IMeleeWeaponItem`,
  `me.HumanController` - read-only context.
- `me.CancelFireAnimation`, `me.CancelSwingSound`, `me.CancelMidSwing` (hit-detection callback),
  `me.CancelServerAction` (the server RPC) - independent `bool` switches.
