# Pençe Harekati - Unity Port Implementation Roadmap

## Overview
10-week implementation plan for porting HTML bullet hell game to Unity. Each phase builds on the previous, with verification checkpoints.

---

## PHASE 1: Core Foundation (Week 1-2)

### Goal
Basic playable prototype with player movement and camera.

### Tasks

#### 1.1 Project Setup
- [ ] Create Unity project (2D template, Unity 2022.3 LTS)
- [ ] Install packages:
  - [ ] 2D Sprite
  - [ ] Input System
  - [ ] TextMeshPro
  - [ ] Unity UI
- [ ] Configure project settings:
  - [ ] Fixed timestep: 0.01666 (60 FPS)
  - [ ] Target frame rate: 60
  - [ ] Physics 2D gravity: 0
- [ ] Create folder structure (see Claude.md section 3.1)

#### 1.2 Core Managers
- [ ] Create GameManager.cs
  - [ ] Singleton pattern
  - [ ] Game state enum (Playing, Paused, GameOver)
  - [ ] Score tracking
  - [ ] Game time tracking
  - [ ] Events: OnGameStart, OnGameOver, OnScoreChange
- [ ] Create PoolManager.cs
  - [ ] Pool dictionary structure
  - [ ] Get() and Return() methods
  - [ ] Prewarm() functionality
  - [ ] Helper methods: GetBullet(), GetEnemy(), GetParticle()
- [ ] Create AudioManager.cs (stub for now)
- [ ] Create SaveManager.cs (stub for now)

#### 1.3 Player Movement
- [ ] Create PlayerController.cs
  - [ ] Component references (Movement, Health, WeaponManager)
  - [ ] Stats: level, XP, damage multipliers
  - [ ] IsMoving property
- [ ] Create PlayerMovement.cs
  - [ ] Rigidbody2D movement
  - [ ] WASD input (New Input System)
  - [ ] Speed: 2.5 (for Altay tank)
  - [ ] World boundary clamping (0-6000, 0-6000)
- [ ] Create Tank_Altay prefab
  - [ ] Sprite (temporary placeholder)
  - [ ] Rigidbody2D (body type: Dynamic, gravity: 0)
  - [ ] CircleCollider2D (radius: 15)
  - [ ] PlayerController, PlayerMovement components

#### 1.4 Camera System
- [ ] Create CameraController.cs
  - [ ] Smooth follow player
  - [ ] Offset: camera.width/2, camera.height/2
  - [ ] Clamp to world boundaries
  - [ ] Zoom controls (optional)
- [ ] Setup Main Camera
  - [ ] Orthographic projection
  - [ ] Size: 10
  - [ ] CameraController component
  - [ ] Target: Player

#### 1.5 World Setup
- [ ] Create Game scene
  - [ ] GameManager object
  - [ ] Player spawn point (300, 300)
  - [ ] Camera
- [ ] Create world boundaries visualization (temporary)
- [ ] Create grid background (optional visual aid)

### Verification Checklist
- [ ] Player moves smoothly with WASD
- [ ] Camera tracks player with smooth follow
- [ ] Player cannot move outside world bounds (0-6000, 0-6000)
- [ ] Game runs at stable 60 FPS
- [ ] GameManager tracks game time
- [ ] No console errors

### Critical Files Reference
- HTML: lines 2350-2400 (player structure)
- HTML: lines 8642-8700 (camera tracking)

---

## PHASE 2: Combat Basics (Week 3-4)

### Goal
Functional shooting and enemy system with 3 weapons.

### Tasks

#### 2.1 Targeting System
- [ ] Create TargetingSystem.cs
  - [ ] FindEnemiesInRange(float range)
  - [ ] GetClosestEnemy()
  - [ ] GetStrongestEnemy()
  - [ ] TargetingMode enum
- [ ] Create TurretController.cs
  - [ ] Rotate to face mouse cursor
  - [ ] Auto-aim at closest enemy (optional toggle)
  - [ ] Visual feedback (turret sprite rotation)
- [ ] Add turret sprite to Tank_Altay prefab

#### 2.2 Weapon System Architecture
- [ ] Create WeaponDataSO.cs
  - [ ] WeaponID, name, icon
  - [ ] Category enum
  - [ ] WeaponLevelData array [10]
  - [ ] Projectile prefab reference
  - [ ] VFX prefab references
- [ ] Create WeaponBase.cs (abstract)
  - [ ] Data, CurrentLevel
  - [ ] UpdateWeapon() (cooldown management)
  - [ ] Fire() (abstract)
  - [ ] CanFire()
  - [ ] SpawnProjectile(), SpawnMuzzleFlash()
- [ ] Create WeaponManager.cs
  - [ ] Dictionary<string, WeaponBase> equipped weapons
  - [ ] AddWeapon(), RemoveWeapon(), UpgradeWeapon()
  - [ ] UpdateWeapons() (calls UpdateWeapon on all)
  - [ ] Max 5 weapon slots

#### 2.3 Implement 3 Weapons

**DefaultGun**
- [ ] Create DefaultGun.cs : WeaponBase
  - [ ] Fire(): Spawn 1 bullet toward closest enemy
  - [ ] Cooldown: 100 frames (1.67 seconds)
  - [ ] Damage: 15 (level 1)
