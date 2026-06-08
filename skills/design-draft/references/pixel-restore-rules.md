# Pixel Restore Rules

Pixel-level restoration means using real values from the design platform whenever available.

For CoDesign, metadata JSON from the target screen's `meta_url` is one of those real values. Treat `rect`, `realRect`, `css`, `fragments`, `fills`, `borders`, `effects`, `radius`, transforms, and parent-child relationships as exact machine-readable restoration data when present.

## Priority

Restore in this order:

1. Layout hierarchy
2. Positioning and spacing
3. Width and height
4. Typography
5. Colors and backgrounds
6. Borders, radius, and shadows
7. Visible text and imagery

## Extraction

For every visited node:

1. For CoDesign, first try to locate the node in metadata JSON and read exact `rect`, `realRect`, `css`, `fragments`, `fills`, `borders`, `effects`, `radius`, transforms, and parent-child relationships
2. When image understanding is available, capture the confirmed artboard or locked unit and use visual understanding first to identify clickable modules, child nodes, groups, repeated items, and safe click hotspots
3. When image understanding is unavailable, identify click candidates from CoDesign metadata JSON, selected-layer data, layer tree, DOM text, accessibility text, right-side panel metadata, and user-confirmed points
4. Click the candidate node in the design draft when panel validation, export controls, or ambiguity resolution are needed
5. Validate the click through selected-node metadata, layer name, right-side panel updates, or CSS extraction results
6. Capture a screenshot of the node or closest available bounding region only when image input is supported or when the screenshot is useful for user confirmation
7. Read the right-side style, property, or CSS panel when metadata is absent, incomplete, or needs verification
8. Record exact values for size, position, spacing, typography, fills, borders, radius, opacity, shadows, and images when available
9. For layout-sensitive nodes, also record parent container CSS, selected bounds, sibling order, and adjacent sibling gaps
10. Convert the values into the target project's style mechanism

Do not replace inspectable values with screenshot guesses.

If CoDesign metadata JSON already provides an exact value, do not continue relying on screenshot estimation for that value. Use screenshot or visual understanding only to assist discovery, classification, or final comparison.

Do not assume the active model can inspect images. If image understanding is unavailable, continue from the selected layer name, hierarchy, right-side panel values, visible text, dimensions, coordinates, and component metadata. Never stop solely because screenshot/image understanding is unavailable.

Set `imageUnderstanding = available` only when the host explicitly supports image input and the current agent can safely reason over image attachments. If capability is unknown, set `imageUnderstanding = unavailable`. Never run a screenshot/image-understanding probe that may crash a text-only host.

## Right Panel CSS Extraction

Right-side property, style, inspect, and CSS panel values must be extracted from Chrome MCP-readable page content whenever possible.

Use these sources before any screenshot-based downgrade:

1. CoDesign metadata JSON when the platform is CoDesign
2. Accessibility snapshot text from the right-side panel
3. DOM text from the right-side panel
4. Selected node metadata exposed by the design tool page
5. `evaluate_script` results that read panel text, attributes, computed data, or design-tool state
6. Copyable CSS text exposed by the design platform

Do not use image understanding, screenshot OCR, or visual estimation as the primary method for extracting CSS values from the right-side panel. If the panel visibly contains values but the first MCP snapshot does not expose them, inspect the right-panel DOM, accessibility tree, selected-node state, or script-readable text before declaring extraction unavailable.

## Evidence Sources

For important layout and style decisions, track the source:

1. `metadata`: extracted from CoDesign metadata JSON, including screen data, node tree, exact geometry, style fields, text fragments, and parent-child relationships
2. `panel`: extracted from right-side property, style, inspect, or CSS panel
3. `layer`: inferred from selected layer name or hierarchy
4. `dom`: read from DOM text or accessibility snapshot
5. `script`: read through evaluated script output that is not specifically CoDesign metadata JSON
6. `project`: matched from existing project conventions
7. `user`: confirmed by the user
8. `estimated`: estimated from screenshot, only when `imageUnderstanding = available`

