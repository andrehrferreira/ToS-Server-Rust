# Entity Behavior System

## Overview

The entity behavior system defines complex AI profiles for creatures and NPCs, creating more challenging and engaging combat encounters. Unlike previous versions with simple behaviors, this system implements strategic, adaptive AI that responds dynamically to combat situations.


## Behavior Profiles

The system supports five main behavior profiles, each with unique combat strategies and decision-making patterns:

1. **Strategic** - Intelligent humanoid creatures with tactical combat
2. **Aggressive** - Predator-like creatures focused on maximizing damage
3. **Stealthy** - Sneaky creatures that use stealth and critical strikes
4. **Powerful** - Boss-level creatures with devastating abilities
5. **Fearful** - Weak creatures that flee from combat

---

## 1. Strategic Behavior

### Overview

Strategic creatures are intelligent humanoids that use tactical combat, positioning, and timing to maximize their effectiveness. They adapt to combat situations and make calculated decisions.

### Characteristics

**Combat Style:**
- Tactical positioning and movement
- Strategic use of abilities at optimal times
- Defensive postures when needed
- Counter-attacks when opportunities arise
- Retreats when health is low
- Uses crowd control and debuffs strategically

**Decision Making:**
- Evaluates threat level of enemies
- Prioritizes targets based on threat and vulnerability
- Calculates optimal ability usage timing
- Adapts strategy based on combat state

**Weaknesses (Intentional):**
- Can be baited into using abilities prematurely
- May retreat too early if health drops quickly
- Can be overwhelmed by multiple attackers
- Predictable patterns that can be learned and exploited

### Behavior States

```rust
pub enum StrategicState {
    Assessing,        // Evaluating threat and positioning
    Engaging,         // Actively attacking
    Defending,        // Using defensive abilities, blocking
    Retreating,       // Moving away to recover
    Countering,       // Waiting for counter-attack opportunity
    Casting,          // Using powerful ability
}

pub struct StrategicBehavior {
    pub state: StrategicState,
    pub threat_assessment: ThreatLevel,
    pub optimal_range: f32,
    pub ability_cooldowns: HashMap<AbilityId, DateTime<Utc>>,
    pub last_position: Vec3,
    pub retreat_threshold: f32,  // Health % to retreat
    pub counter_attack_window: Duration,
}
```

### Implementation