- [ ] Create SO_DefaultGun.asset
  - [ ] Fill in 10 level data (damage: 15→100)
- [ ] Create Bullet_Default prefab
  - [ ] Sprite (small circle)
  - [ ] CircleCollider2D (trigger, radius: 5)
  - [ ] ProjectileBase component

**Laser**
- [ ] Create Laser.cs : WeaponBase
  - [ ] Fire(): Continuous beam to target
  - [ ] Focus mechanic: track time on target
  - [ ] Damage multiplier: 1 → 8x (over 7 seconds)
  - [ ] LineRenderer for visual
- [ ] Create SO_Laser.asset
  - [ ] DPS: 0.5 → 3.5 (10 levels)
- [ ] No projectile (instant hit)

**Missile**
- [ ] Create Missile.cs : WeaponBase
  - [ ] Fire(): Spawn 1-6 homing missiles
  - [ ] Cooldown: 120 → 40 frames
  - [ ] Damage: 25 → 50
- [ ] Create HomingMissile.cs : ProjectileBase
  - [ ] Homing algorithm (see Claude.md section 2.2)
  - [ ] Turn speed: 0.04 radians/frame
- [ ] Create Missile_Homing prefab
- [ ] Create SO_Missile.asset

#### 2.4 Projectile System
- [ ] Create ProjectileBase.cs
  - [ ] Initialize(damage, damageMultiplier, weaponID)
  - [ ] Move (velocity × deltaTime)
  - [ ] OnTriggerEnter2D (collision detection)
  - [ ] Return to pool when off-screen or hit
  - [ ] Pierce support (optional hits before destroy)
- [ ] Create IDamageable interface
  - [ ] TakeDamage(float damage, string weaponID)

#### 2.5 Enemy System
- [ ] Create EnemyDataSO.cs
  - [ ] EnemyType enum
  - [ ] Base stats: health, speed, damage
  - [ ] Attack range, chase range
  - [ ] Score value, XP value
  - [ ] IsBoss flag
- [ ] Create EnemyBase.cs : IDamageable
  - [ ] Initialize(data, playerLevel, score)
  - [ ] Health scaling formula (see Claude.md 2.3)
  - [ ] TakeDamage()
  - [ ] Die(): spawn XP, explosion, return to pool
  - [ ] UpdateHealthColor() (see Claude.md 2.3)
- [ ] Create EnemyAI.cs
  - [ ] Chase behavior
  - [ ] Attack behavior
  - [ ] Teleportation (off-screen > 5 sec)
- [ ] Create TankEnemy.cs : EnemyBase
  - [ ] Size: 30×20
  - [ ] Speed: 1.5
  - [ ] Chase range: 1000
  - [ ] Attack range: 600
- [ ] Create Tank_Enemy prefab
  - [ ] Sprite
  - [ ] Rigidbody2D
  - [ ] CircleCollider2D
  - [ ] EnemyBase, EnemyAI components
- [ ] Create SO_TankEnemy.asset

#### 2.6 Enemy Spawner
- [ ] Create SpawnConfigSO.cs
  - [ ] Spawn interval curve (120 → 30 frames over 5 min)
  - [ ] Max enemies formula: 10 + level × 2
  - [ ] Enemy type weights
- [ ] Create EnemySpawner.cs
  - [ ] UpdateSpawner() (called from GameManager)
  - [ ] SpawnEnemy(type)
  - [ ] GetSpawnPosition() (off-screen, near player)
  - [ ] AdjustSpawnRate() (time-based)
- [ ] Add EnemySpawner to GameManager object

#### 2.7 Collision & Damage
- [ ] Bullet-Enemy collision
  - [ ] Distance check in ProjectileBase
  - [ ] Call enemy.TakeDamage()
  - [ ] Destroy bullet (unless pierce)
- [ ] Enemy-Player collision
  - [ ] Check in EnemyBase or PlayerHealth
  - [ ] Motorcycle kamikaze: 15 damage
  - [ ] Enemy dies on contact (motorcycle only)
- [ ] Player-EnemyBullet collision
  - [ ] ProjectileBase checks for player tag
  - [ ] Apply damage × damageReduction

#### 2.8 VFX Basics
- [ ] Create MuzzleFlash prefab
  - [ ] Sprite animation (3 frames, 0.2 sec)
  - [ ] Auto-destroy after animation
- [ ] Create Explosion particle system
  - [ ] Red/orange emission
  - [ ] Burst: 20 particles
  - [ ] Lifetime: 1 second
  - [ ] Size over lifetime (grow then fade)
- [ ] Create Impact particle system
  - [ ] White sparks
  - [ ] Burst: 5 particles
  - [ ] Lifetime: 0.5 seconds

#### 2.9 Object Pooling Setup
- [ ] Pool definitions in PoolManager:
  - [ ] Bullet_Default (prewarm: 100)
  - [ ] Bullet_Missile (prewarm: 50)
  - [ ] Enemy_Tank (prewarm: 50)
  - [ ] Particle_Explosion (prewarm: 20)
  - [ ] Particle_Impact (prewarm: 50)
- [ ] Test pooling: spawn 100 bullets, verify no lag

