# Paint the World: POC Specifications

## Overview

Three POCs to validate core mechanics:
1. **Static Prerendered Worlds** - Can we generate, stylize, and render splats with brush selection UI?
2. **Interactive Playable** - Can we do real-time locomotion + painting + camera rotation at acceptable frame rates?
3. **User Image Import** - Can users upload images and convert them to splats in practical time?

Each POC is independently testable and validates different risk areas.

---

## POC 1: Static Prerendered Worlds with Stylized Brushes

### Goal
Test the complete offline pipeline: scene generation → style transfer → rendering with UI interaction. Establish baseline quality and understand compute costs.

### Scope
- Use HY-World-2.0 to generate 1-2 base splat scenes from text prompts
- Apply StyleGaussian to create 3-4 style variants per scene (e.g., "marshmallow," "diamond," "kitten," "original")
- Build minimal web viewer: display splat, UI buttons to switch between styles
- No audio, no movement, no painting—just static viewing with brush switching

### Success Criteria
- At least 1 scene generated with visible quality (photorealistic or convincing)
- 3+ style variants created and visibly distinct from each other
- Web viewer loads and renders at 60fps (desktop WebGL)
- Style switching is smooth (< 500ms perceived latency)
- File sizes reasonable for web delivery (< 200MB total per scene)

### What Gets Built
1. **HY-World-2.0 pipeline** - Text prompt generation (4090)
2. **StyleGaussian pipeline** - Style variant generation (4090)
3. **Web viewer** - WebGL splat renderer with style selector UI (M5)
4. **Asset pipeline** - Export/compression/hosting setup

### Unknowns to Resolve
- What is actual HY-World-2.0 generation quality? Is text-to-splat photorealistic enough or cartoonish?
- How distinct are StyleGaussian variants? Do different style images produce visually meaningful differences?
- What is compute time per scene and per style variant? (hours? days?)
- Can splat files be compressed/streamed for web without quality loss?
- Does WebGL rendering handle typical splat complexity (10M+ Gaussians)?

### Hardware
- **4090**: Scene generation + style transfer
- **M5**: Viewer development and testing

### Deliverable
- `pocs/poc1-static-worlds/` folder containing:
  - Generated splat files (at least 2 scenes, 3+ styles each)
  - Web viewer code (HTML/WebGL)
  - Performance metrics (FPS, file sizes, compute times)
  - Quality assessment (screenshots, notes on visual output)

---

## POC 2: Interactive Playable - Forward Motion + Painting + Rotation

### Goal
Test real-time user interaction: constant forward locomotion, camera rotation via swipe, painting effects synchronized to audio timeline, all at playable frame rates.

### Scope
- Use 1 splat scene from POC 1 (or simpler placeholder)
- Implement camera rail: user moves forward at constant speed (1 scene = 30-60 seconds duration)
- Swipe input: rotate camera around rail axis (±45 degrees?)
- Tap input: paint brushstroke at tap location, applies a visual effect (particle system, splat color change, or overlay)
- Audio: Play a 30-60 second song clip, sync painting/scene transitions to beat timestamps
- No style transfer during playback (use pre-computed variants from POC 1 if needed)

### Success Criteria
- Maintains 60fps on high-end web desktop (RTX 3080+, M1+, or equivalent)
- Camera rotation feels responsive (< 100ms latency)
- Painting effect is visible and synced to audio beat (±200ms tolerance)
- Forward movement is smooth (no judder)
- User can reach end of 60-second scene without interaction breaking

### What Gets Built
1. **Camera rail system** - Fixed-speed forward movement + rotation constraints
2. **Input handling** - Swipe gesture detection, tap-to-world raycast
3. **Painting effects** - Visual feedback (trail, particle, overlay, or splat modification)
4. **Audio sync** - Beat detection or hardcoded timeline keyframes
5. **Performance monitoring** - Frame rate counter, frame time profiling

