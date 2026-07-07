# Changelog

All notable changes to this toolkit are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow semantic versioning. The **toolkit version** (this file, git tags, the `TOOLKIT vX.Y.Z` marker in the `studio_primer.md` header) is independent of the `version: 1.0` line inside `style_guide.yaml` — that is the *schema* version, unchanged in 2.0.0 because all enum changes are additive.

## [2.1.0] - 2026-07-08

Driven by a user-run GPT-5.5 review of the Brawl Stars-ref debug series: `shape.language: rounded` alone still pulled generators back to soft candy pills, and shape stated ONCE globally let every panel/card/tab drift rounded.

### Added
- **`shape.ui_panel_geometry`** (OPTIONAL: soft_rounded / square_chunky / arcade_slanted / parallelogram / trapezoid_card) — the panel/card/tab/button silhouette ARCHETYPE (`slant` remains the skew degree). Phrases are silhouette-only (no corner/camera words — token purity); new §2 group (39 headings). Analyzer pairing rule: arcade_slanted/parallelogram ⇒ `slant` filled; soft_rounded ⇒ `slant` absent; the `# CONSISTENCY:` receipt gains a `panel=<archetype/omitted — why>` observed slot.
- **SHAPE LOCK (§4, the per-surface twin of the COLOR LOCK):** when `ui_panel_geometry` and/or `slant` is filled, the prompt ECHOES the silhouette on EVERY UI surface class (main panels, product cards, tabs, buttons, nav buttons, badges/ribbons) instead of one global mention, and the chunky/slanted archetypes append the shape-lock negative "soft pill-shaped panels, overly rounded candy cards" to "Avoid:". `# SELF-CHECK:` gains a `shape lock on surfaces ✓/n-a` item; §5 CHECK flags any single pill-rounded surface on a slanted-archetype guide as `off`. When both fields are absent, nothing changes (existing examples unaffected).

## [2.0.4] - 2026-07-07

### Fixed
- **Negative-contradiction check is now observed-value too.** A v2.0.3 run printed `negatives-vs-filled-fields ✓ (none conflict)` while `dark_theme` sat in `negative` over a `#40144E` deep-purple background — the item was still a rubber-stamp ✓. The `# CONSISTENCY:` receipt item becomes `negatives vs filled: bg=<hex> reads <dark/light> ⇒ <dropped: …/none conflict>`; the §3 bullet states that deep purples/navies/charcoals ARE dark (judge by the emitted hex, not mood). Receipt rule generalized: every `<…>` slot must state what was seen.

### Notes
- The same v2.0.3 run validated the shape gate: the receipt honestly reported `slant=none · lettering=upright` — when that happens on a genuinely slanted ref it is a host **vision** limit, not a compliance failure; the designed remedy is the human `UPDATE:` override (`shape.slant = strong, typography.font_feel = italic_display`), which pins those fields at confidence 1.0 — the shape-trait analogue of eyedropper-verifying hex.

## [2.0.3] - 2026-07-07

### Fixed
- **Slant/italic/corner capture is now gate-enforced, not prose-advised.** A real v2.0.2 run (GPT-5.5, Brawl Stars ref) still emitted `corner_radius: medium`, no `shape.slant`, `font_feel: bold_display` while printing a clean `# CONSISTENCY:` receipt — the receipt had no shape item, so the §3 rule-2 prose was skipped. §3's Consistency check gains a **Shape re-measure** bullet (look at the ref again: slanted/skewed panels ⇒ `shape.slant`; italic lettering ⇒ `italic_display`; corner radius measured, never habit-defaulted to medium) and the `# CONSISTENCY:` receipt gains three **observed-value slots** (`shape re-measured: slant=… · lettering=… · corners=…`) that must state what was seen — a rubber-stamp ✓ is no longer possible. §0.5 STYLE row and FINAL CONTRACT REMINDER line 3 restate it.

## [2.0.2] - 2026-07-07

### Added
- **`shape.slant`** (OPTIONAL: slight / strong) — skewed / parallelogram panels, tabs and banners ("arcade" diagonal energy à la Brawl Stars) are now a capturable trait instead of being normalized to rounded upright shapes; new §2 group `### shape.slant`.
- **`typography.font_feel: italic_display`** — heavy italic display lettering with a forward slant.

### Fixed
- **Every ref no longer gets "rounded-corner-ified."** §3 now instructs the analyzer to treat slanted tabs / skewed banners / italic lettering as a distinct trait and to MEASURE `corner_radius` against the ref (crisp arcade cards = `small`, candy-casual = `large`) instead of defaulting to medium/rounded.

## [2.0.1] - 2026-07-07