### Verification Checklist
- [ ] DefaultGun fires bullets at closest enemy
- [ ] Laser draws beam and damages continuously
- [ ] Laser focus multiplier increases over time (visual color change)
- [ ] Missiles home toward target smoothly
- [ ] Turret rotates to face mouse cursor
- [ ] Tank enemies spawn off-screen
- [ ] Enemies chase player when in range
- [ ] Bullets collide with enemies accurately
- [ ] Enemies take damage (visual flash, health color changes)
- [ ] Enemies die and spawn explosion VFX
- [ ] No instantiation lag (object pooling works)
- [ ] 60 FPS with 10 enemies + 50 bullets on screen

### Critical Files Reference
- HTML: lines 5916-6100 (fireDefaultGun, fireLaser, fireMissile)
- HTML: lines 4458-4550 (createEnemy, enemy AI)
- HTML: lines 6790-6850 (collision detection)

---

## PHASE 3: Progression & Content (Week 5-6)

### Goal
Complete weapon roster, progression loop, all enemy types.

### Tasks

#### 3.1 XP System
- [ ] Create XPOrb.cs
  - [ ] XP value (5, 15, or 50)
  - [ ] Auto-collect when near player (100px)
  - [ ] Animate toward player
  - [ ] Tier colors: green, orange, gold
- [ ] Create XPOrb prefabs (3 tiers)
- [ ] Create XPManager.cs
  - [ ] AddXP(amount)
  - [ ] XP multiplier support
  - [ ] XP requirement formula: 50 × 1.5^level
  - [ ] LevelUp() trigger
  - [ ] RecordWeaponDamage(weaponID, damage)
- [ ] Spawn XP orbs from EnemyBase.Die()
  - [ ] XP value = floor(maxHealth / 2)
  - [ ] Split into multiple orbs (15 XP each)

#### 3.2 Level-Up System
- [ ] Create ProgressionConfigSO.cs
  - [ ] XP curve (AnimationCurve)
  - [ ] Level rewards (every 5 levels: +25 HP, +15% speed, etc.)
- [ ] Create LevelUpManager.cs
  - [ ] ShowLevelUpOptions()
  - [ ] GenerateWeaponOptions() (3 standard, 5 bonus at level 6+)
  - [ ] SelectWeapon(weaponData)
  - [ ] Reroll mechanic
  - [ ] Pause game during selection
- [ ] Create LevelUpUI.cs
  - [ ] Display 3-5 weapon options
  - [ ] Weapon icons + names + stats
  - [ ] Button click → SelectWeapon()
  - [ ] Reroll button (if available)
  - [ ] Animation: grow + shake
- [ ] Create LevelUpUI prefab (Canvas overlay)

#### 3.3 Implement Remaining Weapons

**Bullet Lover Category**
- [ ] Railgun.cs
  - [ ] Pierce: 2 → 11 enemies
  - [ ] Damage: 40 → 150
  - [ ] Straight line projectile
- [ ] Shotgun.cs
  - [ ] Pellets: 3 → 9
  - [ ] Spread: 15° per pellet
  - [ ] Damage: 5 → 15 per pellet
  - [ ] Cooldown: 50 → 20 frames
- [ ] Barrage.cs
  - [ ] Rockets: 3 → 8
  - [ ] Spread: 20° total
  - [ ] Damage: 8 → 18 per rocket
- [ ] Shrapnel.cs
  - [ ] Main projectile explodes into 4-10 fragments
  - [ ] Main damage: 30 → 80
  - [ ] Fragment damage: 15 → 40

**Arsonist Category**
- [ ] Mortar.cs
  - [ ] Arc trajectory
  - [ ] AOE explosion: 40 → 90 radius
  - [ ] Damage: 15 → 110
  - [ ] Apply burning DOT (3 seconds)
- [ ] Flamethrower.cs
  - [ ] Cone attack: 60° spread
  - [ ] Range: 150 → 250
  - [ ] DPS: 0.2 → 1.5
  - [ ] Burning: 2 seconds
- [ ] FlameTrail.cs
  - [ ] Leave burning trail behind player
  - [ ] Trail lifetime: 120 → 300 frames
  - [ ] DPS: 0.2 → 0.7
- [ ] FlameDance.cs
  - [ ] Continuous AOE around player
  - [ ] Radius: 70 → 160
  - [ ] DPS: 15 → 110

**Tech Enthusiast Category**
- [ ] Chain.cs
  - [ ] Arc between enemies
  - [ ] Bounces: 2 → 7
  - [ ] Damage: 10 → 32
  - [ ] Bounce range: 150px
- [ ] PulseCannon.cs
  - [ ] Knockback: 20 → 50 pixels
  - [ ] Damage: 15 → 45
  - [ ] Range: 500 → 600
  - [ ] Wave visual effect

**Toy Lover Category**
- [ ] Mines.cs
  - [ ] Deploy at player location
  - [ ] Trigger on proximity
  - [ ] Damage: 40 → 150
  - [ ] AOE: 60 → 150 radius
  - [ ] Duration: 30 seconds
- [ ] Turret.cs
  - [ ] Spawn stationary turret
  - [ ] Auto-targets enemies
  - [ ] Turret health: 50 → 200
  - [ ] Damage: 10 → 40
  - [ ] Fire rate: 40 frames
