# Visual Customization System - Server

## Overview

The server handles visual customization data for humanoids (players, NPCs, and mobiles). This document defines the JSON format that the server receives, stores, and transmits for character visual appearance.

## Data Format

The visual data is stored as a JSON string in the `visual` field of the character record. The format is optimized for network transmission and database storage.

### Base Structure

```json
{
  "race": "human",
  "gender": "male",
  "head": {
    "preset": 1,
    "size": 0.5,
    "eyeSize": 0.5,
    "mouthSize": 0.5
  },
  "body": {
    "height": 0.5,
    "weight": 0.5,
    "age": 0.5,
    "chest": 0.5,
    "buttocks": 0.5,
    "trapezius": 0.5,
    "biceps": 0.5,
    "legWidth": 0.5,
    "torso": 0.5,
    "belly": 0.5,
    "feet": 0.5
  },
  "skin": {
    "color": [0.8, 0.6, 0.5],
    "freckles": {
      "enabled": false,
      "intensity": 0.0
    },
    "vitiligo": {
      "enabled": false,
      "coverage": 0.0
    }
  },
  "hair": {
    "style": 1,
    "color": [0.2, 0.1, 0.05]
  },
  "beard": {
    "style": 0,
    "color": [0.2, 0.1, 0.05]
  },
  "tattoos": [],
  "makeup": {
    "enabled": false,
    "lipstick": {
      "enabled": false,
      "color": [0.8, 0.0, 0.0],
      "intensity": 0.0
    },
    "eyeshadow": {
      "enabled": false,
      "color": [0.2, 0.1, 0.3],
      "intensity": 0.0
    },
    "blush": {
      "enabled": false,
      "color": [0.8, 0.4, 0.4],
      "intensity": 0.0
    }
  }
}
```

## Field Definitions

### Race
- **Type**: `string`
- **Values**: `"human"`, `"elf"`, `"dwarf"`, `"orc"`, `"undead"`, etc.
- **Description**: Base model that defines the skeleton and base mesh. Each race has separate male and female models.

### Gender
- **Type**: `string`
- **Values**: `"male"`, `"female"`
- **Description**: Determines which model variant to use for the selected race.

### Head
- **preset**: `number` (0-255)
  - Predefined head shape variations
  - Range: 0-255 (1 byte)
- **size**: `number` (0.0-1.0)
  - Head size multiplier
  - Default: 0.5
- **eyeSize**: `number` (0.0-1.0)
  - Eye size multiplier
  - Default: 0.5
- **mouthSize**: `number` (0.0-1.0)
  - Mouth size multiplier
  - Default: 0.5

### Body
All body parameters use normalized values (0.0-1.0) where:
- 0.0 = minimum
- 0.5 = default/average
- 1.0 = maximum

- **height**: `number` (0.0-1.0)
  - Character height (short to tall)
- **weight**: `number` (0.0-1.0)
  - Body weight (thin to fat)
- **age**: `number` (0.0-1.0)
  - Age appearance (young to old)
- **chest**: `number` (0.0-1.0)
  - Chest size (applies to both genders, affects breast size for females)
- **buttocks**: `number` (0.0-1.0)
  - Buttocks size
- **trapezius**: `number` (0.0-1.0)
  - Trapezius muscle size
- **biceps**: `number` (0.0-1.0)
  - Biceps size
- **legWidth**: `number` (0.0-1.0)
  - Leg width
- **torso**: `number` (0.0-1.0)
  - Torso width
- **belly**: `number` (0.0-1.0)
  - Belly size
- **feet**: `number` (0.0-1.0)
  - Feet size

### Skin
- **color**: `array[3]` (RGB values 0.0-1.0)
  - Base skin color
  - Example: `[0.8, 0.6, 0.5]` = light tan
- **freckles**:
  - **enabled**: `boolean`
  - **intensity**: `number` (0.0-1.0)
- **vitiligo**:
  - **enabled**: `boolean`
  - **coverage**: `number` (0.0-1.0) - Percentage of body covered

### Hair
- **style**: `number` (0-255)
  - Hair style ID (0 = no hair/bald)
  - Range: 0-255 (1 byte)
- **color**: `array[3]` (RGB values 0.0-1.0)
  - Hair color

### Beard
- **style**: `number` (0-255)
  - Beard style ID (0 = no beard)
  - Range: 0-255 (1 byte)
  - Only applies to male characters
- **color**: `array[3]` (RGB values 0.0-1.0)
  - Beard color

### Tattoos
- **Type**: `array`
- **Format**: Array of tattoo objects
```json
{
  "id": 1,
  "position": "arm",
  "scale": 0.5,
  "rotation": 0.0,
  "color": [0.0, 0.0, 0.0]
}
```
- **id**: `number` - Tattoo texture/style ID
- **position**: `string` - Body part: `"arm"`, `"chest"`, `"back"`, `"leg"`, `"face"`
- **scale**: `number` (0.0-1.0) - Size multiplier
- **rotation**: `number` (0.0-360.0) - Rotation in degrees
- **color**: `array[3]` (RGB values 0.0-1.0) - Tattoo color (usually black)

