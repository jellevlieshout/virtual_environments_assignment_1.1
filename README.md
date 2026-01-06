# Virtual Environments Assignment 1.1 - Google Cardboard VR Tutorial

This project is a Unity VR application following the [Google Cardboard Unity Quickstart tutorial](https://developers.google.com/cardboard/develop/unity/quickstart). It demonstrates core VR features using Google Cardboard XR Plugin, including head tracking, stereoscopic rendering, and gaze-based interaction.

## Project Overview

**Unity Version:** 6000.2.10f1  
**Cardboard XR Plugin Version:** 1.31.0  
**Project Type:** 3D VR Application

The project implements a demo game called "HelloCardboard" where users can look around a virtual environment to find and collect interactive objects using gaze-based interaction.

---

## Project Structure

```
Assets/
├── Samples/
│   └── Google Cardboard XR Plugin for Unity/
│       └── 1.31.0/
│           └── Hello Cardboard/
│               ├── Environments/          # 3D environment assets
│               ├── Models/                # 3D model meshes
│               ├── Materials/             # Material definitions
│               ├── Textures/              # Texture assets
│               ├── Scenes/                # Unity scene files
│               └── Scripts/               # C# scripts
├── Scenes/                                # Additional scene files
├── Settings/                              # Render pipeline settings
├── XR/                                    # XR configuration
└── Plugins/                               # Platform-specific plugins
    └── Android/                           # Android build configuration
```

---

## Game Scenes

### 1. HelloCardboard Scene
**Path:** `Assets/Samples/Google Cardboard XR Plugin for Unity/1.31.0/Hello Cardboard/Scenes/HelloCardboard.unity`

The main demo scene showcasing VR interaction. Users look around a virtual room and interact with collectible objects using gaze-based selection.

### 2. VrMode Scene
**Path:** `Assets/Samples/Google Cardboard XR Plugin for Unity/1.31.0/Hello Cardboard/Scenes/VrMode.unity`

Demonstrates how to toggle VR mode on and off dynamically during runtime. Users can exit VR mode by tapping the exit button and re-enter by tapping anywhere on the screen.

---

## Assets and Game Objects

### Environment Assets

#### CubeRoom Prefab
**Path:** `Assets/Samples/Google Cardboard XR Plugin for Unity/1.31.0/Hello Cardboard/Environments/CubeRoom.prefab`

- **Purpose:** The main virtual environment - a cube-shaped room
- **Components:**
  - `Transform` - Position and rotation
  - `MeshFilter` - References the CubeRoom mesh
  - `MeshRenderer` - Renders the room with CubeRoomEnvMat material
  - `MeshCollider` - Provides collision detection for the room boundaries
- **Material:** `CubeRoomEnvMat` - Uses baked diffuse lighting texture (`CubeRoom_BakedDiffuse.png`)

### 3D Models

#### Treasure Objects
**Parent GameObject:** `Treasure` (Layer: Interactive)

Contains three child objects representing collectible items:

1. **Icosahedron**
   - **Mesh:** Icosahedron.obj
   - **Materials:** 
     - Inactive: `IcosahedronBlue.mat` (when not gazed at)
     - Active: `IcosahedronPink.mat` (when gazed at)
   - **Components:**
     - `RectTransform`
     - `MeshFilter`
     - `MeshRenderer`
     - `SphereCollider` (radius: 0.5)
     - `ObjectController` script

2. **QuadSphere**
   - **Mesh:** QuadSphere.obj
   - **Materials:**
     - Inactive: `QuadSphereBlue.mat`
     - Active: `QuadSpherePink.mat`
   - **Components:** Same as Icosahedron

3. **TriSphere**
   - **Mesh:** TriSphere.obj
   - **Materials:**
     - Inactive: `TriSphereBlue.mat`
     - Active: `TriSpherePink.mat`
   - **Components:** Same as Icosahedron
   - **Initial State:** Inactive (disabled at start)

**Behavior:** Only one treasure object is active at a time. When collected, it teleports to a random location and activates a different sibling object.

### Player Setup

#### Player GameObject
**Hierarchy:** `Player` → `Main Camera`

- **Transform:** Positioned at origin (0, 0, 0)
- **Components:**
  - `Transform`

#### Main Camera
**Tag:** MainCamera  
**Parent:** Player

- **Components:**
  - `Camera` - Main rendering camera
    - Field of View: 60°
    - Near Clip Plane: 0.03
    - Far Clip Plane: 1000
    - Target Eye: Both (for stereoscopic rendering)
  - `AudioListener` - Audio output
  - `TrackedPoseDriver` - XR head tracking component
    - Tracks head position and rotation from XR input
    - Uses XR HMD input actions for position and rotation
  - `UniversalAdditionalCameraData` - URP camera settings
  - `CardboardReticlePointer` - Gaze-based interaction pointer
    - Reticle Interaction Layer Mask: Interactive layer
    - Provides visual reticle and raycast-based interaction

### Lighting

#### Point Light
- **Type:** Point Light
- **Purpose:** Illuminates the virtual environment
- **Position:** Located within the scene to provide ambient lighting

### UI Elements

#### GraphicsAPIText
- **Component:** `TextMesh`
- **Purpose:** Displays the current graphics API being used (OpenGL ES 3, Vulkan, Metal, etc.)
- **Script:** `GraphicsAPITextController` - Updates text based on detected graphics API

---

## Scripts and Interactions

### 1. CardboardStartup.cs
**Path:** `Assets/Samples/Google Cardboard XR Plugin for Unity/1.31.0/Hello Cardboard/Scripts/CardboardStartup.cs`

**Purpose:** Initializes and manages Cardboard XR Plugin lifecycle.

**Key Functionality:**
- **Start():**
  - Prevents screen from sleeping (`Screen.sleepTimeout = SleepTimeout.NeverSleep`)
  - Sets screen brightness to maximum
  - Scans device parameters if not already stored

- **Update():**
  - Handles gear button press to scan device parameters
  - Handles close button press to quit application
  - Handles trigger hold (3+ seconds) to recenter head tracking
  - Reloads device parameters when new ones are detected
  - Updates screen parameters for proper rendering

**Usage:** Attached to a GameObject in the scene (typically the Player or a dedicated manager object).

### 2. ObjectController.cs
**Path:** `Assets/Samples/Google Cardboard XR Plugin for Unity/1.31.0/Hello Cardboard/Scripts/ObjectController.cs`

**Purpose:** Controls the behavior of collectible treasure objects.

**Key Functionality:**
- **Material Management:**
  - Switches between `InactiveMaterial` (blue) and `GazedAtMaterial` (pink) based on gaze state
  - `SetMaterial(bool gazedAt)` - Updates material when gazed at or not

- **Interaction Methods:**
  - `OnPointerEnter()` - Called when user starts gazing at the object (changes to pink material)
  - `OnPointerExit()` - Called when user stops gazing (changes back to blue material)
  - `OnPointerClick()` - Called when user triggers selection (calls `TeleportRandomly()`)

- **Teleportation:**
  - `TeleportRandomly()` - Moves the parent GameObject to a random position
    - Random angle: -π to π
    - Distance: 2.5m to 3.5m from origin
    - Height: 0.5m to 3.5m
  - Activates a random sibling object and deactivates itself
  - Ensures only one treasure is visible at a time

**Public Fields:**
- `InactiveMaterial` - Material used when object is not being gazed at
- `GazedAtMaterial` - Material used when object is being gazed at

**Usage:** Attached to each treasure object (Icosahedron, QuadSphere, TriSphere).

### 3. VrModeController.cs
**Path:** `Assets/Samples/Google Cardboard XR Plugin for Unity/1.31.0/Hello Cardboard/Scripts/VrModeController.cs`

**Purpose:** Manages entering and exiting VR mode dynamically.

**Key Functionality:**
- **VR Mode Detection:**
  - `_isVrModeEnabled` - Checks if XR is initialized and active

- **Start():**
  - Saves reference to main camera
  - Configures screen settings (sleep timeout, brightness)
  - Scans device parameters if needed

- **Update():**
  - **In VR Mode:**
    - Handles close button to exit VR
    - Handles gear button to scan device parameters
    - Updates screen parameters
  - **Not in VR Mode:**
    - Detects screen touch to enter VR mode

- **VR Mode Management:**
  - `EnterVR()` - Starts XR coroutine and reloads device parameters
  - `ExitVR()` - Stops XR subsystems and resets camera settings
  - `StartXR()` - Coroutine that initializes and starts XR subsystems
  - `StopXR()` - Stops and deinitializes XR subsystems, resets camera FOV

**Usage:** Used in the VrMode scene to demonstrate dynamic VR mode switching.

### 4. GraphicsAPITextController.cs
**Path:** `Assets/Samples/Google Cardboard XR Plugin for Unity/1.31.0/Hello Cardboard/Scripts/GraphicsAPITextController.cs`

**Purpose:** Displays the current graphics API being used by the application.

**Key Functionality:**
- **Start():**
  - Gets the `TextMesh` component
  - Detects graphics device type using `SystemInfo.graphicsDeviceType`
  - Updates text to show:
    - "OpenGL ES 2" (Unity < 2023.1)
    - "OpenGL ES 3"
    - "Metal" (iOS)
    - "Vulkan" (Android)
    - "Unrecognized Graphics API" (fallback)

**Usage:** Attached to the GraphicsAPIText GameObject.

---

## Interaction System

### Gaze-Based Interaction

The interaction system uses a **gaze-based pointer** mechanism:

1. **CardboardReticlePointer Component:**
   - Attached to the Main Camera
   - Projects a visual reticle in the center of the view
   - Performs raycast detection on the "Interactive" layer
   - Detects when objects are being gazed at

2. **Interaction Flow:**
   - User looks at an object → `OnPointerEnter()` called → Material changes to pink
   - User looks away → `OnPointerExit()` called → Material changes back to blue
   - User presses Cardboard trigger while gazing → `OnPointerClick()` called → Object teleports

### Input Handling

**Cardboard Button Controls:**
- **Gear Button:** Scan device parameters (QR code scanning)
- **Close Button:** Exit application (or exit VR mode in VrMode scene)
- **Trigger (held 3+ seconds):** Recenter head tracking
- **Trigger (quick press):** Select/interact with gazed object

**Touch Input:**
- Used in VrMode scene to enter VR mode when not active

### Head Tracking

- **TrackedPoseDriver:** Tracks head position and rotation from XR input
- **Recenter Functionality:** Allows users to recalibrate the forward direction
- **Stereoscopic Rendering:** Camera renders separate views for left and right eyes

---

## Technical Details

### XR Configuration

**XR Plugin:** Google Cardboard XR Plugin for Unity  
**XR Settings:** Configured in `Assets/XR/Settings/XRSettings.asset`  
**XR Loader:** Configured in `Assets/XR/Loaders/XRLoader.asset`

### Render Pipeline

**Pipeline:** Universal Render Pipeline (URP)  
**Settings:**
- Mobile Renderer: `Assets/Settings/Mobile_Renderer.asset`
- PC Renderer: `Assets/Settings/PC_Renderer.asset`
- Render Pipeline Assets: `Mobile_RPAsset.asset` and `PC_RPAsset.asset`

### Build Configuration

**Android:**
- Minimum API Level: Android 8.0 (API 26)
- Target API Level: API 35
- Scripting Backend: IL2CPP
- Graphics APIs: OpenGL ES 3, Vulkan
- Custom Gradle templates configured in `Assets/Plugins/Android/`

**iOS:**
- Minimum iOS Version: 12.0
- Camera permission required for QR code scanning

---

## References

- [Google Cardboard Unity Quickstart Guide](https://developers.google.com/cardboard/develop/unity/quickstart)
- [Google Cardboard XR Plugin Repository](https://github.com/googlevr/cardboard-xr-plugin)
- Unity XR Plugin Management Documentation

---

## Notes

- The project uses Unity's new Input System (`Input System Package (New)`)
- All scripts are part of the `CardboardSamples` assembly definition
- The Interactive layer must be configured in Unity's Layers settings
- Device parameters are scanned via QR code to configure the Cardboard viewer