- [ ] Drone.cs
  - [ ] Orbital movement (see Claude.md 2.2)
  - [ ] Kamikaze attack on enemies
  - [ ] Count: 1 → 5 drones
  - [ ] Explosion damage: 40 → 200
- [ ] RoboSpider.cs
  - [ ] Follows player
  - [ ] Auto-collects XP orbs
  - [ ] Duration: 400 → 900 frames

**Special**
- [ ] Vampire.cs
  - [ ] Continuous self-healing
  - [ ] Healing: 0.5 → 3.5 HP/sec
  - [ ] Visual: red particles around player

#### 3.4 Create ScriptableObject Assets
- [ ] Create 16 WeaponDataSO assets
- [ ] Fill in all 10 level data for each weapon
- [ ] Assign projectile prefabs
- [ ] Assign VFX prefabs
- [ ] Set weapon icons

#### 3.5 Additional Enemy Types
- [ ] Create HelicopterEnemy.cs : EnemyBase
  - [ ] Speed: 2.0 → 3.0
  - [ ] Rapid fire weapon
  - [ ] Can fly over obstacles (layer mask)
- [ ] Create Helicopter_Enemy prefab
- [ ] Create SO_HelicopterEnemy.asset

- [ ] Create MotorcycleEnemy.cs : EnemyBase
  - [ ] Speed: 3.5 → 5.0
  - [ ] Kamikaze: 15 contact damage (7.5 easy mode)
  - [ ] Size: 15×10 (smaller)
  - [ ] Health: 15 + floor(score/250) + (level-1)×8
- [ ] Create Motorcycle_Enemy prefab
- [ ] Create SO_MotorcycleEnemy.asset

- [ ] Update EnemySpawner to spawn all 3 types
  - [ ] Weight distribution (tank: 50%, heli: 30%, motorcycle: 20%)

#### 3.6 Boss System
- [ ] Create BossController.cs : EnemyBase
  - [ ] Multiple weapons (machineGun, missile, flamethrower)
  - [ ] Independent cooldowns per weapon
  - [ ] Stationary or slow patrol
  - [ ] 100% money drop
- [ ] Create Boss_Scaled prefab
- [ ] Boss health scaling formula:
  ```
  Level 1-3: [500, 1000, 2500] fixed
  Level 4+: 2500 × 1.25^(level-2)
  ```
- [ ] Add boss spawn logic to EnemySpawner
  - [ ] Fixed times: 60s, 240s, 420s
  - [ ] Scaling bosses every 10 minutes (count +1)

#### 3.7 Player Health System
- [ ] Create PlayerHealth.cs
  - [ ] MaxHealth: 100 → 500
  - [ ] CurrentHealth
  - [ ] DamageReduction: 1.0 (default)
  - [ ] HealthRegen: 0 → 10/sec
  - [ ] TakeDamage(amount)
  - [ ] Heal(amount)
  - [ ] Die() → GameOver
  - [ ] Events: OnHealthChange, OnDeath
- [ ] Add PlayerHealth to PlayerController

#### 3.8 Visual Feedback
- [ ] Screen shake system
  - [ ] Create ScreenShake.cs
  - [ ] TriggerShake(intensity, duration)
  - [ ] Apply to camera position
  - [ ] Triggers: player hit (10, 10), enemy death (4, 8), boss death (15, 15)
- [ ] Damage overlay
  - [ ] Create DamageOverlay prefab (UI Canvas)
  - [ ] Screen cracks (when health < 1/3 max)
  - [ ] Blood splatters (when hit)
  - [ ] Red flash (5 frames on hit)
- [ ] Floating damage text
  - [ ] Create FloatingText.cs
  - [ ] Spawn at hit position
  - [ ] Animate upward + fade
  - [ ] Color by damage tier
- [ ] Enemy hit flash
  - [ ] Turn red for 0.1 seconds
  - [ ] Already in EnemyBase.TakeDamage()

#### 3.9 HUD Basics
- [ ] Create HUDManager.cs
  - [ ] UpdateHealth(current, max)
  - [ ] UpdateXP(current, max)
  - [ ] UpdateScore(score)
  - [ ] UpdateTimer(seconds)
- [ ] Create HUD prefab (Canvas)
  - [ ] Health bar (top-left)
  - [ ] XP bar (bottom center)
  - [ ] Score (top-right)
  - [ ] Timer (top-center)
  - [ ] Level display (next to XP bar)

### Verification Checklist
- [ ] All 16 weapons implemented and functional
- [ ] Weapon behaviors match HTML original:
  - [ ] Laser focus increases damage over time
  - [ ] Chain lightning bounces between enemies
  - [ ] Drone orbits player and attacks
  - [ ] Mines trigger on proximity
  - [ ] Turret auto-targets
  - [ ] Homing missiles curve toward target
  - [ ] Flamethrower cone attack
  - [ ] Mortar arc trajectory
- [ ] XP orbs spawn from dead enemies
- [ ] XP orbs auto-collect within 100px
- [ ] Level-up modal appears at correct XP thresholds
- [ ] Weapon selection modal displays 3-5 options
- [ ] Weapon upgrades apply (damage, cooldown, count increase)
- [ ] All 3 enemy types spawn and behave correctly:
  - [ ] Tanks chase and shoot
  - [ ] Helicopters fly and rapid fire
  - [ ] Motorcycles kamikaze and explode
