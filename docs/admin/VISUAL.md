# Visual Customization System - Admin Panel

## Overview

This document defines how administrators create and manage visual presets for NPCs and mobiles through the admin panel. The system allows creating complete visual configurations including customization data and equipment, enabling dynamic content creation without client rebuilds.

## Preset Management

### Preset Structure

A visual preset contains:
1. **Visual Customization Data**: Same format as player visual data (see server/VISUAL.md)
2. **Equipment Data**: Additional equipment references that compose the visual
3. **Metadata**: Name, description, tags, creation date, etc.

### Preset Database Schema

```sql
CREATE TABLE visual_presets (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    visual_data TEXT NOT NULL, -- JSON string
    equipment_data TEXT NOT NULL, -- JSON string
    tags VARCHAR(500), -- Comma-separated tags
    created_by VARCHAR(36), -- Admin user ID
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);
```

## Admin Panel Interface

### Preset Creation Screen

#### Layout
```
Visual Preset Editor
├── Header
│   ├── Preset Name Input
│   ├── Description Textarea
│   └── Tags Input
├── Visual Customization Tabs
│   ├── Race & Gender
│   ├── Head
│   ├── Body
│   ├── Skin
│   ├── Hair
│   ├── Beard
│   ├── Tattoos
│   └── Makeup
├── Equipment Panel
│   ├── Equipment Slots
│   │   ├── Helmet
│   │   ├── Chest
│   │   ├── Gloves
│   │   ├── Pants
│   │   ├── Boots
│   │   ├── Robe
│   │   ├── Cloak
│   │   ├── Mainhand
│   │   ├── Offhand
│   │   └── Accessories
│   └── Equipment Search/Select
└── Preview & Actions
    ├── 3D Preview Viewport
    ├── Randomize Button
    ├── Save Preset Button
    └── Load Preset Dropdown
```

### Visual Customization Interface

The admin panel provides the same customization options as the character creation screen, but with additional features:

#### Race & Gender Selection
- Dropdown for race selection
- Toggle for gender (affects available options)
- Preview of base model

#### Head Customization
- Head preset selector (0-255)
- Sliders for:
  - Head size (0.0-1.0)
  - Eye size (0.0-1.0)
  - Mouth size (0.0-1.0)

#### Body Customization
- Primary sliders:
  - Height (0.0-1.0)
  - Weight (0.0-1.0)
  - Age (0.0-1.0)
- Detailed sliders:
  - Chest (0.0-1.0)
  - Buttocks (0.0-1.0)
  - Trapezius (0.0-1.0)
  - Biceps (0.0-1.0)
  - Leg Width (0.0-1.0)
  - Torso (0.0-1.0)
  - Belly (0.0-1.0)
  - Feet (0.0-1.0)

#### Skin Customization
- Color picker (RGB/HSV)
- Freckles:
  - Enable/disable toggle
  - Intensity slider (0.0-1.0)
- Vitiligo:
  - Enable/disable toggle
  - Coverage slider (0.0-1.0)

#### Hair Customization
- Hair style selector (0-255, 0 = bald)
- Color picker

#### Beard Customization
- Beard style selector (0-255, 0 = no beard)
- Color picker
- Only available for male gender

#### Tattoos
- Tattoo library browser
- Add/remove tattoos
- For each tattoo:
  - Style ID selector
  - Position selector (arm, chest, back, leg, face)
  - Scale slider
  - Rotation slider
  - Color picker

#### Makeup
- Global enable/disable
- Lipstick:
  - Enable/disable
  - Color picker
  - Intensity slider
- Eyeshadow:
  - Enable/disable
  - Color picker
  - Intensity slider
- Blush:
  - Enable/disable
  - Color picker
  - Intensity slider

### Equipment Panel

#### Equipment Slots
Each equipment slot has:
- Slot name/label
- Current item display
- "Select Item" button
- "Clear" button
- Item preview thumbnail

#### Equipment Selection Modal
- Search bar for item names
- Filter by equipment type
- Filter by item rarity
- Grid/list view toggle
- Item preview with tooltip
- "Select" button

