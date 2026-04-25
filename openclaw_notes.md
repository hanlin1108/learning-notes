# OpenClaw Learning Notes

> OpenClaw is an open-source C++ reimplementation of the classic 1997 platformer *Captain Claw*, bringing a beloved DOS-era game to modern operating systems.

---

## Table of Contents

1. [What Is OpenClaw?](#1-what-is-openclaw)
2. [Why It's Popular](#2-why-its-popular)
3. [Key Capabilities](#3-key-capabilities)
4. [Architecture Overview](#4-architecture-overview)
5. [How to Get Started](#5-how-to-get-started)
6. [Common Use Cases](#6-common-use-cases)
7. [Tips & Gotchas](#7-tips--gotchas)

---

## 1. What Is OpenClaw?

**Captain Claw** (1997) was a critically acclaimed 2D platformer by Monolith Productions. As the original game became incompatible with modern OSes, the OpenClaw project was born: a ground-up **C++ reimplementation** of the game engine that can run the original game files on Windows, Linux, and macOS.

| | Original Captain Claw | OpenClaw |
|---|---|---|
| Platform | MS-DOS / Win95 | Windows / Linux / macOS |
| Source | Closed, proprietary | Open-source (GitHub) |
| Renderer | DirectDraw | SDL2 |
| Maintainability | Frozen | Actively improved |
| Moddable | No | Yes |

**Key distinction**: OpenClaw is a **game engine clone**, not a ROM hack. It reimplements the engine; you still need the original game's asset files (levels, graphics, sounds) to run it.

---

## 2. Why It's Popular

### For Retro Gamers
- Lets you play a beloved 90s classic on modern hardware — no DOSBox needed
- Better performance and compatibility than emulation
- Proper keyboard/gamepad support with modern remapping

### For Developers & Learners
- One of the best real-world examples of a **complete, working 2D game engine in C++**
- Clean architecture — ECS (Entity-Component-System), physics, rendering, audio all separated
- Shows how to reverse-engineer and document a complex system
- Active codebase with ~10,000 commits — excellent for reading production C++ at scale

### Community
- Regular contributions from game dev enthusiasts
- Detailed documentation on the game's binary file formats (`.wwd`, `.til`, `.pid`)
- Used in university game development courses as a case study

---

## 3. Key Capabilities

```
┌────────────────────────────────────────┐
│              OpenClaw Engine            │
│                                        │
│  ┌──────────┐  ┌──────────────────┐   │
│  │ Renderer │  │  Physics Engine  │   │
│  │  (SDL2)  │  │  (custom AABB)   │   │
│  └──────────┘  └──────────────────┘   │
│  ┌──────────┐  ┌──────────────────┐   │
│  │  Audio   │  │   Game Logic /   │   │
│  │  System  │  │   ECS Entities   │   │
│  └──────────┘  └──────────────────┘   │
│  ┌──────────────────────────────────┐  │
│  │     Asset Loader (.wwd files)    │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

| Feature | Details |
|---|---|
| **All 14 levels** | Complete reimplementation of every original level |
| **Enemies & AI** | All enemy types with original behavior patterns |
| **Physics** | Tile-based AABB collision, slopes, moving platforms |
| **Audio** | MIDI music + WAV sound effects via SDL2_mixer |
| **Modding** | Custom levels via `.wwd` file format tools |
| **Save/Load** | Checkpoint system matching the original |
| **Controls** | Keyboard + gamepad, fully remappable |
| **Cheats** | Original cheat codes supported |

---

## 4. Architecture Overview

### The ECS Pattern (Entity-Component-System)

Every game object (Claw, enemies, pickups) is an **Entity** with attached **Components**:

```
Entity: Captain Claw
├── PositionComponent    (x, y coordinates)
├── PhysicsComponent     (velocity, gravity, collision box)
├── AnimationComponent   (sprite sheet, current frame)
├── HealthComponent      (HP, invincibility frames)
└── InputComponent       (keyboard/gamepad bindings)
```

**Systems** then process all entities that have the right components:

```cpp
// PhysicsSystem processes everything with Position + Physics components
for (auto& entity : entities_with<PositionComponent, PhysicsComponent>()) {
    applyGravity(entity);
    resolveCollisions(entity, tilemap);
    entity.position += entity.velocity * deltaTime;
}
```

### Asset Pipeline

Original game assets are stored in `.wwd` (World, Wap Dungeon) binary files. OpenClaw includes a parser that reads:

```
CLAW.REZ (resource archive)
└── LEVEL1/
    ├── LEVEL1.WWD   ← tile map, object placement, scripting
    ├── IMAGES/      ← sprite PID files
    ├── SOUNDS/      ← WAV files
    └── MUSIC/       ← MIDI files
```

---

## 5. How to Get Started

### Prerequisites

- **C++ compiler** — GCC 7+ or MSVC 2017+ or Clang
- **CMake** ≥ 3.10
- **SDL2** libraries: `SDL2`, `SDL2_image`, `SDL2_ttf`, `SDL2_mixer`
- **Original game assets** — legally required; obtain from GOG.com or your original CD

### Install Dependencies (Ubuntu/Debian)

```bash
sudo apt-get install cmake libsdl2-dev libsdl2-image-dev \
     libsdl2-ttf-dev libsdl2-mixer-dev
```

### Build

```bash
git clone https://github.com/pjsdream/OpenClaw.git
cd OpenClaw
mkdir build && cd build
cmake ..
make -j4
```

### Configure

Create or edit `config.xml` in the same directory as the binary:

```xml
<Config>
    <Assets>
        <RezArchive>CLAW.REZ</RezArchive>  <!-- path to original game assets -->
    </Assets>
    <Display>
        <Width>1280</Width>
        <Height>720</Height>
        <Fullscreen>false</Fullscreen>
    </Display>
    <Audio>
        <SoundVolume>50</SoundVolume>
        <MusicVolume>50</MusicVolume>
    </Audio>
</Config>
```

### Run

```bash
./OpenClaw
```

Place `CLAW.REZ` (from your original game install) next to the binary. The engine reads all assets from it at startup.

### Controls (Default)

| Action | Key |
|---|---|
| Move | Arrow keys / WASD |
| Jump | Alt / Space |
| Attack | Ctrl |
| Fire pistol | Delete |
| Pause | Escape |

---

## 6. Common Use Cases

### 1. Playing the Game
The primary use: play Captain Claw natively without DOSBox.

```bash
./OpenClaw          # launches directly to main menu
```

### 2. Learning C++ Game Development
OpenClaw is widely used as a **reference codebase** for:
- How to structure a 2D game engine
- Real-world ECS implementation in C++
- SDL2 integration patterns
- Game physics without a third-party physics engine

### 3. Modding & Custom Levels

Use community tools to edit `.wwd` files and create custom levels:

```
1. Open LEVEL1.WWD in WapMap (community level editor)
2. Modify tiles, place enemies, set checkpoints
3. Replace the original .wwd file in CLAW.REZ
4. Run OpenClaw — your level loads automatically
```

### 4. Porting & Platform Support
Developers have ported OpenClaw to:
- **Raspberry Pi** — runs at 60fps on Pi 4
- **Android** — community fork with touch controls
- **WebAssembly** — browser-based via Emscripten

---

## 7. Tips & Gotchas

### Common Issues

| Problem | Fix |
|---|---|
| "CLAW.REZ not found" | Make sure `CLAW.REZ` is in the same directory as the binary |
| Black screen on launch | Check SDL2 library versions — needs SDL2 ≥ 2.0.8 |
| No sound | Install `libsdl2-mixer-dev` and ensure MIDI support is compiled in |
| Slow performance | Build in Release mode: `cmake -DCMAKE_BUILD_TYPE=Release ..` |
| Level crashes | Verify your `CLAW.REZ` is the unmodified v1.4 release |

### Build Tips

```bash
# Debug build (with symbols for gdb)
cmake -DCMAKE_BUILD_TYPE=Debug ..

# Release build (optimized, 3-5× faster)
cmake -DCMAKE_BUILD_TYPE=Release ..

# Verify SDL2 is found correctly before building
cmake .. --debug-find
```

### Reading the Code

Best files to start with when studying the codebase:

| File | What to learn |
|---|---|
| `ClawGameApp.cpp` | Application entry point and initialization |
| `ActorFactory.cpp` | How game entities are created from data |
| `PhysicsComponent.cpp` | AABB collision and physics loop |
| `BaseGameLogic.cpp` | The main game loop and state machine |

---

## The 20% That Gives You 80%

If you only remember four things:

1. **OpenClaw = engine, not ROM** — it reimplements the engine; you supply the original `CLAW.REZ` assets
2. **Built on SDL2** — cross-platform graphics, audio, and input via one library
3. **ECS architecture** — everything is entities + components; systems process them each frame
4. **Best use: learning C++ game dev** — the codebase is one of the cleanest real-world 2D game engines you can read and run

---

*Last updated: 2026-04-25*
