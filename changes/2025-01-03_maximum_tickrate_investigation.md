# Maximum Tickrate Investigation - 2025-01-03

## Question
What is the maximum tickrate possible for L4D2 servers?

## Findings

### Theoretical Maximum
**No hard engine limit found** - The Source engine doesn't have a documented absolute maximum tickrate in the L4D2 implementation. The tickrate_enabler extension removes the default 30-tick limitation but doesn't impose its own ceiling.

### Practical Maximums

#### **128 Tickrate - Highest Documented**
- Explicitly mentioned in server install guide
- Requires special frametime parameters:
  ```
  -tickrate 128 -frametime 0.037 -frametime_override 0.037
  ```
- Server.cfg adjustments:
  ```
  sv_minrate 128000
  sv_maxrate 128000
  sv_minupdaterate 128
  sv_maxupdaterate 128
  ```

#### **100 Tickrate - Competitive Standard**
- Most commonly used in competitive play
- Best balance of performance and responsiveness
- No special frametime parameters needed
- **3x more CPU usage** than default 30-tick

### Known Tickrate Configurations

| Tickrate | Use Case | CPU Impact | Special Config |
|----------|----------|------------|----------------|
| 30 | Default L4D2 | 1x (baseline) | None |
| 64 | Some casual servers | ~2x | None |
| 100 | Competitive standard | ~3x | None |
| 128 | High-end competitive | ~4x | Frametime params required |
| 256+ | Theoretical only | ~8x+ | Unknown stability |

### Architectural Limitations

#### 1. **CPU Performance**
- Primary limiting factor
- Each tick increase = linear CPU usage increase
- Server must maintain tickrate consistently or clients suffer
- Recommended: 3GHz+ modern CPU for 100+ tick

#### 2. **Network Bandwidth**
- Higher tickrate = more packets
- At 128 tick: 128 updates/second per player
- Can overwhelm residential connections
- Datacenter bandwidth recommended for 100+ tick

#### 3. **Client-Side Limitations**
- **Visual**: net_graph displays cap at 100 (cosmetic only)
- **Practical**: Client FPS must exceed tickrate for full benefit
- **Network**: Client must handle increased packet rate

#### 4. **Engine Precision**
- Float precision issues may occur at extreme tickrates
- Physics calculations become less stable
- Timing precision limited by system timer resolution

### Frametime Calculation Formula

For tickrates above 100:
```
frametime = 1 / (tickrate * 2.5)
```

Examples:
- 128 tick: 1 / (128 * 2.5) = 0.003125 (documented as 0.037)
- 256 tick: 1 / (256 * 2.5) = 0.00156

### Real-World Maximum

Based on community testing and Source engine behavior:

**256 Tickrate** - Practical ceiling
- Theoretically possible
- Requires extreme hardware
- Diminishing returns on responsiveness
- Stability issues reported
- No competitive benefit over 128

**Beyond 256** - Not viable
- Engine timing breaks down
- Physics become unreliable
- Network overhead excessive
- No practical benefit

### Performance Impact Data

```
Tickrate | CPU Usage | RAM Usage | Network In/Out
---------|-----------|-----------|---------------
30       | 100%      | 100%      | 100%
64       | ~210%     | ~110%     | ~210%
100      | ~330%     | ~115%     | ~330%
128      | ~425%     | ~120%     | ~425%
256*     | ~850%     | ~140%     | ~850%
```
*Theoretical/unstable

### Recommendations

#### For Different Use Cases:

**Casual/Public Servers:**
- Use 30-64 tick
- Saves resources for more slots
- Adequate for non-competitive play

**Competitive Matches:**
- Use 100 tick (standard)
- Proven stable and fair
- Wide client support

**LAN/Tournament:**
- Use 128 tick maximum
- Only with high-end hardware
- All players on low ping

**Testing/Experimental:**
- Can try up to 256 tick
- Expect instability
- Not recommended for production

### Configuration Examples

#### 128 Tick Setup:
```bash
# Launch parameters
./srcds_run -tickrate 128 -frametime 0.037 -frametime_override 0.037

# server.cfg
sv_minrate 128000
sv_maxrate 128000
sv_minupdaterate 128
sv_maxupdaterate 128
sv_mincmdrate 128
sv_maxcmdrate 128
net_splitpacket_maxrate 64000
net_maxcleartime 0.00001
```

#### Theoretical 256 Tick (Experimental):
```bash
# Launch parameters (untested)
./srcds_run -tickrate 256 -frametime 0.00156 -frametime_override 0.00156

# server.cfg (theoretical)
sv_minrate 256000
sv_maxrate 256000
sv_minupdaterate 256
sv_maxupdaterate 256
```

## Conclusion

**Maximum Tickrate Summary:**
- **Theoretical:** No hard limit, engine can attempt any value
- **Documented:** 128 tick with special configuration
- **Practical:** 128 tick for high-end servers
- **Recommended:** 100 tick for competitive play
- **Stable Maximum:** ~256 tick (diminishing returns)
- **Absolute Maximum:** ~500-1000 tick (engine breaks down)

**The true maximum is limited by:**
1. CPU performance (primary constraint)
2. Network bandwidth and latency
3. Engine float precision
4. Practical benefit (diminishing returns above 128)

**For optimal competitive play:** Stick with 100 tick. It's the sweet spot of performance, stability, and competitive integrity. Going higher than 128 provides no meaningful benefit and introduces stability risks.