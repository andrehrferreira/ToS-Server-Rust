# Camera System Documentation

## Overview

The camera system uses a hybrid approach similar to ArcheAge, providing flexible third-person to near top-down view. The camera supports smooth transitions between viewing angles, making it suitable for both exploration and large-scale PvP combat. Players can seamlessly zoom from a close third-person view to an almost top-down perspective using mouse scroll, optimizing the view for different gameplay scenarios.

## Camera Features

**View Modes:**
- **Third-Person**: Standard third-person view for exploration and normal gameplay
- **Near Top-Down**: Zoomed out view (almost top-down) optimized for large-scale PvP
- **Smooth Transition**: Seamless zoom between third-person and top-down views

**Camera Controls:**
- **Scroll Wheel**: Used for camera zoom (scroll back to zoom out, scroll forward to zoom in)
- **Middle Mouse Button**: Hold for freelook (free camera rotation mode)
- **Mouse Scroll Back**: Zoom out to near top-down view (optimal for large-scale PvP)
- **Mouse Scroll Forward**: Zoom in to close third-person view
- **Horizontal Rotation**: Rotate camera horizontally around player
- **Vertical Angle**: Adjustable vertical angle (limited range)
- **Strafe Support**: Camera supports lateral strafing movement (player can move sideways while camera maintains focus)

## Camera Implementation

```cpp
// Unreal Engine C++ Implementation
class UHybridCameraComponent : public UCameraComponent
{
    GENERATED_BODY()

public:
    // Camera mode
    UPROPERTY(BlueprintReadOnly)
    ECameraMode CurrentCameraMode;
    
    // Zoom distance
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float MinZoomDistance = 200.0f;  // Close third-person
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float MaxZoomDistance = 2000.0f; // Near top-down
    
    UPROPERTY(BlueprintReadOnly)
    float CurrentZoomDistance = 500.0f;
    
    // Camera rotation
    UPROPERTY(BlueprintReadOnly)
    float HorizontalRotation = 0.0f;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float VerticalAngle = -45.0f; // Looking down angle
    
    // Zoom speed
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float ZoomSpeed = 50.0f;
    
    // Rotation speed
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float RotationSpeed = 90.0f;
    
    // Update camera
    virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
    
    // Zoom functions
    UFUNCTION(BlueprintCallable)
    void ZoomIn(float DeltaZoom);
    
    UFUNCTION(BlueprintCallable)
    void ZoomOut(float DeltaZoom);
    
    // Rotation functions
    UFUNCTION(BlueprintCallable)
    void RotateHorizontal(float DeltaRotation);
    
private:
    // Update camera position
    void UpdateCameraPosition(float DeltaTime);
    
    // Smooth zoom interpolation
    void InterpolateZoom(float DeltaTime);
};
```

## Camera Modes

**Third-Person Mode:**
- **Distance**: 200-800 units from player
- **Angle**: Standard third-person angle
- **Use Case**: Exploration, normal combat, general gameplay
- **Advantages**: Good visibility of character and immediate surroundings

**Near Top-Down Mode:**
- **Distance**: 1200-2000 units from player
- **Angle**: Almost top-down view (nearly perpendicular to ground)
- **Activation**: Scroll mouse wheel backward to zoom out
- **Use Case**: Large-scale PvP, tactical combat, area awareness
- **Advantages**: Maximum field of view, better situational awareness, optimal for massive PvP battles
- **Transition**: Smooth zoom transition from third-person to top-down

**Zoom Transition:**
- Smooth interpolation between zoom levels
- Configurable zoom speed
- Maintains camera stability during transition
- No jarring camera movements

## Camera Rotation

**Horizontal Rotation:**
- Camera rotates horizontally around player
- Controlled by mouse movement or input
- Smooth rotation with configurable speed
- Maintains distance and vertical angle

**Vertical Angle:**
- Limited vertical angle adjustment
- Prevents camera from going below ground
- Prevents camera from going too high (maintains gameplay view)
- Smooth angle transitions

## Strafe Support

**Lateral Movement:**
- Camera supports lateral strafing (similar to ArcheAge)
- Player can move sideways (A/D keys) while camera maintains focus
- Smooth camera following during strafe movement
- Maintains zoom level during strafe
- Camera rotates smoothly to follow player's lateral movement
- No camera snapping or jarring movements during strafe

## Target Lock Integration

The camera system integrates seamlessly with the target lock system, similar to ArcheAge's implementation.

**Target Lock Features:**
- **Lock on Target**: Lock onto enemies for combat focus or allies for healing
- **TAB Key**: Cycle through nearby targets (enemies prioritized)
- **Z Key**: Lock onto selected target
- **` Key**: Cancel/unlock current target
- **Click Selection**: Click directly on enemy/ally to lock onto them
- **Camera Follow**: Camera automatically follows locked target
- **Maintain Zoom**: Zoom level is maintained during target lock
- **Smooth Transition**: Smooth camera movement when switching targets

**Camera Behavior with Target Lock:**
- Camera follows locked target smoothly
- Maintains zoom level during lock
- Smooth camera movement to target when locking
- Camera rotates to keep target in view
- Unlocks if target moves out of range
- Camera returns to normal position when unlocking

**Target Lock Use Cases:**
- **Combat Focus**: Lock on enemy to maintain focus during combat
- **Healing**: Lock on ally to target healing abilities
- **PvP**: Essential for large-scale PvP combat to track specific targets

**See [PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md) for complete target lock system details.**

## Camera Controls Reference

### Mouse Controls

**Zoom (Scroll Wheel):**
- **Scroll Wheel**: Used for camera zoom
- **Scroll Back (Zoom Out)**: Scroll wheel backward to zoom out to near top-down view (optimal for PvP)
- **Scroll Forward (Zoom In)**: Scroll wheel forward to zoom in to close third-person view
- **Smooth Transition**: Seamless zoom interpolation between levels

**Rotation (Look):**
- **Mouse XY Movement**: Controls camera look/rotation
- **Mouse X Axis**: Rotate camera horizontally around player
- **Mouse Y Axis**: Rotate camera vertically (with negate modifier applied)
- **Negate Modifier**: Y axis uses negate modifier for inverted vertical look
- **Middle Mouse Button**: Hold for freelook (free camera rotation mode)
- **Right Mouse Button**: Hold to rotate camera freely (optional)

### Keyboard Controls

**Camera Rotation (Alternative):**
- **Q**: Rotate camera left (alternative to mouse)
- **E**: Rotate camera right (alternative to mouse)

**Zoom (Alternative):**
- **Page Up**: Zoom in (alternative to scroll)
- **Page Down**: Zoom out (alternative to scroll)

### Target Lock Controls

**Target Selection:**
- **TAB**: Cycle through nearby targets (enemies prioritized, similar to ArcheAge)
- **Z**: Lock onto selected target
- **` (Backtick/Grave Accent)**: Cancel/unlock current target
- **Left Click**: Click on enemy/ally to lock onto them
- **Mouse Wheel**: Cycle targets while locked (if multiple targets available)

**Target Lock Behavior:**
- Camera automatically follows locked target
- Character auto-rotates towards locked target
- Visual indicator shows lock range
- Target health bar displayed when locked

## Related Documentation

- [PLAYER_CONTROLLER.md](./PLAYER_CONTROLLER.md) - Player controller system (target lock, movement, input, pathfinding interaction)

