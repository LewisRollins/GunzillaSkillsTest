# GunzillaSkillsTest
 
# Combat Game README

## Overview
This project is a **2v2 capture point-based shooter** with three combat abilities: **Projectile, Laser, and Shield**. The movement and combat mechanics are designed for smooth multiplayer gameplay, with all replication handled properly.

---

## **Gameplay Features**
### **Movement System**
- **Custom Movement Component (`BP_CustomMovementComponent`)**
  - Input events run **client-side** on `BP_CustomCharacter`.
  - Calls **server-side events** to update velocity.
  - Uses **delta time** for frame rate independence.
  - **Tick functions** handle:
    - Ground detection (trace-based).
    - Wall detection (trace-based).
    - Gravity application.
    - Grapple physics calculations.
    - Updating the server position.
  - Clients **interpolate to server position** for smooth movement.
  - **Ground friction and air resistance**:
    - Ground friction is applied when moving on surfaces.
    - If no movement input for **0.2s**, ground friction increases to stop movement.
    - Air resistance slows movement when airborne.

### **Movement Abilities**
- **Double Jump:** Allows the player to jump twice before landing.
- **Dash:**
  - Provides a quick burst of movement in the direction of input.
  - **Disables gravity, friction, and resets velocity**.
  - Has a cooldown before reuse.
- **Grapple Hook:**
  - Uses a **cable component** and pull the player toward a grapple point.
  - Grapple strength is **mapped to distance**.
  - Fires a **local projectile** to guide the player's aim.
  - Grapple physics handled in `BP_CustomMovementComponent`.
  - Cable visibility and target actor are **Multicast**.

### **Combat Abilities System**
#### **Structure**
- **Three abilities:** `Primary (Projectile)`, `Secondary (Laser)`, and `Tertiary (Shield)`.
- **Each ability is a separate actor**, handled in `BP_CombatAbilityBase`.
- `BP_CustomCharacter` has an **array of default ability classes**.
- On **BeginPlay**, the character **spawns ability BPs** and assigns them to references (`PrimaryAbility`, `SecondaryAbility`, `TertiaryAbility`).
- Inputs call `UseAbility()` on the assigned ability BP.

#### **Replication Handling**
- **All replication is handled inside the Ability BPs.**
- Inputs **only trigger ability usage**, and all logic runs inside the ability actors.
- **Spawned on both Server & Client** for correct visuals.

#### **Abilities**
1. **Projectile Ability (`BP_ProjectileAbility`)**
   - Fires a projectile actor (`BP_Projectile`), replicated on the **server**.
   - A local projectile is spawned for **instant feedback**.
   - A **multicast event** spawns the projectile for non-local clients.
   - **Only the server deals damage** when the projectile hits.

2. **Laser Ability (`BP_LaserAbility`)**
   - Performs a **line trace** on **server** to detect hits.
   - Local laser trace for instant feedback.
   - Multicast effects (`Niagara FX`, impact sparks, etc.).
   - Reflects off **Shields**, bouncing in the correct direction.
   - **Cable update and visual representation** are handled in the Laser BP, but the actual Cable Component remains on the character.

3. **Shield Ability (`BP_ShieldAbility`)**
   - Deploys a **blocking barrier** that reflects projectiles & lasers.
   - Uses a **collision box** to detect and reflect incoming damage.
   - **Multicast plays FX** for all clients.

---

## **Capture Point System**
- The capture point **swaps ownership gradually** over **4 seconds**.
- `TeamACapturePercentage` and `TeamBCapturePercentage` range from **0 to 1**.
- `SwapPercentage` ranges from **-2 to +2** to track control shifts.
- A single player is **enough to contest the point**.
- If a team has control, the **capture progress continues even when contested**.
- If the point is left empty, **the current owner retains control**.

---

## **Hazards**
- **Lava (`BP_Lava`)**
  - Deals damage over time to players standing in it.
  - Uses a **server-side overlap event** to apply damage.

---

## **Respawn & Death Handling**
- **Death is handled on the server controller.**
- **Respawn logic is in the GameMode (`BP_CapturePointGameMode`).**
- When health reaches **≤ 0**, `DestroyCharacter()` is called on the **Controller**.
- `RequestRespawn()` calls `RestartPlayer()`, spawning the player at a valid team spawn point.
- Spawning logic:
  - Finds the **team with the least players**.
  - Overrides `FindPlayerStart()` to pick a **spawn point tagged for the team**.

---

## **Replication Summary**
| Component           | Responsibility                                   | Replicated? |
|--------------------|----------------------------------------------|-------------|
| **Character (`BP_CustomCharacter`)** | Handles input, movement, ability spawning | ✅ Yes |
| **Movement Component (`BP_CustomMovementComponent`)** | Calculates movement, applies physics | ✅ Yes |
| **Ability System (`BP_CombatAbilityBase`)** | Fires projectiles, traces lasers, deploys shields | ✅ Yes |
| **Projectiles (`BP_Projectile`)** | Simulated on clients, server handles damage | ✅ Yes |
| **Laser Traces** | Server handles hits, multicast for FX | ✅ Yes |
| **Shield Reflection** | Detects and reflects incoming lasers & projectiles | ✅ Yes |
| **Capture Point (`BP_CapturePoint`)** | Tracks capture progress and team ownership | ✅ Yes |
| **GameMode (`BP_CapturePointGameMode`)** | Manages respawns, team balancing | ✅ Yes |
| **Lava (`BP_Lava`)** | Deals damage on server overlap | ✅ Yes |

---

## **To-Do / Future Improvements**
- **Improve Grappling Hook:** Currently left as-is.
- **Better Visual Effects:** Placeholder FX for now, needs polishing.
- **Sound Design:** Add sound cues for each action.
- **Map Layout Adjustments:** Balance testing needed.
- **Add UI Widgets** Currently only in world text renders are used. Add UI elements and use repnotify to update.

---
### **📌 Notes:**
- **All combat abilities use separate BPs** for modularity.
- **All core mechanics are replicated properly** for smooth multiplayer gameplay.

---
## **Final Thoughts**
This project is a **fully functional multiplayer shooter** with a structured and modular system for **abilities, movement, and combat mechanics**. Further polish on FX and sounds will enhance the experience, but the foundation is solid.