### Unknowns to Resolve
- What is frame rate ceiling on different hardware? (goal: 60fps on mid-high desktop)
- Does painting effect computation stay within budget? (GPU-side vs CPU-side trade-off)
- How does painting affect frame rate as effect count increases?
- Can splat rendering handle dynamic camera movement without artifacts?
- Is tap-to-3D-world mapping accurate enough for meaningful painting?
- How tight can audio sync be? (real-time or can we use hardcoded beat timestamps?)

### Hardware
- **M5 Max**: Primary development and testing (target platform)
- **4090**: Optional performance comparison/validation

### Deliverable
- `pocs/poc2-interactive/` folder containing:
  - Web app code (camera, input, audio, painting, rendering)
  - Test audio file (30-60 sec song clip with clear beat)
  - Frame rate metrics (min/avg/max FPS under different conditions)
  - Interaction notes (does it feel good? Any usability issues?)
  - Video/screenshot of gameplay

---

## POC 3: User Image Import → Splats

### Goal
Validate whether users can upload images and get splats back in practical time. Understand end-user experience and compute constraints.

### Scope
- Web upload form: user selects 1-5 images
- Backend pipeline: Run COLMAP structure-from-motion, initialize Gaussians, train 3DGS model
- Output: Downloadable .ply file with styled variant (optional)
- Constraint: Aim for < 5 minutes total time per upload (or state actual time honestly)

### Success Criteria
- Upload UI works reliably
- COLMAP initialization succeeds on typical smartphone photos or drone footage
- Training converges to usable splat quality (recognizable as input scene)
- User can download result and load in viewer from POC 1 or 2
- Process time is documented and acceptable (if > 5 min, state the time clearly)

### What Gets Built
1. **Upload interface** - Web form + server handling
2. **COLMAP pipeline** - Wrapper/automation for structure-from-motion
3. **Training pipeline** - Gaussian splat optimization (could use official repo or HY-World-2.0 variant)
4. **Output handling** - Compression, delivery, error handling
5. **User feedback** - Progress bar, status messages, download link

### Unknowns to Resolve
- How many images are required for good reconstruction? (3? 10? 50?)
- What's realistic compute time per upload? (minutes? hours?)
- Can training be parallelized/batched?
- What's the maximum image resolution users should upload?
- How does reconstruction quality vary by input (drone vs. phone, outdoors vs. indoors)?
- Do training hyperparameters need tuning per scene or can one config work generically?

### Hardware
- **4090**: Training backend (required for practical time)
- **M5**: Upload interface development and testing

### Deliverable
- `pocs/poc3-user-import/` folder containing:
  - Web upload form code
  - Backend pipeline (COLMAP + training wrappers)
  - Test images (sample inputs and their outputs)
  - Timing metrics (upload → COLMAP → training → download)
  - Documented failure modes and constraints

---

## Integration Path (If Successful)

If all three POCs validate:
- **POC 1 output** (pre-styled scenes) feeds into **POC 2** (interactive viewer)
- **POC 3 output** (user uploads) can be styled with techniques from **POC 1** and played in **POC 2**
- Full feature: Users upload images → system converts to splats → applies artistic styles → plays in interactive experience synced to music

## Timeline & Sequencing

1. **POC 1 first** (4090): Establish quality baseline and compute costs
2. **POC 2 in parallel** (M5): Viewer development doesn't depend on POC 1 output
3. **POC 3 after POC 1** (4090): Use learned pipeline setup from POC 1 to build training infrastructure

## Risk Summary

| POC | Biggest Risk | Fallback |
|-----|--------------|----------|
| 1 | HY-World-2.0 quality is cartoon-like or blurry | Use manual splat datasets (Polycam, NeRF-360) instead of generation |
| 2 | Splat rendering too slow on desktop WebGL at 60fps | Target lower FPS (30fps) or reduce splat complexity (LOD, decimation) |
| 3 | COLMAP training takes hours per image set | Use commercial service (Polycam, ArcGIS Reality) or pre-trained models |
