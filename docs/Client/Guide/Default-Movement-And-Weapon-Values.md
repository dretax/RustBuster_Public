### Guide
Default movement and weapon values

### Description
Rust Legacy's vanilla, unmodified movement (`CCMotor`) setup and base weapon stats, listed here for
reference.

### CCMotor (movement) defaults

| Field | Default value |
|---|---|
| `movement.setup.gravity` | `35` |
| `movement.setup.maxForwardSpeed` | `4` |
| `movement.setup.maxSidewaysSpeed` | `4` |
| `movement.setup.maxBackwardsSpeed` | `3` |
| `movement.setup.maxGroundAcceleration` | `100` |
| `movement.setup.maxAirAcceleration` | `20` |
| `movement.setup.maxFallSpeed` | `80` |
| `jumping.setup.baseHeight` | `1` |
| `minTimeBetweenJumps` | `1` |

### Bullet weapon defaults

| Weapon | `fireRate` | `bulletRange` | `recoilYawMin` | `recoilYawMax` | `aimSway` | `aimSwaySpeed` | `fireRateSecondary` | `aimingRecoilSubtract` | `recoilPitchMax` |
|---|---|---|---|---|---|---|---|---|---|
| M4 | `0.125` | `140` | `-3` | `3` | `0.5` | `1` | `1` | `0.5` | `5` |
| MP5A4 | `0.115` | `80` | `-3.5` | `3.5` | `0.5` | `1` | `1` | `0.5` | `3.5` |
| P250 | `0.05` | `120` | `-2.25` | `2.25` | `1` | `1` | `1` | `0.35` | `9` |
| 9mm Pistol | `0.05` | `80` | `-1.25` | `1.25` | `1` | `1` | `1` | `0.35` | `6` |
| Bolt Action Rifle | `2.2` | `250` | `-3.5` | `3.5` | `1.5` | `1.25` | `1` | `0.5` | `10` |

### Bow weapon defaults

| Field | Default value |
|---|---|
| `arrowSpeed` | `80` |
| `drawLength` | `2` |
| `fireRate` | `1` |