### Fixed
- **Aspect ratio no longer hard-defaults to 9:16 portrait.** New `style_guide.canvas` block (orientation + "W:H" estimate) that the ANALYZER reads off the ref images themselves; §4 rule 5 now resolves aspect ratio by priority — explicit spec/request → TARGET ref's own canvas → `style_guide.canvas` → per-asset-type default only when all are absent. A landscape ref yields a landscape asset. CHECK compares canvas orientation/ratio for screens; the `# SELF-CHECK:` receipt gains an "aspect ratio sourced" item.

## [2.0.0] - 2026-07-07

The style-neutral + compliance-gated engine. Driven by a real-run debug series (SHOP-screen ref on GPT-5.5): first removing a hidden "casual house style" bias, then hardening the primer against stochastic host-LLM non-compliance.

### Changed
- **Style-neutral engine**: the prompt opener is now SYNTHESIZED from the style_guide (`"Generate a {genre} {platform} game art asset in {rendering-family} style:"`) instead of the hard-coded "casual mobile game" phrase; token phrases express only their enum's meaning (cross-dimension adjectives like "soft/friendly/toy-like/stylized-not-realistic" removed from lighting, shape, metal, proportions phrases); `negative` is ref-derived, never a stock list — a dark/gritty ref bans cute/pastel instead of banning itself.
- `style_guide.color_map` (COLOR LOCK) is now **CONDITIONALLY REQUIRED** — must be filled whenever the refs show UI.
- §5 CHECK is **format-locked**: binary ok/off table (including an entry-points row) + per-fix `TWEAK:` lines + one consolidated `TWEAK:` — no scores, no essays; TWEAKs may only pull the image TOWARD the style_guide (no "art-director" suggestions).
- §5 edge-sharpness judging is conditioned on the style contract (intentional brushwork/grain/pixel/CRT are not defects for styles that declare them).

### Added
- Enum coverage for non-casual styles: `style.mood` dark register (dark, grim, tense, eerie, melancholic), `style.rendering` realistic_painted / inked_comic, `material.character` realistic_painted, `lighting.direction` side / below, optional `lighting.contrast` (soft/medium/hard), `environment.atmosphere` fog / gloom / storm, `project.genre` action / adventure / horror; negative_map entries for dark styles (cute, pastel_palette, toy_gloss, bright_cheerful).
- **§0.6 OUTPUT GATE**: every answer is drafted → verified against its §0.5 checklist row → revised until it passes → printed with a receipt line (`# CONSISTENCY:` for the analyzer, `# SELF-CHECK:` closing every synthesizer ASSUMPTIONS block, the fixed table for CHECK).
- **OUTPUT SKELETON** opening §3/§4/§5 (the entire reply is exactly the defined blocks — no intro sentence, no invented sections) and a **FINAL CONTRACT REMINDER** closing the primer (recency anchor).
- §3 Consistency check: list discipline (only genre/mood/palette/color_map/background.color/negative are lists), negative-vs-filled-field contradiction check, color_map requirement, confidence↔block pairing.
- §4 rules 12–14: prompts are flowing prose and self-contained (never referencing "the style guide"); one entry point per feature per screen (deduped, checked again by CHECK); mandatory self-check receipt. §4 rule 6 contradiction guard skips negatives that contradict filled fields. Closed element lists (no "such as") + "no additional UI elements beyond those listed" tail.
- §0 dispatch rule: an image with no command is never free-form described (no style_guide yet → run STYLE; otherwise ask which command it belongs to).
- Camera hint: flat UI screens/mockups = orthographic.
- `TOOLKIT vX.Y.Z` version marker in the primer header (Ctrl+F it in a chat to know which build is pasted).

### Fixed
- Analyzer self-sabotage: `dark_theme`/`realistic`/`gritty` no longer land in `negative` when the ref itself is dark/realistic/gritty.
- Duplicate feature entry points (e.g. Shop in a floating button AND the bottom nav) in self-suggested screen layouts.
- Detached end-of-prompt color lists — hex now stays inline per surface.
- `platform` and other single-value fields emitted as lists.

## [1.0.0] - 2026-06 (untagged)

Initial draw-fresh toolkit: `studio_primer.md` bundled from `schema/` + `style_tokens/`; commands STYLE / UPDATE: / ASSET: / CHARACTER: / BACKGROUND: / OBJECT: / CHECK / REGEN / TWEAK; four generate branches, UI-kit & character sheets, pose-variation image-edit path, COLOR LOCK, EXECUTION CHECKLIST. (EXTRACT / UPSCALE / design-system board paths removed by design — draw-fresh only.)