```rust
impl StrategicBehavior {
    pub async fn update(&mut self, entity: &mut Entity, world: &World) -> Result<()> {
        // 1. Assess current situation
        let threat = self.assess_threat(entity, world).await?;
        let health_percent = entity.health.current as f32 / entity.health.max as f32;
        
        // 2. Update state based on situation
        match self.state {
            StrategicState::Assessing => {
                if threat.is_high() {
                    self.state = StrategicState::Engaging;
                } else if health_percent < self.retreat_threshold {
                    self.state = StrategicState::Retreating;
                }
            }
            
            StrategicState::Engaging => {
                // Check if should use defensive ability
                if health_percent < 0.5 && self.can_use_defensive_ability() {
                    self.state = StrategicState::Defending;
                    return self.use_defensive_ability(entity).await;
                }
                
                // Check for counter-attack opportunity
                if self.has_counter_opportunity(entity, world).await? {
                    self.state = StrategicState::Countering;
                    return self.execute_counter_attack(entity, world).await;
                }
                
                // Use offensive ability if optimal
                if self.should_use_ability(entity, world).await? {
                    self.state = StrategicState::Casting;
                    return self.use_optimal_ability(entity, world).await;
                }
                
                // Continue attacking
                self.perform_attack(entity, world).await?;
            }
            
            StrategicState::Defending => {
                // Use defensive abilities, block, or dodge
                self.use_defensive_posture(entity).await?;
                
                // Return to engaging after defensive period
                if self.defensive_cooldown_expired() {
                    self.state = StrategicState::Engaging;
                }
            }
            
            StrategicState::Retreating => {
                // Move away from threat
                self.retreat_from_threat(entity, world).await?;
                
                // Use healing abilities if available
                if self.can_heal() {
                    self.use_healing_ability(entity).await?;
                }
                
                // Return to engaging if health recovered
                if health_percent > self.retreat_threshold + 0.2 {
                    self.state = StrategicState::Engaging;
                }
            }
            
            StrategicState::Countering => {
                // Wait for enemy attack, then counter
                if self.enemy_attacking(entity, world).await? {
                    self.execute_counter_attack(entity, world).await?;
                    self.state = StrategicState::Engaging;
                } else {
                    // Counter window expired
                    self.state = StrategicState::Engaging;
                }
            }
            
            StrategicState::Casting => {
                // Continue casting, return to engaging when done
                if self.cast_complete() {
                    self.state = StrategicState::Engaging;
                }
            }
        }
        
        Ok(())
    }
    
    async fn assess_threat(&self, entity: &Entity, world: &World) -> Result<ThreatLevel> {
        let nearby_enemies = world.get_nearby_enemies(entity.position, 10.0).await?;
        
        let mut threat_score = 0.0;
        for enemy in nearby_enemies {
            let distance = (entity.position - enemy.position).length();
            let damage_potential = enemy.calculate_damage_potential();
            let threat_multiplier = 1.0 / (distance + 1.0); // Closer = more threat
            
            threat_score += damage_potential * threat_multiplier;
        }
        
        if threat_score > 100.0 {
            Ok(ThreatLevel::High)
        } else if threat_score > 50.0 {
            Ok(ThreatLevel::Medium)
        } else {
            Ok(ThreatLevel::Low)
        }
    }
    
    async fn should_use_ability(&self, entity: &Entity, world: &World) -> Result<bool> {
        // Check if optimal conditions for ability usage
        let enemies = world.get_nearby_enemies(entity.position, 8.0).await?;
        
        // Use AoE ability if multiple enemies grouped
        if enemies.len() >= 3 && self.can_use_aoe_ability() {
            return Ok(true);
        }
        
        // Use debuff if enemy is strong
        if let Some(enemy) = enemies.first() {
            if enemy.is_strong() && self.can_use_debuff() {
                return Ok(true);
            }
        }
        
        // Use burst ability if enemy health is low
        if let Some(enemy) = enemies.first() {
            let enemy_health_percent = enemy.health.current as f32 / enemy.health.max as f32;
            if enemy_health_percent < 0.3 && self.can_use_finisher() {
                return Ok(true);
            }
        }
        
        Ok(false)
    }
    
    async fn execute_counter_attack(&self, entity: &mut Entity, world: &World) -> Result<()> {
        // Wait for enemy attack animation
        // Then execute counter-attack with bonus damage
        let counter_damage_multiplier = 1.5;
        
        if let Some(enemy) = world.get_nearest_enemy(entity.position).await? {
            let base_damage = entity.calculate_attack_damage();
            let counter_damage = (base_damage as f32 * counter_damage_multiplier) as u32;
            
            entity.attack(enemy.id, counter_damage).await?;
        }
        
        Ok(())
    }
}
```

### Ability Usage Patterns

**Defensive Abilities:**
- Used when health drops below 50%
- Used when multiple enemies are nearby
- Used when enemy is using powerful ability

**Offensive Abilities:**
- AoE abilities when 3+ enemies grouped
- Debuff abilities on strong enemies
- Burst abilities when enemy health is low
- Crowd control when outnumbered

**Counter-Attacks:**
- Triggered when enemy attacks during counter window
- 1.5x damage multiplier
- Short window (0.5-1.0 seconds)

---

## 2. Aggressive Behavior

### Overview

Aggressive creatures are predator-like entities focused on maximizing damage output. They use movement patterns similar to predators, relentlessly pursuing targets while using abilities to enhance their attacks.

### Characteristics

