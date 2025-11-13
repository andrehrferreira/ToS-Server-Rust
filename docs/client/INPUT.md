# Input System Documentation

## Overview

The input system handles all player input including movement, camera, targeting, and interactions. This document focuses on movement controls and input handling.

## Movement Input Architecture

The movement system uses **two separate input axes** to properly support lateral movement (strafe):

1. **Forward/Backward Input**: Handles W (forward) and S (backward) keys
2. **Left/Right Input**: Handles A (left/strafe left) and D (right/strafe right) keys

**Why Separate Inputs:**
- **Lateral Movement Support**: Separating inputs allows proper strafe movement
- **Independent Control**: Forward/backward and left/right movements are independent
- **Better Functionality**: Using a single combined input does not work well for lateral movement
- **Combat Positioning**: Essential for combat where players need to strafe while maintaining forward movement

**Implementation Note:**
This separation is critical for the movement system to function correctly. Combining all movement into a single input axis prevents proper lateral movement and strafe functionality.

## Movement Controls

The player controller supports standard WASD movement controls for character movement, implemented with separate input axes for optimal lateral movement support.

**WASD Movement:**

**Forward/Backward Input:**
- **W**: Move forward
- **S**: Move backward

**Left/Right Input:**
- **A**: Strafe left (lateral movement)
- **D**: Strafe right (lateral movement)

**Movement Combinations:**
- **W + A**: Move forward-left (diagonal movement)
- **W + D**: Move forward-right (diagonal movement)
- **S + A**: Move backward-left (diagonal movement)
- **S + D**: Move backward-right (diagonal movement)

**Additional Movement:**
- **Space**: Jump (when on ground)
- **Left Shift**: Sprint (consumes stamina, faster movement)
- **Left Ctrl**: Roll (dodge/evasion move)

**Strafe Movement:**
- Player can strafe laterally (A/D keys) while camera maintains focus
- Camera supports smooth lateral movement (see [CAMERA.md](./CAMERA.md))
- Strafe movement is essential for combat positioning

## Actionbar Controls

The number keys **1 through 0** are allocated for actionbar functions, allowing players to quickly access skills, items, and abilities.

**Actionbar Keys:**
- **1**: Actionbar slot 1
- **2**: Actionbar slot 2
- **3**: Actionbar slot 3
- **4**: Actionbar slot 4
- **5**: Actionbar slot 5
- **6**: Actionbar slot 6
- **7**: Actionbar slot 7
- **8**: Actionbar slot 8
- **9**: Actionbar slot 9
- **0**: Actionbar slot 10

**Actionbar Functionality:**
- Players can assign skills, items, or abilities to actionbar slots
- Pressing the corresponding number key activates the assigned action
- Essential for combat and quick access to frequently used abilities
- Supports both keyboard and mouse clicking on actionbar slots

## Combat Controls

**Target Selection:**
- **TAB**: Cycle through nearby targets (enemies prioritized)
- **Z**: Lock onto selected target
- **` (Backtick/Grave Accent)**: Cancel/unlock current target
- **Left Click**: Select target by clicking directly on enemy/ally

**Autoattack:**
- **Left Mouse Button**: Autoattack (automatic attacks on locked target)
- Autoattack activates when target is locked
- Continuous attacks while target remains locked and in range
- Essential for combat efficiency

## Interaction Controls

**Object Interaction:**
- **E Key**: Interact with object/NPC/chest
- **F Key**: Alternative interaction key
- **Click Interaction**: Click directly on interactive object to interact

**See [PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md) for pathfinding interaction details.**

## System Controls

**Settings:**
- **F12**: Open settings menu

## Related Documentation

- [PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md) - Player controller system (target lock, interactions, motion matching)
- [CAMERA.md](./CAMERA.md) - Camera system (hybrid camera, zoom, rotation)

