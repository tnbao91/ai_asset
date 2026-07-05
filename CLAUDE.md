# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **prompt-engineering toolkit** (no application code) that turns reference images into on-style 2D mobile-game assets. The deliverable is `studio_primer.md` — a self-contained mega-prompt the user pastes into a vision-capable LLM chat, which then answers short commands (`STYLE`, `UPDATE:`, `ASSET:`, `CHARACTER:`, `BACKGROUND:`, `OBJECT:`, `CHECK`, `REGEN`, `TWEAK`) to produce a `style_guide.yaml` and, from it, image prompts. The primer only ever outputs **text** — it never invokes an image tool even when the host chat has one.

**Draw-fresh only (deliberate design decision, 07/2026):** every asset is generated FRESH from the style_guide (eyedropper-verified hex pinned in the prompt) — that is the path generative models are good at. The former EXTRACT / UPSCALE / `STYLE: design system` board paths (re-rendering existing pixels) were removed after real-world testing showed them producing mushy, broken output on gpt-image; do not re-add "keep the source pixels" paths. The one image-edit path kept is the CHARACTER pose variation (identity lock). **The STYLE ref is attached only to `STYLE`** — later commands inherit the style_guide from context, and their attachment slot belongs to that command's own ref (TARGET layout / CHARACTER / the generated result for CHECK); don't reintroduce "re-attach the STYLE ref when generating" wording.