- [ ] Boss scaling formula working (verify boss health)
- [ ] Bosses use multiple weapons
- [ ] Screen shake on damage
- [ ] Damage overlays show when health < 1/3 max
- [ ] Floating damage numbers appear
- [ ] HUD updates correctly (health, XP, score)
- [ ] 60 FPS with 20 enemies + 200 bullets + 1000 particles

### Critical Files Reference
- HTML: lines 5916-6970 (all 16 weapon fire functions)
- HTML: lines 4550-4650 (boss creation and scaling)
- HTML: lines 7019-7200 (level-up options generation)

---

## PHASE 4: Meta Systems (Week 7-8)

### Goal
Full game experience with menus, save system, and final phase.

### Tasks

#### 4.1 Additional Vehicles
- [ ] Create VehicleDataSO.cs
  - [ ] VehicleID, name, icon
  - [ ] VehicleType enum
  - [ ] MaxHealth, Speed, BaseDamage
  - [ ] Size (width, height)
  - [ ] Unlock cost
- [ ] Create 9 VehicleDataSO assets:
  - **Tanks:**
    - [ ] SO_Altay: speed 2.5, health 100, damage 25
    - [ ] SO_Tulpar: speed 3.0, health 90, damage 15
    - [ ] SO_KaplanSTA: speed 3.2, health 85, damage 20
  - **Helicopters:**
    - [ ] SO_T129: speed 4.2, health 70, damage 3
    - [ ] SO_T625: speed 3.8, health 75, damage 4
    - [ ] SO_ATAK2: speed 4.0, health 80, damage 5
  - **Armored:**
    - [ ] SO_Ejder: speed 3.5, health 80, damage 1
    - [ ] SO_Cobra2: speed 3.6, health 75, damage 2
    - [ ] SO_Arma: speed 3.7, health 85, damage 3
- [ ] Create 9 vehicle prefabs (duplicate Tank_Altay, replace sprites)
- [ ] Update PlayerController to apply vehicle stats on spawn

#### 4.2 Permanent Upgrades
- [ ] Create UpgradeDataSO.cs
  - [ ] UpgradeID, name, icon
  - [ ] Effect enum (MaxHealth, Speed, Damage, HealthRegen, MapUnlock)
  - [ ] Value (amount to add)
- [ ] Create PermanentUpgrade.cs (world pickup)
  - [ ] OnTriggerEnter2D: apply upgrade
  - [ ] Visual: rotating icon
  - [ ] Spawn every 5 levels
- [ ] Create upgrade assets:
  - [ ] +25 MaxHealth
  - [ ] +15% Speed
  - [ ] +5 Damage
  - [ ] +1 Health Regen/sec
  - [ ] Map unlock
- [ ] Spawn upgrades in world (random positions)
  - [ ] Trigger: every 5 levels (5, 10, 15, 20, 25, ...)

#### 4.3 Save System
- [ ] Create SaveManager.cs
  - [ ] SaveData struct: money, unlocks, achievements
  - [ ] SaveGame(data)
  - [ ] LoadGame() → SaveData
  - [ ] AddPersistentMoney(amount)
  - [ ] UnlockVehicle(vehicleID)
  - [ ] JSON serialization (PlayerPrefs or file)
- [ ] Save triggers:
  - [ ] On game end (win or lose)
  - [ ] On vehicle unlock
  - [ ] On achievement unlock
- [ ] Load on game start

#### 4.4 Main Menu
- [ ] Create MainMenu scene
- [ ] Create MainMenuUI.cs
  - [ ] Play button → load Game scene
  - [ ] Vehicle selection screen
  - [ ] Difficulty selection (Normal / Easy)
  - [ ] Settings (volume, controls)
  - [ ] Quit button
- [ ] Create VehicleSelectionUI.cs
  - [ ] Display 9 vehicle cards
  - [ ] Show stats (health, speed, damage)
  - [ ] Lock/unlock icons
  - [ ] Select button → save choice, start game
- [ ] Create UI prefabs:
  - [ ] MainMenuCanvas
  - [ ] VehicleCard (9 instances)
  - [ ] DifficultyToggle

#### 4.5 Pause Menu
- [ ] Create PauseMenuUI.cs
  - [ ] ESC key toggles pause
  - [ ] Resume button
  - [ ] Restart button
  - [ ] Main menu button
  - [ ] Settings
- [ ] Create PauseMenu prefab (Canvas)
- [ ] Integrate with GameManager.PauseGame()

#### 4.6 Weapon Slot UI
- [ ] Create WeaponSlotUI.cs
  - [ ] Display 5 weapon slots
  - [ ] Show icon, level, damage
  - [ ] Update on weapon add/upgrade
- [ ] Create WeaponSlot prefab (UI element)
- [ ] Add 5 slots to HUD

#### 4.7 Final Phase System

**Flag Capture**
- [ ] Create Flag.cs
  - [ ] CaptureProgress (0 → 1)
  - [ ] CaptureTime (10s, 20s, 30s, 40s for flags 1-4)
  - [ ] CaptureRadius: 150px
  - [ ] Decay rate: half of capture rate
  - [ ] OnCaptured event
- [ ] Create Flag prefab
  - [ ] Sprite (flag icon)
  - [ ] Trigger collider (radius 150)
  - [ ] Visual progress bar
