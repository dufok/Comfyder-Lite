# Comfyder

**AI rendering for Blender via ComfyUI + fal.ai — from one-button refine to a full zone-based pipeline.**

> ⚠️ **Requires: a running [ComfyUI](https://github.com/comfyanonymous/ComfyUI) server + a [fal.ai](https://fal.ai) API key.**
> ComfyUI is only the orchestrator (CPU-only is fine) — all generation runs on FAL as paid API calls (typically a few cents per image).

Two add-ons in this repo:

| Add-on | What it does |
|---|---|
| **Comfyder Lite** | One button: render → mood prompt → refined AI render back in Blender |
| **Comfyder Pro** | The full pipeline: per-material zones with masks, engines and sliders — blocking in, art-directed board out |

## Requirements

| What | Why |
|---|---|
| **ComfyUI** reachable over HTTP (localhost or LAN) | Orchestrator. Never samples locally — a CPU box is fine |
| **[gokayfem/ComfyUI-fal-API](https://github.com/gokayfem/ComfyUI-fal-API)** custom nodes | `FluxGeneral_fal`, `FluxPro1Fill_fal`, `FluxProKontext_fal`, `VLM_fal`, … |
| **ComfyUI-FAL** (Image Edit pack) custom nodes | `FalGeminiFlashEdit`, `FalQwenImageEditInpaint`, `FalZImageTurboInpaint`, … |
| **`FAL_KEY`** env var in the ComfyUI process | All generation is billed per call on [fal.ai](https://fal.ai) |
| **Blender 5.0+** | Both add-ons use the 5.x compositor API |

## Install

Grab a zip from [`dist/`](dist/) and either:

- **drag & drop** the zip into the Blender window → Install → enable, or
- `Edit → Preferences → Get Extensions → ⌄ (top-right) → Install from Disk…` → pick the zip.

Classic single-file install also works: `Preferences → Add-ons → Install from Disk…` → `addon/comfyder_*.py`.

Then set your ComfyUI address in the panel.

---

## Comfyder Lite

Refine any render (or a viewport snapshot) with a single prompt.

**Use:** press **F12** → in the render window press **N** → **Comfyder** tab → type a mood prompt (any language) → **Generate**. ~30–90 s later the result loads as a new image *"Comfyder Result"*.

**Chain:** frame → `VLM_fal` (Gemini vision describes the scene, merges your mood prompt) → `FalGeminiFlashEdit` (edits the frame; optional depth map as a geometry reference) → back to Blender. Background polling — the UI never freezes.

| Setting | Default | Notes |
|---|---|---|
| Prompt (✏ wide editor) | — | Mood/style instruction |
| VLM scene description | on | Better scene fidelity |
| Source: Render / Viewport | Render | Viewport = OpenGL snapshot, quick drafts without F12 |
| 1K / 2K / 4K | 2K | Output resolution |
| Attach depth + Auto-render depth | off / on | Depth locks geometry hard; auto mode renders a fresh Z-pass map itself (your compositor is restored) |
| Seed | 7 | FAL backends are not fully deterministic — keep results you like |

---

## Comfyder Pro

The zone pipeline: assign placeholder `mat_*` materials to objects, describe each zone, press Generate. Every zone gets its own AI pass through its own Cryptomatte mask.

**Chain:** pass pack (depth + per-zone masks, rendered by the add-on) → global pass (Flux + depth ControlNet — *this pass sets the colors of the whole image*) → sequential zone passes → final refine (Gemini: frame + depth + mood + auto-protect list). Every intermediate step is saved to the results folder.

**The three-prompt pyramid:**

1. **Zone prompts** — what each material is (`old red brick wall…`, `ring of clear water…`).
2. **Scene prompt** — the global pass. Build it from zones with one click, then append environment/atmosphere at the end. Colors are decided HERE.
3. **Mood** (final pass) — quality/light/haze/DoF. Do not describe materials here: zones marked *Protect in final* are appended automatically as "Keep exactly: …".

**Zone engines** (each zone picks its own):

| Engine | Best for | Notes |
|---|---|---|
| Fill — repaint | color flips, thin structures, flowers | pixel-exact to the mask |
| Qwen — texture | refining texture on large zones | `strength` + negative prompt; resamples the frame — not for thin things |
| Gemini — wide zone | walls, water, backgrounds | smart 2K edit composited back by mask |
| Kontext — structure | keep exact shapes | tends to "lacquer" organics |
| Z-Turbo — draft | cheap quick checks | |

**Per-zone mask sliders:** `Dilate` / `Blur` (defaults 6/4; use 15–25/15+ for flowers and glow so petals are not clipped by the silhouette).

**Workflow tips (battle-tested):**

- List order = pass order: large zones first, small last.
- A telephoto lens from far away gives a flat depth map — the panel warns you; 24mm closer works much better.
- Resolution is auto-capped to 1536 px / multiple of 16 (FluxGeneral limit).
- Fix the seed and iterate a single zone — re-running one zone costs cents.
- FAL is not deterministic even with a fixed seed: when you like a result, keep it and iterate *from* it rather than re-running the whole chain.

---

## Also in this repo

- `blender/setup_passes.py` — standalone pass-pack renderer (depth + Cryptomatte masks)
- `driver/comfy_graph_v2.py` + `driver/run.py` + `driver/materials.yaml` — the same pipeline as a scriptable CLI driver
- `docs/Blocking2Render.md` — field notes the add-ons are built on (20 rules: mask sizes, engine choice, aspect-ratio traps, pinning)

## Roadmap

- Light zones: volumetric beams outside the depth map + procedural overlay with an intensity slider
- Reference swatches per zone (NanoBanana / Kontext Multi)
- Result history with **Pin** — iterate from any kept frame
- "Open in ComfyUI" — inspect the exact executed graph

## License

[MIT](LICENSE) © 2026 Stepan Vladovskiy
