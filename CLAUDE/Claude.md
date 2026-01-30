# PENÇE HAREKATI - Comprehensive Game Analysis & Unity Architecture

## Table of Contents
1. [Game Overview](#game-overview)
2. [Complete Mechanics Analysis](#complete-mechanics-analysis)
3. [Unity Architecture Design](#unity-architecture-design)
4. [Implementation Strategy](#implementation-strategy)
5. [Critical Code References](#critical-code-references)

---

## 1. GAME OVERVIEW

**Pençe Harekati** is a top-down bullet hell shooter featuring Turkish military vehicles. The player controls tanks, helicopters, or armored vehicles with auto-targeting turrets, fighting waves of enemies while leveling up and collecting weapons.

### Core Features
- **16 Unique Weapons** across 4 categories with 10 upgrade levels each
- **9 Playable Vehicles** (3 tanks, 3 helicopters, 3 armored vehicles)
- **3 Enemy Types** + scalable boss system
- **XP & Level-Up System** with weapon selection
- **Final Phase** with flag capture and closing firewall
- **Persistent Progression** (money, unlocks between sessions)
- **6000×6000 World** with obstacles and capture points

---

## 2. COMPLETE MECHANICS ANALYSIS

### 2.1 Player Movement & Control

#### Input System
- **WASD Keys**: Directional movement
  - W/S: Forward/backward
  - A/D: Strafe left/right
- **Mouse Cursor**: Turret auto-aim target
- **Space**: Dash ability (cooldown-based)
- **E**: Toggle headlights
- **M**: Map view (if unlocked)
- **ESC**: Pause menu

#### Vehicle Types & Physics

**Tanks** (Heavy, slow, high damage)
```javascript
ALTAY:        { speed: 2.5, health: 100, damage: 25, size: 30×20 }
TULPAR:       { speed: 3.0, health: 90,  damage: 15, size: 28×18 }
KAPLAN STA:   { speed: 3.2, health: 85,  damage: 20, size: 26×18 }
```

**Helicopters** (Fast, evasive, low damage)
```javascript
T129 ATAK:    { speed: 4.2, health: 70,  damage: 3, size: 25×15 }
T625 Gökbey:  { speed: 3.8, health: 75,  damage: 4, size: 24×16 }
ATAK-2:       { speed: 4.0, health: 80,  damage: 5, size: 26×16 }
```

**Armored Vehicles** (Balanced)
```javascript
Ejder Yalçın: { speed: 3.5, health: 80,  damage: 1, size: 22×14 }
Cobra II:     { speed: 3.6, health: 75,  damage: 2, size: 22×14 }
Arma:         { speed: 3.7, health: 85,  damage: 3, size: 24×15 }
```

#### Player Stats (Upgradeable)
```javascript
maxHealth:           100 → 500 (via level-ups)
speed:               2.5-4.2 (vehicle-dependent)
damageBonus:         0 → ∞ (additive)
damageBonusMultiplier: 1.0 → 2.0 (multiplicative)
damageReduction:     1.0 → 0.5 (incoming damage)
healthRegen:         0 → 10/sec
fireRateBonus:       1.0 → 0.5 (reduces cooldown)
xpGainMultiplier:    1.0 → 2.0
```

#### Critical Mechanic: Stationary Damage Bonus
```javascript
finalDamage = (baseDamage + damageBonus) * damageBonusMultiplier
if (!player.isMoving) {
    finalDamage *= 2  // 2x damage when stationary
}
```

### 2.2 Weapon System (16 Weapons)

#### Weapon Categories

**1. Mermi Manyağı (Bullet Lover)**
- defaultGun, railgun, shotgun, barrage, shrapnel

**2. Kundakçı (Arsonist)**
- mortar, flamethrower, flameTrail, flameDance

**3. Teknoloji Meraklısı (Tech Enthusiast)**
- laser, chain, missile, pulseCannon

**4. Oyuncak Sever (Toy Lover)**
- mines, turret, drone, roboSpider

**5. Special**
- vampire (self-healing)

#### Detailed Weapon Specs

##### 1. DefaultGun
```javascript
Level:     1 → 10
Cooldown:  100 frames (constant)
Damage:    Tanks:     15 → 100
           Armored:   2 → 4
           Helicopter: 3 → 5
Count:     1 bullet (constant)
Behavior:  Single shot toward closest enemy
```

##### 2. Laser (Focus Mechanic)
```javascript
Level:     1 → 10
Cooldown:  1 frame (continuous beam)
DPS:       0.5 → 3.5
Range:     200 → 400
Focus:     Damage multiplier increases while locked on target
           Multiplier = min(8, 1 + (focusTime / 60) * 1)
           Max: 8x damage after ~7 seconds of continuous focus
```

##### 3. Railgun (Piercing)
```javascript
Level:     1 → 10
Cooldown:  100 frames
Damage:    40 → 150
Pierces:   2 → 11 enemies
Speed:     12
Behavior:  Straight line, passes through multiple enemies
```

##### 4. Shotgun (Spread)
```javascript
Level:     1 → 10
Cooldown:  50 → 20 frames
Damage:    5 → 15 (per pellet)
Pellets:   3 → 9
Spread:    15° between pellets
```

##### 5. Chain Lightning
```javascript
Level:     1 → 10
Cooldown:  100 → 40 frames
Damage:    10 → 32
Bounces:   2 → 7 enemies
Range:     150 (per bounce)
Behavior:  Arcs between enemies within range
```

##### 6. Homing Missile
```javascript
Level:     1 → 10
Cooldown:  120 → 40 frames
Damage:    25 → 50
Count:     1 → 6 missiles
TurnSpeed: 0.04 radians/frame
Behavior:  Smooth homing toward target
Algorithm:
    targetAngle = atan2(target.y - y, target.x - x) + PI/2
    diff = targetAngle - missile.angle
    // Normalize to [-PI, PI]
    missile.angle += clamp(diff, -turnSpeed, turnSpeed)
```

##### 7. Mortar (AOE + Burning)
```javascript
Level:     1 → 10
Cooldown:  120 → 55 frames
Damage:    15 → 110 (impact)
AOE:       40 → 90 radius
Burning:   3 seconds DOT
BurnDmg:   Same as impact damage
Behavior:  Arc trajectory, explodes on landing
```

##### 8. Flamethrower (Cone)
```javascript
Level:     1 → 10
Cooldown:  3 → 1 frame
DPS:       0.2 → 1.5
Range:     150 → 250
Width:     60° cone
Burning:   2 seconds DOT
```

##### 9. Mines (Deployable)
```javascript
Level:     1 → 10
Cooldown:  150 → 60 frames
Damage:    40 → 150
AOE:       60 → 150 radius
Duration:  30 seconds
Behavior:  Drop at player location, trigger on proximity
```

##### 10. Turret (Autonomous)
```javascript
Level:     1 → 10
Cooldown:  200 → 120 frames (respawn)
Damage:    10 → 40
FireRate:  40 frames between shots
Duration:  Infinite (until destroyed)
Health:    50 → 200
Behavior:  Stationary, auto-targets enemies
```

##### 11. Drone (Orbital)
```javascript
Level:     1 → 10
Cooldown:  500 → 300 frames (per drone)
Count:     1 → 5 drones
Damage:    40 → 200 (on collision/explosion)
Behavior:  Orbits player, kamikaze attacks enemies
Algorithm:
    orbitAngle += 0.02
    targetX = player.x + cos(orbitAngle) * 100
    targetY = player.y + sin(orbitAngle) * 100
    x += (targetX - x) * 0.1  // Smooth interpolation
```

##### 12. Barrage (Rocket Volley)
```javascript
Level:     1 → 10
Cooldown:  80 → 35 frames
Damage:    8 → 18 (per rocket)
Count:     3 → 8 rockets
Spread:    20° total spread
```

##### 13. Shrapnel (Fragmentation)
```javascript
Level:     1 → 10
Cooldown:  60 → 30 frames
Damage:    30 → 80 (main) + 15 → 40 (fragments)
Fragments: 4 → 10 pieces
Behavior:  Main projectile explodes into fragments on impact
```

##### 14. Pulse Cannon (Knockback)
```javascript
Level:     1 → 10
Cooldown:  50 → 30 frames
Damage:    15 → 45
Range:     500 → 600
Knockback: 20 → 50 pixels
Behavior:  Wave pushes enemies back
```

##### 15. Flame Trail (DOT Trail)
```javascript
Level:     1 → 10
Cooldown:  20 → 7 frames
DPS:       0.2 → 0.7
Duration:  120 → 300 frames (trail lifetime)
Behavior:  Leaves burning trail behind player
```

##### 16. Flame Dance (AOE Spin)
```javascript
Level:     1 → 10
Cooldown:  1 frame (continuous)
DPS:       15 → 110
AOE:       70 → 160 radius
Behavior:  Constant fire damage around player
```

##### 17. Vampire (Heal)
```javascript
Level:     1 → 10
Cooldown:  1 frame (continuous)
Healing:   0.5 → 3.5 HP/sec
Behavior:  Passive health regeneration
```

##### 18. Robo Spider (Collector)
```javascript
Level:     1 → 10
Cooldown:  500 frames
Duration:  400 → 900 frames
Behavior:  Follows player, auto-collects XP orbs
```

### 2.3 Enemy System

#### Enemy Types

##### Tank Enemy
```javascript
Size:       30×20 to 40×40 (health-based)
Speed:      1.5 → 2.5
AttackRange: 600
ChaseRange: 1000
Weapon:     Machine gun (5 damage bullets)
Behavior:   Chase player when in range, maintain distance
```

##### Helicopter Enemy
```javascript
Size:       Similar to tank
Speed:      2.0 → 3.0
AttackRange: 500
ChaseRange: 1000
Weapon:     Rapid fire (3 damage bullets)
Behavior:   Aerial movement (can pass over obstacles)
```

##### Motorcycle Enemy (Kamikaze)
```javascript
Size:       15×10
Speed:      3.5 → 5.0
AttackRange: 50 (melee)
ChaseRange: 1000
ContactDmg: 15 (7.5 in easy mode)
Behavior:   Suicide rush toward player
Health:     15 + floor(score/250) + (level-1)*8
```

#### Boss System

##### Fixed Bosses (Levels 0-2)
```javascript
Boss Level 1: { health: 500,  weapons: [machineGun] }
Boss Level 2: { health: 1000, weapons: [machineGun, missile] }
Boss Level 3: { health: 2500, weapons: [machineGun, missile, flamethrower] }
```

##### Scaling Bosses (Level 3+)
```javascript
baseHealth = 2500
scalingFactor = 1.25
bossHealth = floor(baseHealth * pow(scalingFactor, bossLevel - 2))

Example:
Level 4: 3125 HP
Level 5: 3906 HP
Level 6: 4883 HP
Level 10: 11920 HP
```

**Boss Behavior:**
- Stationary or slow patrol
- AttackRange: 600
- Multiple weapons with independent cooldowns
- Guaranteed money drop (100% chance)

#### Enemy Spawning Algorithm

```javascript
// Spawn rate increases over time
baseInterval = 120 frames (2 seconds)
minInterval = 30 frames (0.5 seconds)
rampTime = 300 seconds (5 minutes)

currentInterval = max(minInterval, baseInterval - (gameTime / rampTime) * (baseInterval - minInterval))

// Max enemies scales with level
maxEnemies = 10 + player.level * 2

// Spawn if below cap
if (activeEnemies.length < maxEnemies && timer >= currentInterval) {
    spawnEnemy()
    timer = 0
}
```

#### Enemy Health Scaling

```javascript
// Regular Enemies
baseHealth = 20
scoreMultiplier = floor(score / 250)
levelMultiplier = player.level * 20

health = (baseHealth + scoreMultiplier + levelMultiplier) * difficultyMultiplier

// Difficulty modifiers
Normal: difficultyMultiplier = 1.0
Easy:   difficultyMultiplier = 0.6

// XP value
xpValue = floor(health / 2)
```

#### Teleportation Mechanic

```javascript
// Teleport enemies that are off-screen too long
if (distanceToPlayer > 900 && offScreenTime > 300 frames) {
    // Spawn near player based on movement direction
    angle = player.movementAngle || player.turretAngle
    teleportDistance = max(camera.width, camera.height) / 2 + 150

    newX = player.x + cos(angle) * teleportDistance
    newY = player.y + sin(angle) * teleportDistance

    enemy.position = {newX, newY}
    enemy.offScreenTime = 0
}
```

### 2.4 Progression System

#### XP Collection

```javascript
// XP Orbs
Tier 1 (Green):  5 XP  (common)
Tier 2 (Orange): 15 XP (uncommon)
Tier 3 (Gold):   50 XP (rare)

// Auto-collection radius
magnetRange = 100 pixels (default)
magnetRange = 999999 (with magnet powerup - 5 seconds)

// XP from enemies
xpValue = floor(enemy.health / 2)
```

#### Level-Up System

```javascript
// XP requirement escalation
initialXP = 50
xpToNextLevel *= 1.5 (after each level)

Levels:
1:  50 XP
2:  75 XP
3:  112 XP
4:  169 XP
5:  253 XP
10: 1927 XP
20: 522,769 XP
```

#### Level-Up Rewards

```javascript
// Standard level-up (levels 1-5, 75% chance after level 6)
optionsCount = 3
choicesAllowed = 1

// Bonus level-up (level 6, 25% chance after level 6)
optionsCount = 5
choicesAllowed = 3

// Permanent stat increases (every 5 levels)
Level 5:  +25 maxHealth
Level 10: +15% speed
Level 15: +5 damage bonus
Level 20: +1 health regen/sec
Level 25: Unlock map view
```

### 2.5 Collision Detection

#### Bullet-Enemy Collision
```javascript
distance = sqrt((bullet.x - enemy.x)^2 + (bullet.y - enemy.y)^2)
if (distance < enemy.width/2 + bullet.radius) {
    // Hit detected
    enemy.takeDamage(bullet.damage)
    if (!bullet.pierce) bullet.destroy()
}
```

#### Player-Enemy Contact
```javascript
// Motorcycle kamikaze damage
distance = sqrt((player.x - enemy.x)^2 + (player.y - enemy.y)^2)
if (distance < player.width/2 + enemy.width/2) {
    if (enemy.type === "motorcycle") {
        player.takeDamage(15 * difficultyModifier)
        enemy.die()
    }
}
```

#### Player-Bullet Collision
```javascript
// Enemy bullets damage player
distance = sqrt((player.x - bullet.x)^2 + (player.y - bullet.y)^2)
if (distance < player.width/2 + bullet.radius) {
    damage = bullet.damage * player.damageReduction
    player.takeDamage(damage)
    bullet.destroy()
}
```

#### Obstacle Collision
```javascript
// AABB for rectangular obstacles
if (x < obstacle.x + obstacle.width &&
    x + width > obstacle.x &&
    y < obstacle.y + obstacle.height &&
    y + height > obstacle.y) {
    // Collision - push entity out
}

// Circle collision for round obstacles
distance = sqrt((x - obstacle.x)^2 + (y - obstacle.y)^2)
if (distance < radius + obstacle.radius) {
    // Collision
}
```

### 2.6 Final Phase System

#### Flag Capture Mechanic

```javascript
// 4 flags at world corners
flags = [
    {x: 1000, y: 1000, captureTime: 10},  // 10 seconds
    {x: 5000, y: 1000, captureTime: 20},  // 20 seconds
    {x: 5000, y: 5000, captureTime: 30},  // 30 seconds
    {x: 1000, y: 5000, captureTime: 40}   // 40 seconds
]

// Capture progress
captureRadius = 150 pixels
if (distance(player, flag) < captureRadius) {
    flag.progress += 1 / (captureTime * 60)  // 60 FPS
} else {
    flag.progress -= (1 / (captureTime * 60)) / 2  // Decay at half speed
}

if (flag.progress >= 1.0) {
    flag.captured = true
    spawnGuardianBosses(flag)
}
```

#### Firewall Mechanic

```javascript
// Triggered after all 4 flags captured
initialRadius = 3000
shrinkRate = 5 pixels/second
centerX = 3000
centerY = 3000

currentRadius -= shrinkRate * deltaTime

// Damage player if outside
distanceFromCenter = sqrt((player.x - centerX)^2 + (player.y - centerY)^2)
if (distanceFromCenter > currentRadius) {
    player.takeDamage(5 * deltaTime)  // 5 damage/second
}
```

#### Guardian Bosses

```javascript
// 4 stationary bosses spawn at captured flags
guardians = 4
perGuardian = {
    health: bossScalingFormula(currentBossLevel),
    weapons: [machineGun, missile, flamethrower],
    stationary: true
}

// Win condition
if (guardians.all(g => g.isDead)) {
    triggerVictory()
}
```

### 2.7 Visual Effects

#### Particle System

```javascript
ParticleTypes:
- explosion:   Red/orange bursts (life: 1.0s, fade out)
- smoke:       Gray clouds (life: 1.5-2.0s, expand + fade)
- impact:      White sparks (life: 0.5s)
- flame:       Orange animated (life: 0.4-0.8s)
- confetti:    Multi-color squares with gravity (life: 1.5s)
- blood:       Red splatter (life: 1.0s)
- track:       Tank tread marks (life: variable)
```

#### Screen Shake

```javascript
function triggerScreenShake(intensity, duration) {
    shakeIntensity = intensity  // 4-15 pixels
    shakeDuration = duration    // 8-15 frames

    // Apply each frame
    offsetX = (random() - 0.5) * shakeIntensity
    offsetY = (random() - 0.5) * shakeIntensity

    ctx.translate(offsetX, offsetY)
}

// Triggered by:
- Player taking damage: shake(10, 10)
- Enemy explosion: shake(4, 8)
- Boss death: shake(15, 15)
```

#### Damage Overlay

```javascript
// Critical health warning (< 1/3 max health)
if (player.health < player.maxHealth / 3) {
    drawCrackOverlay()
    drawBloodSplatters()
}

// Hit flash (5 frames)
if (player.isHit) {
    ctx.globalAlpha = 0.3
    ctx.fillStyle = 'red'
    ctx.fillRect(0, 0, canvas.width, canvas.height)
    hitTimer--
}
```

### 2.8 Camera System

```javascript
// Camera follows player
camera.x = player.x - camera.width / 2
camera.y = player.y - camera.height / 2

// Clamp to world boundaries
camera.x = clamp(camera.x, 0, worldWidth - camera.width)
camera.y = clamp(camera.y, 0, worldHeight - camera.height)

// Transform world objects to screen space
screenX = worldX - camera.x
screenY = worldY - camera.y
```

---

## 3. UNITY ARCHITECTURE DESIGN

### 3.1 Folder Structure

```
Assets/_Project/
├── Prefabs/
│   ├── Player/
│   │   ├── Vehicles/
│   │   │   ├── Tank_Altay.prefab
│   │   │   ├── Helicopter_ATAK.prefab
│   │   │   └── ... (9 vehicles total)
│   │   └── Components/
│   │       ├── Turret.prefab
│   │       ├── Shield_VFX.prefab
│   │       └── DashTrail.prefab
│   ├── Enemies/
│   │   ├── Tank_Enemy.prefab
│   │   ├── Helicopter_Enemy.prefab
│   │   ├── Motorcycle_Enemy.prefab
│   │   └── Bosses/
│   │       └── Boss_Scaled.prefab
│   ├── Weapons/
│   │   ├── Projectiles/
│   │   │   ├── Bullet_Default.prefab
│   │   │   ├── Laser_Beam.prefab
│   │   │   ├── Missile_Homing.prefab
│   │   │   └── ... (16 projectile types)
│   │   └── Effects/
│   │       ├── MuzzleFlash.prefab
│   │       ├── Explosion.prefab
│   │       └── ImpactEffect.prefab
│   ├── Collectibles/
│   │   ├── XPOrb_Tier1.prefab
│   │   ├── XPOrb_Tier2.prefab
│   │   ├── XPOrb_Tier3.prefab
│   │   ├── Flag.prefab
│   │   └── PermanentUpgrade.prefab
│   ├── VFX/
│   │   ├── Particles/
│   │   │   ├── PS_Explosion.prefab
│   │   │   ├── PS_Fire.prefab
│   │   │   ├── PS_Smoke.prefab
│   │   │   └── PS_Blood.prefab
│   │   └── UI/
│   │       ├── DamageOverlay.prefab
│   │       └── ScreenCrack.prefab
│   └── World/
│       ├── Obstacle_Rect.prefab
│       ├── Obstacle_Circle.prefab
│       ├── Prison.prefab
│       └── Firewall.prefab
├── Scripts/
│   ├── Core/
│   │   ├── GameManager.cs
│   │   ├── PoolManager.cs
│   │   ├── AudioManager.cs
│   │   └── SaveManager.cs
│   ├── Player/
│   │   ├── PlayerController.cs
│   │   ├── PlayerHealth.cs
│   │   ├── PlayerMovement.cs
│   │   ├── TurretController.cs
│   │   ├── WeaponManager.cs
│   │   └── AbilityManager.cs
│   ├── Enemies/
│   │   ├── EnemyBase.cs
│   │   ├── EnemyAI.cs
│   │   ├── EnemySpawner.cs
│   │   ├── BossController.cs
│   │   └── EnemyTypes/
│   │       ├── TankEnemy.cs
│   │       ├── HelicopterEnemy.cs
│   │       └── MotorcycleEnemy.cs
│   ├── Weapons/
│   │   ├── WeaponBase.cs
│   │   ├── ProjectileBase.cs
│   │   ├── TargetingSystem.cs
│   │   └── WeaponTypes/
│   │       ├── DefaultGun.cs
│   │       ├── Laser.cs
│   │       ├── Missile.cs
│   │       ├── Drone.cs
│   │       └── ... (16 weapons)
│   ├── Progression/
│   │   ├── XPManager.cs
│   │   ├── LevelUpManager.cs
│   │   ├── AchievementManager.cs
│   │   └── UpgradeManager.cs
│   ├── World/
│   │   ├── WorldGenerator.cs
│   │   ├── CameraController.cs
│   │   ├── ObstacleManager.cs
│   │   └── FinalPhaseController.cs
│   ├── UI/
│   │   ├── HUDManager.cs
│   │   ├── LevelUpUI.cs
│   │   ├── MainMenuUI.cs
│   │   ├── PauseMenuUI.cs
│   │   └── WeaponSlotUI.cs
│   └── Utilities/
│       ├── ObjectPool.cs
│       ├── FloatingText.cs
│       ├── ScreenShake.cs
│       └── Extensions.cs
├── ScriptableObjects/
│   ├── Weapons/
│   │   ├── WeaponDataSO.cs (base class)
│   │   └── Weapons/
│   │       ├── SO_DefaultGun.asset
│   │       ├── SO_Laser.asset
│   │       └── ... (16 weapons)
│   ├── Enemies/
│   │   ├── EnemyDataSO.cs
│   │   └── Enemies/
│   │       ├── SO_TankEnemy.asset
│   │       ├── SO_HelicopterEnemy.asset
│   │       ├── SO_MotorcycleEnemy.asset
│   │       └── SO_Boss.asset
│   ├── Vehicles/
│   │   ├── VehicleDataSO.cs
│   │   └── Vehicles/
│   │       ├── SO_Altay.asset
│   │       ├── SO_ATAK.asset
│   │       └── ... (9 vehicles)
│   ├── Upgrades/
│   │   ├── UpgradeDataSO.cs
│   │   └── Upgrades/
│   │       ├── SO_HealthUpgrade.asset
│   │       ├── SO_SpeedUpgrade.asset
│   │       └── SO_DamageUpgrade.asset
│   └── GameConfig/
│       ├── SpawnConfigSO.cs
│       ├── BalanceConfigSO.cs
│       └── ProgressionConfigSO.cs
├── Sprites/
│   ├── Vehicles/
│   ├── Enemies/
│   ├── Weapons/
│   ├── UI/
│   └── World/
├── Audio/
│   ├── SFX/
│   └── Music/
└── Scenes/
    ├── MainMenu.unity
    ├── Game.unity
    └── Testing/
```

### 3.2 Core Systems

#### GameManager.cs

```csharp
using UnityEngine;
using System;

public enum GameMode { Normal, Easy }
public enum GamePhase { Combat, FinalPhase }

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    [Header("References")]
    public PlayerController Player;
    public EnemySpawner EnemySpawner;
    public XPManager XPManager;
    public FinalPhaseController FinalPhase;

    [Header("Game State")]
    public GameMode CurrentMode = GameMode.Normal;
    public GamePhase CurrentPhase = GamePhase.Combat;
    public float GameTime;
    public int Score;
    public int SessionMoney;

    [Header("Configuration")]
    public BalanceConfigSO BalanceConfig;

    // Managers
    private PoolManager poolManager;
    private AudioManager audioManager;
    private SaveManager saveManager;

    // Events
    public event Action OnGameStart;
    public event Action OnGameOver;
    public event Action OnGameWin;
    public event Action<GamePhase> OnPhaseChange;
    public event Action<int> OnScoreChange;

    private bool isPaused;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;

        poolManager = GetComponent<PoolManager>();
        audioManager = GetComponent<AudioManager>();
        saveManager = GetComponent<SaveManager>();
    }

    private void Start()
    {
        StartGame();
    }

    private void Update()
    {
        if (!isPaused)
        {
            GameTime += Time.deltaTime;
        }
    }

    public void StartGame()
    {
        GameTime = 0;
        Score = 0;
        SessionMoney = 0;
        CurrentPhase = GamePhase.Combat;

        OnGameStart?.Invoke();
    }

    public void AddScore(int points)
    {
        Score += points;
        OnScoreChange?.Invoke(Score);
    }

    public void AddMoney(int amount)
    {
        SessionMoney += amount;
    }

    public void StartFinalPhase()
    {
        CurrentPhase = GamePhase.FinalPhase;
        OnPhaseChange?.Invoke(GamePhase.FinalPhase);
        FinalPhase.StartFinalPhase();
    }

    public void GameOver()
    {
        isPaused = true;
        OnGameOver?.Invoke();
    }

    public void Victory()
    {
        isPaused = true;
        saveManager.AddPersistentMoney(SessionMoney);
        OnGameWin?.Invoke();
    }

    public void PauseGame()
    {
        isPaused = true;
        Time.timeScale = 0;
    }

    public void ResumeGame()
    {
        isPaused = false;
        Time.timeScale = 1;
    }

    public float GetDifficultyMultiplier()
    {
        return CurrentMode == GameMode.Easy ? 0.6f : 1.0f;
    }
}
```

#### PoolManager.cs

```csharp
using UnityEngine;
using System.Collections.Generic;

public class PoolManager : MonoBehaviour
{
    [System.Serializable]
    public class Pool
    {
        public string poolName;
        public GameObject prefab;
        public int prewarmCount = 50;
    }

    [Header("Pool Definitions")]
    public Pool[] pools;

    private Dictionary<string, Queue<GameObject>> poolDictionary;
    private Dictionary<string, GameObject> prefabDictionary;
    private Transform poolParent;

    private void Awake()
    {
        poolDictionary = new Dictionary<string, Queue<GameObject>>();
        prefabDictionary = new Dictionary<string, GameObject>();
        poolParent = new GameObject("ObjectPools").transform;

        foreach (var pool in pools)
        {
            PrewarmPool(pool.poolName, pool.prefab, pool.prewarmCount);
        }
    }

    private void PrewarmPool(string poolName, GameObject prefab, int count)
    {
        Queue<GameObject> objectPool = new Queue<GameObject>();
        prefabDictionary[poolName] = prefab;

        Transform parent = new GameObject($"Pool_{poolName}").transform;
        parent.SetParent(poolParent);

        for (int i = 0; i < count; i++)
        {
            GameObject obj = Instantiate(prefab, parent);
            obj.SetActive(false);
            objectPool.Enqueue(obj);
        }

        poolDictionary[poolName] = objectPool;
    }

    public GameObject Get(string poolName)
    {
        if (!poolDictionary.ContainsKey(poolName))
        {
            Debug.LogWarning($"Pool {poolName} doesn't exist!");
            return null;
        }

        GameObject obj;

        if (poolDictionary[poolName].Count > 0)
        {
            obj = poolDictionary[poolName].Dequeue();
        }
        else
        {
            obj = Instantiate(prefabDictionary[poolName]);
        }

        obj.SetActive(true);
        return obj;
    }

    public void Return(string poolName, GameObject obj)
    {
        if (!poolDictionary.ContainsKey(poolName))
        {
            Debug.LogWarning($"Pool {poolName} doesn't exist!");
            Destroy(obj);
            return;
        }

        obj.SetActive(false);
        poolDictionary[poolName].Enqueue(obj);
    }

    // Convenience methods
    public GameObject GetBullet(string bulletType) => Get($"Bullet_{bulletType}");
    public GameObject GetEnemy(string enemyType) => Get($"Enemy_{enemyType}");
    public GameObject GetParticle(string particleType) => Get($"Particle_{particleType}");

    public void ReturnBullet(string bulletType, GameObject obj) => Return($"Bullet_{bulletType}", obj);
    public void ReturnEnemy(string enemyType, GameObject obj) => Return($"Enemy_{enemyType}", obj);
    public void ReturnParticle(string particleType, GameObject obj) => Return($"Particle_{particleType}", obj);
}
```

### 3.3 Player System

#### PlayerController.cs

```csharp
using UnityEngine;
using System.Collections.Generic;

public class PlayerController : MonoBehaviour
{
    [Header("Components")]
    public PlayerMovement Movement;
    public PlayerHealth Health;
    public WeaponManager WeaponManager;
    public AbilityManager AbilityManager;
    public TurretController Turret;

    [Header("Vehicle Data")]
    public VehicleDataSO CurrentVehicle;

    [Header("Stats")]
    public float DamageBonusMultiplier = 1f;
    public float FireRateBonus = 1f;
    public float DamageReduction = 1f;
    public float XPGainMultiplier = 1f;

    [Header("Progression")]
    public int Level = 1;
    public float CurrentXP;
    public float XPToNextLevel = 50;

    public bool IsMoving => Movement.IsMoving;

    private void Start()
    {
        Initialize();
    }

    private void Update()
    {
        Movement.UpdateMovement();
        Turret.UpdateTargeting();
        WeaponManager.UpdateWeapons();
        AbilityManager.UpdateAbilities();
    }

    public void Initialize()
    {
        Health.Initialize(CurrentVehicle.MaxHealth);
        Movement.Initialize(CurrentVehicle.Speed);
        WeaponManager.Initialize();
    }

    public float GetFinalDamageMultiplier()
    {
        float multiplier = DamageBonusMultiplier;

        // 2x damage when stationary
        if (!IsMoving)
        {
            multiplier *= 2f;
        }

        return multiplier;
    }
}
```

#### PlayerMovement.cs

```csharp
using UnityEngine;

public enum VehicleType { Tank, Helicopter, Armored }

public class PlayerMovement : MonoBehaviour
{
    [Header("Settings")]
    public VehicleType VehicleType;
    public float Speed = 3f;
    public float DashSpeed = 8f;
    public float DashDuration = 0.3f;

    [Header("World Bounds")]
    public Vector2 WorldMin = Vector2.zero;
    public Vector2 WorldMax = new Vector2(6000, 6000);

    private Rigidbody2D rb;
    private Vector2 moveInput;
    private bool isDashing;
    private float dashTimer;

    public bool IsMoving => moveInput.sqrMagnitude > 0.01f;

    private void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    public void Initialize(float speed)
    {
        Speed = speed;
    }

    public void UpdateMovement()
    {
        GetInput();
        HandleDash();
        ApplyMovement();
        ClampToBounds();
    }

    private void GetInput()
    {
        float horizontal = Input.GetAxisRaw("Horizontal");
        float vertical = Input.GetAxisRaw("Vertical");
        moveInput = new Vector2(horizontal, vertical).normalized;

        if (Input.GetKeyDown(KeyCode.Space))
        {
            StartDash();
        }
    }

    private void HandleDash()
    {
        if (isDashing)
        {
            dashTimer -= Time.deltaTime;
            if (dashTimer <= 0)
            {
                isDashing = false;
            }
        }
    }

    private void ApplyMovement()
    {
        float currentSpeed = isDashing ? DashSpeed : Speed;
        Vector2 velocity = moveInput * currentSpeed;

        rb.velocity = velocity;
    }

    private void StartDash()
    {
        if (GetComponent<PlayerController>().AbilityManager.CanDash())
        {
            isDashing = true;
            dashTimer = DashDuration;
        }
    }

    private void ClampToBounds()
    {
        Vector3 pos = transform.position;
        pos.x = Mathf.Clamp(pos.x, WorldMin.x, WorldMax.x);
        pos.y = Mathf.Clamp(pos.y, WorldMin.y, WorldMax.y);
        transform.position = pos;
    }
}
```

### 3.4 Weapon System

#### WeaponDataSO.cs

```csharp
using UnityEngine;

public enum WeaponCategory { BulletLover, Arsonist, TechEnthusiast, ToyLover, Special }
public enum TargetingMode { Closest, Strongest, Random }

[CreateAssetMenu(fileName = "Weapon_", menuName = "PenceHarekati/Weapon")]
public class WeaponDataSO : ScriptableObject
{
    [Header("Identity")]
    public string WeaponID;
    public string WeaponName;
    public WeaponCategory Category;
    public Sprite Icon;

    [Header("Level Data")]
    public WeaponLevelData[] LevelData = new WeaponLevelData[10];

    [Header("Projectile")]
    public GameObject ProjectilePrefab;
    public string ProjectilePoolName;

    [Header("VFX")]
    public GameObject MuzzleFlashPrefab;
    public GameObject ImpactEffectPrefab;

    [Header("Behavior")]
    public bool AutoTarget = true;
    public TargetingMode TargetingMode = TargetingMode.Closest;

    public WeaponLevelData GetLevelData(int level)
    {
        int index = Mathf.Clamp(level - 1, 0, LevelData.Length - 1);
        return LevelData[index];
    }
}

[System.Serializable]
public class WeaponLevelData
{
    public int Level;
    public float Damage;
    public float Cooldown;
    public float Range;
    public int ProjectileCount;
    public float SpecialValue; // For focus time, burn duration, etc.
}
```

#### WeaponBase.cs

```csharp
using UnityEngine;

public abstract class WeaponBase : MonoBehaviour
{
    [Header("Data")]
    public WeaponDataSO Data;
    public int CurrentLevel = 1;

    [Header("State")]
    protected float cooldownTimer;
    protected PlayerController player;
    protected TargetingSystem targeting;
    protected PoolManager poolManager;

    protected WeaponLevelData CurrentLevelData => Data.GetLevelData(CurrentLevel);

    protected virtual void Awake()
    {
        player = GetComponentInParent<PlayerController>();
        targeting = player.GetComponent<TargetingSystem>();
        poolManager = GameManager.Instance.GetComponent<PoolManager>();
    }

    public virtual void UpdateWeapon()
    {
        if (cooldownTimer > 0)
        {
            cooldownTimer -= Time.deltaTime;
        }

        if (CanFire())
        {
            Fire();
        }
    }

    protected virtual bool CanFire()
    {
        return cooldownTimer <= 0 && targeting.HasTarget();
    }

    public abstract void Fire();

    protected void ResetCooldown()
    {
        float baseCooldown = CurrentLevelData.Cooldown / 60f; // Convert frames to seconds
        cooldownTimer = baseCooldown * player.FireRateBonus;
    }

    protected void SpawnProjectile(Vector3 position, Quaternion rotation)
    {
        GameObject projectile = poolManager.Get(Data.ProjectilePoolName);
        projectile.transform.position = position;
        projectile.transform.rotation = rotation;

        ProjectileBase proj = projectile.GetComponent<ProjectileBase>();
        proj.Initialize(CurrentLevelData.Damage, player.GetFinalDamageMultiplier(), Data.WeaponID);
    }

    protected void SpawnMuzzleFlash()
    {
        if (Data.MuzzleFlashPrefab != null)
        {
            GameObject flash = Instantiate(Data.MuzzleFlashPrefab, transform.position, transform.rotation);
            Destroy(flash, 0.2f);
        }
    }

    public void UpgradeWeapon()
    {
        if (CurrentLevel < Data.LevelData.Length)
        {
            CurrentLevel++;
        }
    }
}
```

#### Example: Laser.cs

```csharp
using UnityEngine;

public class Laser : WeaponBase
{
    [Header("Laser Specific")]
    private LineRenderer laserBeam;
    private Transform currentTarget;
    private float focusTime;
    private float maxFocusMultiplier = 8f;

    protected override void Awake()
    {
        base.Awake();
        laserBeam = GetComponent<LineRenderer>();
    }

    public override void Fire()
    {
        Transform target = targeting.GetClosestEnemy();

        if (target == null)
        {
            DisableLaser();
            return;
        }

        // Update focus time
        if (target == currentTarget)
        {
            focusTime += Time.deltaTime;
        }
        else
        {
            currentTarget = target;
            focusTime = 0;
        }

        // Calculate focus multiplier
        float focusMultiplier = Mathf.Min(maxFocusMultiplier, 1f + focusTime);

        // Apply damage
        float dps = CurrentLevelData.Damage;
        float finalDamage = dps * focusMultiplier * player.GetFinalDamageMultiplier() * Time.deltaTime;

        IDamageable damageable = target.GetComponent<IDamageable>();
        damageable?.TakeDamage(finalDamage, Data.WeaponID);

        // Visual
        DrawLaser(target.position);

        // No cooldown for laser (continuous beam)
    }

    private void DrawLaser(Vector3 targetPos)
    {
        laserBeam.enabled = true;
        laserBeam.SetPosition(0, transform.position);
        laserBeam.SetPosition(1, targetPos);

        // Color based on focus
        float t = focusTime / 7f; // Max focus at 7 seconds
        laserBeam.startColor = Color.Lerp(Color.red, Color.white, t);
        laserBeam.endColor = laserBeam.startColor;
    }

    private void DisableLaser()
    {
        laserBeam.enabled = false;
        currentTarget = null;
        focusTime = 0;
    }
}
```

### 3.5 Enemy System

#### EnemyBase.cs

```csharp
using UnityEngine;

public interface IDamageable
{
    void TakeDamage(float damage, string weaponID);
}

public abstract class EnemyBase : MonoBehaviour, IDamageable
{
    [Header("Data")]
    public EnemyDataSO Data;

    [Header("Stats")]
    public float Health;
    public float MaxHealth;
    public float Speed;
    public float Damage;

    [Header("Components")]
    public EnemyAI AI;
    public Rigidbody2D Rb;
    public SpriteRenderer Renderer;

    [Header("VFX")]
    public ParticleSystem BloodEffect;

    protected PlayerController target;
    protected bool isHit;
    protected float hitTimer;

    public virtual void Initialize(EnemyDataSO data, int playerLevel, int score)
    {
        Data = data;

        // Calculate scaled health
        float difficultyMult = GameManager.Instance.GetDifficultyMultiplier();
        MaxHealth = (20 + Mathf.Floor(score / 250f) + playerLevel * 20) * difficultyMult;
        Health = MaxHealth;

        Speed = data.Speed;
        Damage = data.Damage;

        UpdateHealthColor();

        target = GameManager.Instance.Player;
        AI.Initialize(target.transform);
    }

    protected virtual void Update()
    {
        if (isHit)
        {
            hitTimer -= Time.deltaTime;
            if (hitTimer <= 0)
            {
                isHit = false;
                Renderer.color = Color.white;
            }
        }
    }

    public virtual void TakeDamage(float damage, string weaponID)
    {
        Health -= damage;
        isHit = true;
        hitTimer = 0.1f;
        Renderer.color = Color.red;

        // Track weapon damage
        GameManager.Instance.XPManager.RecordWeaponDamage(weaponID, damage);

        // Spawn floating text
        FloatingText.Create(transform.position, damage.ToString("0"));

        if (Health <= 0)
        {
            Die();
        }
    }

    protected virtual void Die()
    {
        SpawnXPOrbs();
        SpawnExplosion();
        SpawnMoney();

        GameManager.Instance.AddScore(Data.ScoreValue);

        PoolManager poolManager = GameManager.Instance.GetComponent<PoolManager>();
        poolManager.ReturnEnemy(Data.EnemyType, gameObject);
    }

    protected void SpawnXPOrbs()
    {
        float xpValue = Mathf.Floor(MaxHealth / 2f);

        // Spawn multiple orbs
        int orbCount = Mathf.CeilToInt(xpValue / 15f);
        for (int i = 0; i < orbCount; i++)
        {
            Vector2 randomOffset = Random.insideUnitCircle * 0.5f;
            Vector3 spawnPos = transform.position + (Vector3)randomOffset;

            XPOrb orb = Instantiate(Resources.Load<GameObject>("Prefabs/XPOrb"), spawnPos, Quaternion.identity)
                .GetComponent<XPOrb>();
            orb.Initialize(15f);
        }
    }

    protected void SpawnExplosion()
    {
        PoolManager poolManager = GameManager.Instance.GetComponent<PoolManager>();
        GameObject explosion = poolManager.GetParticle("Explosion");
        explosion.transform.position = transform.position;
    }

    protected void SpawnMoney()
    {
        // 5% chance for regular enemies, 100% for bosses
        float dropChance = Data.IsBoss ? 1f : 0.05f;

        if (Random.value < dropChance)
        {
            // Spawn money collectible
        }
    }

    protected void UpdateHealthColor()
    {
        if (Health >= 800)
            Renderer.color = new Color(1f, 0.4f, 0.7f); // Pink
        else if (Health >= 500)
            Renderer.color = new Color(1f, 0.84f, 0f); // Gold
        else if (Health >= 300)
            Renderer.color = Color.cyan;
        else if (Health >= 200)
            Renderer.color = Color.white;
        else if (Health >= 120)
            Renderer.color = new Color(0.8f, 0.2f, 1f); // Purple
        else if (Health >= 60)
            Renderer.color = new Color(1f, 0.6f, 0.2f); // Orange
        else
            Renderer.color = new Color(1f, 0.27f, 0.27f); // Red
    }
}
```

---

## 4. IMPLEMENTATION STRATEGY

### 4.1 Development Phases

#### Phase 1: Core Foundation (Week 1-2)
**Goal:** Basic playable prototype

**Tasks:**
1. Unity project setup (2D template, Unity 2022.3 LTS)
2. Folder structure creation
3. Core managers: GameManager, PoolManager, AudioManager, SaveManager
4. Basic player movement (one vehicle type: Tank_Altay)
5. Camera controller with player tracking
6. Input system setup (New Input System)
7. World boundaries (6000×6000)

**Deliverable:** Player can move around with WASD, camera follows, world boundaries work

**Verification:**
- [ ] Player moves smoothly with WASD
- [ ] Camera tracks player position
- [ ] World boundaries prevent player from leaving map
- [ ] 60 FPS stable

---

#### Phase 2: Combat Basics (Week 3-4)
**Goal:** Functional shooting and enemy system

**Tasks:**
8. Weapon system architecture (WeaponBase, WeaponDataSO)
9. Implement 3 weapons: DefaultGun, Laser, Missile
10. TurretController (mouse aim, auto-target closest enemy)
11. TargetingSystem component
12. Enemy system: EnemyBase, EnemyAI
13. Implement Tank enemy only
14. Collision detection (circle-circle)
15. ProjectileBase class
16. Object pooling for bullets (100+ capacity)
17. Basic VFX: muzzle flash, explosion particles
18. Damage system (IDamageable interface)

**Deliverable:** Player can shoot enemies, enemies chase and die

**Verification:**
- [ ] 3 weapons fire correctly
- [ ] Turret aims at mouse cursor
- [ ] Auto-target finds closest enemy
- [ ] Enemies take damage and die
- [ ] Collision detection accurate
- [ ] Particle effects spawn on hit/death
- [ ] Object pooling working (no lag from instantiation)

---

#### Phase 3: Progression & Content (Week 5-6)
**Goal:** Full weapon roster and progression loop

**Tasks:**
19. XP system: XPManager, XPOrb prefab
20. XP collection (auto-collect within 100px radius)
21. Level-up system: LevelUpManager, LevelUpUI
22. Weapon selection modal (3-5 options UI)
23. Implement remaining 13 weapons:
    - Railgun, Shotgun, Barrage, Shrapnel
    - Mortar, Flamethrower, FlameTrail, FlameDance
    - Chain, PulseCannon
    - Mines, Turret, Drone, RoboSpider
    - Vampire
24. Implement Helicopter enemy
25. Implement Motorcycle enemy (kamikaze)
26. Boss system: BossController, scaling health formula
27. EnemySpawner with time-based spawn rate
28. Screen shake on hits
29. Damage overlay (screen cracks, blood)
30. FloatingText for damage numbers
31. PlayerHealth component with visual feedback

**Deliverable:** Complete progression loop with all weapons and enemies

**Verification:**
- [ ] XP orbs spawn and are collectable
- [ ] Level-up modal appears at correct XP thresholds
- [ ] All 16 weapons implemented and functional
- [ ] Weapon behaviors match HTML original (laser focus, chain bounce, drone orbit, etc.)
- [ ] 3 enemy types spawn and behave correctly
- [ ] Boss scaling formula working (health increases correctly)
- [ ] Screen shake on damage
- [ ] Damage overlays show when health < 1/3 max

---

#### Phase 4: Meta Systems (Week 7-8)
**Goal:** Full game experience with menus and final phase

**Tasks:**
32. Implement remaining 8 vehicles:
    - TULPAR, KAPLAN STA (tanks)
    - T625 Gökbey, ATAK-2 (helicopters)
    - Ejder Yalçın, Cobra II, Arma (armored)
33. VehicleDataSO for each vehicle
34. Permanent upgrades system (world pickups every 5 levels)
35. SaveManager: persistence between sessions
36. SaveData structure: money, unlocks, achievements
37. Main menu: MainMenuUI scene
38. Vehicle selection screen
39. Weapon discovery tracking
40. Pause menu: PauseMenuUI
41. Final phase system: FinalPhaseController
42. Flag prefabs (4 flags at corners)
43. Flag capture mechanic (timed capture with decay)
44. Firewall system (shrinking circle)
45. Guardian bosses (4 stationary bosses)
46. Escape plane (victory condition)
47. HUDManager: health bar, XP bar, score, timer, weapon slots

**Deliverable:** Full game with menus, save system, and final phase

**Verification:**
- [ ] All 9 vehicles selectable and playable
- [ ] Vehicle stats differ correctly
- [ ] Save/load system works (money persists)
- [ ] Main menu navigation functional
- [ ] Pause menu works (ESC key)
- [ ] Final phase triggers after 4 flags captured
- [ ] Firewall damages player outside radius
- [ ] Firewall shrinks correctly over time
- [ ] Guardian bosses spawn and are beatable
- [ ] Victory screen appears on win
- [ ] All UI elements visible and updating

---

#### Phase 5: Optimization & Polish (Week 9-10)
**Goal:** Release-ready game with performance optimization

**Tasks:**
48. Object pool optimization (prewarm counts tuning)
49. Sprite Atlas creation for batch rendering
50. Particle system limits (max 5000 particles)
51. Camera culling for off-screen entities
52. Performance profiling with Unity Profiler
53. Optimize enemy AI (skip updates for far enemies)
54. Memory leak detection and fixes
55. Balance tuning:
    - Enemy health scaling
    - Weapon damage values
    - XP requirements
    - Spawn rates
56. Audio implementation:
    - Weapon fire sounds
    - Explosion sounds
    - UI click sounds
    - Background music
57. UI polish:
    - Animations (level-up modal grow/shake)
    - Button hover effects
    - Smooth transitions
58. Bug fixing pass
59. Build testing (Windows, Mac, Linux)
60. WebGL build (optional)

**Deliverable:** Polished, optimized, release-ready game

**Verification:**
- [ ] 60 FPS with 1000+ bullets + 100+ enemies + 5000 particles
- [ ] Memory usage < 100MB
- [ ] No memory leaks (stable over 30 min session)
- [ ] All audio working
- [ ] UI responsive and animated
- [ ] No game-breaking bugs
- [ ] Builds successfully for target platforms
- [ ] Game feels polished (particles, sounds, feedback)

---

### 4.2 Critical Success Factors

1. **Object Pooling** - Essential for 1000+ bullets performance
2. **Data-Driven Design** - ScriptableObjects enable easy balancing
3. **Modular Weapons** - WeaponBase architecture allows easy additions
4. **Accurate Porting** - Preserve exact mechanics from HTML (formulas, timings)
5. **Performance First** - Profile early, optimize continuously

---

## 5. CRITICAL CODE REFERENCES

### 5.1 HTML File Sections

**Player & Vehicle Definitions**
- **Location:** `/home/user/Pence_Harekati/HTML/Pence_Harekati.html` (lines 2350-2650)
- **Contains:** 9 vehicle configs (speed, health, damage), player stats structure
- **Unity Mapping:** → VehicleDataSO assets, PlayerController stats

**Enemy Spawning & Scaling**
- **Location:** `/home/user/Pence_Harekati/HTML/Pence_Harekati.html` (lines 4458-4650)
- **Contains:** createEnemy() function, boss scaling formula, enemy types
- **Unity Mapping:** → EnemySpawner.cs, EnemyBase.Initialize(), BossController.cs

**Weapon Fire Functions**
- **Location:** `/home/user/Pence_Harekati/HTML/Pence_Harekati.html` (lines 5916-6970)
- **Contains:** All 16 weapon fire functions (fireLaser, fireMissile, fireDrone, etc.)
- **Unity Mapping:** → WeaponTypes/*.cs (Laser.cs, Missile.cs, Drone.cs, etc.)

**Level-Up System**
- **Location:** `/home/user/Pence_Harekati/HTML/Pence_Harekati.html` (lines 7019-7540)
- **Contains:** showLevelUpOptions(), XP calculation, weapon selection logic
- **Unity Mapping:** → LevelUpManager.cs, LevelUpUI.cs, XPManager.cs

**Game Loop Structure**
- **Location:** `/home/user/Pence_Harekati/HTML/Pence_Harekati.html` (lines 8642-8840)
- **Contains:** Update order (player, enemies, bullets, collisions, camera)
- **Unity Mapping:** → GameManager.Update(), component Update() order

---

## 6. PERFORMANCE TARGETS

| Metric | Target | Strategy |
|--------|--------|----------|
| Frame Rate | 60 FPS constant | Object pooling, culling, ECS for bullets |
| Bullets | 1000+ simultaneous | Pool prewarm: 1000, DOTS system |
| Enemies | 100+ active | Pool prewarm: 100, LOD system |
| Particles | 5000+ active | Pool prewarm: 500, limit lifetime |
| Memory | < 100MB | Sprite atlas, texture compression |
| Load Time | < 5 seconds | AssetBundle loading, async scenes |

---

## 7. TECHNOLOGY STACK

- **Engine:** Unity 2022.3 LTS
- **Rendering:** 2D Renderer Pipeline (URP optional)
- **Input:** New Input System
- **UI:** Unity UI (uGUI) + TextMeshPro
- **Audio:** Unity Audio System
- **Physics:** Unity 2D Physics (Rigidbody2D, CircleCollider2D)
- **Optimization:** Unity DOTS (optional for bullet system)
- **Version Control:** Git + GitHub
- **Platform:** PC (Windows/Mac/Linux), WebGL (secondary)

---

## 8. TESTING CHECKLIST

### Gameplay Tests
- [ ] Player movement responsive (WASD)
- [ ] Turret aims correctly (mouse cursor)
- [ ] All 16 weapons fire and behave correctly
- [ ] Auto-targeting finds closest enemy
- [ ] Stationary damage bonus (2x) works
- [ ] Collision detection accurate (no ghost hits)
- [ ] XP collection and level-up triggers
- [ ] Level-up modal shows 3-5 weapon options
- [ ] Weapon upgrades apply correctly (1-10 levels)
- [ ] Enemy spawning rate increases over time
- [ ] Boss scaling formula correct (health increases)
- [ ] Final phase triggers after 4 flags captured
- [ ] Firewall damages player correctly
- [ ] Victory condition works (all guardians dead)

### Performance Tests
- [ ] 60 FPS with 1000 bullets on screen
- [ ] 60 FPS with 100 enemies on screen
- [ ] No frame drops during explosions (20+ simultaneous)
- [ ] Memory stable over 30 minute session
- [ ] No garbage collection spikes (< 5ms)

### Save System Tests
- [ ] Money persists between sessions
- [ ] Unlocked vehicles saved
- [ ] Weapon discovery tracked
- [ ] Achievements saved

### UI Tests
- [ ] HUD updates correctly (health, XP, score)
- [ ] Level-up modal animates smoothly
- [ ] Main menu navigation works
- [ ] Pause menu freezes game
- [ ] Weapon slot UI shows correct icons/levels

---

## CONCLUSION

This document provides a comprehensive blueprint for porting the HTML Pençe Harekati game to Unity. The architecture prioritizes:

1. **Performance** - Object pooling, culling, data-driven design
2. **Modularity** - Easy to add new weapons, enemies, vehicles
3. **Accuracy** - Preserves exact mechanics, formulas, and behaviors
4. **Scalability** - Supports 1000+ bullets, 100+ enemies at 60 FPS

Follow the 5-phase roadmap for systematic implementation over 10 weeks.

---

**Document Version:** 1.0
**Last Updated:** 2026-01-30
**Project:** Pençe Harekati Unity Port
