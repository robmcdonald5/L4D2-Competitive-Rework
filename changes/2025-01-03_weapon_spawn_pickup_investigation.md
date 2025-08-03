# Weapon Spawn and Pickup Rules Investigation - 2025-01-03

## Issue
Understanding how weapon spawn and pickup limits work in competitive configs, particularly why hunting rifles can't be picked up and deagles are limited to 1 player in Zonemod.

## System Overview

The weapon control system uses three interconnected plugins:

### 1. **l4d_weapon_limits.smx** - Controls pickup limits
### 2. **l4d2_weaponrules.smx** - Controls spawn replacements
### 3. **l4d2_melee_spawn_control.smx** - Controls melee weapon spawns

## How Weapon Limits Work

### Command: `l4d_wlimits_add <limit> <ammo> <weapon1> [weapon2...]`

**Parameters:**
- `limit` - Maximum number of players who can carry this weapon type
- `ammo` - Ammo given when weapon is denied:
  - `-1` = Given for primary weapon spawns only
  - `0` = No ammo given ever
  - `1+` = Ammo always given

### Zonemod Configuration Analysis

From `/cfg/cfgogl/zonemod/zonemod.cfg`:

```
l4d_wlimits_add 3 1 weapon_smg_silenced weapon_smg      // 3 players max can have SMGs
l4d_wlimits_add 3 1 weapon_pumpshotgun weapon_shotgun_chrome  // 3 players max can have shotguns
l4d_wlimits_add 1 0 weapon_pistol_magnum                // 1 player max can have deagle
l4d_wlimits_add 0 1 weapon_hunting_rifle weapon_sniper_scout weapon_sniper_awp  // 0 players (banned)
```

**This explains your observations:**
- **Hunting rifles**: Limit = 0 = Nobody can pick them up (but they give ammo)
- **Deagles**: Limit = 1 = Only 1 player can have one
- **SMGs/Shotguns**: Limit = 3 = Up to 3 players can have them

## How Weapon Spawn Rules Work

### Command: `l4d2_addweaponrule <match> <replace>`

Replaces weapon spawns automatically when the map loads.

### Zonemod Spawn Replacements

From `/cfg/cfgogl/zonemod/shared_settings.cfg`:

```
l4d2_addweaponrule rifle                  smg           // M16 → SMG
l4d2_addweaponrule rifle_ak47             smg_silenced  // AK47 → Silenced SMG
l4d2_addweaponrule autoshotgun            pumpshotgun   // Auto shotgun → Pump
l4d2_addweaponrule sniper_military        hunting_rifle // Military sniper → Hunting rifle
l4d2_addweaponrule rifle_m60              pistol_magnum // M60 → Deagle
```

**Result:** T2 weapons are replaced with T1 weapons, except hunting rifles (which spawn but can't be picked up).

## How to Modify These Rules

### 1. Change Weapon Pickup Limits

**Allow hunting rifles (1 per team):**
```
l4d_wlimits_clear  // Clear existing limits
l4d_wlimits_add 1 1 weapon_hunting_rifle weapon_sniper_scout weapon_sniper_awp
l4d_wlimits_lock
```

**Allow 2 deagles:**
```
l4d_wlimits_clear
l4d_wlimits_add 2 0 weapon_pistol_magnum
l4d_wlimits_lock
```

### 2. Change Weapon Spawns

**Keep T2 weapons (rifles):**
```
l4d2_resetweaponrules  // Remove all replacement rules
```

**Replace hunting rifles with SMGs:**
```
l4d2_addweaponrule hunting_rifle smg
```

**Remove weapons entirely:**
```
l4d2_addweaponrule hunting_rifle none  // Deletes hunting rifles from map
```

### 3. Control Melee Weapons

**Set available melees:**
```
confogl_addcvar l4d2_melee_spawn "fireaxe,cricket_bat,katana"
```

**Add melees to existing spawn list:**
```
confogl_addcvar l4d2_add_melee "golfclub,machete"
```

## Configuration Comparison

### Strict Configs (Zonemod, Retro):
- 0 snipers allowed
- 1 deagle max
- T2 weapons replaced with T1

### Moderate Configs (NextMod, NeoMod):
- 0 snipers allowed
- 2 deagles allowed
- 3 melees allowed
- T2 weapons replaced, hunting rifles kept

### Liberal Configs (Equilibrium):
- 1-2 scout snipers allowed
- 2 deagles allowed
- More weapon variety

## Practical Examples

### To allow hunting rifles in Zonemod:

1. **Edit** `/cfg/cfgogl/zonemod/zonemod.cfg`
2. **Change:**
   ```
   l4d_wlimits_add 0 1 weapon_hunting_rifle weapon_sniper_scout weapon_sniper_awp
   ```
   **To:**
   ```
   l4d_wlimits_add 1 1 weapon_hunting_rifle weapon_sniper_scout weapon_sniper_awp
   ```

### To spawn more weapon variety:

1. **Edit** `/cfg/cfgogl/zonemod/shared_settings.cfg`
2. **Comment out or remove** the `l4d2_addweaponrule` lines you don't want
3. **Example:** Remove the line `l4d2_addweaponrule rifle smg` to keep M16 rifles

### To add custom weapon spawns per map:

Use server commands or add to config:
```
// Force specific weapon spawns
l4d2_addweaponrule smg rifle_ak47  // Replace all SMGs with AK47s
```

## Important Notes

1. **Weapon limits are per-team**, not total players
2. **The `l4d_wlimits_lock` command** must be called after setting limits
3. **Weapon rules apply on map load** - changes require map restart
4. **Some weapons still spawn** even if limited to 0 (they just can't be picked up)
5. **Ammo is given** when a limited weapon is denied (if ammo parameter > 0)

## Summary

The system provides complete control over weapon spawning and pickup without map editing:
- **l4d_weapon_limits** controls who can pick up what
- **l4d2_weaponrules** controls what spawns where  
- **l4d2_melee_spawn_control** controls available melee weapons

All configuration is done through config files and console commands, making it easy to customize weapon availability for different competitive modes.