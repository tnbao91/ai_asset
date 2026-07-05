# End-to-end example: Settings Screen

A real run of the chain `style_guide.yaml` + `asset_spec.yaml` + `layout_spec.yaml` → image prompt (generator of your choice), using `style_tokens/` to translate the enums. This also serves as a verification of the process.

## Inputs (in this folder — clean, schema-conformant)
- `./style_guide.yaml` — style DNA (analyzer output, manually reviewed).
- `./asset_spec.yaml` — asset = settings_screen.
- `./layout_spec.yaml` — screen layout.

## Enum → token translation (excerpt)
| Field (style_guide) | Value | Source token |
|---|---|---|
| rendering | semi_painted | render_shape.yaml → "semi-painted... stylized baked texture look" |
| shape.language / corner | rounded / large | "rounded hand-rolled silhouettes" / "large soft rounded corners" |
| material.button (conf .78 ↓) | glossy_plastic | materials.yaml → "smooth glossy plastic... airbrushed sheen" |
| material.icon | painted | "semi-painted icon, baked texture" |
| material.currency | polished_gold | "polished gold, precious sheen" |
| material.container | soft_plastic | "soft plastic panel, rounded chunky edges" |
| lighting (conf .73 ↓) | top / strong / soft_bottom | light_color.yaml → "soft directional light from top" + "strong highlights" + "soft contact shadow at bottom" |
| effects | bevel | "soft beveled edges" |
| button | depth medium / gloss high | render_shape.yaml |
| typography (conf .80) | rounded_sans / bold / uppercase | ui_components.yaml → "bold rounded soft sans-serif, ALL-CAPS title, neutral ink" |
| controls.toggle (conf .76) | pill / on=secondary / knob glossy | ui_components.yaml → "pill toggle track, green #42D70F when ON, round glossy knob" |
| layout | large / low / mobile | layout_negative.yaml |
| background (style) / background (asset) | transparent / popup | light_color.yaml + asset_spec → "soft dimmed popup background" |
| aspect_ratio (asset_spec) | 9:16 | direct → opening line "Aspect ratio 9:16" |
| negative | realistic, flat_design, sharp_corner, thin_icon... | layout_negative.yaml → "Avoid:" + `negative_tail_always` (watermark, jpeg artifacts…) |

> `material` (0.78) and `lighting` (0.73) have low confidence → phrased **softly** ("leaning toward...") in the prompt so you can steer.

## OUTPUT — image prompt (paste straight into your generator of choice)

See `prompt.txt`.
