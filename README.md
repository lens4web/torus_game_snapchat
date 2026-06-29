# Technical Documentation: AR Torus Game

## 1. Project Overview
This project is a Face-Tracking AR game built in Snapchat's Lens Studio. The user controls the game by opening their mouth to shoot projectiles (cubes/foam) at a rotating goal (a Torus). 

The game uses Lens Studio's built-in **Physics Engine** for projectile movement, but uses **coordinate-based math** for scoring to prevent physics bugs (like objects passing through solid walls at high speeds).

---

## 2. Scene Hierarchy & Key Objects
To maintain this project, it is important to understand the main objects in the **Scene Hierarchy**:

* **Camera / Orthographic Camera:** Renders the 3D environment and the 2D UI Canvas.
* **GameController (Empty Object):** This object holds the main JavaScript and connects all the inputs.
* **Torus (`objectToRotate`):** The 3D goal object. It rotates continuously. 
  * *Required Components:* Needs a `Physics Collider` (set to Static or Kinematic) so projectiles can bounce off its edges.
* **ScoreZone (`scoreZone`):** An invisible 3D Cylinder placed exactly inside the Torus hole. It is a child of the Torus (it moves with it).
  * *Important Setup:* Its `Mesh Renderer` must be **disabled** (unchecked) so it remains invisible. It does NOT use physical colliders for scoring. The script uses its World Position to calculate hits.
* **UI Canvas:**
  * `hintText`: Displays instructions ("Open mouth to start").
  * `scoreText`: Displays the current score.
  * `timerText`: Displays the remaining time.

---

## 3. Assets & Materials
* **CubePrefab (`cubePrefab`):** The projectile object stored in the **Assets Browser**.
  * *Required Components:* * `Mesh Renderer` and `Material` (defines how the projectile looks).
    * `Physics Collider` (Box or Sphere shape).
    * `Physics Body` (The script automatically sets this to dynamic when spawned).

---

## 4. Adjustable Properties (Inspector Setup)
The main script has several `@input` variables. You can change these in the **Inspector** panel without editing the code. This makes balancing the game easy.

**Object Links:**
* `cubePrefab` *(Asset.ObjectPrefab)*: The projectile to spawn.
* `objectToRotate` *(SceneObject)*: The Torus goal.
* `scoreZone` *(SceneObject)*: The invisible reference point for scoring.
* `scoreText`, `hintText`, `timerText` *(Component.Text)*: Links to the UI text elements.

**Game Balance Settings:**
* `gameDuration` *(float, default: 30.0)*: Total game time in seconds.
* `holeRadius` *(float, default: 3.0)*: The size of the scoring area. If a projectile is within this distance from the `scoreZone` center, it counts as a hit.
* `missPenalty` *(float, default: 1.0)*: Points deducted for a missed shot.
* `rotationSpeed` *(vec3, default: {0.0, 1.0, 0.0})*: The XYZ rotation speed of the Torus.
* `spawnInterval` *(float, default: 0.5)*: Time delay (in seconds) between each spawned projectile while the mouth is open.

---

## 5. Core Logic & Code Mechanics

### A. Event System (Interactions)
* **Face Tracking:** The script listens for `FaceFoundEvent` and `FaceLostEvent`. If the face is lost, the game pauses spawning.
* **Mouth Controls:** The game uses `MouthOpenedEvent` and `MouthClosedEvent`. The very first time the user opens their mouth, the script hides the Hint Text, shows the Score/Timer, and starts the game loop.

### B. Spawning & Physics
When the mouth is open, the `spawnCube()` function runs every `spawnInterval` seconds:
1. It creates a copy of the `cubePrefab`.
2. It adds a small random X and Z offset (`offsetRange = 0.5`) so projectiles don't fly in a perfectly straight line.
3. It uses a `DelayedCallbackEvent` (set to 0.0 seconds). This is a required technical fix in Lens Studio to make sure the object is fully loaded before applying physics forces.
4. It applies a physical velocity (`launchVector = new vec3(5.0, 10.0, -10.0)`). 

### C. Scoring System (Anti-Tunneling)
To prevent fast projectiles from glitching through the Torus, the script avoids physics triggers and uses a math-based approach:
1. The `checkCubesStatus()` function constantly checks the Y-axis (height) of every active projectile.
2. If a projectile falls below the `scoreZone` height, the script calculates the 2D distance (X and Z axis) between the projectile and the center of the hole.
3. If the distance is less than or equal to `holeRadius`, the player gets **+1 point**.
4. If the distance is greater (meaning it bounced off the rim or missed entirely), the player loses points (`missPenalty`).

### D. Memory Optimization (Garbage Collection)
To keep the frame rate high and prevent device crashes, the script cleans up old objects. Once a projectile falls 50 units below the Torus (`zonePos.y - 50.0`), the script calls `cube.destroy()` to remove it from the device memory.

---

## 6. Developer Guide: How to Customize
If you need to make changes to the game feel, look for these specific areas in the code:
* **Change Shooting Direction/Speed:** Find the `launchVector` variable inside the `spawnCube()` function. Change the X, Y, and Z values to make the projectiles shoot higher, faster, or further.
* **Change the Spawn Spread:** Adjust the `offsetRange` variable inside `spawnCube()`. A higher number makes the projectiles scatter more.
* **Fix Unfair Misses:** If valid hits are counting as misses, go to the script Inspector and increase the `holeRadius` value slightly.
