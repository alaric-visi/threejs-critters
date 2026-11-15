# ThreeJs Critters

A collection of 21 procedural 3D and wireframe biological/creature models built with **Three.js (`r128`)** as self-contained, zero-dependency HTML documents. Every specimen—from microscopic diatoms and tardigrades to articulated arthropods and a winged dragon—is synthesized entirely in code using parametric geometries, spline curves, vertex displacement, and procedural animation loops.

- **Series 01 — Aether Wireframe Models (`wireframe-models/`)**: [archive.mikehiggins.me.uk/notes/critter-models-01/](https://archive.mikehiggins.me.uk/notes/critter-models-01/) ([About Mesh Insects](https://archive.mikehiggins.me.uk/notes/critter-models-01/about-mesh-insects))
- **Series 02 — PBR 3D Specimen Models (`3d-models/`)**: [archive.mikehiggins.me.uk/notes/critter-models-02/](https://archive.mikehiggins.me.uk/notes/critter-models-02/)

---

## Repository Structure

```text
threejs-critters/
├── 3d-models/           # 14 physically-based (PBR) shaded 3D models
├── wireframe-models/    # 7 unlit "Aether" wireframe & edge-mesh models
└── .gitignore
```

---

## Technical Architecture

Every file in this repository is a standalone HTML page with **no build step, no bundler, and no external `.glb`/`.gltf` or image texture assets**. All geometry, surface detail, and motion are generated at runtime.

### 1. Dual-Canvas Layered Rendering
Each page stacks two full-viewport `<canvas>` elements:
- **`#bgCanvas` (`z-index: 1`, 2D Context)**: Renders the backdrop independently from the WebGL pass.
  - In `wireframe-models/`, `#bgCanvas` draws an animated 8×8 px grid of shimmering dark-cyan pixels using a precomputed 49-step RGB lookup ramp (`RAMP_STEPS = 48`) updated at half the display frame rate (`(bgFrame++ & 1) === 0`) to halve 2D fill cost.
  - In `3d-models/`, `#bgCanvas` renders a specimen-specific `createRadialGradient` studio backdrop.
- **`#animationCanvas` (`z-index: 2`, WebGL Context)**: Hosts `THREE.WebGLRenderer({ antialias: true, alpha: true })` with `devicePixelRatio` capped at `2` (`Math.min(devicePixelRatio, 2)`) for crisp rendering without excessive fill-rate overhead on high-DPI displays.

### 2. Responsive Camera & Viewport Scaling
Both series adapt camera distance and vertical field-of-view (`PerspectiveCamera`) across three viewport breakpoints (`< 480px`, `< 768px`, `>= 768px`) on load and `resize`, keeping specimens framed cleanly on mobile and desktop screens.

### 3. Series 01: Aether Wireframe Series (`wireframe-models/`)
- **Materials**: Uses unlit `THREE.MeshBasicMaterial` (with `wireframe: true` or solid dark occlusion cores) and `THREE.LineBasicMaterial` over `THREE.EdgesGeometry`. Because `MeshBasicMaterial` ignores scene lights, materials are shared by color across meshes.
- **Spline & Shape Construction**: Limbs, antennae, wing venation, and segmented tails are built with `THREE.TubeGeometry` along `THREE.CatmullRomCurve3` (and `QuadraticBezierCurve3` in `wireframe-fly.html`), paired with `THREE.ShapeGeometry` membranes and `ConeGeometry` / `TorusGeometry` / `CylinderGeometry` accents.
- **Embed Theme Synchronization (`syncTheme`)**: A `MutationObserver` watches `window.parent.document.documentElement` for `class` changes (e.g., toggling `.dark` when embedded in an iframe), dynamically switching `document.body.style.backgroundColor` and `scene.background` between `#000000` (dark) and `#ffffff` (light).
- **Interaction**: Pointer/touch drag directly rotates the specimen root group with smooth damping and idle auto-rotation.

### 4. Series 02: PBR 3D Specimen Series (`3d-models/`)
- **Lighting & Shading**: Uses `THREE.MeshStandardMaterial` lit by a multi-point studio rig combining `HemisphereLight`, key/rim `DirectionalLight`s, and colored `PointLight` accents.
- **Procedural Surface Detail**:
  - `fuzz()` / `wrinkle()` / `dimpledSphere()`: Perturb `BufferGeometry` vertex positions via sinusoidal harmonics plus stochastic noise, then recompute smooth normals (`geo.computeVertexNormals()`).
  - `shimmer()` / `iri()`: Cycles material HSL hue over time (`m.color.setHSL(...)`) to simulate structural iridescence on wings, carapaces, and silica frustules.
  - `segmentBetween(p0, p1, r0, r1, ...)`: Computes midpoint and unit-vector quaternion orientation (`setFromUnitVectors`) to join arbitrary 3D joint coordinates with tapered `CylinderGeometry` segments.
  - `chitin()` / `matte()`: Tuned roughness/metalness presets (`DoubleSide`) for exoskeleton plates, hairs, and translucent membranes (`depthWrite: false`).
- **Damped Spherical Orbit Camera**: Mouse, touch, and wheel events update `targetYaw`, `targetPitch` (clamped to `[-1.35, 1.35]` rad), and `targetDist`, interpolated each frame (`0.08` smoothing factor) alongside an `autoYaw` idle orbit.

---

## Specimen Catalog & Live Links

### Series 01 — Wireframe Models (`wireframe-models/`)
Downstream of [`https://archive.mikehiggins.me.uk/notes/critter-models-01/`](https://archive.mikehiggins.me.uk/notes/critter-models-01/)

| File | Live Project | Key Primitives & Materials | Technical Highlights |
| :--- | :--- | :--- | :--- |
| [wireframe-bee.html](wireframe-models/wireframe-bee.html) | [Wireframe Bee](https://archive.mikehiggins.me.uk/notes/critter-models-01/wireframe-bee) | `SphereGeometry`, `CylinderGeometry`, `ConeGeometry`, `TorusGeometry`, `TubeGeometry` (`CatmullRomCurve3`), `ShapeGeometry`, `EdgesGeometry`, `MeshBasicMaterial`, `LineBasicMaterial` | Procedural `addFuzz` hair spines, segmented abdomen rings, veined translucent/wireframe wings (`makeWing`), articulated legs & antennae. |
| [wireframe-butterfly.html](wireframe-models/wireframe-butterfly.html) | [Wireframe Butterfly](https://archive.mikehiggins.me.uk/notes/critter-models-01/wireframe-butterfly) | `SphereGeometry`, `CircleGeometry`, `TorusGeometry`, `TubeGeometry` (`CatmullRomCurve3`), `ShapeGeometry`, `EdgesGeometry`, `MeshBasicMaterial`, `LineBasicMaterial` | Separate `makeForewing` & `makeHindwing` profiles with `addEyespot` concentric rings, clubbed antennae, and synchronized wing-beat animation. |
| [wireframe-fly.html](wireframe-models/wireframe-fly.html) | [Wireframe Fly](https://archive.mikehiggins.me.uk/notes/critter-models-01/wireframe-fly) | `SphereGeometry`, `CylinderGeometry`, `TorusGeometry`, `TubeGeometry` (`CatmullRomCurve3`, `QuadraticBezierCurve3`), `ShapeGeometry`, `MeshBasicMaterial` | Oversized compound-eye hemispheres, proboscis curve, halteres, and rapid wing oscillation. |
| [wireframe-locust.html](wireframe-models/wireframe-locust.html) | [Wireframe Locust](https://archive.mikehiggins.me.uk/notes/critter-models-01/wireframe-locust) | `BoxGeometry`, `CircleGeometry`, `CylinderGeometry`, `SphereGeometry`, `TorusGeometry`, `TubeGeometry` (`CatmullRomCurve3`), `ShapeGeometry`, `EdgesGeometry`, `MeshBasicMaterial`, `LineBasicMaterial` | Largest wireframe model (626 lines); armored pronotum, `makeTegmen` forewings, `makeHindwingFolded` fan geometry, and enlarged jumping `makeHindLeg` femur/tibia. |
| [wireframe-mosquito.html](wireframe-models/wireframe-mosquito.html) | [Wireframe Mosquito](https://archive.mikehiggins.me.uk/notes/critter-models-01/wireframe-mosquito) | `CircleGeometry`, `ConeGeometry`, `CylinderGeometry`, `SphereGeometry`, `TorusGeometry`, `TubeGeometry` (`CatmullRomCurve3`), `ShapeGeometry`, `EdgesGeometry`, `MeshBasicMaterial`, `LineBasicMaterial` | Elongated proboscis, plumose `makeAntenna`, narrow veined wings, and long multi-jointed stilt legs. |
| [wireframe-scorpion.html](wireframe-models/wireframe-scorpion.html) | [Wireframe Scorpion](https://archive.mikehiggins.me.uk/notes/critter-models-01/wireframe-scorpion) | `ConeGeometry`, `CylinderGeometry`, `SphereGeometry`, `TorusGeometry`, `TubeGeometry` (`CatmullRomCurve3`), `MeshBasicMaterial` | Articulated 5-segment metasoma tail curving over the prosoma, telson stinger, and twin pedipalp pincers. |
| [wireframe-spider.html](wireframe-models/wireframe-spider.html) | [Wireframe Spider](https://archive.mikehiggins.me.uk/notes/critter-models-01/wireframe-spider) | `CylinderGeometry`, `OctahedronGeometry`, `SphereGeometry`, `TubeGeometry` (`CatmullRomCurve3`), `MeshBasicMaterial` | Faceted `OctahedronGeometry` eye cluster, chelicerae, pedipalps, and 8 multi-segment `CatmullRomCurve3` legs with walking gait phase offsets. |

### Series 02 — PBR 3D Models (`3d-models/`)
Downstream of [`https://archive.mikehiggins.me.uk/notes/critter-models-02/`](https://archive.mikehiggins.me.uk/notes/critter-models-02/)

| File | Live Project | Key Primitives & Curves | Technical Highlights |
| :--- | :--- | :--- | :--- |
| [treehopper.html](3d-models/treehopper.html) | [Brazilian Treehopper](https://archive.mikehiggins.me.uk/notes/critter-models-02/brazilian-treehopper) | `CylinderGeometry`, `ShapeGeometry`, `SphereGeometry`, `TubeGeometry` (`CatmullRomCurve3`, `QuadraticBezierCurve3`) | *Bocydium globulare* pronotal helmet: 4 `dimpledSphere` chitinous globes (`makeGlobe`) suspended on curved stalks (`makeProcess`) above iridescent wings. |
| [bumblebee.html](3d-models/bumblebee.html) | [Bumblebee](https://archive.mikehiggins.me.uk/notes/critter-models-02/bumblebee) | `ConeGeometry`, `CylinderGeometry`, `ShapeGeometry`, `SphereGeometry` | Vertex-displaced `fuzz()` thorax/abdomen, `scatterHairs` & `legHairs` bristles, quaternion-aligned `segmentBetween` legs, and `shimmer()` translucent wings. |
| [butterfly.html](3d-models/butterfly.html) | [Butterfly](https://archive.mikehiggins.me.uk/notes/critter-models-02/butterfly) | `CylinderGeometry`, `ExtrudeGeometry`, `SphereGeometry`, `TubeGeometry` (`CatmullRomCurve3`, `QuadraticBezierCurve3`) | Extruded `makeForewing` / `makeHindwing` membranes with `wingSpot` markings, `iri()` structural color shift, and articulated wing-hinge animation. |
| [cockroach.html](3d-models/cockroach.html) | [Cockroach](https://archive.mikehiggins.me.uk/notes/critter-models-02/cockroach) | `CylinderGeometry`, `ExtrudeGeometry`, `SphereGeometry`, `TorusGeometry`, `TubeGeometry` (`QuadraticBezierCurve3`) | Flattened pronotum shield, `addSpines` tibial spurs along `makeLeg`, long sweeping `makeAntenna` splines, and overlapping `makeWing` tegmina. |
| [diatoms.html](3d-models/diatoms.html) | [Diatoms](https://archive.mikehiggins.me.uk/notes/critter-models-02/diatoms) | `BoxGeometry`, `CircleGeometry`, `ConeGeometry`, `CylinderGeometry`, `ExtrudeGeometry`, `SphereGeometry`, `TorusGeometry` | Multi-morphotype silica frustule colony (`makePennateCell`, `makeCentricCell`, `makeTriangleCell`, `makeRodCell`) arranged into `makeChain`, `makeZigzag`, `makeFan`, `makeRibbon`, and `buildRosette` formations with `iri()` opalescence. |
| [dragon.html](3d-models/dragon.html) | [Dragon](https://archive.mikehiggins.me.uk/notes/critter-models-02/dragon) | `BoxGeometry`, `BufferGeometry`, `CylinderGeometry`, `ShapeGeometry`, `SphereGeometry` | Parametric `radiusAt` / `spineAt` body tapering, custom `BufferGeometry` wing membranes stretched across articulated wing fingers (`makeWing`), horned `makeHead`, and undulating flight motion. |
| [weevil.html](3d-models/weevil.html) | [Giraffe-Necked Weevil](https://archive.mikehiggins.me.uk/notes/critter-models-02/giraffe-weevil) | `ConeGeometry`, `CylinderGeometry`, `SphereGeometry` | Elongated articulated rostrum/neck, glossy red elytra over a dark thorax, and jointed `makeLeg` / `makeAntenna` assemblies. |
| [hammerhead.html](3d-models/hammerhead.html) | [Hammerhead Worm](https://archive.mikehiggins.me.uk/notes/critter-models-02/hammerhead-worm) | `SphereGeometry` | Continuous multi-segment body driven by `widthAt`, `spineX`, and `spineY` traveling sinusoidal wave functions for realistic planarian gliding locomotion. |
| [locust.html](3d-models/locust.html) | [Locust](https://archive.mikehiggins.me.uk/notes/critter-models-02/locust) | `CircleGeometry`, `ConeGeometry`, `CylinderGeometry`, `ExtrudeGeometry`, `ShapeGeometry`, `SphereGeometry`, `TorusGeometry`, `TubeGeometry` (`QuadraticBezierCurve3`) | Most detailed PBR model (625 lines); `makeWalkLeg` vs. muscular `makeHindLeg` with `addSpines`, `makeTegmen` forewings, `makeHindWing` fan, and compound eyes. |
| [scorpion.html](3d-models/scorpion.html) | [Scorpion](https://archive.mikehiggins.me.uk/notes/critter-models-02/scorpion) | `ConeGeometry`, `CylinderGeometry`, `SphereGeometry`, `TorusGeometry`, `TubeGeometry` (`QuadraticBezierCurve3`) | Segmented `makeTail` metasoma with dynamic defensive curl, `makeClaw` chelae with articulating pincers, and 8 walking legs. |
| [spider.html](3d-models/spider.html) | [Spider](https://archive.mikehiggins.me.uk/notes/critter-models-02/spider) | `CylinderGeometry`, `SphereGeometry`, `TubeGeometry` (`QuadraticBezierCurve3`) | `addHairs` leg bristles, `makeChelicera` fangs, `makePalp` pedipalps, 8-eye cluster, and phase-shifted 8-leg locomotion. |
| [tardigrade.html](3d-models/tardigrade.html) | [Tardigrade](https://archive.mikehiggins.me.uk/notes/critter-models-02/tardigrade) | `CircleGeometry`, `ConeGeometry`, `CylinderGeometry`, `SphereGeometry`, `TorusGeometry` | Cuticle `wrinkle()` vertex displacement, telescopic `makeMouth` stylet ring, and 8 clawed `makeLobopodLeg` limbs with slow cryptobiotic gait. |
| [trilobite-beetle.html](3d-models/trilobite-beetle.html) | [Trilobite Beetle](https://archive.mikehiggins.me.uk/notes/critter-models-02/trilobite-beetle) | `CylinderGeometry`, `ExtrudeGeometry`, `SphereGeometry` | Overlapping `ExtrudeGeometry` thoracic tergite plates (`plateOutline` / `plateShape` / `makePlate`) with `dimpledSphere` tubercles and marginal spines. |
| [wasp.html](3d-models/wasp.html) | [Wasp](https://archive.mikehiggins.me.uk/notes/critter-models-02/wasp) | `ConeGeometry`, `CylinderGeometry`, `ShapeGeometry`, `SphereGeometry` | Narrow petiole waist, banded yellow-and-black abdomen segments, `scatterHairs`, `makeWingShape` membranes with `shimmer()`, and rapid wing-beat cycle. |

---

## Running Locally

Because each specimen is a self-contained HTML file (loading Three.js `r128` from `cdnjs.cloudflare.com`), you can open any file directly in a browser or serve the directory locally:

```bash
# Serve locally with Python
python3 -m http.server 8000
# Then open http://localhost:8000/3d-models/bumblebee.html
# or       http://localhost:8000/wireframe-models/wireframe-bee.html
```

### Controls
- **Orbit / Rotate**: Click and drag (or touch drag)
- **Zoom** (`3d-models/`): Scroll wheel
- **Auto-orbit**: Releases smoothly back into idle rotation

