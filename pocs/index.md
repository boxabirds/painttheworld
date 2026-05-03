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

## Verified Answers to Pre-Implementation Questions

### Verified: Tools & Code Availability

**1. HY-World-2.0 — AVAILABLE**
- Repository: https://github.com/Tencent-Hunyuan/HY-World-2.0
- License: Tencent-HY-World-2.0-Community License
- Weights: On Hugging Face (tencent/HY-World-2.0), auto-downloaded on first run
- Installation: `git clone` + `pip install` from repo
- Output: Mesh, 3DGS, point clouds directly usable
- Status: Full public release, code + weights both available

**2. StyleGaussian — AVAILABLE**
- Repository: https://github.com/Kunhao-Liu/StyleGaussian
- Paper: SIGGRAPH Asia 2024
- License: **Unspecified** (no LICENSE file; assume research-use-only or contact author before commercial use)
- Code: Full PyTorch implementation available
- Weights: Train from scratch or contact author (no pre-trained weights documented)
- Status: Research code, maintained, but license unclear

**3. COLMAP — AVAILABLE (Open Source)**
- Repository: https://github.com/colmap/colmap
- License: BSD
- Installation: Homebrew (macOS), conda, system packages, Docker, or from source
- Binaries: https://demuc.de/colmap/
- Python bindings: PyPI package `pycolmap`
- Status: Actively maintained, all platforms supported

**4. WebGL Splat Renderers — Multiple Options Available**

| Library | License | Status | Best For |
|---------|---------|--------|----------|
| **gsplat.js** (Hugging Face) | MIT | Active (v1.2.9) | General-purpose, well-maintained |
| **GaussianSplats3D** (mkkellogg) | Non-commercial | Active (2025) | Three.js integration, mature |
| **antimatter15/splat** | Unspecified | Active (2025) | Zero dependencies, minimal code |
| **Cesium** | Apache 2.0 | New (2025) | Geographic scale, LOD streaming, overkill for POCs |

**Recommendation for POC 2:** Use `gsplat.js` (MIT, active, no dependency constraints) or `GaussianSplats3D` (mature, three.js ecosystem).

---

### Still Unverified: Hardware & User Preferences

These require operator to answer (R cannot determine):

1. **Hardware topology:**
   - Is the 4090 this Linux machine, or a separate host?
   - Is the M5/M5 Max a separate device, or accessible via SSH?
   - Should R assume direct shell access to the 4090, or will you run commands manually?

2. **Audio for POC 2:**
   - Do you have a specific song clip, or should R use any test audio?

3. **Asset hosting:**
   - Where should splat files be served from? (local dev, GitHub releases, cloud storage, etc.)

4. **Renderer preference:**
   - `gsplat.js` (recommended: MIT, active) or `GaussianSplats3D` (alternative: mature three.js)?

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

---

## Getting Started: POC 1 on 4090

### Prerequisites Check

Before starting, verify on your 4090:

```bash
# Check CUDA/PyTorch
python3 -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}'); print(f'GPU: {torch.cuda.get_device_name(0)}')"

# Check git
git --version

# Check Python version (3.8+)
python3 --version
```

### Step 1: Set Up HY-World-2.0

```bash
cd /home/julian/sambashare/expts/painttheworld
mkdir -p experiments/poc1-static-worlds
cd experiments/poc1-static-worlds

# Clone HY-World-2.0
git clone https://github.com/Tencent-Hunyuan/HY-World-2.0
cd HY-World-2.0

# Create environment
conda env create -f environment.yml
conda activate hy-world-2.0

# Install dependencies
pip install -r requirements.txt
```

### Step 2: Generate First Splat Scene

```bash
# Inside HY-World-2.0 repo
python scripts/inference.py \
  --prompt "a vibrant forest with sunlight filtering through trees" \
  --output_dir ../output_scenes/forest_v1 \
  --format 3dgs  # Outputs .ply Gaussian splat files
```

**Expected output:** Directory `output_scenes/forest_v1/` containing:
- `splats.ply` (3D Gaussian splat file, ~50-500MB depending on complexity)
- `metadata.json` (scene info)
- Preview images

**Time estimate:** 10-30 minutes depending on model size and GPU load

### Step 3: Set Up StyleGaussian

```bash
cd ..
git clone https://github.com/Kunhao-Liu/StyleGaussian
cd StyleGaussian

# Create environment
conda env create -f environment.yml
conda activate stylegaussian

pip install -r requirements.txt
```

### Step 4: Create Style Variants

StyleGaussian requires:
1. Base splat scene (.ply from HY-World-2.0)
2. Style reference image (a photo, painting, or artwork defining the visual style)

```bash
# Prepare style images
mkdir ../style_references
# Place image files here: marshmallow.jpg, diamond.jpg, kitten.jpg, etc.

# Run style transfer
python style_transfer.py \
  --splat_path ../output_scenes/forest_v1/splats.ply \
  --style_image ../style_references/marshmallow.jpg \
  --output_path ../output_scenes/forest_v1/forest_marshmallow.ply
```

**Expected output:** Stylized splat file `forest_marshmallow.ply` with texture applied

**Time estimate:** 5-15 minutes per style variant

### Step 5: Document Outputs

After each successful run, record in `pocs/poc1-static-worlds/results.md`:

```markdown
## POC 1 Results

### Scene 1: Forest
- **Base scene generation time:** XX minutes
- **Base file size:** XXX MB
- **Quality assessment:** [notes on visual output]

### Style Variants
| Style | Generation Time | File Size | Visual Result |
|-------|-----------------|-----------|---------------|
| Marshmallow | X min | XXX MB | [notes] |
| Diamond | X min | XXX MB | [notes] |
| Kitten | X min | XXX MB | [notes] |

### Observations
- [What worked well]
- [What was unexpectedly slow/fast]
- [Quality surprises (better/worse than expected)]
- [Blockers encountered]
```

### Step 6: Prepare for POC 2

Once you have 1-2 styled scenes:

1. Copy best `.ply` files to: `pocs/poc1-static-worlds/splat_assets/`
2. Run on M5 for POC 2 web viewer:
   ```bash
   # Transfer via GitHub or direct copy
   scp -r pocs/poc1-static-worlds/splat_assets/ <m5>:~/expts/painttheworld/
   ```

---

## Potential Blockers & Solutions

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError` in HY-World-2.0 | Ensure conda env activated: `conda activate hy-world-2.0` |
| Out of memory (OOM) on 4090 | Reduce model precision or batch size; see HY-World-2.0 docs for quantization |
| StyleGaussian training diverges | Check style image size (recommend 512x512); try different learning rate in config |
| `.ply` files too large (>1GB) | Use ply compression or decimation tools; document file sizes for web feasibility |
| CUDA/PyTorch version mismatch | Use `conda` environments to pin versions; don't mix pip/conda |

---

## Success Criteria for POC 1

When done, you should have:

1. ✓ At least 1 base splat scene generated from text prompt
2. ✓ 3+ style variants per scene created
3. ✓ All outputs documented with timing and file sizes
4. ✓ Quality assessment (are the splats recognizable? Are styles visually distinct?)
5. ✓ Splat files ready to transfer to M5 for web viewer (POC 2)

Commit results to `pocs/poc1-static-worlds/` folder with:
- `.ply` files
- `results.md` (timing, sizes, quality notes)
- Any failure logs or error messages (for debugging later)

**Then:** Move to POC 2 on M5 to build the interactive viewer that loads these splats.