**Combat Style:**
- Constant forward pressure
- Predator-like movement (circling, flanking)
- Ability usage focused on maximizing damage
- Relentless pursuit
- Retreats only when critically wounded
- Fights to the death (doesn't flee completely)

**Decision Making:**
- Prioritizes closest or weakest target
- Uses damage-boosting abilities frequently
- Maintains aggressive positioning
- Adapts movement to maximize attack opportunities

**Weaknesses:**
- Can be kited (moved away from)
- Predictable attack patterns
- Vulnerable when using abilities
- May overcommit to kills

### Behavior States

```rust
pub enum AggressiveState {
    Hunting,          // Searching for target
    Pursuing,         // Chasing target
    Attacking,        // In melee range, attacking
    UsingAbility,     // Casting damage ability
    Wounded,          // Low health, temporary retreat
}

pub struct AggressiveBehavior {
    pub state: AggressiveState,
    pub target_id: Option<EntityId>,
    pub last_attack_time: DateTime<Utc>,
    pub ability_usage_cooldown: Duration,
    pub wounded_threshold: f32,  // Health % to temporarily retreat
    pub pursuit_range: f32,
    pub attack_range: f32,
}
```

### Implementation

```rust
impl AggressiveBehavior {
    pub async fn update(&mut self, entity: &mut Entity, world: &World) -> Result<()> {
        let health_percent = entity.health.current as f32 / entity.health.max as f32;
        
        match self.state {
            AggressiveState::Hunting => {
                // Find nearest target
                if let Some(target) = world.get_nearest_enemy(entity.position).await? {
                    self.target_id = Some(target.id);
                    self.state = AggressiveState::Pursuing;
                }
            }
            
            AggressiveState::Pursuing => {
                if let Some(target_id) = self.target_id {
                    let target = world.get_entity(target_id).await?;
                    let distance = (entity.position - target.position).length();
                    
                    if distance <= self.attack_range {
                        // In attack range
                        self.state = AggressiveState::Attacking;
                    } else {
                        // Use predator movement to close distance
                        self.predator_movement(entity, &target, world).await?;
                    }
                } else {
                    self.state = AggressiveState::Hunting;
                }
            }
            
            AggressiveState::Attacking => {
                if let Some(target_id) = self.target_id {
                    let target = world.get_entity(target_id).await?;
                    let distance = (entity.position - target.position).length();
                    
                    // Check if should use damage ability
                    if self.should_use_damage_ability() && distance <= self.attack_range {
                        self.state = AggressiveState::UsingAbility;
                        return self.use_damage_ability(entity, target_id, world).await;
                    }
                    
                    // Perform attack
                    self.perform_attack(entity, target_id, world).await?;
                    
                    // Check if critically wounded
                    if health_percent < self.wounded_threshold {
                        self.state = AggressiveState::Wounded;
                    }
                    
                    // If target moved away, pursue
                    if distance > self.attack_range {
                        self.state = AggressiveState::Pursuing;
                    }
                }
            }
            
            AggressiveState::UsingAbility => {
                // Continue casting
                if self.ability_complete() {
                    self.state = AggressiveState::Attacking;
                }
            }
            
            AggressiveState::Wounded => {
                // Temporary retreat (not full flight)
                if let Some(target_id) = self.target_id {
                    let target = world.get_entity(target_id).await?;
                    
                    // Move away slightly
                    self.tactical_retreat(entity, &target, world).await?;
                    
                    // Use defensive ability if available
                    if self.can_use_defensive_ability() {
                        self.use_defensive_ability(entity).await?;
                    }
                    
                    // Return to attacking if health recovered or enough time passed
                    if health_percent > self.wounded_threshold + 0.1 {
                        self.state = AggressiveState::Attacking;
                    }
                }
            }
        }
        
        Ok(())
    }
    
    async fn predator_movement(
        &self,
        entity: &mut Entity,
        target: &Entity,
        world: &World,
    ) -> Result<()> {
        // Predator movement: circle around target, approach from side/behind
        let to_target = target.position - entity.position;
        let distance = to_target.length();
        
        // Calculate optimal approach angle (flanking)
        let approach_angle = self.calculate_flank_angle(entity, target, world).await?;
        
        // Move in arc towards target
        let movement_direction = self.calculate_predator_path(entity, target, approach_angle);
        
        entity.move_towards(movement_direction, entity.stats.move_speed).await?;
        
        Ok(())
    }
    
    async fn calculate_flank_angle(
        &self,
        entity: &Entity,
        target: &Entity,
        world: &World,
    ) -> Result<f32> {
        // Find angle that approaches from side/behind
        // Avoids frontal approach when possible
        
        let to_target = target.position - entity.position;
        let target_forward = target.rotation.forward();
        
        // Calculate angle between our position and target's forward
        let angle_to_target = to_target.normalize().angle_between(target_forward);
        
        // Prefer approaching from side (90 degrees) or behind (180 degrees)
        let optimal_angle = if angle_to_target < 0.5 {
            // Currently in front, move to side
            1.57  // 90 degrees
        } else {
            // Can approach from side or behind
            angle_to_target
        };
        
        Ok(optimal_angle)
    }
    
    async fn use_damage_ability(
        &self,
        entity: &mut Entity,
        target_id: EntityId,
        world: &World,
    ) -> Result<()> {
        // Use ability that maximizes damage
        // Examples: damage boost, multi-hit, critical strike chance
        
        let ability = self.select_damage_ability(entity).await?;
        entity.cast_ability(ability, target_id).await?;
        
        Ok(())
    }
    
    async fn tactical_retreat(
        &self,
        entity: &mut Entity,
        target: &Entity,
        world: &World,
    ) -> Result<()> {
        // Move away but stay in combat range
        // Not full flight, just repositioning
        
        let away_direction = (entity.position - target.position).normalize();
        let retreat_distance = 5.0; // Short retreat
        
        let retreat_position = entity.position + away_direction * retreat_distance;
        entity.move_to(retreat_position).await?;
        
        Ok(())
    }
}
```

### Ability Usage Patterns

**Damage-Boosting Abilities:**
- Used frequently to maximize damage output
- Prioritized over defensive abilities
- Used when target is in optimal range

**Movement Abilities:**
- Used to close distance quickly
- Used to maintain pursuit
- Used for flanking maneuvers

**Defensive Abilities (Limited):**
- Only used when critically wounded
- Used to survive long enough to kill target

---

## 3. Stealthy Behavior

### Overview

Stealthy creatures use stealth, positioning, and critical strikes to eliminate targets. They prefer hit-and-run tactics, attacking from behind for maximum damage, and using evasion to avoid taking damage.

### Characteristics

**Combat Style:**
- Stealth and invisibility abilities
- Backstab attacks for critical damage
- Hit-and-run tactics
- High evasion and dodge chance
- Less aggressive, waits for opportunities
- Focuses on precise, high-damage strikes

**Decision Making:**
- Evaluates positioning opportunities
- Waits for optimal attack windows
- Prioritizes targets that can't see them
- Uses stealth to reposition

**Weaknesses:**
- Vulnerable when revealed
- Lower health pool
- Less effective in prolonged combat
- Can be detected by certain abilities

### Behavior States

```rust
pub enum StealthyState {
    Hidden,           // In stealth, positioning
    Stalking,         // Following target, waiting for opportunity
    Attacking,        // Executing attack
    Evading,          // Dodging attacks
    ReStealthing,     // Returning to stealth
}

pub struct StealthyBehavior {
    pub state: StealthyState,
    pub is_stealthed: bool,
    pub target_id: Option<EntityId>,
    pub last_backstab_time: DateTime<Utc>,
    pub backstab_cooldown: Duration,
    pub stealth_duration: Duration,
    pub reveal_threshold: f32,  // Damage % to reveal
    pub evasion_chance: f32,
}
```

### Implementation

```rust
impl StealthyBehavior {
    pub async fn update(&mut self, entity: &mut Entity, world: &World) -> Result<()> {
        match self.state {
            StealthyState::Hidden => {
                // Find target and position for backstab
                if let Some(target) = world.get_nearest_enemy(entity.position).await? {
                    self.target_id = Some(target.id);
                    self.state = StealthyState::Stalking;
                } else {
                    // Patrol or wait
                    self.patrol(entity, world).await?;
                }
            }
            
            StealthyState::Stalking => {
                if let Some(target_id) = self.target_id {
                    let target = world.get_entity(target_id).await?;
                    
                    // Position behind target
                    let behind_position = self.calculate_behind_position(entity, &target);
                    let distance = (entity.position - behind_position).length();
                    
                    if distance < 2.0 {
                        // In position for backstab
                        self.state = StealthyState::Attacking;
                        return self.execute_backstab(entity, target_id, world).await;
                    } else {
                        // Move to position (staying stealthed)
                        self.move_to_position(entity, behind_position, world).await?;
                    }
                }
            }
            
            StealthyState::Attacking => {
                // Execute backstab or critical strike
                if let Some(target_id) = self.target_id {
                    // Check if can backstab (behind target)
                    if self.is_behind_target(entity, target_id, world).await? {
                        self.execute_backstab(entity, target_id, world).await?;
                    } else {
                        // Regular critical strike
                        self.execute_critical_strike(entity, target_id, world).await?;
                    }
                    
                    // After attack, check if should re-stealth
                    if self.should_restealth(entity, world).await? {
                        self.state = StealthyState::ReStealthing;
                    } else {
                        self.state = StealthyState::Evading;
                    }
                }
            }
            
            StealthyState::Evading => {
                // Dodge attacks, use evasion abilities
                if let Some(target_id) = self.target_id {
                    // Check if being attacked
                    if self.is_under_attack(entity, world).await? {
                        self.dodge_attack(entity, world).await?;
                    }
                    
                    // Try to re-stealth
                    if self.can_restealth() {
                        self.state = StealthyState::ReStealthing;
                    }
                }
            }
            
            StealthyState::ReStealthing => {
                // Use stealth ability
                if self.use_stealth_ability(entity).await? {
                    self.is_stealthed = true;
                    self.state = StealthyState::Hidden;
                }
            }
        }
        
        Ok(())
    }
    
    async fn execute_backstab(
        &self,
        entity: &mut Entity,
        target_id: EntityId,
        world: &World,
    ) -> Result<()> {
        // Backstab: 3x damage, guaranteed critical
        let base_damage = entity.calculate_attack_damage();
        let backstab_damage = base_damage * 3;
        
        // Check if behind target
        if self.is_behind_target(entity, target_id, world).await? {
            entity.attack(target_id, backstab_damage).await?;
            
            // Apply debuff (stunned or slowed)
            world.apply_debuff(target_id, DebuffType::Stunned, Duration::seconds(2)).await?;
        } else {
            // Regular critical strike (2x damage)
            let crit_damage = base_damage * 2;
            entity.attack(target_id, crit_damage).await?;
        }
        
        Ok(())
    }
    
    async fn is_behind_target(
        &self,
        entity: &Entity,
        target_id: EntityId,
        world: &World,
    ) -> Result<bool> {
        let target = world.get_entity(target_id).await?;
        
        let to_target = target.position - entity.position;
        let target_forward = target.rotation.forward();
        
        // Check if we're behind target (angle > 90 degrees)
        let angle = to_target.normalize().angle_between(-target_forward);
        Ok(angle > 1.57) // > 90 degrees
    }
    
    async fn calculate_behind_position(
        &self,
        entity: &Entity,
        target: &Entity,
    ) -> Vec3 {
        let target_back = -target.rotation.forward();
        let behind_distance = 1.5; // Close enough for backstab
        
        target.position + target_back * behind_distance
    }
    
    async fn dodge_attack(&self, entity: &mut Entity, world: &World) -> Result<()> {
        // Use evasion ability or dodge roll
        if self.can_use_evasion() {
            // Roll dodge
            let dodge_direction = self.calculate_dodge_direction(entity, world).await?;
            entity.dodge_roll(dodge_direction).await?;
        } else {
            // Use evasion ability (increased dodge chance)
            entity.use_ability(AbilityId::Evasion).await?;
        }
        
        Ok(())
    }
    
    async fn should_restealth(&self, entity: &Entity, world: &World) -> Result<bool> {
        // Re-stealth if:
        // - Health is low
        // - Multiple enemies nearby
        // - Stealth cooldown expired
        // - Not currently being targeted
        
        let health_percent = entity.health.current as f32 / entity.health.max as f32;
        let nearby_enemies = world.get_nearby_enemies(entity.position, 5.0).await?;
        
        if health_percent < 0.5 {
            return Ok(true);
        }
        
        if nearby_enemies.len() > 2 {
            return Ok(true);
        }
        
        if !world.is_entity_targeted(entity.id).await? {
            return Ok(true);
        }
        
        Ok(false)
    }
}
```

### Ability Usage Patterns

**Stealth Abilities:**
- Used to initiate combat
- Used to escape when low on health
- Used to reposition during combat

**Critical Strike Abilities:**
- Used when behind target (backstab)
- Used when target is vulnerable
- Used to finish off low-health enemies

**Evasion Abilities:**
- Used when under attack
- Used to avoid area-of-effect abilities
- Used to create distance for re-stealthing

---

## 4. Powerful Behavior

### Overview

Powerful creatures are typically bosses with devastating abilities and mechanics. They focus on weakening enemies through debuffs and area denial, then finishing them with massive attacks. They have high health and are less concerned with taking damage.

### Characteristics

**Combat Style:**
- Massive area-of-effect attacks
- Debuff and weakening abilities
- Damage reduction and mitigation
- Phase-based combat (different abilities per phase)
- Environmental hazards and mechanics
- High health pool, tank-like

**Decision Making:**
- Prioritizes area control
- Uses debuffs to weaken groups
- Saves powerful abilities for optimal moments
- Adapts based on combat phase

**Weaknesses:**
- Slower movement
- Predictable ability patterns
- Vulnerable during ability casting
- Can be interrupted

### Behavior States

```rust
pub enum PowerfulState {
    Phase1,           // Initial phase
    Phase2,           // Mid-health phase
    Phase3,           // Low-health phase (enraged)
    Casting,          // Using powerful ability
    Weakening,        // Applying debuffs
    Enraged,          // Low health, increased aggression
}

pub struct PowerfulBehavior {
    pub state: PowerfulState,
    pub current_phase: u32,
    pub phase_thresholds: Vec<f32>,  // Health % for phase transitions
    pub ability_queue: Vec<AbilityId>,
    pub last_massive_attack: DateTime<Utc>,
    pub massive_attack_cooldown: Duration,
    pub enrage_threshold: f32,
}
```

### Implementation

```rust
impl PowerfulBehavior {
    pub async fn update(&mut self, entity: &mut Entity, world: &World) -> Result<()> {
        let health_percent = entity.health.current as f32 / entity.health.max as f32;
        
        // Check phase transitions
        self.update_phase(health_percent);
        
        match self.state {
            PowerfulState::Phase1 | PowerfulState::Phase2 | PowerfulState::Phase3 => {
                // Check if should use massive attack
                if self.should_use_massive_attack(entity, world).await? {
                    self.state = PowerfulState::Casting;
                    return self.use_massive_attack(entity, world).await;
                }
                
                // Check if should weaken enemies
                if self.should_weaken_enemies(entity, world).await? {
                    self.state = PowerfulState::Weakening;
                    return self.apply_weakening_abilities(entity, world).await;
                }
                
                // Use phase-appropriate abilities
                self.use_phase_abilities(entity, world).await?;
            }
            
            PowerfulState::Casting => {
                // Continue casting massive ability
                if self.cast_complete() {
                    self.state = self.get_current_phase_state();
                }
            }
            
            PowerfulState::Weakening => {
                // Apply debuffs and area denial
                self.apply_debuffs(entity, world).await?;
                self.create_area_hazards(entity, world).await?;
                
                self.state = self.get_current_phase_state();
            }
            
            PowerfulState::Enraged => {
                // Increased aggression, more frequent abilities
                if health_percent < self.enrage_threshold {
                    self.enraged_combat(entity, world).await?;
                } else {
                    self.state = self.get_current_phase_state();
                }
            }
        }
        
        Ok(())
    }
    
    fn update_phase(&mut self, health_percent: f32) {
        for (phase_index, threshold) in self.phase_thresholds.iter().enumerate() {
            if health_percent <= *threshold {
                self.current_phase = phase_index as u32 + 1;
                
                match self.current_phase {
                    1 => self.state = PowerfulState::Phase1,
                    2 => self.state = PowerfulState::Phase2,
                    3 => {
                        self.state = PowerfulState::Phase3;
                        if health_percent < self.enrage_threshold {
                            self.state = PowerfulState::Enraged;
                        }
                    }
                    _ => {}
                }
                break;
            }
        }
    }
    
    async fn should_use_massive_attack(
        &self,
        entity: &Entity,
        world: &World,
    ) -> Result<bool> {
        // Use massive attack if:
        // - Cooldown expired
        // - Multiple enemies grouped
        // - Optimal positioning
        
        if !self.can_use_massive_attack() {
            return Ok(false);
        }
        
        let nearby_enemies = world.get_nearby_enemies(entity.position, 10.0).await?;
        if nearby_enemies.len() >= 3 {
            return Ok(true);
        }
        
        Ok(false)
    }
    
    async fn use_massive_attack(
        &self,
        entity: &mut Entity,
        world: &World,
    ) -> Result<()> {
        // Cast devastating area attack
        // Examples: meteor, earthquake, explosion
        
        let ability = self.select_massive_ability().await?;
        let target_position = self.calculate_optimal_aoe_position(entity, world).await?;
        
        entity.cast_ability_at_position(ability, target_position).await?;
        
        Ok(())
    }
    
    async fn apply_weakening_abilities(
        &self,
        entity: &mut Entity,
        world: &World,
    ) -> Result<()> {
        // Apply debuffs that reduce enemy effectiveness
        // Examples: damage reduction, slow, vulnerability
        
        let nearby_enemies = world.get_nearby_enemies(entity.position, 15.0).await?;
        
        for enemy in nearby_enemies {
            // Apply damage reduction debuff
            world.apply_debuff(
                enemy.id,
                DebuffType::DamageReduction { percent: 0.3 },
                Duration::seconds(10),
            ).await?;
            
            // Apply slow debuff
            world.apply_debuff(
                enemy.id,
                DebuffType::Slow { percent: 0.5 },
                Duration::seconds(8),
            ).await?;
        }
        
        Ok(())
    }
    
    async fn create_area_hazards(
        &self,
        entity: &mut Entity,
        world: &World,
    ) -> Result<()> {
        // Create environmental hazards
        // Examples: fire zones, poison clouds, spikes
        
        let hazard_positions = self.calculate_hazard_positions(entity, world).await?;
        
        for position in hazard_positions {
            world.create_hazard(
                HazardType::Fire,
                position,
                Duration::seconds(15),
            ).await?;
        }
        
        Ok(())
    }
    
    async fn enraged_combat(
        &self,
        entity: &mut Entity,
        world: &World,
    ) -> Result<()> {
        // Enraged: faster ability usage, more damage, less defensive
        
        // Use abilities more frequently
        if self.can_use_ability() {
            self.use_phase_abilities(entity, world).await?;
        }
        
        // Increased attack speed
        entity.stats.attack_speed_multiplier = 1.5;
        
        Ok(())
    }
    
    fn get_current_phase_state(&self) -> PowerfulState {
        match self.current_phase {
            1 => PowerfulState::Phase1,
            2 => PowerfulState::Phase2,
            3 => PowerfulState::Phase3,
            _ => PowerfulState::Phase1,
        }
    }
}
```

### Phase-Based Abilities

**Phase 1 (100-66% Health):**
- Basic attacks
- Single-target debuffs
- Area denial abilities

**Phase 2 (66-33% Health):**
- Increased ability frequency
- Multi-target debuffs
- Environmental hazards
- Defensive abilities

**Phase 3 (33-0% Health):**
- Massive area attacks
- Enrage mechanics
- Increased damage
- Desperation abilities

---

## 5. Fearful Behavior

### Overview

Fearful creatures are typically weak animals or creatures that flee from combat. They may attack when cornered or when enemies get too close, but their primary instinct is to escape.

### Characteristics

**Combat Style:**
- Primary behavior: flee
- Attacks only when cornered or very close
- Uses escape abilities
- Low health and damage
- Group behavior (flee together)

**Decision Making:**
- Evaluates threat distance
- Chooses escape route
- Attacks only as last resort
- May group with other fearful creatures

**Weaknesses:**
- Very low health
- Low damage output
- Predictable behavior
- Easy to corner and kill

### Behavior States

```rust
pub enum FearfulState {
    Idle,             // Not threatened
    Alerted,          // Detected threat, preparing to flee
    Fleeing,          // Running away
    Cornered,         // Cannot flee, may attack
    Attacking,        // Forced to attack (cornered or very close)
}

pub struct FearfulBehavior {
    pub state: FearfulState,
    pub threat_distance: f32,
    pub flee_distance: f32,
    pub attack_distance: f32,  // Very close, will attack
    pub cornered_threshold: f32,  // Health % when cornered
    pub group_id: Option<GroupId>,  // May flee in groups
}
```

### Implementation

```rust
impl FearfulBehavior {
    pub async fn update(&mut self, entity: &mut Entity, world: &World) -> Result<()> {
        let nearest_enemy = world.get_nearest_enemy(entity.position).await?;
        
        match self.state {
            FearfulState::Idle => {
                if let Some(enemy) = nearest_enemy {
                    let distance = (entity.position - enemy.position).length();
                    
                    if distance < self.threat_distance {
                        self.state = FearfulState::Alerted;
                    }
                } else {
                    // Patrol or idle behavior
                    self.idle_behavior(entity, world).await?;
                }
            }
            
            FearfulState::Alerted => {
                if let Some(enemy) = nearest_enemy {
                    let distance = (entity.position - enemy.position).length();
                    
                    if distance < self.attack_distance {
                        // Too close, may attack
                        self.state = FearfulState::Attacking;
                    } else if distance < self.flee_distance {
                        // Start fleeing
                        self.state = FearfulState::Fleeing;
                    }
                } else {
                    self.state = FearfulState::Idle;
                }
            }
            
            FearfulState::Fleeing => {
                if let Some(enemy) = nearest_enemy {
                    let distance = (entity.position - enemy.position).length();
                    
                    if distance < self.attack_distance {
                        // Cornered, forced to attack
                        self.state = FearfulState::Cornered;
                        return self.cornered_attack(entity, enemy.id, world).await;
                    }
                    
                    // Calculate escape route
                    let escape_direction = self.calculate_escape_route(entity, &enemy, world).await?;
                    
                    // Use escape ability if available
                    if self.can_use_escape_ability() {
                        self.use_escape_ability(entity, escape_direction, world).await?;
                    } else {
                        // Run away
                        entity.move_in_direction(escape_direction, entity.stats.move_speed * 1.5).await?;
                    }
                    
                    // Check if safe distance reached
                    if distance > self.flee_distance * 2.0 {
                        self.state = FearfulState::Idle;
                    }
                } else {
                    self.state = FearfulState::Idle;
                }
            }
            
            FearfulState::Cornered => {
                // Cannot escape, must fight
                if let Some(enemy) = nearest_enemy {
                    self.cornered_attack(entity, enemy.id, world).await?;
                    
                    // Try to escape after attack
                    if self.can_escape(entity, world).await? {
                        self.state = FearfulState::Fleeing;
                    }
                }
            }
            
            FearfulState::Attacking => {
                // Very close enemy, attack then flee
                if let Some(enemy) = nearest_enemy {
                    let distance = (entity.position - enemy.position).length();
                    
                    if distance < self.attack_distance {
                        // Quick attack then flee
                        self.quick_attack(entity, enemy.id, world).await?;
                        self.state = FearfulState::Fleeing;
                    } else {
                        self.state = FearfulState::Fleeing;
                    }
                }
            }
        }
        
        Ok(())
    }
    
    async fn calculate_escape_route(
        &self,
        entity: &Entity,
        threat: &Entity,
        world: &World,
    ) -> Result<Vec3> {
        // Calculate direction away from threat
        let away_direction = (entity.position - threat.position).normalize();
        
        // Check for obstacles and find clear path
        let escape_position = self.find_clear_path(entity.position, away_direction, world).await?;
        
        Ok((escape_position - entity.position).normalize())
    }
    
    async fn cornered_attack(
        &self,
        entity: &mut Entity,
        enemy_id: EntityId,
        world: &World,
    ) -> Result<()> {
        // Desperate attack when cornered
        // May have increased damage due to desperation
        
        let base_damage = entity.calculate_attack_damage();
        let desperation_multiplier = 1.3; // Slight damage boost when cornered
        let damage = (base_damage as f32 * desperation_multiplier) as u32;
        
        entity.attack(enemy_id, damage).await?;
        
        // Try to create opening to escape
        if self.can_create_opening(entity, world).await? {
            self.use_escape_ability(entity, Vec3::zero(), world).await?;
        }
        
        Ok(())
    }
    
    async fn quick_attack(
        &self,
        entity: &mut Entity,
        enemy_id: EntityId,
        world: &World,
    ) -> Result<()> {
        // Fast attack, then immediately flee
        entity.attack(enemy_id, entity.calculate_attack_damage()).await?;
        
        // Small knockback or stun to create escape window
        world.apply_debuff(enemy_id, DebuffType::Stunned, Duration::milliseconds(500)).await?;
        
        Ok(())
    }
    
    async fn can_escape(&self, entity: &Entity, world: &World) -> Result<bool> {
        // Check if there's a clear path to escape
        let nearby_enemies = world.get_nearby_enemies(entity.position, 5.0).await?;
        
        // Can escape if no enemies blocking path
        Ok(nearby_enemies.len() < 2)
    }
}
```

### Group Behavior

Fearful creatures may flee in groups:

```rust
impl FearfulBehavior {
    async fn group_flee_behavior(&mut self, entity: &mut Entity, world: &World) -> Result<()> {
        if let Some(group_id) = self.group_id {
            let group_members = world.get_group_members(group_id).await?;
            
            // Follow group leader or move with group
            if let Some(leader) = group_members.first() {
                if leader.id != entity.id {
                    // Follow leader's escape route
                    let leader_direction = leader.position - entity.position;
                    entity.move_in_direction(leader_direction.normalize(), entity.stats.move_speed).await?;
                }
            }
        }
        
        Ok(())
    }
}
```

---

## Behavior System Integration

### Behavior Selection

Entities are assigned behaviors based on their type and role:

```rust
pub fn assign_behavior(entity_type: EntityType, role: EntityRole) -> Box<dyn Behavior> {
    match (entity_type, role) {
        (EntityType::Humanoid, EntityRole::Warrior) => {
            Box::new(StrategicBehavior::new())
        }
        (EntityType::Beast, EntityRole::Predator) => {
            Box::new(AggressiveBehavior::new())
        }
        (EntityType::Humanoid, EntityRole::Rogue) => {
            Box::new(StealthyBehavior::new())
        }
        (EntityType::Boss, _) => {
            Box::new(PowerfulBehavior::new())
        }
        (EntityType::Animal, EntityRole::Prey) => {
            Box::new(FearfulBehavior::new())
        }
        _ => {
            Box::new(BasicBehavior::new()) // Fallback
        }
    }
}
```

### Behavior Updates

Behaviors are updated each game tick:

```rust
pub struct BehaviorSystem {
    behaviors: HashMap<EntityId, Box<dyn Behavior>>,
}

impl BehaviorSystem {
    pub async fn update(&mut self, world: &mut World, delta_time: Duration) -> Result<()> {
        for (entity_id, behavior) in &mut self.behaviors {
            if let Some(mut entity) = world.get_entity_mut(*entity_id).await? {
                behavior.update(&mut entity, world).await?;
            }
        }
        
        Ok(())
    }
}
```

---

## See Also

- [SPELLS.md](./SPELLS.md) - Spells and actions system
- [ENTITIES.md](./ENTITIES.md) - Entity system overview
- [COMBAT.md](../combat/COMBAT.md) - Combat system
- [AI.md](./AI.md) - AI system architecture

