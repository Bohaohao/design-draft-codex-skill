# Style Conversion

The goal is one-to-one restoration.

## Rules

1. Preserve exact numeric values whenever possible
2. Prefer CoDesign metadata JSON and extracted right-side panel values over screenshot estimation
3. Preserve typography, spacing, colors, borders, radius, and shadows with minimal loss
4. Convert styles into the mechanism already used by the project
5. Use absolute positioning only when necessary for fidelity
6. Prefer normal flow, flex, or grid when they preserve the visual result without loss
7. Apply extracted styles to every clicked node, including containers, groups, layout nodes, text, icons, images, and atomic controls
8. Optimize style organization for the current stack without changing the visual result

## Capability Boundary

Set `imageUnderstanding = available` only when the host explicitly supports image input and the current agent can safely reason over image attachments. If capability is unknown, set `imageUnderstanding = unavailable`.

Do not require image understanding, screenshot reading, screenshot OCR, or visual estimation for style conversion. Use screenshot-based estimation only as a last resort and only when `imageUnderstanding = available`.

## Right Panel Source Of Truth

Extract right-side property, style, inspect, and CSS panel values from Chrome MCP-readable page content: accessibility snapshot text, DOM text, selected node metadata, evaluated script output, or copyable CSS text exposed by the design platform.

Do not use image understanding, screenshot OCR, or visual estimation as the primary method for extracting CSS values from the right-side panel. If a value is available in the panel as readable text or metadata, that value is the source of truth for style conversion.

## CoDesign Metadata Conversion

For CoDesign, metadata JSON from `meta_url` is also a source of truth for style conversion.

Use metadata fields this way:

1. `css`: convert declarations directly into scoped CSS, CSS modules, utility-compatible classes, or inline style objects according to the target project
2. `fragments`: preserve mixed text content and fragment-level typography such as color, `fontSize`, `fontFace`, `fontWeight`, `lineHeight`, and `textAlign`
3. `content`: use for plain text when fragments are absent or do not vary styles
4. `rect` and `realRect`: lock local width, height, x/y offsets, sibling gaps, and restoration-unit bounds
5. `fills`: map to background colors, gradients, image fills, or placeholder asset decisions
6. `borders`: map to border width, color, style, side-specific borders, and line strokes
7. `radius`: map to `border-radius`
8. `effects` and `shadows`: map to `box-shadow`, blur, filters, or related CSS effects where applicable
9. `opacity`: map to CSS opacity
10. `rotation`, `absoluteTransform`, and `relativeTransform`: map to CSS transform only when visible and needed

When `fragments` are present, do not flatten the text into a single style if fragment typography differs visibly. Use nested spans or project-native text wrappers so each fragment keeps its own font, line-height, color, and weight.

Use `rect` and `realRect` for local layout locking. If normal flow does not reproduce metadata-derived bounds within 2 px, use absolute positioning inside the local unit, CSS grid tracks, fixed flex basis, or explicit spacers derived from those bounds.

Metadata-derived values must be marked as `metadata`, not `estimated`.

## Project-Aware Output

Follow the target project's existing conventions before introducing a new style pattern:

1. Vue SFC with scoped CSS
2. CSS modules
3. Utility-class projects
4. Inline style objects where already established
5. Preprocessors such as Sass or Less when already used

## Strategy Selection

Use visual understanding to classify the confirmed restoration area or locked unit before node traversal when image understanding is available. The classification chooses the best implementation strategy for that region and may split a mixed region into smaller strategy-specific units.

Region strategy classes:

1. Pure DOM: ordinary layout, text, cards, lists, tables, simple tags, separators, simple icons, and static UI blocks; restore with HTML/CSS and exact panel extraction
2. Form/UI component: input, textarea, select, checkbox, radio, date picker, upload, switch, segmented control, and button workflows; prefer the current project's component library when it preserves fidelity
3. Chart/ECharts: axes, legends, series, KPI visualization, pie, bar, line, radar, map, and other data visualizations; prefer ECharts when available or authorized
4. Canvas: dense custom graphics, complex visual effects, highly irregular drawing, or regions where DOM/component restoration would be brittle; use canvas only for that region
5. Asset/slice: photos, logos, bitmap illustrations, complex icons, exported images, and image-like backgrounds; search slice/artifact sources before approximating
6. Mixed: split the region into smaller units and assign each subregion its own strategy

If image understanding is unavailable, choose from selected layer name, hierarchy, visible text, role, dimensions, component metadata, and right-side panel values. Keep exact extracted style values as the fidelity anchor whenever available.

Visual classification controls implementation strategy, traversal plan, and primitive choice. It must not replace exact CSS extraction, selected bounds extraction, parent layout extraction, or asset lookup.

Do not stop solely because screenshot/image understanding is unavailable. If non-visual signals do not identify the strategy clearly, ask the user to select or confirm the intended element or strategy.

## Forbidden Style Shortcuts

Do not:

1. Use visual estimation when Chrome MCP-readable values exist
2. Invent exact CSS values that were not extracted, computed, or confirmed
3. Treat screenshot OCR as a reliable style source
4. Convert approximate values as if they were exact panel values

If a component library primitive is used, create class hooks or wrapper classes so the extracted style values can still be applied precisely.
