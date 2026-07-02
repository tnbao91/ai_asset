# ai_asset — AI pipeline for 2D UI/UX mobile-game assets

Turn **reference images** into a structured **style guide**, then into a ready-to-use **image prompt** for producing on-style 2D UI/UX assets. **Generator-neutral** — which image generator you use for the prompt (gpt-image / Gemini "Nano Banana" / Midjourney…) is entirely up to you.

> **How to use:** paste **[`studio_primer.md`](studio_primer.md)** as the first message of a fresh chat with any vision-capable LLM (a regular ChatGPT account is enough), attach your reference images, then type `STYLE` and `ASSET:` to get a style guide and prompt right in the chat. Full step-by-step below.

The toolkit **stops at the prompt** — choosing a generator and generating the image is on you.

---

## Quick start

1. Open a fresh chat in a vision-capable LLM (ChatGPT, Claude, or Gemini).
2. Paste the entire contents of `studio_primer.md` as the first message.
3. Attach your **STYLE references** (images whose art style you want to learn) and type `STYLE` → you get a `style_guide.yaml`.
4. Review it (fix hex colors with an eyedropper; confirm low-confidence fields) and send the fixes back with `UPDATE:`.
5. Type `ASSET: <description>` → you get a finished image prompt.
6. Take the prompt to the image generator of your choice (attach the STYLE reference there for color consistency).

---

## Repo structure

```
README.md            ← you are here (overview + full usage guide)
studio_primer.md     ← THE TOOL: a self-contained mega-prompt you paste into an LLM chat
schema/              ← SOURCE: field + enum definitions (the primer is built from these)
  style_guide.schema.yaml
  asset_spec.schema.yaml   layout_spec.schema.yaml   (optional structured asset description)
style_tokens/        ← SOURCE: STYLE DICTIONARY, enum → English phrase (built into the primer; seeded from the PDFs)
  materials.yaml  render_shape.yaml  light_color.yaml  layout_negative.yaml
doc/                 ← local reference material (third-party prompt-collection PDFs; not included in this repo)
examples/
  settings_screen/   ← sample output (style_guide.yaml + prompt)
  analyzer_smoke_test/ ← proof the analyzer runs on a real asset image
```

---

## How it works

`studio_primer.md` is **the tool** — a self-contained mega-prompt **built from** `schema/` (valid enums) + `style_tokens/` (enum → phrase dictionary). It bundles two jobs into one chat:

```
STYLE ref ──► [ STYLE ] ──► style_guide.yaml (enums)  ─┐
                                                       │
   asset (text | TARGET ref | empty→AI-suggested) ─────┴─► [ ASSET ] ──► prompt (generator of your choice)
```

