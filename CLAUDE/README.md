# Pençe Harekati - Unity Port Project

## 🎮 Project Overview

This project is a complete port of the **Pençe Harekati** bullet hell game from HTML5/Canvas to Unity. The original game features Turkish military vehicles in a top-down shooter with auto-targeting turrets, 16 unique weapons, and a progression system with level-ups and boss battles.

### Original Game
- **Location:** `/HTML/Pence_Harekati.html`
- **Platform:** HTML5 Canvas + Vanilla JavaScript
- **Status:** Complete and playable
- **World Size:** 6000×6000 pixels
- **Frame Rate:** 60 FPS
- **Weapons:** 16 weapons across 4 categories
- **Vehicles:** 9 playable vehicles (tanks, helicopters, armored)
- **Enemies:** 3 enemy types + scalable boss system

### Unity Port Goals
- **Engine:** Unity 2022.3 LTS (2D)
- **Platform:** PC (Windows/Mac/Linux), WebGL (optional)
- **Performance:** 60 FPS with 1000+ bullets, 100+ enemies, 5000+ particles
- **Fidelity:** 100% mechanic preservation from original
- **Architecture:** Data-driven, modular, optimized

---

## 📁 Repository Structure

```
Pence_Harekati/
├── HTML/                           # Original HTML5 game
│   └── Pence_Harekati.html         # Source game (complete, playable)
├── CLAUDE/                         # Unity port documentation
│   ├── README.md                   # This file (project overview)
│   ├── Claude.md                   # Game analysis + Unity architecture
│   └── Todo.md                     # 10-week implementation roadmap
├── Unity/                          # (To be created) Unity project
│   └── PenceHarekati/              # Unity project root
│       ├── Assets/
│       ├── ProjectSettings/
│       └── Packages/
├── .gitignore
├── LICENSE
└── README.md                       # Repository root README
```

---

## 📚 Documentation

### 1. Claude.md
**Comprehensive game analysis and Unity architecture design**

Contains:
- **Complete mechanics analysis**: All 16 weapons, enemy behaviors, progression formulas
- **Unity architecture**: Folder structure, core systems, component design
- **Code references**: Line-by-line mapping from HTML to Unity
- **Performance targets**: 60 FPS with 1000+ bullets
- **Implementation examples**: C# code snippets for core systems

### 2. Todo.md
**10-week implementation roadmap**

Contains:
- **5 development phases**: Core, Combat, Progression, Meta, Polish
- **60+ detailed tasks**: Step-by-step implementation guide
- **Verification checklists**: Testing requirements for each phase
- **Critical file references**: HTML line numbers for reference

### 3. README.md
**This file - project overview and quick start**

---

## 🎯 Key Features

### Gameplay
- **16 Unique Weapons** with 10 upgrade levels each:
  - **Bullet Lover**: DefaultGun, Railgun, Shotgun, Barrage, Shrapnel
  - **Arsonist**: Mortar, Flamethrower, FlameTrail, FlameDance
  - **Tech Enthusiast**: Laser, Chain, Missile, PulseCannon
  - **Toy Lover**: Mines, Turret, Drone, RoboSpider
  - **Special**: Vampire (self-heal)

- **9 Playable Vehicles**:
  - **Tanks**: ALTAY, TULPAR, KAPLAN STA (heavy, slow, high damage)
  - **Helicopters**: T129 ATAK, T625 Gökbey, ATAK-2 (fast, evasive)
  - **Armored**: Ejder Yalçın, Cobra II, Arma (balanced)

- **Enemy System**:
  - **Tank Enemy**: Slow, high health, long-range
  - **Helicopter Enemy**: Fast, aerial, rapid fire
  - **Motorcycle Enemy**: Kamikaze, high speed, contact damage
  - **Bosses**: Scalable health (2500 × 1.25^level), multiple weapons

### Progression
- **XP System**: Collect orbs, level up every ~5 minutes
- **Level-Up Choices**: Select 1-3 weapons from 3-5 options
- **Permanent Upgrades**: +Health, +Speed, +Damage (every 5 levels)
- **Persistent Unlocks**: Money, vehicles, achievements saved between sessions

### Final Phase
- **Flag Capture**: Capture 4 flags at map corners (timed capture)
- **Closing Firewall**: Shrinking circle deals damage outside
- **Guardian Bosses**: 4 final bosses must be defeated
- **Victory Condition**: Destroy all guardians, escape via plane

---

## 🚀 Quick Start

### Prerequisites
- Unity 2022.3 LTS or later
- Git
- Text editor (VS Code recommended)

### Setup Steps

1. **Clone Repository**
   ```bash
   git clone https://github.com/televizyonist/Pence_Harekati.git
   cd Pence_Harekati
   ```

2. **Play Original Game**
   ```bash
   # Open HTML/Pence_Harekati.html in browser
   # Study mechanics, test weapons, understand gameplay
   ```

3. **Read Documentation**
   ```bash
   # Start with Claude.md for architecture understanding
   # Review Todo.md for implementation plan
   ```

4. **Create Unity Project** (when ready to implement)
   ```bash
   # Create new Unity 2D project in Unity/PenceHarekati/
   # Follow Phase 1 tasks in Todo.md
   ```

---

## 🛠️ Technology Stack

### Original Game
- HTML5 Canvas
- Vanilla JavaScript
- 60 FPS game loop (requestAnimationFrame)
- 2D vector graphics

