# Visual Customization System - Client

## Overview

This document defines the client-side implementation for character visual customization. The system is based on Unreal Engine's Metahuman technology and Chaos Cloth for realistic character creation and outfit systems.

## Architecture

### Base System: Metahuman

The character customization system uses Unreal Engine's Metahuman framework as the foundation:
- **Metahuman Creator**: Base character generation
- **Metahuman Animator**: Facial animation and expressions
- **Metahuman DNA**: Genetic data storage for character appearance

### Outfit System: Chaos Cloth

Equipment and clothing use Chaos Cloth physics:
- Realistic cloth simulation
- Dynamic outfit deformation
- Equipment layering system

## Character Creation Screen

### UI Flow

1. **Race Selection**
   - Display available races with preview models
   - Show male/female toggle
   - Preview base model for selected race/gender

2. **Head Customization**
   - Head preset selector (grid of thumbnails)
   - Head size slider
   - Eye size slider
   - Mouth size slider
   - Real-time preview

3. **Body Customization**
   - Primary sliders:
     - Height
     - Weight
     - Age
   - Detailed sliders:
     - Chest
     - Buttocks
     - Trapezius
     - Biceps
     - Leg Width
     - Torso
     - Belly
     - Feet
   - Real-time preview with rotation controls

4. **Skin Customization**
   - Skin color picker (HSV color wheel)
   - Freckles toggle and intensity slider
   - Vitiligo toggle and coverage slider
   - Real-time preview

5. **Hair Customization**
   - Hair style selector (grid with preview)
   - Hair color picker
   - Real-time preview

6. **Beard Customization** (Male only)
   - Beard style selector
   - Beard color picker
   - Real-time preview

7. **Tattoos**
   - Tattoo library browser
   - Placement system (click on body part)
   - Scale and rotation controls
   - Color picker
   - Multiple tattoos support

8. **Makeup** (Optional, mainly for female)
   - Lipstick: color and intensity
   - Eyeshadow: color and intensity
   - Blush: color and intensity
   - Real-time preview

9. **Final Review**
   - 360° rotation preview
   - Lighting options (day/night/indoor)
   - Save and confirm

## Implementation Details

### Metahuman Integration

#### Character Mesh Setup
```cpp
// Load base Metahuman mesh based on race and gender
AMetahumanCharacter* Character = LoadMetahumanBase(Race, Gender);

// Apply DNA modifications
FMetahumanDNA DNA;
DNA.HeadPreset = HeadPreset;
DNA.HeadSize = HeadSize;
DNA.EyeSize = EyeSize;
DNA.MouthSize = MouthSize;
Character->ApplyDNA(DNA);
```

#### Body Morphing
```cpp
// Apply body morph targets
UMorphTargetComponent* MorphTarget = Character->GetMorphTargetComponent();
MorphTarget->SetMorphTarget("Height", HeightValue);
MorphTarget->SetMorphTarget("Weight", WeightValue);
MorphTarget->SetMorphTarget("Chest", ChestValue);
// ... etc
```

#### Skin Material
```cpp
// Dynamic material instance for skin
UMaterialInstanceDynamic* SkinMaterial = Character->GetSkinMaterial();
SkinMaterial->SetVectorParameterValue("SkinColor", SkinColorRGB);
SkinMaterial->SetScalarParameterValue("FrecklesIntensity", FrecklesIntensity);
SkinMaterial->SetScalarParameterValue("VitiligoCoverage", VitiligoCoverage);
```

#### Hair System
```cpp
// Hair mesh component
UStaticMeshComponent* HairMesh = Character->GetHairComponent();
HairMesh->SetStaticMesh(HairStyleMeshes[HairStyleID]);
UMaterialInstanceDynamic* HairMaterial = HairMesh->CreateDynamicMaterialInstance(0);
HairMaterial->SetVectorParameterValue("HairColor", HairColorRGB);
```

#### Beard System
```cpp
// Beard mesh component (male only)
if (Gender == EMale)
{
    UStaticMeshComponent* BeardMesh = Character->GetBeardComponent();
    BeardMesh->SetStaticMesh(BeardStyleMeshes[BeardStyleID]);
    UMaterialInstanceDynamic* BeardMaterial = BeardMesh->CreateDynamicMaterialInstance(0);
    BeardMaterial->SetVectorParameterValue("BeardColor", BeardColorRGB);
}
```

### Chaos Cloth Integration

#### Equipment Layering
```cpp
// Equipment mesh with Chaos Cloth
UChaosClothComponent* EquipmentMesh = NewObject<UChaosClothComponent>(Character);
EquipmentMesh->SetClothAsset(EquipmentClothAsset);
EquipmentMesh->SetSimulationSettings(ClothSimulationSettings);
EquipmentMesh->AttachToComponent(Character->GetMesh(), FAttachmentTransformRules::KeepRelativeTransform);
```

#### Outfit System
```cpp
// Outfit manager
class FOutfitManager
{
    TArray<UChaosClothComponent*> EquipmentLayers;
    
    void AddEquipmentLayer(UChaosClothAsset* ClothAsset, int32 LayerIndex)
    {
        UChaosClothComponent* Layer = CreateClothLayer(ClothAsset);
        Layer->SetLayerIndex(LayerIndex);
        EquipmentLayers.Add(Layer);
    }
    
    void UpdateClothSimulation()
    {
        for (UChaosClothComponent* Layer : EquipmentLayers)
        {
            Layer->UpdateSimulation();
        }
    }
};
```

