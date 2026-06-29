# AR Torus Game: Project Documentation

## 1. Overview
This project is an interactive Augmented Reality (AR) game developed for Snapchat using Lens Studio. The core concept revolves around a rotating goal (a Torus) circling the user's head. The player's objective is to shoot projectiles (cubes/foam) into the moving goal within a limited timeframe to achieve the highest score possible.

## 2. User Controls & Interaction
The game is completely hands-free and controlled via Snap's built-in Face Tracking technology:
* **Face Tracking:** The lens anchors the game environment to the user's face.
* **Mouth Open:** Acts as the primary trigger. Opening the mouth for the first time starts the game timer. Keeping the mouth continuously open generates a steady stream of projectiles.
* **Mouth Close:** Instantly pauses the generation of projectiles.

## 3. Project Structure
The Scene Hierarchy and Asset Browser are organized into a few essential components to ensure smooth performance:

* **The Goal (Torus):** A 3D Torus object that continuously rotates. It acts as the physical rim of the goal.
* **The Projectile (Cube Prefab):** A dynamic 3D object stored in the Assets browser. It possesses physics properties (velocity, gravity) and is instantiated dynamically during gameplay.
* **The Score Zone (Invisible Cylinder):** A transparent 3D cylinder nested precisely inside the Torus hole. This is a critical structural element used to calculate accurate scoring without relying on heavy physics collisions.
* **UI Canvas:** * *Hint Text:* Prompts the user to start the game.
    * *Score Text:* Tracks the live score.
    * *Timer Text:* Displays the countdown.
* **Game Manager / Spawner:** An empty 3D object in the scene that holds the main logic, linking the UI, physical objects, and user interactions together.

## 4. Core Mechanics & Functionality
* **Game Initialization:** Upon launching the lens, the UI prompts the user to open their mouth. The timer and score are hidden until the first interaction.
* **Spawning & Physics:** While the user's mouth is open, projectiles spawn at a set interval (e.g., every 0.5 seconds). They are assigned an immediate physical velocity, launching them toward the rotating Torus.
* **Scoring System (Anti-Tunneling Logic):** To prevent fast-moving projectiles from glitching through colliders (a common physics issue called "tunneling"), the scoring relies on coordinate-based math. 
    * The system constantly tracks the Y-axis position of falling projectiles.
    * When a projectile drops below the invisible Score Zone's height, the game calculates its exact distance from the center of the hole.
    * **Hit (+1 point):** The projectile falls cleanly inside the hole's radius.
    * **Miss (-1 point):** The projectile falls outside the radius or bounces off the Torus edge.
* **Resource Optimization:** To maintain a high frame rate and prevent device overheating, any projectile that falls far below the screen is automatically destroyed and wiped from the device's memory.
* **Game Over:** Once the timer reaches zero, the game ends. Spawning is disabled, all active projectiles are cleared from the screen, and the final score is locked.