### Makeup
- **enabled**: `boolean` - Global makeup toggle
- **lipstick**:
  - **enabled**: `boolean`
  - **color**: `array[3]` (RGB values 0.0-1.0)
  - **intensity**: `number` (0.0-1.0)
- **eyeshadow**:
  - **enabled**: `boolean`
  - **color**: `array[3]` (RGB values 0.0-1.0)
  - **intensity**: `number` (0.0-1.0)
- **blush**:
  - **enabled**: `boolean`
  - **color**: `array[3]` (RGB values 0.0-1.0)
  - **intensity**: `number` (0.0-1.0)

## Network Protocol

### Client to Server

When a player creates or updates their character visual:

**Packet**: `CreateCharacter` or `UpdateVisual`
**Data**: JSON string containing the visual data structure above.

The server validates:
1. All numeric values are within their specified ranges
2. Race and gender combinations are valid
3. Array lengths match expected sizes
4. Required fields are present

### Server to Client

When transmitting visual data to clients:

**Packet**: `CreateCharacter` or `UpdateVisual`
**Data**: JSON string (same format as above)

The visual data is sent:
- When a character is created
- When a character enters another player's area of interest
- When a character's visual is updated
- In the full character data packet

## Database Storage

### Character Table
- **Field**: `visual`
- **Type**: `TEXT` or `VARCHAR(MAX)`
- **Format**: JSON string
- **Indexing**: Not indexed (low query frequency)

### NPC/Mobile Presets
- **Table**: `visual_presets`
- **Fields**:
  - `id`: Primary key
  - `name`: Preset name
  - `visual_data`: JSON string (same format)
  - `equipment_data`: JSON string (equipment references)
  - `created_at`: Timestamp
  - `updated_at`: Timestamp

## Validation Rules

### Range Validation
All numeric values must be clamped to their specified ranges:
- Normalized values (0.0-1.0): Clamp to [0.0, 1.0]
- Integer IDs (0-255): Clamp to [0, 255]
- RGB values: Clamp to [0.0, 1.0]

### Race/Gender Validation
- Each race must support both male and female models
- Invalid combinations are rejected

### Hair/Beard Validation
- Beard styles only apply to male characters
- Female characters with beard style > 0 are rejected

## Optimization

### Compression
- Visual data is stored as compact JSON
- Consider using binary format for network transmission if size becomes an issue
- Current JSON format is ~500-800 bytes per character

### Caching
- Visual data is cached in memory for online players
- NPC presets are loaded at server startup
- Presets are cached in a Map structure for O(1) lookup

### Update Frequency
- Visual data rarely changes after character creation
- Updates only sent when visual is modified
- Broadcast to area of interest when visual changes

## NPC Presets

NPCs and mobiles use preset visual data stored in the database. The preset includes:
1. Visual customization data (same format as above)
2. Equipment references (additional to visual customization)

Presets are loaded at server startup and cached for performance.

## Error Handling

### Invalid Data
- If visual data is invalid, server uses default visual for the race/gender
- Error is logged but does not prevent character creation/login

### Missing Data
- If visual field is null or empty, server generates default visual based on race/gender
- Default values are applied for all missing fields

### Data Migration
- Old characters without visual data are assigned default visuals on first login
- Migration script can batch-update existing characters

## Security

### Input Validation
- All numeric values are validated against ranges
- String values are sanitized
- JSON structure is validated before parsing
- Maximum JSON size: 2KB per visual data

### Rate Limiting
- Visual updates are rate-limited (max 1 per minute per character)
- Prevents abuse and spam

## Example Usage

### Creating Character with Visual
```typescript
const visualData = {
  race: "human",
  gender: "male",
  head: { preset: 5, size: 0.5, eyeSize: 0.5, mouthSize: 0.5 },
  body: { height: 0.6, weight: 0.4, age: 0.3, /* ... */ },
  skin: { color: [0.7, 0.5, 0.4], freckles: { enabled: true, intensity: 0.3 }, vitiligo: { enabled: false, coverage: 0.0 } },
  hair: { style: 12, color: [0.1, 0.05, 0.02] },
  beard: { style: 3, color: [0.1, 0.05, 0.02] },
  tattoos: [],
  makeup: { enabled: false, /* ... */ }
};

character.visual = JSON.stringify(visualData);
```

### Loading NPC Preset
```typescript
const preset = VisualPresets.get("npc_merchant_001");
const npc = new NPC();
npc.visual = preset.visual_data;
// Apply equipment from preset.equipment_data
```