**Generator-neutral by design**: the toolkit stops at the prompt. It never generates images and must not be coupled to any specific image generator (gpt-image / Gemini "Nano Banana" / Midjourney are the user's choice). Scope is 2D assets in four classes: **UI/UX** (screens, icons, buttons, panels), **backgrounds**, **characters** (static art + pose variations via a CHARACTER ref; animation sheets are out of scope), and **objects/props**. Repo content stays in English (prompts target English-language generators).

## The one rule that matters: the primer is GENERATED

`studio_primer.md` is bundled from two source directories — never hand-edit it independently of them:

- `schema/` — field + enum definitions. `style_guide.schema.yaml` is canonical; `asset_spec.schema.yaml` / `layout_spec.schema.yaml` describe the optional structured UI asset/layout inputs; `background_spec.schema.yaml` / `character_spec.schema.yaml` / `object_spec.schema.yaml` the optional structured inputs for the three non-UI classes.
- `style_tokens/` — the style dictionary mapping each enum to an English prompt phrase (`materials`, `render_shape`, `light_color`, `layout_negative`, `character_environment`).

Change flow: edit the source file first → rebuild the affected primer section → verify against the **BUILD MANIFEST** in the primer's header comment (it lists every source section that must be present; grep each item). The schemas are documentation-style pseudo-YAML (fields + enums, not JSON Schema); there is no build/validator script — the repo intentionally stays code-free.

## Architecture (pipeline inside the primer)

STYLE ref image(s) → **§3 ANALYZER** → `style_guide.yaml` (enums from §1 only, plus per-dimension `confidence`; `<0.75` = user must review; hex values are estimates the user fixes via eyedropper + `UPDATE:`; the `character`/`environment` blocks are OPTIONAL — filled only when the refs actually contain characters/scenes, never guessed) → **§4 SYNTHESIZER** (the four generate commands map 1:1 to its four prompt branches — `ASSET:`=UI, `CHARACTER:`, `BACKGROUND:`, `OBJECT:`; each branch translates every enum through the §2 dictionary, in its fixed prompt order ending with "Avoid:") → one natural-language prompt → user generates the image elsewhere → optional **§5 CHECKER** compares the generated image back against the style guide — extra-strict on palette hex (per-role expected vs observed) and UI edge sharpness (blur/mush/soft text = off) — and emits per-fix `TWEAK:` lines plus **ONE consolidated `TWEAK:`** the user copies to fix everything in one go.

Character **pose variation** is a sub-branch of §4: `CHARACTER:` + an attached CHARACTER ref (the already-generated character; or `character_spec.character_ref`) → an image-EDIT prompt that opens with the §2 identity-lock phrase (face/outfit/colors frozen, only pose/expression change). It is the ONE image-edit path in the toolkit — it needs an image-*editing* generator (the §4 IMAGE-EDIT PATH NOTE covers this) but still translates lighting/outline through §2.

Two consistency sub-modes mirror each other (both stay inside their parent command — no new command, so §0/README stay 1:1): the **character sheet** (`character_spec.sheet`) draws one character's expressions/turnaround on a single sheet; the **UI-KIT sheet** (`asset_spec.kit` / `type: ui_kit`, or "UI kit" in text) is the `ASSET:` analogue — the whole UI component set (button/toggle/slider/checkbox/progress/panel/icons/text) drawn on ONE canvas in one pass so widgets can't drift apart. Both lean on the §2 word **consistent** as the anti-drift anchor. UI widget/text style itself lives in the OPTIONAL `typography` / `controls` style_guide blocks.

Distinctions that must not be blurred:

- **STYLE ref** ("how it looks" → style_guide) vs **TARGET ref** ("what to make / its layout" → asset content) vs **CHARACTER ref** ("who this is" → pose variation, identity locked). Different inputs to different commands/branches.
- `style_guide.background` (overall style background) vs `asset_spec.background` (popup/fullscreen/transparent/scene for one asset) vs the `background` asset CLASS (a scene made via background_spec).
- Negative tails: `negative_tail_always` is appended to every prompt; `negative_tail_non_screen` ("no text, no UI overlays") only to single assets — screens ARE UI, so it must be skipped there; `negative_tail_character` (anatomy) only to characters — BUT when `feature_exaggeration: high` (mitten/stump styles) use `negative_tail_character_simplified` INSTEAD ("no distinct fingers", Banana2 Zero-Digits — "no EXTRA fingers" is the wrong guard there); `negative_tail_background` (no actors) only to backgrounds.

## Consistency invariants (check after any change)

- Every enum in `schema/style_guide.schema.yaml` appears in primer §1; every token group in `style_tokens/*.yaml` appears in primer §2 (currently 36 `###` headings). The OPTIONAL UI blocks `typography` and `controls` (toggle/slider/checkbox/progress, in `style_tokens/ui_components.yaml`) must appear in both §1 and §2 — they anchor UI-widget/text consistency the same way `material.button/container/icon` anchor button/panel/icon. Likewise the OPTIONAL `color_map` block (the **COLOR LOCK**, per-surface hex; tokens in `style_tokens/light_color.yaml`) must appear in §1, §2 (`### color lock`), §3 (analyzer fills it per surface, with scope notes), §4 (rule 4: contract sentence once + hex INLINE per surface + color-lock negatives in rule 6) and §5 (CHECK compares every filled surface) — it exists because text-only hex is followed loosely; per-surface inline hex is the strongest text-side color device.
- The primer §0 command table and README's "Command reference" table must match 1:1. The §0.5 EXECUTION CHECKLIST rows must cover the same commands as §0 (it's a per-command anti-miss contract, not a second command list).
- The **IMAGE-EDIT PATH NOTE** lives once in §4, scoped to the pose-variation branch (the toolkit's only image-edit path) — don't duplicate it elsewhere when editing.
- §4 has a **Rule 11 coverage self-check**: every filled style_guide dimension must be translated or explicitly `# [skipped]` in ASSUMPTIONS. The per-branch prompt orders name outline/mood/button.depth/icon.* explicitly — keep them named when editing the orders.
- `examples/settings_screen/` is the end-to-end proof for the UI branch: `prompt.txt` must remain derivable from its three spec files under the current §4 rules. `examples/mascot_character/` is the proof for the character branch (both prompts derivable from its specs, incl. the pose-variation image-edit prompt). `examples/analyzer_smoke_test/` is a captured analyzer artifact — leave as-is.

## Verification commands

```bash
# all YAML parses
python3 -c "import yaml, glob; [yaml.safe_load(open(f)) for f in glob.glob('schema/*.yaml') + glob.glob('style_tokens/*.yaml') + glob.glob('examples/**/*.yaml', recursive=True)]"

# primer contains the BUILD MANIFEST items (spot-check; full list in the primer header)
for p in "version: 1.0" "asset sheet layout" "Always-append tail" "orthographic" "Generate a casual mobile game art asset" "chibi proportions" "identity lock" "layered_parallax" "size_class" "EXECUTION CHECKLIST" "IMAGE-EDIT PATH NOTE" "Coverage self-check" "UI-kit reference sheet" "color lock"; do grep -c "$p" studio_primer.md; done
```

## doc/

Third-party copyrighted PDFs (Katya Agarskaya style dictionaries, prompt collections) that seeded `style_tokens/`. Kept local only — gitignored; never commit or publish them.