#### Equipment Data Format
```json
{
  "helmet": "ItemRef_Helmet_001",
  "chest": "ItemRef_Chest_002",
  "gloves": "ItemRef_Gloves_001",
  "pants": "ItemRef_Pants_001",
  "boots": "ItemRef_Boots_001",
  "robe": null,
  "cloak": "ItemRef_Cloak_001",
  "mainhand": "ItemRef_Sword_001",
  "offhand": null,
  "ring01": "ItemRef_Ring_001",
  "ring02": null,
  "necklace": "ItemRef_Necklace_001"
}
```

### Preview System

#### 3D Preview Viewport
- Real-time 3D character preview
- Rotate camera (mouse drag)
- Zoom (mouse wheel)
- Lighting options (day/night/indoor)
- Animation preview (idle, walk, attack)

#### Preview Updates
- Updates in real-time as sliders change
- Equipment changes update immediately
- Smooth transitions between changes

### Randomization System

#### Randomize Button
Generates random values for all visual parameters within valid ranges.

#### Randomization Options
- **Full Random**: Randomize everything
- **Partial Random**: 
  - Randomize head only
  - Randomize body only
  - Randomize colors only
  - Randomize hair/beard only
- **Smart Random**: 
  - Maintains aesthetic coherence
  - Avoids extreme combinations
  - Suggests complementary values

#### Randomization Algorithm
```javascript
function randomizeVisual(options) {
    const visual = {
        race: randomElement(availableRaces),
        gender: randomElement(['male', 'female']),
        head: {
            preset: randomInt(0, 255),
            size: randomFloat(0.2, 0.8), // Avoid extremes
            eyeSize: randomFloat(0.3, 0.7),
            mouthSize: randomFloat(0.3, 0.7)
        },
        body: {
            height: randomFloat(0.3, 0.7),
            weight: randomFloat(0.3, 0.7),
            age: randomFloat(0.2, 0.8),
            // ... other body parameters
        },
        // ... other sections
    };
    
    return visual;
}
```

## Preset Management Features

### Preset List View

#### Table Columns
- Name
- Race
- Gender
- Tags
- Created By
- Created Date
- Actions (Edit, Duplicate, Delete, Preview)

#### Filters
- Search by name
- Filter by race
- Filter by gender
- Filter by tags
- Filter by creator
- Filter by date range

#### Bulk Actions
- Delete multiple presets
- Export multiple presets
- Duplicate multiple presets
- Tag management

### Preset Editor

#### Load Preset
- Dropdown/autocomplete to search presets
- Load button loads preset data into editor
- "New Preset" button clears editor

#### Save Preset
- Validates all data
- Checks for duplicate names (with warning)
- Saves to database
- Updates preview thumbnail
- Shows success/error message

#### Duplicate Preset
- Creates copy with "_copy" suffix
- Opens in editor for modification
- Preserves original

#### Delete Preset
- Confirmation dialog
- Checks if preset is in use
- Soft delete (marks as inactive)
- Hard delete option for admins

### Preset Preview

#### Thumbnail Generation
- Automatic thumbnail generation on save
- 256x256px preview image
- Shows in preset list
- Updates when preset is modified

#### Full Preview
- Click preset to open full preview
- 3D model viewer
- Equipment display
- Animation controls
- Export image option

## Integration with NPC/Mobile System

### Assigning Presets to NPCs

#### NPC Editor Integration
- Visual preset selector in NPC creation/editing
- Preview of selected preset
- Override options (can modify preset for specific NPC)

#### Mobile Editor Integration
- Visual preset selector in mobile creation/editing
- Batch assignment (assign same preset to multiple mobiles)
- Random preset selection option

### Preset Loading at Server Startup

#### Loading Process
```typescript
class VisualPresetManager {
    private presets: Map<string, VisualPreset> = new Map();
    
    async loadPresets() {
        const presets = await database.query(
            'SELECT * FROM visual_presets WHERE is_active = true'
        );
        
        for (const preset of presets) {
            this.presets.set(preset.id, {
                id: preset.id,
                name: preset.name,
                visualData: JSON.parse(preset.visual_data),
                equipmentData: JSON.parse(preset.equipment_data)
            });
        }
        
        logger.info(`Loaded ${this.presets.size} visual presets`);
    }
    
    getPreset(id: string): VisualPreset | null {
        return this.presets.get(id) || null;
    }
}
```