### Unity Port
- **Engine**: Unity 2022.3 LTS
- **Rendering**: 2D Renderer, Universal Render Pipeline (optional)
- **Input**: New Input System
- **UI**: Unity UI (uGUI) + TextMeshPro
- **Physics**: Unity 2D Physics (Rigidbody2D, CircleCollider2D)
- **Audio**: Unity Audio System
- **Optimization**: Object Pooling, Sprite Atlases, Unity DOTS (optional for bullets)
- **Save System**: JSON serialization (PlayerPrefs or file)
- **Version Control**: Git + GitHub

---

## 📊 Performance Targets

| Metric | Target | Strategy |
|--------|--------|----------|
| Frame Rate | 60 FPS constant | Object pooling, culling, ECS |
| Bullets | 1000+ simultaneous | Prewarm pool: 1000, DOTS system |
| Enemies | 100+ active | Prewarm pool: 100, LOD system |
| Particles | 5000+ active | Prewarm pool: 500, lifetime limits |
| Memory | < 100MB | Sprite atlas, texture compression |
| Draw Calls | < 100 | Batching via sprite atlas |

---

## 🗓️ Development Roadmap

### Phase 1: Core Foundation (Week 1-2)
- Unity project setup
- Core managers (GameManager, PoolManager)
- Basic player movement
- Camera system
- **Deliverable:** Player can move, camera follows

### Phase 2: Combat Basics (Week 3-4)
- Weapon system architecture
- 3 weapons (DefaultGun, Laser, Missile)
- Enemy system (Tank enemy only)
- Collision detection
- VFX basics
- **Deliverable:** Player can shoot enemies

### Phase 3: Progression & Content (Week 5-6)
- XP and level-up system
- Remaining 13 weapons
- Helicopter and Motorcycle enemies
- Boss system
- Screen shake, damage overlays
- **Deliverable:** Full progression loop

### Phase 4: Meta Systems (Week 7-8)
- Remaining 8 vehicles
- Save/load system
- Main menu and vehicle selection
- Pause menu
- Final phase (flags, firewall, guardians)
- **Deliverable:** Complete game

### Phase 5: Optimization & Polish (Week 9-10)
- Performance profiling and optimization
- Audio implementation
- UI polish and animations
- Balance tuning
- Bug fixing
- Build testing
- **Deliverable:** Release-ready game

**Total Estimated Time:** 105-130 hours (10-13 weeks part-time)

---

## 🎨 Game Mechanics Highlights

### Stationary Damage Bonus
When player is not moving, all weapon damage is **2x multiplied**. Encourages tactical positioning.

### Laser Focus Mechanic
Laser damage increases the longer it's focused on the same target:
```
Damage Multiplier = min(8, 1 + focusTime)
Max 8x damage after ~7 seconds
```

### Boss Scaling Formula
```
Level 1-3: Fixed health [500, 1000, 2500]
Level 4+: 2500 × 1.25^(level-2)

Example:
Level 5: 3906 HP
Level 10: 11920 HP
```

### XP Requirement Growth
```
XP Required = 50 × 1.5^(level-1)

Level 1: 50 XP
Level 5: 253 XP
Level 10: 1927 XP
```

### Enemy Teleportation
Enemies off-screen for >5 seconds teleport near player to prevent infinite kiting.

---

## 🧪 Testing Strategy

### Performance Tests
- [ ] 60 FPS with 1000 bullets on screen
- [ ] 60 FPS with 100 enemies on screen
- [ ] No frame drops during 20+ simultaneous explosions
- [ ] Memory stable over 30 minute session
- [ ] No GC spikes (< 5ms)

### Gameplay Tests
- [ ] All 16 weapons fire and behave correctly
- [ ] Auto-targeting finds closest enemy
- [ ] Collision detection accurate (no ghost hits)
- [ ] Level-up system triggers at correct XP thresholds
- [ ] Final phase completable (flags → firewall → guardians → victory)

### Save System Tests
- [ ] Money persists between sessions
- [ ] Unlocked vehicles saved
- [ ] Achievements tracked correctly

---

## 🤝 Contributing

This is a personal porting project, but suggestions and feedback are welcome!

### How to Contribute
1. Play the original HTML game (`HTML/Pence_Harekati.html`)
2. Review the architecture (`CLAUDE/Claude.md`)
3. Suggest improvements or report issues via GitHub Issues

### Code Style
- Follow Unity C# conventions
- Use meaningful variable names
- Comment complex algorithms
- Keep classes focused (Single Responsibility Principle)

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](../LICENSE) file for details.

---

## 📞 Contact

- **Developer:** televizyonist
- **GitHub:** https://github.com/televizyonist/Pence_Harekati
- **Project Board:** (To be created)

---

## 🙏 Acknowledgments

- Original HTML game creator (if different from current developer)
- Unity community for optimization techniques
- Turkish Armed Forces for vehicle inspiration

---

## 📖 Additional Resources

### Unity Learning
- [Unity 2D Tutorial](https://learn.unity.com/tutorial/2d-game-development)
- [Object Pooling](https://learn.unity.com/tutorial/object-pooling)
- [ScriptableObjects](https://learn.unity.com/tutorial/introduction-to-scriptable-objects)
- [Unity Profiler](https://docs.unity3d.com/Manual/Profiler.html)

### Game Development Patterns
- [Game Programming Patterns](https://gameprogrammingpatterns.com/)
- [Unity Best Practices](https://unity.com/how-to/programmer-s-guide-unity-best-practices)

### Bullet Hell Design
- [Bullet Patterns](https://store.steampowered.com/app/513610/Danmaku_Unlimited_3/)
- [Top-Down Shooters](https://en.wikipedia.org/wiki/Shoot_%27em_up#Top-down)

---

**Last Updated:** 2026-01-30
**Version:** 1.0
**Status:** Documentation Phase - Implementation Pending