### Tattoo System

#### Tattoo Application
```cpp
// Tattoo decal component
UDecalComponent* TattooDecal = NewObject<UDecalComponent>(Character);
TattooDecal->SetupAttachment(Character->GetMesh(), BoneName);
TattooDecal->SetDecalMaterial(TattooMaterial);
TattooDecal->SetWorldLocation(TattooPosition);
TattooDecal->SetWorldRotation(TattooRotation);
TattooDecal->SetWorldScale3D(FVector(TattooScale));
```

#### Tattoo Material
```cpp
// Dynamic tattoo material
UMaterialInstanceDynamic* TattooMaterial = UMaterialInstanceDynamic::Create(TattooBaseMaterial, this);
TattooMaterial->SetTextureParameterValue("TattooTexture", TattooTexture);
TattooMaterial->SetVectorParameterValue("TattooColor", TattooColorRGB);
```

### Makeup System

#### Makeup Application
```cpp
// Makeup material parameters
UMaterialInstanceDynamic* FaceMaterial = Character->GetFaceMaterial();
FaceMaterial->SetVectorParameterValue("LipstickColor", LipstickColor);
FaceMaterial->SetScalarParameterValue("LipstickIntensity", LipstickIntensity);
FaceMaterial->SetVectorParameterValue("EyeshadowColor", EyeshadowColor);
FaceMaterial->SetScalarParameterValue("EyeshadowIntensity", EyeshadowIntensity);
FaceMaterial->SetVectorParameterValue("BlushColor", BlushColor);
FaceMaterial->SetScalarParameterValue("BlushIntensity", BlushIntensity);
```

## UI Implementation

### Widget Structure

```
CharacterCreationWidget
├── RaceSelectionPanel
│   ├── RaceGrid (Grid of race buttons with preview)
│   └── GenderToggle (Male/Female)
├── CustomizationTabs
│   ├── HeadTab
│   │   ├── HeadPresetGrid
│   │   ├── HeadSizeSlider
│   │   ├── EyeSizeSlider
│   │   └── MouthSizeSlider
│   ├── BodyTab
│   │   ├── PrimarySliders (Height, Weight, Age)
│   │   └── DetailedSliders (Chest, Buttocks, etc.)
│   ├── SkinTab
│   │   ├── SkinColorPicker
│   │   ├── FrecklesToggle
│   │   └── VitiligoToggle
│   ├── HairTab
│   │   ├── HairStyleGrid
│   │   └── HairColorPicker
│   ├── BeardTab (Male only)
│   │   ├── BeardStyleGrid
│   │   └── BeardColorPicker
│   ├── TattoosTab
│   │   ├── TattooLibrary
│   │   └── PlacementControls
│   └── MakeupTab
│       ├── LipstickControls
│       ├── EyeshadowControls
│       └── BlushControls
└── PreviewPanel
    ├── CharacterViewport (3D preview)
    ├── RotationControls
    ├── LightingOptions
    └── ConfirmButton
```

### Real-time Preview

The preview system updates in real-time as the user adjusts sliders:

```cpp
void UCharacterCreationWidget::OnSliderValueChanged(float NewValue, FString ParameterName)
{
    // Update character parameter
    PreviewCharacter->SetParameter(ParameterName, NewValue);
    
    // Update preview immediately
    PreviewViewport->Invalidate();
}
```

### Randomization

Randomize button generates random values within valid ranges:

```cpp
void UCharacterCreationWidget::RandomizeVisual()
{
    VisualData.Race = GetRandomRace();
    VisualData.Gender = GetRandomGender();
    VisualData.Head.Preset = FMath::RandRange(0, 255);
    VisualData.Head.Size = FMath::RandRange(0.0f, 1.0f);
    // ... randomize all parameters
    
    ApplyVisualDataToPreview();
}
```

## Data Serialization

### Sending to Server

When the player confirms their character creation:

```cpp
FString VisualJSON = SerializeVisualData(VisualData);
SendCreateCharacterPacket(CharacterName, VisualJSON);
```

### Visual Data Structure

Matches the server format (see server/VISUAL.md):
```cpp
struct FVisualData
{
    FString Race;
    FString Gender;
    FHeadData Head;
    FBodyData Body;
    FSkinData Skin;
    FHairData Hair;
    FBeardData Beard;
    TArray<FTattooData> Tattoos;
    FMakeupData Makeup;
};
```

## Performance Optimization

### LOD System
- Use lower LOD for preview when multiple characters visible
- Full quality only for player's own character
- Distance-based LOD for other players

### Async Loading
- Load hair/beard meshes asynchronously
- Preload common styles in background
- Stream textures on demand

### Caching
- Cache generated materials
- Reuse morph target calculations
- Pool decal components for tattoos

## Validation

### Client-side Validation
- Validate all ranges before sending to server
- Prevent invalid race/gender combinations
- Ensure required fields are set

### Preview Validation
- Show warnings for extreme values
- Highlight invalid combinations
- Provide suggestions for improvements

## Accessibility

### Controls
- Keyboard navigation for all UI elements
- Controller support for sliders
- Touch-friendly controls for mobile

### Visual Feedback
- Clear labels for all parameters
- Tooltips explaining each option
- Preview updates immediately

## Future Enhancements

### Advanced Features
- Face sculpting mode (direct vertex manipulation)
- Preset save/load system
- Import/export character data
- Photo mode for character screenshots

### Social Features
- Share character codes
- Character showcase gallery
- Vote on community creations