#### Server Startup Integration
```typescript
// In server startup
await VisualPresetManager.loadPresets();
```

### Applying Presets to Entities

#### NPC Creation
```typescript
function createNPC(npcData: NPCCreationData) {
    const npc = new NPC();
    
    // Apply visual preset
    if (npcData.visualPresetId) {
        const preset = VisualPresetManager.getPreset(npcData.visualPresetId);
        if (preset) {
            npc.visual = JSON.stringify(preset.visualData);
            
            // Apply equipment from preset
            const equipmentData = preset.equipmentData;
            if (equipmentData.helmet) npc.helmet = equipmentData.helmet;
            if (equipmentData.chest) npc.chest = equipmentData.chest;
            // ... etc
        }
    }
    
    return npc;
}
```

#### Mobile Creation
```typescript
function createMobile(mobileData: MobileCreationData) {
    const mobile = new Mobile();
    
    // Apply visual preset
    if (mobileData.visualPresetId) {
        const preset = VisualPresetManager.getPreset(mobileData.visualPresetId);
        if (preset) {
            mobile.visual = JSON.stringify(preset.visualData);
            
            // Apply equipment
            applyEquipmentFromPreset(mobile, preset.equipmentData);
        }
    }
    
    return mobile;
}
```

## Use Cases

### Creating Quest NPCs

1. Admin opens preset editor
2. Selects race and gender
3. Customizes appearance to match quest theme
4. Adds appropriate equipment
5. Saves preset with quest name
6. Assigns preset to quest NPC in quest editor

### Creating Enemy Variants

1. Admin creates base enemy preset
2. Duplicates preset multiple times
3. Modifies each duplicate (different colors, equipment)
4. Assigns variants to different spawn points
5. Creates visual variety without duplicating code

### Creating Dynamic Content

1. Admin creates preset library
2. Quest system randomly selects presets
3. NPCs spawn with varied appearances
4. Content feels fresh without manual work

## API Endpoints

### Preset Management
- `GET /api/admin/visual-presets` - List all presets
- `GET /api/admin/visual-presets/:id` - Get preset details
- `POST /api/admin/visual-presets` - Create new preset
- `PUT /api/admin/visual-presets/:id` - Update preset
- `DELETE /api/admin/visual-presets/:id` - Delete preset
- `POST /api/admin/visual-presets/:id/duplicate` - Duplicate preset
- `POST /api/admin/visual-presets/randomize` - Generate random preset

### Preview
- `GET /api/admin/visual-presets/:id/preview` - Get preset preview image
- `POST /api/admin/visual-presets/preview` - Generate preview from data

## Security

### Permissions
- Only admins with `visual_presets:create` permission can create presets
- Only admins with `visual_presets:edit` permission can edit presets
- Only admins with `visual_presets:delete` permission can delete presets

### Validation
- All input validated server-side
- Range checks for all numeric values
- Equipment references validated against item database
- Preset names sanitized

### Audit Log
- Log all preset creation/modification/deletion
- Track which admin made changes
- Timestamp all actions

## Performance Considerations

### Caching
- Presets cached in memory at server startup
- Cache invalidated on preset update
- Thumbnail images cached with CDN

### Database Optimization
- Index on `is_active` for faster queries
- Index on `name` for search
- Index on `created_at` for date filtering

### Preview Generation
- Thumbnails generated asynchronously
- Queue system for preview generation
- Cache generated thumbnails

## Best Practices

### Preset Naming
- Use descriptive names: `quest_npc_merchant_001`
- Include race/gender: `human_male_warrior_001`
- Use tags for organization

### Preset Organization
- Group related presets with tags
- Create preset templates for common types
- Document preset usage in description

### Equipment Selection
- Use appropriate equipment for NPC role
- Consider equipment visibility
- Test equipment combinations

### Randomization Usage
- Use randomization to create variants
- Fine-tune randomized results
- Save good random results as presets