- [ ] Spawn 4 flags at corners:
  - [ ] (1000, 1000)
  - [ ] (5000, 1000)
  - [ ] (5000, 5000)
  - [ ] (1000, 5000)

**Firewall**
- [ ] Create Firewall.cs
  - [ ] Center: (3000, 3000)
  - [ ] InitialRadius: 3000
  - [ ] ShrinkRate: 5 pixels/second
  - [ ] DamagePerSecond: 5
  - [ ] CheckPlayerInside()
- [ ] Create Firewall visual
  - [ ] Line renderer (circle)
  - [ ] Red/orange glow
  - [ ] Animate shrinking
- [ ] Trigger after all 4 flags captured

**Guardian Bosses**
- [ ] Spawn 4 guardian bosses (1 per flag)
  - [ ] Stationary positions
  - [ ] High health (current boss level scaling)
  - [ ] Multiple weapons
- [ ] Victory condition: all guardians dead
- [ ] Spawn escape plane (visual only)

**Final Phase Controller**
- [ ] Create FinalPhaseController.cs
  - [ ] StartFinalPhase()
  - [ ] UpdateFirewall()
  - [ ] SpawnGuardians()
  - [ ] CheckVictoryCondition()
  - [ ] Events: OnFinalPhaseStart, OnVictory
- [ ] Add to GameManager

#### 4.8 Victory/Defeat Screens
- [ ] Create GameOverUI.cs
  - [ ] Display final score
  - [ ] Weapon damage breakdown (pie chart or list)
  - [ ] Money earned
  - [ ] Restart button
  - [ ] Main menu button
- [ ] Create VictoryUI.cs
  - [ ] Same as GameOverUI but with victory message
  - [ ] Bonus money for victory
- [ ] Create UI prefabs

#### 4.9 Achievement System (Optional)
- [ ] Create AchievementManager.cs
  - [ ] Track weapon category completions
  - [ ] Unlock bonuses for completing sets
  - [ ] Examples:
    - [ ] "Bullet Lover" set: +15% damage
    - [ ] "Arsonist" set: +20% burn duration
    - [ ] "Tech Enthusiast" set: +25% fire rate
    - [ ] "Toy Lover" set: +1 extra drone/turret
- [ ] Display achievements in main menu

### Verification Checklist
- [ ] All 9 vehicles selectable in main menu
- [ ] Vehicle stats differ correctly (speed, health, damage)
- [ ] Vehicle sprites visible and distinct
- [ ] Save/load system works:
  - [ ] Money persists between sessions
  - [ ] Unlocked vehicles saved
  - [ ] Achievements saved
- [ ] Main menu navigation works:
  - [ ] Play → Game scene
  - [ ] Vehicle selection → saves choice
  - [ ] Quit → closes app
- [ ] Pause menu works:
  - [ ] ESC toggles pause
  - [ ] Game freezes when paused
  - [ ] Resume, restart, main menu buttons functional
- [ ] Weapon slot UI shows equipped weapons
- [ ] Final phase triggers after 4 flags captured:
  - [ ] Flag capture progress bars work
  - [ ] Flags decay when player leaves radius
  - [ ] All 4 flags can be captured
- [ ] Firewall appears after flags captured:
  - [ ] Firewall shrinks over time
  - [ ] Player takes damage outside firewall
  - [ ] Firewall visual is clear
- [ ] 4 guardian bosses spawn at flags
- [ ] Victory screen appears when all guardians dead
- [ ] Defeat screen appears when player dies
- [ ] Weapon damage breakdown displays on game end
- [ ] Permanent upgrades spawn every 5 levels
- [ ] Upgrades apply correctly when picked up

### Critical Files Reference
- HTML: lines 2350-2650 (vehicle definitions)
- HTML: lines 7540-7800 (flag capture logic)
- HTML: lines 8100-8300 (final phase firewall)

---

## PHASE 5: Optimization & Polish (Week 9-10)

### Goal
Release-ready game with optimized performance and polish.

### Tasks

#### 5.1 Performance Optimization

**Object Pooling Tuning**
- [ ] Prewarm counts optimization:
  - [ ] Bullets: Test with 500, 1000, 1500
  - [ ] Enemies: Test with 50, 100, 150
  - [ ] Particles: Test with 500, 1000, 2000
- [ ] Pool growth strategy: double size when exhausted
- [ ] Pool cleanup: return objects after 30 seconds off-screen

**Sprite Atlas**
- [ ] Create Sprite Atlas for all game sprites:
  - [ ] Vehicles
  - [ ] Enemies
  - [ ] Weapons/Projectiles
  - [ ] UI icons
- [ ] Enable "Include in Build" for atlases
- [ ] Verify batch rendering in Profiler (check draw calls)

**Particle System Limits**
- [ ] Max particles per system: 500
- [ ] Stop emission when off-screen
- [ ] Pool particle system game objects

**Camera Culling**
- [ ] Implement frustum culling in EnemyBase
  - [ ] Disable AI update when off-screen
  - [ ] Enable renderer only when visible
- [ ] Culling margin: camera bounds + 200px

**LOD System (Optional)**
- [ ] Distant enemies (> 30 units): simplified sprite
- [ ] Reduce update frequency for far enemies

