# Style presets

A preset is a **complete, pre-authored `style_guide.yaml`** — pure DATA for the primer's `LOAD` command. Loading one skips the `STYLE` step entirely: no reference image, no eyedropper pass — the hex values and enums are already final (`confidence: 1.0` throughout).

Presets are inputs the *user* pastes at runtime; they are never bundled into `studio_primer.md` and they are not a render path — every asset is still drawn fresh from the guide, exactly as after a `STYLE` run.

## How to use a preset

1. Open a fresh chat with a vision-capable LLM and paste all of [`studio_primer.md`](../studio_primer.md) as the first message.
2. Open the preset file, copy the **entire YAML**.
3. Send a message that starts with `LOAD` followed by the pasted YAML. The primer validates it against its §1 schema, adopts it verbatim, and reprints it — from there, use `ASSET:` / `CHARACTER:` / `BACKGROUND:` / `OBJECT:` / `CHECK` as usual.

The same `LOAD` command also resumes a previous game: save the final `style_guide.yaml` from any chat to a file and paste it back later.

## Catalog

| Preset | Look | Inspiration (described in words) | Status |
|---|---|---|---|
| [`turkish-casual.yaml`](turkish-casual.yaml) | Glossy saturated cute-casual: chunky rounded UI, candy-bright palette on a light sky backdrop, heavy button gloss, bold outlined uppercase display text, toon-3D characters | The polished casual-mobile look popularized by Turkish studios (match-3 / puzzle hits in the Royal Match / Toon Blast lineage) — described from publicly known genre characteristics, not extracted from any bundled asset | DRAFT |

**DRAFT** = enum choices are settled but hex values are best-effort estimates, not yet verified against reference screenshots. **verified** = refined via a `STYLE` pass on real refs + eyedropper-checked hex.

## Authoring your own preset

1. Collect reference screenshots **locally** in `presets/refs/` (gitignored — see the copyright note below).
2. In a scratch chat, paste the primer, attach the refs, run `STYLE`.
3. Eyedropper-verify every hex (palette AND color_map) and fix them via `UPDATE:`; confirm every `confidence < 0.75` dimension.
4. Save the final printed `style_guide.yaml` as `presets/<style-slug>.yaml` (kebab-case), set every `confidence` to 1.0 (the guide is now human-final), and add a catalog row above.
5. A preset must stay valid against `schema/style_guide.schema.yaml` — run the preset validator in [`CLAUDE.md`](../CLAUDE.md) → *Verification commands*.

## Copyright note

Reference screenshots of commercial games are copyrighted. They live **only** in the local, gitignored `presets/refs/` folder (same policy as `doc/`) and are never committed or published. What gets committed is only the derived style *parameters* — enum values and hex colors plus a plain-text description — never the images themselves.
