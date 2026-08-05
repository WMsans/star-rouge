# Falling-sand architectures for a destructible 2D arena

Research findings for [Star Rouge #3](https://github.com/WMsans/star-rouge/issues/3). **Facts only — no architecture is chosen here.** Where sources disagree or a number does not exist in any source I could verify, that is stated explicitly rather than filled in.

- Scope constraints from the map: pure 2D (`Node2D`/`RigidBody2D`), Godot 4.7, Windows/Linux, 60fps on modest hardware, pixel art.
- Date of research: 2026-08-05. Godot version used for local measurements: **4.7.1.stable** (`godot --version` → `4.7.1.stable.arch_linux.a13da4feb`, built 2026-07-13).

Trust tiers used below: **[P]** primary (talk by the author, official docs, source code, shipped game data), **[M]** measured by me on this machine, **[S]** secondary (community wiki / third-party write-up), **[?]** unverified or disputed.

---

## 1. Noita / "Falling Everything" — what the authors actually said

Source **[P]**: Petri Purho, *Exploring the Tech and Design of Noita*, GDC 2019 — <https://www.youtube.com/watch?v=prXuyMCgbTc>. All quotes below are from the auto-transcript of that talk (retrieved 2026-08-05 with `yt-dlp --write-auto-subs`), so wording is approximate but the technical content is the speaker's own.

### The core cell rule is trivial

> "if there's a sand pixel here it's looking at the line below it if that's empty then move down and then if that's occupied it looks … to the down left and the right and it moves in that direction"

Water is "essentially the same algorithm" plus a left/right spread step at the end; gases are "just like inverse of this". Purho: **"this is like 95 percent of our tech is this."**

### Single-buffered, in-place, bottom-up

Asked directly about double buffering, Purho said the simulation is single-buffered and gave the reason **[P]**:

> "if you do a double buffer then you have to actually update everything right unless you sort of double buffer every junk separately … right now pixels can only occupy one place"

I.e. single-buffering is what makes the dirty-rect skip legal at all. A double-buffered CA has to write every output cell every step.

### Chunking, dirty rects, and the multithreading scheme

- World divided into **64 × 64 chunks** **[P]**.
- **Each chunk keeps a dirty rect** of what needs updating. Purho on its value even without threads: "even if you don't multithread this what this alone does is like it removes so many of the pixels that you have to test because very often the world ends up in some sort of semi-stable state" **[P]**.
- Threading is a **checkerboard in 4 update rounds**: gather work, dispatch every 4th chunk to a thread pool, wait, repeat 4× per frame **[P]**.
- The safety property that makes it correct on a shared single buffer: **"any pixel that's getting updated in this can go 32 pixels in any direction and it's guaranteed that another thread won't update it"** **[P]**. So the 64-chunk size and the 32-pixel maximum displacement per update are the same decision.
- Streaming (separate from simulation chunks): the world is kept in **512 × 512 areas**, **12 resident at a time**, furthest written to disk and read back on return **[P]**.

### Sand → rigid body geometry: a 3-stage pipeline

For rigid bodies, Noita uses **Box2D** integrated with the grid **[P]**. Per frame:

1. Take rigid-body pixels **out** of the world grid.
2. Step the falling-sand simulation once.
3. Step Box2D once.
4. Put the rigid-body pixels **back** at their new positions (each pixel knows its **UV coordinate inside its triangle** and which body it belongs to, which is how it finds its new grid position).
5. Any world pixel now occupied by a rigid-body pixel is **removed from the grid and handed to a separate velocity/gravity particle system**, then re-inserted into the grid when it next hits something. This is where splashes come from, and it is the same trick Purho wrote for *Bloody Zombies* (2008, a 256×256 game).

Geometry for a body is built as **marching squares → Ramer–Douglas–Peucker simplification → triangulation → Box2D** **[P]**. Two consequences the talk states outright:

- **"if one of these pixels gets destroyed you have to recalculate all of the marching squares and the box 2d body stuff"** — destruction cost is per-body re-meshing, not per-pixel.
- **Static world geometry only runs the first two stages** (marching squares + RDP) producing **hollow** bodies, and **only in places where rigid bodies are or will be** — "that saves you a lot" **[P]**.

### Numbers from shipped game data

Source **[S]** (community wiki documenting the shipped `magic_numbers` data — *not* first-party, but it is transcribed engine config, and it is the only place these values are written down): <https://noita.wiki.gg/wiki/Modding:_Magic_Numbers>

| Key | Value |
| --- | --- |
| `VIRTUAL_RESOLUTION_X` / `_Y` | **427 × 242** |
| `GRID_RENDER_TILE_SIZE` | 32 |
| `GRID_MAX_UPDATES_PER_FRAME` | **128** |
| `GRID_MIN_UPDATES_PER_FRAME` | **40** |
| `GRID_FLEXIBLE_MAX_UPDATES` | 0 (disabled) |
| `CELLFACTORY_CELLDATA_MAX_COUNT` | 1024 (max distinct materials) |
| `PARTICLE_EMITTER_MAX_PARTICLES` | 100000 |
| `BOX2D_THREAD_MAX_WAIT_IN_MS` | **12** |

Derived, arithmetic only (flagged as **derived**, not sourced):

- 427 × 242 = **103,334 cells on screen** at 1 sim cell = 1 game pixel.
- On-screen area covers roughly **7 × 4 ≈ 28** of the 64×64 chunks.
- `GRID_MAX_UPDATES_PER_FRAME` = 128 chunk-updates × 64² = **up to ~524,000 cells simulated per frame** as a hard cap, with a floor of 40 × 64² ≈ 164,000. Caveat: I could not verify from a primary source that a "grid update" is exactly one 64×64 chunk update rather than one chunk's dirty rect — treat the multiplication as an upper bound of that interpretation.
- `BOX2D_THREAD_MAX_WAIT_IN_MS` = 12 implies Box2D runs off the main thread with a wait cap of 12 ms against a 16.67 ms frame **[?]** — plausible reading of the name, not stated anywhere I found.

**Gaps I could not close on Noita:** the talk gives **no ms-per-frame figures, no total simulated cell count, no CPU spec, and no thread count**. The only performance statement in the talk is the design-level one: "is it running at 60 frames a second yes it's kind of done." Anyone quoting a "Noita simulates N pixels in X ms" number is not quoting this talk.

Disputed **[?]**: a Steam/forum claim that Noita's internal resolution is "480×270" conflicts with `VIRTUAL_RESOLUTION_X/Y = 427/242` above. The magic-numbers values are closer to primary (game data) and are also consistent with `config.xml`'s separate `internal_size_w` window setting being a different knob. Do not treat 480×270 as established.

---

## 2. CPU cellular automaton vs GPU compute — the real trade

### GPU: block CA with a Margolus neighborhood, and the one hard number that exists

Source **[P]** (author's own README + shader, Godot project): <https://github.com/Clocktown/Godot-Falling-Sand-Compute-Shader> — states **Godot 4.6**.

Design, verbatim from the README:

- Data lives in a **2-channel 8-bit UNORM texture**: red = sand present (0/1), green = "already fell last step" mask.
- Parallelised with a **Block Cellular Automaton / Margolus neighborhood**: the grid is cut into **independent 2×2 blocks**, and the block grid is **shifted by (1,1) on odd steps**. Each thread owns one 2×2 block, reads and writes its four pixels.
- A 2×2 block is a **4-bit value → 16 possible states → a branch-free integer transition table**. "No ping-pong buffering is needed and the amount of invocations are cut by a factor of four with no redundant neighborhood accesses between threads."
- One "whole" step = **two substeps** (shifted + unshifted), so sand falls one cell per step; `_physics_process` schedules two dispatches.
- Randomised transitions are needed because the block scheme otherwise produces "obvious patterns and gaps".

Performance, verbatim: at **4000×4000** (16,000,000 cells) — "On my GPU, this runs in **0.1-0.2ms per `_physics_process`** (i.e. for two substeps)", and at 16× playback speed on the same 4000×4000 grid it "requires **under one millisecond** for the compute." **The GPU model is not stated** ("my GPU"), so this number has no hardware attached to it — it establishes an order of magnitude, not a budget for modest hardware.

Limits of that project as evidence: **one material (sand) only**, no liquids/gases/heat, **no rigid bodies and no collision generation at all** ("the bottom border acts as a wall, while left/right/top borders are the void"), and it profiles via `RenderingDevice.capture_timestamp()` using an undocumented naming convention to surface entries in the visual profiler.

Second GPU data point **[P]**: <https://github.com/ARez2/sandengine> — also Margolus 2×2 with offset shifting, compute shaders, **requires OpenGL 4.3**. It is a **Rust/OpenGL standalone application, not a Godot project** (a search result described it as Godot; the README does not). No grid size and no timings are published; "performance optimizations (profiling, work group size, dispatch size etc.)" is still on its TODO. Its planned physics section states the constraint plainly: **"only static, non moving materials can be part of a rigidbody."**

### What the GPU path costs you, from Godot's own docs **[P]**

<https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html>

- "Compute shaders can only be used from RenderingDevice-based renderers (the **Forward+ or Mobile** renderer)" — **not** the Compatibility renderer. (`project.godot` already declares `Forward Plus`, so this is satisfied today.)
- "Ideally, you would not call `sync()` to synchronize the RenderingDevice right away as it will cause the **CPU to wait for the GPU** to finish working." This is the crux for sand↔rigidbody coupling: any CPU-side consumer of the sand state (collision polygons, damage queries, gameplay logic) needs a readback, and a same-frame readback is a stall.
- "Long computations can cause Windows graphics drivers to 'crash' due to **TDR** being triggered … you may need to split it into multiple dispatches."
- "Local RenderingDevices cannot be debugged using tools such as RenderDoc."

**Not measured here:** I could not get any GPU number on this machine. `RenderingServer.create_local_rendering_device()` returned **null** under both `--headless` and `--display-driver headless --rendering-driver vulkan` in Godot 4.7.1, so no compute-shader benchmark was run. Treat all GPU throughput figures in this document as other people's hardware.

### CPU in Godot: measured throughput **[M]**

All figures below are **mine, measured on this machine**, not from any source. Method: a throwaway Godot 4.7.1 project run with `godot --headless --script`, GDScript, grid as a flat `PackedByteArray` (1 byte per cell), single-buffered bottom-up scan, Noita's exact rule (down, then down-left/down-right). Timings via `Time.get_ticks_usec()`, 20 iterations, grid re-seeded each iteration so every cell is active. Appendix A has the kernel.

Hardware: **Intel Core i7-13620H** (16 logical CPUs), Mesa Intel Graphics (RPL-P) iGPU, Linux. This is a mid-range 2023 laptop CPU — arguably close to the "modest hardware" target, but a *laptop* one; it is not a floor.

**Single-threaded full-grid step (worst case: every cell moving), GDScript:**

| Grid | Cells | avg ms | worst ms | Mcell/s |
| --- | --- | --- | --- | --- |
| 64 × 64 | 4,096 | 0.238 | 0.256 | 17.2 |
| 128 × 128 | 16,384 | 0.959 | 1.052 | 17.1 |
| 320 × 180 | 57,600 | 3.385 | 3.522 | 17.0 |
| 256 × 256 | 65,536 | 3.944 | 4.633 | 16.6 |
| 640 × 360 | 230,400 | 13.501 | 13.922 | 17.1 |
| 512 × 512 | 262,144 | 15.400 | 15.760 | 17.0 |
| 1280 × 720 | 921,600 | 54.126 | 55.146 | 54 ms — 3.2× over budget |

The headline: **GDScript sustains ~17 million cell-visits per second** on this CPU, flat across grid sizes (no cache cliff at these sizes). At a 16.67 ms frame that is **~283,000 cell-visits per frame for the whole frame budget**, and realistically a fraction of that — a 10 % budget (1.67 ms) buys **~28,000 cell-visits per frame**, i.e. roughly a **170 × 170** fully-active region, single-threaded, with nothing left for rendering, physics or gameplay.

**Counter-intuitive result worth carrying forward: settled sand is more expensive than falling sand.**

| Grid | Full-grid step (all falling) | Settled grid scan (nothing moves) |
| --- | --- | --- |
| 256 × 256 | 3.94 ms | **7.98 ms** |
| 640 × 360 | 13.50 ms | **28.01 ms** |
| 1280 × 720 | 54.13 ms | **112.12 ms** |

Reason: a moving cell exits on the first `below == EMPTY` test, while a settled cell pays all three failed neighbour tests. So a static, "quiet" arena is the *expensive* case for a naive full scan — which is exactly the case dirty rects eliminate, and it is why Purho's "even if you don't multithread this" remark is load-bearing rather than an aside.

**Noita's 4-pass checkerboard over 64×64 chunks, reimplemented in GDScript with `WorkerThreadPool`** **[M]**: 640 × 384 grid (60 chunks, 15 per pass), half filled with sand, 10 frames, tasks mutating one shared `PackedByteArray`:

- single-threaded, same chunk decomposition: **11.252 ms/frame**
- `WorkerThreadPool.add_group_task` + `wait_for_group_task_completion`, 4 passes/frame: **2.188 ms/frame**
- **speedup 5.14×** on 16 logical cores; sand count conserved exactly (122,880 before and after, both paths).

Caveats on that measurement, stated so it is not over-read: mass conservation over 10 frames is evidence, not proof, of thread-safety; sharing a `PackedByteArray` across group tasks worked here but Godot documents no guarantee for it; and the four `wait_for_group_task_completion` barriers per frame are pure overhead that grows as chunk count shrinks. 5.14× on 16 cores ≈ 32 % scaling efficiency.

### Language choice, per Godot's own FAQ **[P]**

<https://docs.godotengine.org/en/stable/about/faq.html>

- "In most games, the *scripting language* itself is not the cause of performance problems."
- C# is faster for pure language operations, but incurs **marshalling overhead** on Godot API calls, and its **garbage collection can cause unpredictable stuttering**.
- C++ via GDExtension "**will almost always be faster** than either C# or GDScript", at the cost of dev time.
- "All languages supported by Godot are fast enough for general-purpose scripting."

**No official benchmark numbers exist** for GDScript vs C# vs GDExtension on tight numeric loops — I looked, and the third-party comparisons I found are blog posts without published methodology or hardware, so nothing quantitative is reported here. The only hard numbers in this document for the CPU path are my ~17 Mcell/s GDScript measurements above; the C#/GDExtension multiples over that are **unmeasured**.

Prior art for the GDExtension route **[P]**: <https://github.com/kiwijuice56/sand-slide> — a shipped falling-sand game whose "simulation logic is implemented in C++ as a GDextension", UI in Godot 4.3. Reading `extension/sand_simulation.cpp`:

- Grid is a flat `std::vector<int> cells` indexed `row * width + col`; element IDs 0–4096 (arrays sized 4097), IDs > 4096 are indestructible "taps".
- **Chunked by occupancy counter, not dirty rects**: a `chunks` vector counts non-empty cells per chunk and empty chunks are skipped wholesale (`if (chunks[chunk] == 0) { continue; }`). Chunk size is configurable (`set_chunk_size`).
- Update order: chunks back-to-front, rows bottom-to-top, columns left-to-right. Two PRNGs — a global `randf()` and a position-hashed `randf_loc(row, col)` for deterministic per-cell noise.
- Rendering: `get_color_image()` walks every pixel and packs RGB into a `PackedByteArray` each frame.
- **Single-threaded**; no published grid size or timings.

---

## 3. Sand → collision geometry in Godot specifically

### Marching squares is in the engine already **[P]**

<https://docs.godotengine.org/en/stable/classes/class_bitmap.html>

`BitMap.opaque_to_polygons(rect, epsilon)`: "Creates an Array of polygons covering a rectangular portion of the bitmap. It uses a **marching squares algorithm, followed by Ramer-Douglas-Peucker (RDP) reduction** of the number of vertices." That is stages 1–2 of Noita's pipeline, built in, with `epsilon` as the simplification knob. Companions: `BitMap.create_from_image_alpha(image, threshold)` and `BitMap.grow_mask(pixels, rect)` (morphological dilation for positive values, erosion for negative — useful for closing single-cell noise before meshing).

**Measured cost of that call** **[M]** (same machine/method; blobby `FastNoiseLite` terrain, `frequency = 0.03`, ~50 % solid):

| Bitmap | Solid cells | epsilon | Polygons | Vertices | ms |
| --- | --- | --- | --- | --- | --- |
| 128 × 128 | 8,520 | 0.0 | 20 | 1,332 | 0.746 |
| 128 × 128 | 8,520 | 1.0 | 20 | 663 | 0.609 |
| 128 × 128 | 8,520 | 2.0 | 19 | 392 | 0.552 |
| 256 × 256 | 33,131 | 0.0 | 83 | 4,982 | 2.722 |
| 256 × 256 | 33,131 | 1.0 | 83 | 2,517 | 2.386 |
| 256 × 256 | 33,131 | 2.0 | 80 | 1,513 | 2.219 |
| 512 × 512 | 131,308 | 0.0 | 319 | 19,374 | 10.822 |
| 512 × 512 | 131,308 | 1.0 | 319 | 9,745 | 9.628 |
| 512 × 512 | 131,308 | 2.0 | 308 | 6,025 | 9.074 |

Readings: cost is **~linear in bitmap area (~0.04 µs/cell)** and dominated by the marching-squares pass, not by RDP — `epsilon` 0→2 cuts vertices **3.2×** but time only **~16 %**. A whole-arena re-mesh at 512×512 costs **~10 ms**, i.e. 60 % of a frame; at 128×128 it is ~0.6 ms. This is the number that makes Noita's "only build static geometry where rigid bodies are or will be" restriction, and its per-body re-mesh on destruction, look like necessity rather than polish.

### The shape you get is the slow one **[P]**

<https://docs.godotengine.org/en/stable/classes/class_concavepolygonshape2d.html>

- "A 2D polyline shape … Used internally in `CollisionPolygon2D` when it's in `BUILD_SEGMENTS` mode."
- "intended to work with **static** `CollisionShape2D` nodes like `StaticBody2D` and will **likely not behave well for `CharacterBody2D`s or `RigidBody2D`s in a mode other than Static**."
- It is **hollow**: "Physics bodies that are small have a chance to **clip through this shape when moving fast**" — it cannot detect collision from the inside.
- "`ConcavePolygonShape2D` is the **slowest 2D collision shape** to check collisions against. Its use should generally be limited to level geometry."

So Noita's "hollow bodies" for static world geometry has a direct Godot analogue **with documented tunnelling and cost caveats**, and for anything that is a moving `RigidBody2D` the docs point at `CollisionPolygon2D` in `BUILD_SOLIDS` mode, which decomposes into `ConvexPolygonShape2D`s — i.e. Noita's stage 3 (triangulation/convex decomposition) is not optional if the sand-derived body moves.

### Physics budget facts **[P]**, read out of Godot 4.7.1 itself

Queried via `ProjectSettings.get_setting()` on a bare 4.7.1 project:

```
physics/common/physics_ticks_per_second = 60      → 16.67 ms per physics tick
physics/common/max_physics_steps_per_frame = 8
physics/common/physics_jitter_fix = 0.5
physics/common/physics_interpolation = false
physics/2d/run_on_separate_thread = false
physics/2d/physics_engine = DEFAULT
Engine.physics_ticks_per_second = 60
```

Notes: the 60 Hz tick is the frame budget for anything driven from `_physics_process`; 2D physics is **on the main thread by default** (`run_on_separate_thread = false`) — contrast Noita's `BOX2D_THREAD_MAX_WAIT_IN_MS = 12`, which suggests it does not run physics on the main thread; and 2D physics interpolation exists in 4.7 but is **off by default**, which matters if the sim runs at a different rate from rendering.

### Third-party 2D physics backends, if Box2D parity is ever wanted **[P]**

- <https://github.com/appsinacup/godot-box2d> — GDExtension physics server replacing Godot's 2D physics, Box2D **v2.4.1**, badges target Godot **4.2**. Documented limitations: no thread-safety, no double-precision builds, **no cross-platform determinism**, "polygons have a small skin, which can result in differences from Godot Physics". **The README states the project is unmaintained** and points users at Rapier instead.
- <https://github.com/appsinacup/godot-rapier-physics> — Rapier + Salva fluids, badge says **Godot 4.7** (Rust bindings note 4.4–4.7), 2D described as "pretty stable, though there are some issues", installs as a physics-server replacement. Limitations listed: "no support for asymmetric collisions", 3D incomplete, double builds need manual compilation. Offers locally-deterministic and cross-platform-deterministic variants. **No benchmarks published.**

Neither publishes performance numbers, so any claim that swapping the 2D physics server helps or hurts a sand game is currently **unsupported by data**.

### Third route: no collision geometry at all

The Noita talk documents a mechanism that sidesteps meshing entirely for the fast-moving minority of cells: pull a cell **out** of the grid into a **separate velocity+gravity particle simulation**, and put it back into the grid when it next hits something **[P]**. Godot's equivalent for the visual-only variant of this is GPU particles colliding against a **real-time signed distance field**: `LightOccluder2D.sdf_collision` — "If enabled, the occluder will be part of a real-time generated signed distance field that can be used in custom shaders" **[P]** (<https://docs.godotengine.org/en/stable/classes/class_lightoccluder2d.html>). I did **not** find documentation stating that this SDF is usable for CPU-side gameplay queries, nor numbers for its generation cost, so the SDF option is recorded here as **existing but unquantified**; the `particle_systems_2d` tutorial page does not cover it.

Signed-distance-field-based sand→collision, as a *general* architecture, produced **no primary source with numbers** in this search. Recorded as a gap.

---

## 4. Rendering the sand

- The cheap path is one texture, not one node per cell: `ImageTexture.update()` — "Replaces the texture's data with a new Image … faster than allocating additional memory for a new texture each time", with the restriction that "the new image dimensions, format, and mipmaps configuration should match the existing texture's image configuration" **[P]** (<https://docs.godotengine.org/en/stable/classes/class_imagetexture.html>). Both prior-art projects do exactly this: sand-slide packs a `PackedByteArray` per frame; Clocktown's GPU version displays a 2-channel texture on a `TextureRect` with a custom shader reading only the red channel (Godot's handling of 2-channel textures "messes with the visuals" otherwise) **[P]**.
- **The Godot docs contain no performance warning about `Image.set_pixel()`/`get_pixel()`** — I checked both the class reference and `doc/classes/Image.xml` in the engine source. The common advice to avoid it is not documented; the docs only note that "depending on the image's format, the color set here may be clamped or lose precision."
- Measured, GDScript, filling an image every frame **[M]**:

| Image | Pixels | `set_pixel` loop | `PackedByteArray` fill + `create_from_data` |
| --- | --- | --- | --- |
| 256 × 256 | 65,536 | **1.587 ms** | 3.327 ms |
| 640 × 360 | 230,400 | **5.465 ms** | 11.564 ms |
| 1280 × 720 | 921,600 | **21.918 ms** | 46.218 ms |

  Note the direction: in **GDScript**, `Image.set_pixel()` was **~2× faster** than assembling the frame byte-by-byte in a `PackedByteArray`, because the byte path pays four indexed GDScript writes per pixel instead of one call. That inverts the usual folklore, and it is a GDScript-specific result — in C++/GDExtension the byte path has no such per-element overhead (that is what sand-slide does), so **do not carry this table over to a compiled implementation**.
- `ImageTexture.update()` measured **0.000 ms** under `--headless`. That is the **dummy rendering driver, not a real number** — texture upload cost was not measured, and no GPU-side figure in this document was measured locally.

---

## 5. Explicit gaps and disagreements

1. **No published Noita frame budget.** No ms figures, no cell totals, no hardware in the GDC talk. The `GRID_*_UPDATES_PER_FRAME` values (40–128) are the closest thing to a budget and they come from a **community wiki**, not first-party.
2. **What one "grid update" means in Noita is unconfirmed** — the 128 × 64² ≈ 524k cells/frame figure above is my arithmetic on an assumed interpretation.
3. **480×270 vs 427×242** for Noita's internal resolution: sources conflict; the game-data value (427×242) is better sourced.
4. **No GPU number measured here at all**; the only GPU figure cited (0.1–0.2 ms for 16 M cells) is on an **unidentified GPU**, for **one material with no rigid bodies**.
5. **No official or credible third-party GDScript vs C# vs GDExtension benchmark** for tight cell loops. The gap between my 17 Mcell/s GDScript figure and a compiled implementation is unquantified.
6. **SDF-based sand→collision has no sourced numbers.** Godot's 2D SDF is documented as a shader input, not a physics/gameplay query surface.
7. **Rigid-body coupling on the GPU path is unsolved in every project examined.** sandengine states outright that "only static, non moving materials can be part of a rigidbody"; Clocktown's has no collision at all. Nobody in this search published a working GPU-sim ↔ CPU-rigidbody scheme with timings.
8. **My measurements are single-machine, GDScript, synthetic.** An i7-13620H is not the modest-hardware floor, `--headless` excludes all rendering contention, and the synthetic grids are 100 % active — the opposite of what dirty rects are for. Treat them as a ceiling on throughput and a floor on frame cost, not as a budget.

---

## Appendix A — the measured kernel (for reproduction)

GDScript, Godot 4.7.1, `godot --headless --script`. Grid is `PackedByteArray`, `0 = EMPTY`, `1 = SAND`, single buffer, bottom-up:

```gdscript
static func sand_step(g: PackedByteArray, w: int, h: int, rng_bit: int) -> int:
	var moved := 0
	var y := h - 2
	while y >= 0:
		var row := y * w
		var below := row + w
		var x := 0
		while x < w:
			var i := row + x
			if g[i] == SAND:
				if g[below + x] == EMPTY:
					g[below + x] = SAND
					g[i] = EMPTY
					moved += 1
				else:
					var dir := 1 if ((x + y + rng_bit) & 1) == 0 else -1
					var nx := x + dir
					if nx >= 0 and nx < w and g[below + nx] == EMPTY:
						g[below + nx] = SAND
						g[i] = EMPTY
						moved += 1
					else:
						nx = x - dir
						if nx >= 0 and nx < w and g[below + nx] == EMPTY:
							g[below + nx] = SAND
							g[i] = EMPTY
							moved += 1
			x += 1
		y -= 1
	return moved
```

The threading benchmark builds 64×64 chunk origins, buckets them into 4 lists by `(cy % 2) * 2 + (cx % 2)`, and runs `WorkerThreadPool.add_group_task(step_chunk, n, -1, true)` + `wait_for_group_task_completion()` once per bucket per frame, all tasks mutating one member `PackedByteArray`. Benchmark scripts were throwaway and are not committed.

## Appendix B — sources

Primary:
- Petri Purho, *Exploring the Tech and Design of 'Noita'*, GDC 2019 — <https://www.youtube.com/watch?v=prXuyMCgbTc> (GDC Vault: <https://www.gdcvault.com/play/1025695/Exploring-the-Tech-and-Design>)
- Godot docs: [compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html), [FAQ / language choice](https://docs.godotengine.org/en/stable/about/faq.html), [BitMap](https://docs.godotengine.org/en/stable/classes/class_bitmap.html), [ConcavePolygonShape2D](https://docs.godotengine.org/en/stable/classes/class_concavepolygonshape2d.html), [ImageTexture](https://docs.godotengine.org/en/stable/classes/class_imagetexture.html), [Image](https://docs.godotengine.org/en/stable/classes/class_image.html), [LightOccluder2D](https://docs.godotengine.org/en/stable/classes/class_lightoccluder2d.html)
- Godot engine source: `doc/classes/Image.xml`
- <https://github.com/Clocktown/Godot-Falling-Sand-Compute-Shader> (README + project, Godot 4.6)
- <https://github.com/kiwijuice56/sand-slide> (`extension/sand_simulation.cpp`, C++ GDExtension)
- <https://github.com/ARez2/sandengine> (README; Rust/OpenGL, not Godot)
- <https://github.com/appsinacup/godot-box2d>, <https://github.com/appsinacup/godot-rapier-physics>

Secondary / lower trust, used only where flagged:
- <https://noita.wiki.gg/wiki/Modding:_Magic_Numbers> (transcribed shipped game data)
- <https://blog.macuyiko.com/post/2020/an-exploration-of-cellular-automata-and-graph-based-game-systems-part-4.html> (independent write-up of the same talk; corroborates single-buffering, checkerboard threading, and the 32-pixel guarantee)
- <https://jason.today/falling-sand> (p5.js tutorial implementation; flat 1D array, reverse iteration, no timings or grid sizes published — no numbers taken from it)