**Unity Profiler Analysis**
- [ ] Profile game with 1000 bullets + 100 enemies
- [ ] Identify bottlenecks (CPU, GPU, memory)
- [ ] Optimize top 5 expensive operations
- [ ] Target: < 5ms per frame

**ECS for Bullets (Optional)**
- [ ] Convert ProjectileBase to ECS component
- [ ] BulletMovementSystem (parallel jobs)
- [ ] BulletCollisionSystem (with Physics)
- [ ] Expected gain: 2-3x bullet capacity

#### 5.2 Memory Optimization
- [ ] Texture compression:
  - [ ] Sprites: ASTC or ETC2 (mobile)
  - [ ] UI: Truecolor for quality
- [ ] Audio compression: Vorbis for music, ADPCM for SFX
- [ ] Reduce unused assets in build
- [ ] Memory profiler: check for leaks
  - [ ] Run 30 min session, track memory growth
  - [ ] Should be < 100MB stable

#### 5.3 Audio Implementation
- [ ] Import audio assets:
  - [ ] Weapon fire sounds (16 weapons)
  - [ ] Explosion sounds (3 sizes: small, medium, large)
  - [ ] Hit/impact sounds
  - [ ] UI click/hover sounds
  - [ ] Level-up sound (ding)
  - [ ] Victory/defeat stings
  - [ ] Background music (combat, final phase)
- [ ] Create AudioManager.cs
  - [ ] PlaySFX(clipName, volume, pitch)
  - [ ] PlayMusic(clipName, loop)
  - [ ] Audio pooling for SFX (avoid dynamic allocation)
  - [ ] Volume controls (master, SFX, music)
- [ ] Hook up audio triggers:
  - [ ] Weapon fire → WeaponBase.Fire()
  - [ ] Enemy death → EnemyBase.Die()
  - [ ] Player hit → PlayerHealth.TakeDamage()
  - [ ] Level-up → XPManager.LevelUp()
  - [ ] UI clicks → Button.onClick

#### 5.4 UI Polish

**Animations**
- [ ] Level-up modal:
  - [ ] Grow animation (0.5s ease-out)
  - [ ] Shake animation (1.5s loop)
- [ ] Button hover effects:
  - [ ] Scale up 1.1x
  - [ ] Glow outline
- [ ] Health bar:
  - [ ] Smooth lerp (not instant)
  - [ ] Flash red on damage
- [ ] XP bar:
  - [ ] Smooth fill animation
  - [ ] Glow pulse at 90%+
- [ ] Weapon slot:
  - [ ] Highlight when upgraded
  - [ ] Shake when added

**Visual Feedback**
- [ ] Damage numbers:
  - [ ] Color by amount (white < 50, yellow < 100, orange < 300, red 300+)
  - [ ] Critical hit (2x stationary bonus): larger, special color
- [ ] Screen shake intensity tuning:
  - [ ] Player hit: 10
  - [ ] Enemy death: 4
  - [ ] Boss death: 15
  - [ ] Explosion (mine, mortar): 8
- [ ] Muzzle flashes:
  - [ ] Scale by weapon type
  - [ ] Color by weapon category (red: bullet, orange: fire, blue: tech, green: toy)

**Transitions**
- [ ] Scene transitions:
  - [ ] Fade out/in (0.5s)
  - [ ] Loading screen (if needed)
- [ ] Smooth camera zoom (if map view unlocked)
- [ ] Pause menu: blur background

#### 5.5 Balance Tuning

**Enemy Difficulty**
- [ ] Playtest 10 min session:
  - [ ] Target: player level 5-7
  - [ ] Adjust spawn rate if too easy/hard
- [ ] Playtest 20 min session:
  - [ ] Target: player level 10-12
  - [ ] Check boss difficulty
- [ ] Playtest final phase:
  - [ ] Flag capture should take ~2 minutes total
  - [ ] Firewall should close in ~8 minutes
  - [ ] Guardian bosses should be challenging but beatable

**Weapon Balance**
- [ ] Track weapon usage stats (pick rate, damage dealt)
- [ ] Buff underused weapons
- [ ] Nerf overperforming weapons
- [ ] Goal: all weapons viable, no "must-picks"

**XP Curve**
- [ ] Verify level-up frequency:
  - [ ] Target: level-up every ~3-5 minutes early game
  - [ ] Target: level-up every ~5-8 minutes late game
- [ ] Adjust XP requirement multiplier if needed (1.5 → 1.4 or 1.6)

**Economy**
- [ ] Money drop rate: 5% should yield ~100 money per 10 min
- [ ] Vehicle unlock costs: balance with earning rate
- [ ] Permanent upgrade costs: should feel valuable but attainable

#### 5.6 Bug Fixing

**Common Issues to Check**
- [ ] Bullets stuck on screen (off-screen detection)
- [ ] Enemies not despawning (teleport bug)
- [ ] Weapons not firing (cooldown reset bug)
- [ ] UI overlap (z-order issues)
- [ ] Save data corruption (test edge cases)
- [ ] Memory leaks (particles not returning to pool)
- [ ] Physics tunneling (bullets passing through enemies)
- [ ] Collision detection false positives/negatives
- [ ] Audio pops/clicks (volume/pitch limits)

