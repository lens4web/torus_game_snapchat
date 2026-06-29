# Technical Documentation: AR Torus Game

## 1. Project Overview
This project is a Face-Tracking AR game built in Snapchat's Lens Studio. The user controls the game by opening their mouth to shoot projectiles at a rotating goal (a Torus). 

The game combines Lens Studio's built-in **Physics Engine** for projectile movement with **coordinate-based math** for scoring. This hybrid approach prevents physics glitches (like fast-moving objects passing through solid walls) and ensures highly accurate gameplay.

---

## 2. Scene Hierarchy & Key Objects
The scene is structured to handle UI, 3D tracking, and physics independently. Here are the core elements in the **Scene Hierarchy**:

* **Orthographic Camera:** Renders the 2D UI elements over the screen.
  * **UI Canvas:** Contains the text components: `Text_Hint` ("Open mouth to start"), `Text_Score`, and `Text_Timer`.
* **Camera:** The main 3D camera rendering the AR environment.
  * **GameController:** An empty object that holds the `Main_script`. It acts as the central brain of the game, connecting all inputs and managing the game loop.
* **3D Upper Body Tracking:** Tracks the user's movements in 3D space.
  * **Upper Body Mesh (Occluder):** This is a critical visual component. It is an invisible mesh that tracks to the user's head and shoulders. It acts as an **occluder**—meaning it masks out objects behind it. When the Torus rotates around the user, this mesh hides the back half of the Torus behind the user's head, creating a realistic 3D depth effect.
* **Floating Rush Funplex:** A 3D floating logo exported directly from the original project without modifications. It serves as static branding/decoration in the AR space.
* **Torus (`objectToRotate`):** The 3D goal object. It rotates continuously around the user's head. It contains a `Physics Collider` so that projectiles can physically bounce off its rim.
* **ScoreZone (`scoreZone`):** An invisible 3D Cylinder placed exactly inside the Torus hole (nested as a child so it moves with the Torus). 
  * *Important Setup:* Its `Mesh Renderer` is disabled. It does NOT use physical trigger colliders. Instead, the script uses its exact 3D coordinates as a mathematical reference point to calculate whether a projectile passed through the hoop.

---

## 3. Assets & The Projectile Prefab
* **Box_prefab (Linked to `cubePrefab`):** This is the projectile asset stored in the **Assets Browser**.
  * **What it is:** A pre-configured 3D object (foam/cube) that is spawned dynamically into the scene during gameplay.
  * **How it works:** It contains a `Mesh Renderer`, a `Material` (visuals), a `Physics Collider` (shape), and a `Physics Body`. It sits quietly in the Asset Browser until the user opens their mouth. When spawned by the `Main_script`, the script forces its `Physics Body` to become dynamic (`isDynamic = true`) and instantly applies a directional velocity vector to it, shooting it toward the Torus.

---

## 4. Adjustable Properties (Inspector Setup)
The `Main_script` has several inputs exposed in the **Inspector** panel. This allows developers to tweak the game balance without opening the code.

**Object Links:**
* `Cube Prefab`: Linked to `Box_prefab` (The projectile).
* `Object To Rotate`: Linked to `Torus` (The goal).
* `Score Zone`: Linked to `ScoreZone` (The invisible scoring reference).
* `Score Text`, `Hint Text`, `Timer Text`: Linked to their respective UI elements on the Canvas.

**Game Balance Variables:**
* `Game Duration` *(default: 30.0)*: Total gameplay time in seconds.
* `Hole Radius` *(default: 3.0)*: The mathematical radius of the goal. If a `Box_prefab` falls within this distance from the center of the `ScoreZone`, it counts as a point.
* `Miss Penalty` *(default: 1.0)*: Points deducted if a projectile misses the hole or bounces away.
* `Rotation Speed` *(default: {0.0, 1.0, 0.0})*: The vector defining how fast the Torus spins on the Y-axis.
* `Spawn Interval` *(default: 0.5)*: The delay in seconds between each projectile spawn while the user's mouth is open.

---

## 5. Core Logic & Mechanics

### A. Event System (Interactions)
* **Face Tracking:** The script listens for `FaceFoundEvent` and `FaceLostEvent`. If the camera loses the user's face, the game pauses spawning projectiles to prevent errors.
* **Mouth Controls:** Controlled via `MouthOpenedEvent` and `MouthClosedEvent`. The very first time the mouth opens, the game hides the Hint Text, shows the Score/Timer, and starts the countdown. Keeping the mouth open generates a continuous flow of `Box_prefab` projectiles.

### B. Spawning & Physics
When the mouth is open, the `spawnCube()` function runs based on the `Spawn Interval`:
1. It instantiates a copy of the `Box_prefab`.
2. It applies a random X and Z offset (`offsetRange = 0.5`) so the projectiles scatter slightly rather than flying in a single, predictable line.
3. It uses a `DelayedCallbackEvent` (set to 0.0s). This is a technical requirement in Lens Studio to ensure the cloned object is fully initialized in the scene before applying physics forces.
4. It sets a physical velocity (`launchVector = new vec3(5.0, 10.0, -10.0)`), launching the box upward and forward.

### C. Scoring System (Anti-Tunneling)
To prevent fast projectiles from glitching through the Torus mesh:
1. The `checkCubesStatus()` function tracks the height (Y-axis) of every spawned `Box_prefab`.
2. When a box drops below the height of the `ScoreZone`, the script calculates its 2D horizontal distance (X and Z axis) from the center of the hole.
3. **Hit (+1 point):** If the distance is less than or equal to `Hole Radius`.
4. **Miss (-1 point):** If the distance is greater than the radius (meaning it bounced off the Torus or missed entirely).

### D. Memory Optimization (Garbage Collection)
To keep the game optimized and prevent it from crashing on older phones, old projectiles must be deleted. Once a `Box_prefab` falls 50 units below the Torus, the script calls `cube.destroy()`, completely removing it from the device's memory.