`metadata` is exact machine-readable data. It is not screenshot OCR and must not be reported as `estimated`.

Use combined labels such as `metadata+layout` when placement is derived from metadata geometry plus parent or sibling relationships, and `metadata+panel` when right-panel CSS confirms metadata values.

Do not invent exact CSS values that were not extracted, computed, or confirmed.

## Layout Coupling

Pixel placement depends on more than the selected node's CSS. A node's final browser position may be changed by parent flex/grid layout, parent padding, sibling gaps, line-height, font fallback, and natural document flow.

For layout-sensitive nodes, collect a coupled layout dataset before implementation:

1. Selected node declarations
2. Selected node bounds: `x`, `y`, `width`, `height`
3. Parent bounds and parent layout declarations
4. Previous and next visible sibling bounds
5. Horizontal or vertical sibling gaps
6. Local restoration unit origin

For CoDesign, build this dataset from metadata JSON first when available. Use `parent_id`, shared parent ids, traversal parents, `rect`, `realRect`, sibling rects, `relativeTransform`, and `absoluteTransform` to reconstruct local parent-child layout. If the parent group is not directly useful, derive the local container from the union bounds of related child nodes and keep child offsets relative to that inferred container.

Layout-sensitive nodes include buttons below forms, modal footer actions, form fields, upload/file rows, card footers, tabs, table rows, list items, and toolbar actions.

Use normal HTML flow only when it reproduces the extracted bounds within 2 px. If normal flow drifts, lock the layout inside the nearest restoration unit with absolute positioning, grid tracks, fixed flex-basis, or explicit spacer dimensions derived from extracted bounds.

Do not tune spacing by arbitrary visual guesses when panel or selected-layer bounds are available. Use extracted bounds and sibling gaps.

After implementing a layout-sensitive node, measure the local browser bounding box and compare it to the design bounds before moving to the next unit.

## Canvas-Rendered Content

Canvas-rendered business content is not an automatic reason to use screenshot approximation.

If the page content is drawn on a canvas:

1. For CoDesign, attempt metadata JSON discovery and node lookup before blind canvas clicking
2. When image understanding is available, screenshot the canvas artboard or locked unit first and visually identify candidate modules, child blocks, and click hotspots
3. Click the visible module, block, or child element inside the canvas when validation or panel extraction is needed
4. Wait for the design platform's right-side property, style, or inspect popup/panel to appear or update
5. Read any available values from metadata JSON or that panel
6. Retry with a more precise hotspot inside the module if the first click only selects the canvas root or wrong object
7. Use the layer tree only after metadata and direct canvas module clicks fail

Only use screenshot-based estimation after those attempts fail to expose usable style data and only when image understanding is available. If image understanding is unavailable, report the missing value as unresolved or ask the user to select or confirm a more inspectable node.

## Downgrade Path

If direct clicking does not reveal stable annotation or style data:

1. For CoDesign, attempt metadata JSON discovery, parsing, and node lookup before downgrading to canvas-only probing
2. If image understanding is available, revisit the screenshot-derived click map and choose a more precise visual hotspot before using non-visual fallback
3. For canvas-rendered content, click the visible module or block inside the canvas before treating the canvas root as uninspectable
4. Retry a more precise direct click on the target hotspot
5. Wait for the right-side property, style, or inspect popup/panel to appear or update
6. Use the layer tree only as a locating aid
7. Stay within the smallest user-approved target node
8. Use computed browser styles only when the design panel does not expose the needed value
9. Use screenshot-based estimation only as a last resort when image understanding is available and report it as an unresolved or approximate difference
10. If image understanding is unavailable, ask for a more precise selection or user confirmation instead of guessing from a screenshot

Once the minimum implementable style set for the current target has been collected, stop probing and implement the unit.

## Safety

Default to style-only changes. Do not alter existing detail logic, data flow, routing, component communication, API calls, permissions, or validation behavior unless explicitly requested.
