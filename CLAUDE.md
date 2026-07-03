# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **prompt-engineering toolkit** (no application code) that turns reference images into on-style 2D UI/UX mobile-game assets. The deliverable is `studio_primer.md` — a self-contained mega-prompt the user pastes into a vision-capable LLM chat, which then answers short commands (`STYLE`, `UPDATE:`, `ASSET:`, `EXTRACT:`, `UPSCALE`, `CHECK`, `REGEN`, `TWEAK`) to produce a `style_guide.yaml` and, from it, image prompts (plus cutout/upscale image-edit prompts). The primer only ever outputs **text** — it never invokes an image tool even when the host chat has one.

**Generator-neutral by design**: the toolkit stops at the prompt. It never generates images and must not be coupled to any specific image generator (gpt-image / Gemini "Nano Banana" / Midjourney are the user's choice). Scope is 2D UI/UX assets (screens, icons, buttons, panels) — characters/sprites are out of scope. Repo content stays in English (prompts target English-language generators).

## The one rule that matters: the primer is GENERATED

`studio_primer.md` is bundled from two source directories — never hand-edit it independently of them:

- `schema/` — field + enum definitions. `style_guide.schema.yaml` is canonical; `asset_spec.schema.yaml` / `layout_spec.schema.yaml` describe the optional structured asset/layout inputs; `extract_spec.schema.yaml` / `upscale_spec.schema.yaml` describe the optional structured EXTRACT/UPSCALE inputs (which keep the source art as-is and never use the style_guide).
- `style_tokens/` — the style dictionary mapping each enum to an English prompt phrase (`materials`, `render_shape`, `light_color`, `layout_negative`).

Change flow: edit the source file first → rebuild the affected primer section → verify against the **BUILD MANIFEST** in the primer's header comment (it lists every source section that must be present; grep each item). The schemas are documentation-style pseudo-YAML (fields + enums, not JSON Schema); there is no build/validator script — the repo intentionally stays code-free.

## Architecture (pipeline inside the primer)

STYLE ref image(s) → **§3 ANALYZER** → `style_guide.yaml` (enums from §1 only, plus per-dimension `confidence`; `<0.75` = user must review; hex values are estimates the user fixes via eyedropper + `UPDATE:`) → **§4 SYNTHESIZER** (translates every enum through the §2 dictionary, in the fixed prompt order ending with "Avoid:") → one natural-language prompt → user generates the image elsewhere → optional **§7 CHECKER** compares the generated image back against the style guide and emits ready-made `TWEAK:` lines.

Two side pipelines operate on an EXISTING image and deliberately BYPASS the style_guide/§2 (they keep the source art unchanged, only emitting an image-edit prompt): **§5 EXTRACTOR** (`EXTRACT`) isolates an item onto a transparent background (a cutout), and **§6 UPSCALER** (`UPSCALE`) enlarges + sharpens a generated asset. Both note that they need an image-*editing* generator and that the primer still only outputs text.

Distinctions that must not be blurred:

- **STYLE ref** ("how it looks" → style_guide) vs **TARGET ref** ("what to make / its layout" → asset content). Two different inputs to two different commands.
- `style_guide.background` (overall style background) vs `asset_spec.background` (popup/fullscreen/transparent/scene for one asset).
- Negative tails: `negative_tail_always` is appended to every prompt; `negative_tail_non_screen` ("no text, no UI overlays") only to single assets — screens ARE UI, so it must be skipped there.

## Consistency invariants (check after any change)

- Every enum in `schema/style_guide.schema.yaml` appears in primer §1; every token group in `style_tokens/*.yaml` appears in primer §2 (currently 24 `###` headings).
- The primer §0 command table and README's "Command reference" table must match 1:1.
- `examples/settings_screen/` is the end-to-end proof: `prompt.txt` must remain derivable from its three spec files under the current §4 rules. `examples/analyzer_smoke_test/` is a captured analyzer artifact — leave as-is.

## Verification commands

```bash
# all YAML parses
python3 -c "import yaml, glob; [yaml.safe_load(open(f)) for f in glob.glob('schema/*.yaml') + glob.glob('style_tokens/*.yaml') + glob.glob('examples/**/*.yaml', recursive=True)]"

# primer contains the BUILD MANIFEST items (spot-check; full list in the primer header)
for p in "version: 1.0" "asset sheet layout" "Always-append tail" "orthographic" "Generate a casual mobile game art asset"; do grep -c "$p" studio_primer.md; done
```

## doc/

Third-party copyrighted PDFs (Katya Agarskaya style dictionaries, prompt collections) that seeded `style_tokens/`. Kept local only — gitignored; never commit or publish them.
