# Gridshotto

A high-performance web-based aim trainer built with **Three.js**. Practice your precision and speed directly in your browser.

## Technical Documentation

This project is a single-page application (SPA) that uses **Three.js** to create a 3D environment for aim training. Below is a detailed breakdown of how the code works.

### 1. Styling & Themes (CSS)

The application uses CSS variables for theme management and an absolute-positioned UI overlay.

```css
:root {
    --bg: #0b0e14;
    --text-color: #ffffff;
    --ui-accent: #2563eb;
    --panel: rgba(0, 0, 0, 0.9);
}

.light-theme {
    --bg: #f0f2f5;
    --text-color: #000000;
    --panel: rgba(255, 255, 255, 0.95);
}
```

**How it works:**
-   **CSS Variables**: Defined in `:root`, these allow for instant theme switching by toggling the `.light-theme` class on the `<body>`.
-   **UI Scaling**: Stats (score, time, accuracy) are positioned using `flexbox` and absolute coordinates to stay pinned to the corners of the viewport regardless of screen size.

### 2. Audio System

Sound effects are generated dynamically using the Web Audio API to avoid external asset loading latencies.

```javascript
function playSound(freq, type, duration, vol) {
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = type;
    osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + duration);
}
```

**How it works:**
-   **Oscillators**: Create sine or square waves at specific frequencies (e.g., 600Hz for hits, 150Hz for shots).
-   **Gain Nodes**: Handle the volume envelope, ramping down quickly to create a "click" or "pop" sound effect.

### 3. Three.js Environment

The 3D scene consists of a camera, a floor, and a "meme" wall that serves as the backdrop for the targets.

```javascript
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true });

const memeUrl = 'images.jfif';
const texLoader = new THREE.TextureLoader();
texLoader.load(memeUrl, (tex) => {
    memeMat.map = tex;
    memeMat.needsUpdate = true;
});
```

**How it works:**
-   **PerspectiveCamera**: Set with a 75-degree FOV to mimic standard FPS view angles.
-   **TextureLoader**: Asynchronously loads the `images.jfif` file and applies it to a plane positioned behind the grid.
-   **Lighting**: An `AmbientLight` provides even illumination across all targets and the floor.

### 4. Target & Grid Logic

Targets are spawned at fixed grid positions to ensure fairness and consistent training patterns.

```javascript
const gridPositions = [];
for (let x = -2.25; x <= 2.25; x += 1.5) {
    for (let y = -1.5; y <= 1.5; y += 1.5) gridPositions.push({ x: x, y: y });
}

function spawnTarget() {
    const sphere = new THREE.Mesh(new THREE.SphereGeometry(0.4, 32, 32), material);
    const idx = avail[Math.floor(Math.random() * avail.length)];
    sphere.position.set(gridPositions[idx].x, gridPositions[idx].y, -6);
    scene.add(sphere);
}
```

**How it works:**
-   **Grid Generation**: A double loop creates a coordinate system for a 4x3 grid.
-   **Collision Awareness**: `spawnTarget` checks the `targets` array to ensure new targets don't overlap with existing ones.
-   **Lerping**: When spawned, targets use `sphere.scale.lerp` in the animation loop to smoothly grow to full size.

### 5. Input & Aiming

The game uses Pointer Lock to allow for infinite mouse movement, translating mouse delta into camera rotation.

```javascript
document.onmousemove = (e) => {
    if (document.pointerLockElement === document.body) {
        const sens = (sensInput / engine) * 0.0065;
        yaw -= e.movementX * sens;
        pitch -= e.movementY * sens;
        camera.rotation.set(pitch, yaw, 0);
    }
};
```

**How it works:**
-   **Sensitivity Scaling**: Users can select "Valorant" or "CS2" presets. The code divides the raw input by the engine's base sensitivity factor and applies a constant for consistent pixel-to-degree movement.
-   **Clamping**: Pitch is clamped to +/- 90 degrees to prevent the camera from flipping over.

### 6. Raycasting & Scoring

The `shoot` function handles the core gameplay loop: detecting hits and updating the score.

```javascript
function shoot() {
    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
    const hit = raycaster.intersectObjects(targets)[0];

    if (hit) {
        score += 500;
        hits++;
        scene.remove(hit.object);
        spawnTarget();
    } else {
        score = Math.max(0, score - 150);
    }
}
```

**How it works:**
-   **Raycaster**: Projects a line from the camera center (0,0) into the 3D space.
-   **Hit Detection**: Checks for intersections with the `targets` array. If the first intersected object is a target, it's a hit.
-   **Scoring**: Adds 500 points for hits and subtracts 150 for misses, mirroring the "Aim Lab Gridshot" scoring model.

## Deployment

The project is live at: [**https://afnanww.github.io/gridshotto/**](https://afnanww.github.io/gridshotto/)

To run locally, simply open `index.html` in any modern web browser. Requires an internet connection to load the `Three.js` library.

---

## Meta Information
**Last Updated**: 8 December 2023
**Developer**: afnanww  
**Web Name**: Gridshotto
