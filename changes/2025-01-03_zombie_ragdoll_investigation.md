# Zombie Ragdoll Disappearance Investigation - 2025-01-03

## Issue
Zombie bodies disappear instantly when killed in certain configurations.

## Root Cause Identified
The plugin **`l4d_common_ragdolls_be_gone.smx`** is responsible for removing common infected ragdolls immediately upon death.

### Plugin Details
- **Location:** `/addons/sourcemod/plugins/optional/l4d_common_ragdolls_be_gone.smx`
- **Source:** `/addons/sourcemod/scripting/l4d_common_ragdolls_be_gone.sp`
- **Author:** Sir
- **Purpose:** "Make ragdolls for common infected vanish into thin air server-side on death"
- **How it works:** Hooks the `player_death` event and immediately removes common infected entities

## Affected Configurations

### Configs that LOAD the ragdoll removal plugin:
Only the Zonemod family of configs load this plugin via their `shared_plugins.cfg`:
1. **Zonemod** - `/cfg/cfgogl/zonemod/shared_plugins.cfg:108`
2. **Zonehunters** - `/cfg/cfgogl/zonehunters/shared_plugins.cfg:94`
3. **Zoneretro** - `/cfg/cfgogl/zoneretro/shared_plugins.cfg:98`

### Configs that DO NOT load the ragdoll removal plugin:
All other matchmodes keep normal ragdoll behavior:
- Acemod Revamped (acemodrv)
- All AMR variants (amrv1v1, amrv2v2, amrv3v3)
- Apex
- Deadman
- All Equilibrium variants (eq, eq1v1, eq2v2, eq3v3)
- NeoMod
- All NextMod variants (nextmod, nextmod1v1, nextmod2v2, nextmod3v3)
- Promod Elite (pmelite)
- All ZH variants (zh1v1, zh2v2, zh3v3)
- All ZM variants (zm1v1, zm2v2, zm3v3)

## Important Notes
- The plugin is NOT loaded in `generalfixes.cfg` (global fixes loaded by all configs)
- The plugin is NOT loaded in `sharedplugins.cfg` (user-defined global plugins)
- This is intentionally only enabled for Zonemod-based configurations

## Solution Options

### To DISABLE ragdoll removal (keep bodies) in Zonemod configs:
Comment out or remove the following line from these files:
- `/cfg/cfgogl/zonemod/shared_plugins.cfg`
- `/cfg/cfgogl/zonehunters/shared_plugins.cfg`
- `/cfg/cfgogl/zoneretro/shared_plugins.cfg`

Line to remove/comment:
```
sm plugins load optional/l4d_common_ragdolls_be_gone.smx
```

### To ENABLE ragdoll removal (remove bodies) globally:
Add to `/cfg/generalfixes.cfg` (affects ALL configs):
```
sm plugins load optional/l4d_common_ragdolls_be_gone.smx
```

### To ENABLE for specific configs only:
Add the above line to the specific config's `shared_plugins.cfg` or `confogl_plugins.cfg`

## Performance Considerations
The plugin was likely added to Zonemod for performance reasons - removing ragdolls reduces:
- Server-side physics calculations
- Network traffic (ragdoll sync)
- Client-side rendering load

This can be beneficial in competitive play where performance and consistency are prioritized over visual effects.