- **Two roles of a reference image:** a *STYLE ref* (how it looks → `style_guide`) is different from a *TARGET ref* (what to make / its layout → asset).
- **Source of truth:** if you change `schema/` or `style_tokens/`, the primer must be **rebuilt** from them (don't hand-edit the same content in two places).

---

## Detailed usage

### S1. Open a chat & load the primer
1. Open a chat with a **vision-capable** LLM (ChatGPT — a regular account is fine — Claude, or Gemini).
2. Copy the whole `studio_primer.md` and paste it as the **first message**. Send it.
3. The LLM takes on the role and waits for commands. (Use one chat per game — see S6.)

### S2. Extract a style guide from references — command `STYLE`
Attach your **STYLE references** (1..N images in the same art style) and type:

```
STYLE
```

The LLM returns a `style_guide.yaml`, for example (abridged):

```yaml
style:
  rendering: semi_painted
  mood: [cheerful, friendly]
material:
  button: glossy_plastic
lighting:
  direction: top
  highlight: strong
palette:
  primary: ["#0D6DB8"]     # estimate — verify with an eyedropper
confidence:
  shape: 0.95
  material: 0.74           # < 0.75 → review this
  lighting: 0.70
# REVIEW NOTES
# material (0.74): button could be candy_jelly instead of glossy_plastic — double-check.
# lighting (0.70): top vs top_left is uncertain.
# palette: hex values are estimates — eyedropper-verify primary/accent.
```

> With multiple references, the LLM extracts the **common style denominator** rather than describing each image.

### S3. Review & adjust (important — don't skip)
1. **Read the `# REVIEW NOTES`.** Any dimension with `confidence < 0.75` is something the model is unsure about — confirm or correct it.
2. **Fix hex values with an eyedropper** (the model only *guesses* colors):
   - Open the reference in Figma / Photoshop, or use the macOS "Digital Color Meter" / any pipette tool.
   - Pick the 3–5 main colors (primary, accent…) and read their `#RRGGBB`.
3. **Send the corrections with `UPDATE:`**, for example:
   ```
   UPDATE: palette.primary = #0D6DB8, palette.accent = #FFD34A,
   material.button = glossy_plastic, lighting.direction = top
   ```
   The LLM reprints the corrected `style_guide.yaml` with those fields marked final (`confidence` → 1.0). This style guide is shared by every asset in the game.

### S4. Make an asset — command `ASSET:`
There are **three ways** to specify what to make — use whichever is convenient:

**(a) Describe in text:**
```
ASSET: a gold coin currency icon, single coin
```

**(b) Attach a TARGET ref** (an image of the asset/layout you want to mirror — *different* from the STYLE ref in S2):
```
ASSET:        ← then attach an image of a settings screen whose layout you want to follow
```

**(c) Give only a vague type — or nothing at all — and let the AI propose the details:**
```
ASSET: settings screen      ← or just "ASSET:" — the AI proposes the details, tagged [AI-suggested]
```

The LLM returns **one natural-language prompt**, plus an ASSUMPTIONS block if it invented anything:

```
Generate a casual mobile game art asset: a gold coin currency icon, square 1:1
canvas; semi-painted with a stylized baked texture look, rounded chunky
silhouette; polished gold with a rich precious sheen and clean highlights; soft
light from the top, strong highlights, soft contact shadow; clean pure colors —
accent yellow #FFD34A; isolated on a transparent background, generous centered
padding.
Avoid: photorealism, realistic metal, flat design, sharp corners, thin lines,
text or UI overlays, watermark, signature, jpeg artifacts.

# ASSUMPTIONS
# [AI-suggested] square 1:1 canvas; single standing coin, not a stack — change if you want a pile.
```

> **Read the `# ASSUMPTIONS` block** and correct anything wrong (e.g. `TWEAK: make it a stack of 3 coins`).

### S5. Generate the image (outside this toolkit — your choice of generator)
1. Copy the prompt.
2. Open the image generator **of your choice** (gpt-image in ChatGPT, Gemini "Nano Banana", Midjourney…).
3. **Attach the STYLE reference again** + paste the prompt → generate. *(Attaching the reference is the main way to keep color/feel consistent — more reliable than text alone.)*
4. Not happy? Go back to the chat and type `TWEAK <change>` or `REGEN` for a new prompt — or attach the generated image and type `CHECK` for a per-dimension conformance report with ready-made `TWEAK` lines.

### S6. Many assets & many games
- **Same game:** keep typing `ASSET: ...` in the **same chat** — the style guide stays in context, so all assets stay on-style.
- **New game:** open a **new chat**, paste `studio_primer.md` again, attach that game's STYLE references. (Name the chat after the game for easy retrieval.)

### Command reference
| Command | Effect |
|---------|--------|
| `STYLE` (+ attach STYLE ref) | Analyze the image(s) → `style_guide.yaml` |
| `UPDATE: <field = value, …>` | Apply your manual corrections (eyedropper hex, final enums) → reprints the corrected `style_guide.yaml` |
| `ASSET: <description>` / `ASSET:` (+ TARGET ref) / `ASSET:` (empty) | Build a prompt for one asset |
| `CHECK` (+ attach the image you generated) | Per-dimension conformance report vs the style guide + ready-made `TWEAK` lines |
| `REGEN` | Regenerate the last prompt |
| `TWEAK <change>` | Adjust the prompt as requested |

---

## Tips & troubleshooting

- **Assets drift in style across images:** attach the same STYLE reference every time you generate; keep the style sentences identical across prompts; generate in small batches. (Cross-asset consistency is a common weakness of today's generators.)
- **Icons that must really match each other:** generate them as ONE **asset sheet** instead of separate images — `ASSET: an asset sheet of 6 booster icons, 3 columns x 2 rows, consistent items` (the word *consistent* is what holds one style across cells) — then slice it. Strongest anti-drift trick.
- **Control the output ratio:** set `aspect_ratio` in `asset_spec.yaml` (`"1:1"` icon, `"9:16"` portrait screen…) or just say it in the `ASSET:` line; if you don't, the AI picks a default per asset type and tags it `[AI-suggested]`.
- **Colors come out wrong vs. the reference:** almost always because the model guessed the hex — the **eyedropper** step (S3) is mandatory, and attach the reference when generating.
- **The LLM emits odd/invalid style_guide values:** remind it to "use only the enums in §1 of the primer." If the primer scrolled out of context, paste it again.
- **Prompt too long/short:** ~150–250 words for a single icon/asset; ~300–400 for a screen with layout. Type `TWEAK: make it tighter` if it's bloated.
- **Want to follow a specific screen's layout:** use option (b) in S4 — attach a TARGET ref.
- **Manage many games more neatly:** if you want a record, save each game's `style_guide.yaml` to its own file (e.g. under `examples/<game>/`) so you can paste it back next time instead of re-running `STYLE`.

---

## Expectations & limits

1. **Always attach the STYLE reference when generating.** `style_guide.yaml` is a *style contract* + prompt seed, **not** the sole consistency mechanism. The attached reference is what keeps colors/feel accurate.
2. **Cross-asset consistency is a common weakness of today's generators.** One reference set used for a settings screen, a currency icon, and a button may still drift slightly. Reduce drift by keeping the style description identical, attaching the same reference, and generating in small batches.
3. **The prompt is descriptive prose** (suited to most modern generators), not weighted tags or `--flags`. Negatives go in an "Avoid: …" sentence. For Midjourney, convert to tags + `--sref` yourself.
4. **Vision-guessed hex values are never exact** — the eyedropper step (S3) is required.

A full sample output (style guide + prompt) for reference: [`examples/settings_screen/`](examples/settings_screen/).

---

> Reminder: this toolkit **stops at the prompt**. Choosing a generator and generating the image is on your side.
