# Tank and Witch Spawn Control Investigation - 2025-01-03

## Overview
This investigation reveals the comprehensive system for controlling tank and witch spawns in competitive L4D2 without using a map editor.

## Key Findings

### 1. Boss Spawn Control System

The system uses multiple interconnected components:

#### Primary Control Plugins:
1. **witch_and_tankifier.smx** (`/addons/sourcemod/scripting/witch_and_tankifier.sp`)
   - Main plugin controlling tank/witch spawn percentages
   - ConVars:
     - `sm_tank_can_spawn` (0/1) - Enable/disable tank spawns
     - `sm_witch_can_spawn` (0/1) - Enable/disable witch spawns
     - `sm_witch_avoid_tank_spawn` - Distance witches avoid tank spawns

2. **l4d_boss_percent.smx** - Displays boss percentages on ready-up
3. **l4d_boss_vote.smx** - Allows players to vote for custom boss spawn percentages
4. **BossSpawning module** (in confoglcompmod) - Ensures consistent spawn positions between rounds

### 2. Spawn Percentage Control

#### Flow-Based Spawning:
Tanks and witches spawn based on "flow percentage" - the progress through the map (0-100%).

**Key CVars:**
- `versus_boss_flow_min` - Minimum flow percentage (typically 0.10-0.20)
- `versus_boss_flow_max` - Maximum flow percentage (typically 0.80-0.90)
- `versus_boss_buffer` - Buffer zone before spawn trigger

**Examples from configs:**
```
// Zonemod (more restricted)
versus_boss_flow_min 0.20
versus_boss_flow_max 0.85

// Equilibrium (wider range)
versus_boss_flow_min 0.10
versus_boss_flow_max 0.90
```

### 3. Map-Specific Customization (mapinfo.txt)

Each matchmode has a `mapinfo.txt` file that allows per-map customization:

**Location:** `/cfg/cfgogl/[modename]/mapinfo.txt`

**Features:**
- `tank_ban_flow` - Blocks tank spawns in specific flow ranges
- `tank_warpto` - Teleports tank to specific coordinates if spawned badly
- `tank_z_fix` - Fixes stuck tank spawns
- `witch_ban_flow` - Similar to tank_ban_flow for witches
- Map-specific flow overrides

**Example (c1m1_hotel):**
```
"tank_ban_flow"
{
    "Instant spawns"
    {
        "min"    "0"
        "max"    "25"
    }
    "Before elevator top, after elevator bottom"
    {
        "min"    "33"
        "max"    "79"
    }
}
```

### 4. Boss Difficulty Settings

#### Tank Settings (per config):
- `z_tank_health` - Tank health (varies by player count)
  - 1v1: ~1333 HP
  - 2v2: ~2000-2320 HP
  - 3v3: ~3000-3350 HP
  - 4v4: ~4200 HP (Apex)
- `z_tank_speed_vs` - Tank movement speed (200-210)
- `tank_burn_duration` - How long tank burns

#### Witch Settings:
- `z_witch_health` - Witch health (default 1000)
- Witch behavior controlled by AI Director

#### Global Difficulty:
- `z_difficulty "normal"` - Forces normal difficulty (prevents co-op settings)

### 5. Commands for Control

#### Admin Commands:
- `sm_ftank [percent]` - Force tank spawn at specific percentage
- `sm_fwitch [percent]` - Force witch spawn at specific percentage
- `static_tank_map` - Mark map as having static (unchangeable) tank spawn
- `static_witch_map` - Mark map as having static witch spawn

#### Player Commands:
- `sm_voteboss [tank%] [witch%]` - Vote for boss spawn percentages
- `sm_boss` - View current boss spawn percentages
- `sm_tank` / `sm_witch` - View specific boss percentage

### 6. Dynamic Control Methods

#### Without Map Editor, You Can:

1. **Change spawn percentages via config:**
   - Edit `versus_boss_flow_min/max` in config files
   - Use `confogl_addcvar` in matchmode configs

2. **Disable/enable bosses:**
   ```
   sm_tank_can_spawn 0  // Disable tanks
   sm_witch_can_spawn 0 // Disable witches
   ```

3. **Vote for custom percentages:**
   ```
   sm_voteboss 25 50  // Vote for 25% tank, 50% witch
   ```

4. **Customize per map via mapinfo.txt:**
   - Add tank_ban_flow sections
   - Set tank_warpto coordinates
   - Override flow percentages

5. **Adjust difficulty:**
   ```
   z_tank_health 5000      // Custom tank health
   z_tank_speed_vs 220     // Custom tank speed
   ```

### 7. Important Notes

- **Static Maps:** Some maps have "static" spawns that cannot be voted on
- **Flow Calculation:** Flow is calculated from map start to end, not physical distance
- **Round Consistency:** BossSpawning module ensures same spawn locations in both rounds
- **Witch Avoidance:** Witches automatically avoid spawning too close to tanks
- **Finale Maps:** Different rules apply to finale maps

## Practical Usage

### To customize boss spawns for your server:

1. **Quick changes (per session):**
   ```
   sm_cvar sm_tank_can_spawn 0     // Disable tanks
   sm_cvar versus_boss_flow_min 0.30  // Change minimum spawn
   ```

2. **Permanent changes:**
   - Edit `/cfg/cfgogl/[modename]/confogl.cfg`
   - Add/modify mapinfo.txt entries
   - Create custom configs

3. **Map-specific adjustments:**
   - Edit mapinfo.txt for the specific config
   - Add tank_ban_flow sections for problem areas
   - Set tank_warpto for better spawn positions

### Example: Disable tank on specific map section
In mapinfo.txt:
```
"c1m1_hotel"
{
    "tank_ban_flow"
    {
        "No tank in hotel"
        {
            "min"    "0"
            "max"    "50"
        }
    }
}
```

This comprehensive system allows full control over boss spawning without needing to edit the actual map files.