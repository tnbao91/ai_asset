# Example: mascot character (Character branch of §4)

End-to-end proof for the **character** asset class — same game/style as
[`examples/settings_screen/`](../settings_screen/), extended with the optional
`character` block + `material.character` (the analyzer only fills those when the
STYLE refs contain characters; here they were set human-final via `UPDATE:`).

Files:

| File | Role |
|------|------|
| `style_guide.yaml` | Style DNA incl. the optional `character` block, `material.character`, `camera` |
| `character_spec.yaml` | NEW-character run (`CHARACTER:`, no `character_ref`) |
| `prompt.txt` | Output of the Character branch (§4) for that spec |
| `character_spec.pose_variation.yaml` | POSE VARIATION run — `CHARACTER:` with `character_ref` set, CHARACTER ref attached |
| `prompt_pose_variation.txt` | Output: an image-EDIT prompt (identity locked, only pose/expression change) |

## How prompt.txt derives from the specs (enum → §2 phrase)

| Spec field | Value | §2 phrase used |
|------------|-------|----------------|
| spec `archetype` + `physique` + `outfit` | chef cat, round-bellied, hat + apron | subject sentence (content, not restyled) |
| spec `aspect_ratio` | "3:4" | "Portrait 3:4 canvas" |
| spec `framing` | full_body | "show the full body clearly from head to feet" |
| `character.proportions` | toon_3head | "cartoon proportions, about 3 heads tall…" |
| `character.feature_exaggeration` | medium | "clearly exaggerated features…" |
| spec `pose` + `expression` | waving, smile | pose/expression sentence |
| `style.rendering` | semi_painted | "semi-painted render with a stylized baked texture look…" |
| `material.character` | painted_soft | "soft hand-painted character surfaces…" |
| `shape.language` | rounded | "rounded, hand-rolled silhouette…" |
| `camera` | three_quarter | "3/4 view" |
| `lighting.*` | top / strong / soft_bottom | lighting sentence |
| `outline.*` | true / medium / darker_variant | outline sentence |
| `effects.bevel` | true | "soft beveled edges" |
| `palette` + spec `palette_role` | #0D6DB8 apron, #FFD34A details | palette sentence with hex |
| spec `background` | transparent | "isolated on a transparent background" |
| `negative` + tails | §2 NEGATIVE map | "Avoid: …" + character tail + non-screen tail + always tail |

The pose-variation prompt skips the whole style translation for the character's
DESIGN (the attached CHARACTER ref carries it) and opens with the §2 identity-lock
phrase instead; only pose/expression + "keep lighting/outline/background" remain.
