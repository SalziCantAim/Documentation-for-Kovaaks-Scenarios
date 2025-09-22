# Kovaak's Scenario Categorization Guide

## Overview
This guide provides a comprehensive system for categorizing Kovaak's aim trainer scenarios (.sce files) based on their properties and mechanics.

## Primary Categories
### 1. Clicking


**Identification Rules:**

- `AimTypeTag=Clicking` OR
- `AimTypeTag=Other` AND (`MagazineMax` < 20 OR `Category=SemiAuto`)
- Generally semi-automatic weapon behavior
- Targets typically don't regenerate health (`HealthRegenPerSec=0.0` or negative)

#### Subcategories:

##### Static

Targets are stationary.
- Targets don't move (simple dodge profiles like vbr or no movement)
- `HealthRegenPerSec=0.0` (no regeneration)

##### Dynamic

Targets are moving.
- Bot profiles have `DodgeProfileNames` specified AND `NoDodging=false`
- Complex dodge profiles with movement patterns

##### Pokeball (Special Static Subtype)

- Full automatic weapon (`Category=FullyAuto`)
- Very fast health regeneration after not being hit
- `HealthRegenDelay` < 1.0 seconds


##### Reflex (Special Subtype)

**Identification:**
- Scenario name contains "reflex"
- `MaxHealth=100` and `HealthRegenPerSec=-100.0` = 1 second to click
- `MaxHealth=100` and `HealthRegenPerSec=-200.0` = 500ms to click as example
- Generally timebased clicking with health decay (sometimes also seen as 1w1t clicking/switching)
- Can be combined with Dynamic movement if dodge profiles present

**SubSubcategories:**
- **Reflex (Fast)**: ≤500ms to eliminate target
- **Reflex (Medium)**: 500ms-1s to eliminate target  
- **Reflex (Slow)**: 1s-2s to eliminate target
- **Reflex (Dynamic)**: Reflex with movement patterns

##### Nevermiss (Special Subtype)

**Identification:**
- Scenario name contains "nevermiss" or "never miss"
- `PlayerMaxLives=1` (player dies on first miss)
- `Explosive=true` in weapon profile
- Player character: `MainBBRadius` <= `ProjBBRadius` 
- `InvinciblePlayer=false`

### 2. Tracking
An aiming category where the crosshair is maintained on the target for an extended period while reacting to direction changes.

**Identification Rules:**
- `AimTypeTag=Tracking` OR
- `InvincibleBots=true` (tracking invincible targets) OR
- Rotation profiles with set timer eg. 1900 3 timies like VT PGT S5 
- Very fast firing weapon (`TimeBetweenShots` < 0.02) with continuous tracking
- currently no conclusive way of differentiating between different subcategories 

#### Subcategories:

##### Precise

Focused on precise tracking scenarios.
- Often tagged with "Precision" in `SearchTags`


##### Reactive

Fast directional changes requiring quick reactivity.
- Tagged with "Reactive" in `SearchTags`
- Higher bot movement speeds and frequent direction changes

##### Smoothness

Focus on smooth tracking motion.
- Tagged with "Smoothness" in `SearchTags` or `AimSubTypeTag`

#### Special Tracking Types:

##### TI (Tracking Invincible)

- `InvincibleBots=true`
- Targets cannot be eliminated
- Pure tracking without killing

##### Time-based Tracking

- `Timelimit` > 60 seconds (more time than normal)
- Scoring based on completion time rather than hits
- `ScorePerTime` > 0 or time-based scoring system
- Multiple targets in rotation (gauntlet style)

##### Rotational Tracking

- Multiple bot profiles that rotate/change
- Each target alive for set duration: `HealthRegenPerSec` < 0 (health decreases over time)
- Example: 1900 health with -100 health/sec = 19 seconds per target
- Scoring typically `ScorePerHit` > 0

### 3. Target Switching

Fast target changes requiring time before each target can be eliminated.

**Identification Rules:**

- `AimTypeTag=Target Switching` OR
- Multiple simultaneous targets that respawn
- Fast target switching between multiple active targets

