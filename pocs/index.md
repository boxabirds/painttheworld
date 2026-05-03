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

---

## Restated Understanding of the Brief

**Three POCs, each a separate folder under `pocs/`:**

- **POC 1** — Offline pipeline: HY-World-2.0 generates base splats from text prompts, StyleGaussian produces 3-4 style variants per scene, web viewer toggles between them. Static. No interaction beyond style switching.
- **POC 2** — Interactive playable: rail-cam forward motion through a splat scene, swipe rotation, tap painting, audio-synced visuals. Built on POC 1's splats (or a placeholder).
- **POC 3** — User upload: web form → COLMAP structure-from-motion → 3DGS training → downloadable .ply. Loads in POC 1/2 viewer.

**Stated sequencing:** POC 1 first on 4090; POC 2 in parallel on M5; POC 3 after POC 1 on 4090.

---

## Unverified Assumptions (R Has Not Checked These)

R has made several pattern-match guesses below. None have been verified. Several are fabrication risks if R is wrong.

1. **HY-World-2.0 availability** — R does not know whether weights/code are publicly released, what license applies, or where to obtain them. Pattern-match suggests Tencent's Hunyuan family, but R has not verified a "HY-World-2.0" variant exists or is downloadable.

2. **StyleGaussian availability** — R has not verified the repo, pre-trained weights, installation requirements, or its compatibility with HY-World-2.0 outputs.

3. **Hardware topology** — Brief names "4090" and "M5/M5 Max" but current environment is this Linux box. R does not know:
   - Is the 4090 this machine, or a separate host?
   - Is the M5 a separate device you'll work on directly?
   - Should R assume it has access to the 4090 via SSH/remote, or is it entirely off-limits?

4. **WebGL splat renderer choice** — Candidates exist: `@mkkellogg/gaussian-splats-3d`, `gsplat.js`, `antimatter15/splat`, Spark, others. Brief did not specify, and R should not unilaterally pick one.

5. **Audio for POC 2** — No song clip specified. R assumed 30-60 seconds with "clear beat" but operator may have a specific track in mind.

6. **Asset hosting** — No preference stated for where to host/serve splat files.

7. **COLMAP for POC 3** — R assumed COLMAP is available and operator is comfortable with it. No verification of installation, version, or operational knowledge.

---

## Alternatives by Scope (R Is Not Ranking These)

Operator asked for "specs in the index doc." R reads three plausible scopes for what "the plan" means. Each is a different commitment size; R is not recommending one.

### Scope A: Single POC Start
Pick one POC (brief says POC 1 or POC 2 can begin immediately) and write a detailed task breakdown for only that one. Defer the others until that POC is in progress or complete.

**Output:** Task list for chosen POC, with all dependencies/prerequisites explicit.

**Effort:** < 1 hour.

### Scope B: Environment + Dependency Audit
Before any POC code: verify HY-World-2.0 obtainability, StyleGaussian obtainability, COLMAP install path, splat renderer choice, hardware access topology, audio asset. Do not implement anything yet.

**Output:** Matrix of verified-vs-unverified inputs per POC. Explicit list of "these tools must be installed before we start."

**Effort:** 2-4 hours of research + tool testing.

### Scope C: Full Three-POC Task Graph
Decompose every POC into ordered tasks: folder scaffolds, dependency installs, pipeline scripts, viewer code, measurement harness, deliverable formatting. Operator picks where to start.

**Output:** Numbered task list across all POCs, with dependencies and sequencing rules.

**Effort:** 4-6 hours of planning + scaffolding.

---

## Questions Before R Proceeds

R will not write any files or code until operator clarifies:

1. **Which scope?** A, B, or C above? If A, which POC (1, 2, or 3)?

2. **Hardware access:**
   - Is the 4090 this Linux machine (`/home/julian/sambashare/expts/painttheworld`), or a separate host?
   - Is the M5/M5 Max a separate device you'll work on directly, or should R assume access via SSH/etc?
   - Can R assume it has direct shell access to the 4090 during these POCs, or will you run commands there manually?

3. **Tool verification:**
   - Has operator already confirmed HY-World-2.0 and StyleGaussian are obtainable? Or should R verify both are publicly released + documented?
   - Is COLMAP already installed on the 4090, or does R need to plan for installation?

4. **Renderer choice:**
   - Does operator have a preferred WebGL splat renderer library, or should R propose one based on performance/simplicity?

5. **Audio:**
   - Does operator have a specific 30-60 second song clip for POC 2, or should R assume any test audio works?

**R has written no new files and made no code changes. Awaiting direction.**