**Testing Pass**
- [ ] 10 minute session (no bugs)
- [ ] 30 minute session (no crashes, stable memory)
- [ ] Victory playthrough (final phase → win)
- [ ] Defeat playthrough (player dies)
- [ ] Restart multiple times (save/load works)
- [ ] Test all 9 vehicles
- [ ] Test all 16 weapons
- [ ] Test all 3 enemy types + bosses

#### 5.7 Build Testing
- [ ] Windows build:
  - [ ] Standalone build (64-bit)
  - [ ] Test on clean Windows machine
  - [ ] Verify controls work
  - [ ] Check performance
- [ ] Mac build:
  - [ ] Universal (Intel + Apple Silicon)
  - [ ] Test on Mac machine
- [ ] Linux build:
  - [ ] Test on Ubuntu/Debian
- [ ] WebGL build (optional):
  - [ ] Optimize texture sizes
  - [ ] Test in Chrome, Firefox, Safari
  - [ ] Check loading time (< 10s)
  - [ ] Verify input works (keyboard + mouse)

#### 5.8 Final Polish
- [ ] Add game credits screen
- [ ] Add tutorial tooltips (first time hints)
- [ ] Add death recap (what killed you)
- [ ] Add statistics screen (total enemies killed, bullets fired, etc.)
- [ ] Add options menu (volume, resolution, fullscreen)
- [ ] Add key rebinding (optional)
- [ ] Add gamepad support (optional)

### Verification Checklist

**Performance**
- [ ] 60 FPS stable with:
  - [ ] 1000 bullets on screen
  - [ ] 100 enemies on screen
  - [ ] 5000 particles on screen
  - [ ] All above simultaneously
- [ ] Memory usage < 100MB
- [ ] No memory leaks (30 min session, memory stable)
- [ ] No GC spikes (< 5ms)
- [ ] Draw calls < 100 (with sprite atlas)

**Audio**
- [ ] All weapon sounds play correctly
- [ ] Explosion sounds vary (small/medium/large)
- [ ] UI clicks audible
- [ ] Background music loops smoothly
- [ ] Volume controls work (master, SFX, music)
- [ ] No audio pops or distortion

**UI**
- [ ] All animations smooth (60 FPS)
- [ ] Buttons have hover effects
- [ ] Health/XP bars animate smoothly
- [ ] Damage numbers are readable
- [ ] UI scales correctly (different resolutions)
- [ ] No UI overlap or clipping

**Balance**
- [ ] All weapons feel useful
- [ ] No weapons are "must-pick" or "never-pick"
- [ ] Enemy difficulty scales appropriately
- [ ] Boss fights are challenging but fair
- [ ] Final phase is completable with skill
- [ ] Economy feels balanced (money drop rate vs unlock costs)

**Stability**
- [ ] No crashes (30 min session)
- [ ] No bugs from testing checklist
- [ ] Save/load works reliably
- [ ] All 9 vehicles work
- [ ] All 16 weapons work
- [ ] All 3 enemy types + bosses work
- [ ] Final phase completable

**Builds**
- [ ] Windows build works (tested on 2+ machines)
- [ ] Mac build works (if targeting Mac)
- [ ] Linux build works (if targeting Linux)
- [ ] WebGL build works (if targeting WebGL)
- [ ] Controls work correctly on all platforms
- [ ] Performance acceptable on all platforms

### Critical Files Reference
- All HTML sections (final verification pass)
- Unity Profiler data (performance benchmarks)
- Playtest feedback notes

---

## Post-Launch (Optional)

### Potential Features
- [ ] Additional vehicles (naval, drone, etc.)
- [ ] New weapons (20+ total)
- [ ] New enemy types (air support, artillery)
- [ ] Co-op multiplayer (2-4 players)
- [ ] Procedural world generation
- [ ] Roguelike mode (permadeath, meta-progression)
- [ ] Steam achievements
- [ ] Leaderboards (high scores)
- [ ] Mobile port (touch controls)

---

## Notes

### Time Estimates
- Phase 1: 10-15 hours
- Phase 2: 20-25 hours
- Phase 3: 30-35 hours
- Phase 4: 25-30 hours
- Phase 5: 20-25 hours
- **Total:** ~105-130 hours (10-13 weeks part-time, 5-7 weeks full-time)

### Dependencies
- Unity 2022.3 LTS or later
- Input System package
- TextMeshPro package
- (Optional) DOTS packages for ECS optimization

### Risks & Mitigation
- **Risk:** Performance issues with 1000+ bullets
  - **Mitigation:** Object pooling, ECS, camera culling
- **Risk:** Complex weapon behaviors hard to replicate
  - **Mitigation:** Reference HTML code directly, test each weapon individually
- **Risk:** Balance issues (game too easy/hard)
  - **Mitigation:** Extensive playtesting, tunable ScriptableObjects
- **Risk:** Save corruption
  - **Mitigation:** Backup saves, robust error handling, versioning

### Success Metrics
- 60 FPS stable (1000 bullets + 100 enemies)
- < 100MB memory usage
- All 16 weapons functional and balanced
- Final phase completable
- No game-breaking bugs
- Positive playtest feedback

---

**Document Version:** 1.0
**Last Updated:** 2026-01-30
**Project:** Pençe Harekati Unity Port