#### Subcategories:

##### Speed Switching

Low time-to-kill with fast-paced targets.
- Tagged with "Speed" or "Fast" in `SearchTags`
- Quick target elimination

##### Evasive Switching

Harder-to-hit movement patterns requiring reading ability.
- Tagged with "Evasive" in `SearchTags`
- Complex movement patterns and evasive behavior
- generally more health than speed switching

#### Special Target Switching Types:

##### Static TS

- Target switching but targets don't move
- `HealthRegenPerSec=0.0` (targets don't regenerate normally)
- Multiple stationary targets that must be switched between


## Potential Properties for Identification of types

### Weapon Profile Properties
- `MagazineMax`: 0 = infinite ammo, <20 = likely clicking
- `Category`: "FullyAuto" vs "SemiAuto"  
- `TimeBetweenShots`: Very low (0.01-0.012) = tracking, higher = clicking
- `Explosive`: true = potentially nevermiss

### Character Profile Properties  
- `InvinciblePlayer`: false for nevermiss scenarios
- `InvincibleBots`: true = tracking invincible
- `HealthRegenPerSec`: 0 = no regen, negative = timed targets, positive = regenerating
- `HealthRegenDelay`: Fast (<1s) = pokeball style
- `MainBBRadius` vs `ProjBBRadius`: For nevermiss detection

### Scenario Properties
- `PlayerMaxLives`: 1 = nevermiss potential
- `Timelimit`: >60s = potentially time-based
- `ScorePerHit` vs `ScorePerTime` vs `ScorePerDamage`: Determines scoring method
- `AimTypeTag` / `AimSubTypeTag`: Primary classification always the first thing to check
- `SearchTags`: Keywords for subcategorization

### Bot Behavior
- Multiple simultaneous bots = likely target switching
- Bot rotation profiles = rotational scenarios  
- `DodgeProfileNames` specified AND `NoDodging=false` = dynamic movement
- `NoDodging=true` or empty `DodgeProfileNames` = static targets

### Main Target Identification (Multi-Bot Scenarios)

Some scenarios have multiple bots where only one is the actual target, while others are "blocker" bots or environmental elements. The main target is identified by:

**Primary Method - DisableScoring:**
- Main target: `DisableScoring=false` (allows scoring)
- Blocker bots: `DisableScoring=true` (no scoring)

**Secondary Method - Bot Teams:**
- not conclusive!
- Example: `BotTeams=2;1;1;1;1` with `AddedBots=regen_dot.bot;Launcher.bot;Launcher2.bot;Launcher3.bot;Launcher4.bot`
- `regen_dot.bot` is team 2 (minority team) = main target
- `Launcher.bot` through `Launcher4.bot` are team 1 = blocker bots

**Main properties/profiles to focus on:**
- Mainly focus the main target's properties for categorization
- Health regeneration, dodge profiles, and target size should be based on the main target's character profile
- Ignore blocker bots when determining movement patterns and timing

## Adaptive Mechanics

Scenarios can include adaptive difficulty:
- `IsTargetSizeActive=true`: Target size adapts to performance
- `PerformanceTarget`: Target accuracy threshold  
- `MaxTargetSizeMultiplier` / `MinTargetSizeMultiplier`: Size scaling range the same for the speed based multiplier
- `AdjustmentRate` / `AdjustmentInterval`: Adaptation speed and frequency

## Algorithm Priority

1. **Check scenario name** for special keywords ("nevermiss", "pokeball", "reflex")
2. **PRIORITIZE `AimTypeTag` and `AimSubTypeTag`** for direct classification
3. **Check dodge profiles** (`DodgeProfileNames` + `NoDodging=false`) for dynamic movement
4. **Analyze health regeneration timing** for reflex scenarios
5. **Analyze weapon properties** for clicking vs tracking distinction (fallback)
6. **Check invincibility settings** for tracking invincible
7. **Analyze target count and respawn** for target switching
8. **Use `SearchTags`** for subcategory refinement
9. **Analyze timing and scoring** for special mechanics
