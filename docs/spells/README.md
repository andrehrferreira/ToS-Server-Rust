# Spells System Documentation

## Overview

The Spells system provides a comprehensive collection of magical abilities organized into twelve distinct skillsets (Codex trees). Players select three skillsets to form their character's build, similar to ArcheAge's class system. Each skillset contains 12 spells total: 7 common spells, 4 rare spells, and 1 ancient spell.

## Skillset Structure

Each skillset is organized in its own directory, with individual markdown files for each spell. This modular structure allows for easy navigation, maintenance, and expansion of the spell system.

## Skillsets

1. **[Marksmanship](./marksmanship/)** - Ranged combat with bows and crossbows
2. **[Auraweaving](./auraweaving/)** - Buffs, debuffs, and mobility enhancements
3. **[Warcraft](./warcraft/)** - Powerful melee combat abilities
4. **[Fortitude](./fortitude/)** - Defensive techniques and damage mitigation
5. **[Cursemancy](./cursemancy/)** - Dark magic and curses from the abyss
6. **[Occultism](./occultism/)** - Mind-affecting magic, debuffs, and crowd control
7. **[Shadowcraft](./shadowcraft/)** - Stealth, quick attacks, and evasion
8. **[Bardic Arts](./bardic_arts/)** - Music-based buffs, debuffs, and support
9. **[Elementalism](./elementalism/)** - Devastating elemental magic
10. **[Dual Blade](./dual_blade/)** - Dual-wielding and rapid strikes
11. **[Lifebinding](./lifebinding/)** - Healing and support abilities
12. **[Hexcraft](./hexcraft/)** - Mental manipulation and crowd control

## Spell Organization

### Directory Structure

```
spells/
├── README.md
├── marksmanship/
│   ├── README.md
│   ├── precise_shot.md
│   ├── volley.md
│   ├── evasive_shot.md
│   └── ... (12 spells total)
├── auraweaving/
│   ├── README.md
│   ├── teleport.md
│   ├── protective_aura.md
│   └── ... (12 spells total)
└── ... (other skillsets)
```

### Spell Files

Each spell has its own markdown file containing:
- Spell name and description
- Rarity tier (Common, Rare, Ancient)
- Spell type (Projectile, Target Area, Camera Direction, Single Target, Target Self, Area Centered)
- Damage/effect details
- Cooldown and resource costs
- Conditions and effects
- Visual effects description
- Usage notes

## Codex System Integration

**Tree Selection:**
- Players select three skillsets at character creation
- Can be changed via NPC mage (gold cost)
- Selection limits available spells
- Build diversity through combinations

**Spell Learning:**
- Spells learned via scrolls
- Common scrolls (green): Drop from magical creatures
- Rare scrolls (blue): Drop from bosses
- Ancient scrolls (orange): Drop from raids
- Scrolls are consumable (one-time use)

## Related Documentation

- [Player Spells](../player/SPELLS.md) - Player spells system overview
- [Server Spells](../server/entities/SPELLS.md) - Server-side spell mechanics
- [Client Spells](../client/SPELLS.md) - Client-side spell implementation

## See Also

- [Server README](../server/README.md) - Server documentation overview
- [Player README](../player/README.md) - Player systems overview

