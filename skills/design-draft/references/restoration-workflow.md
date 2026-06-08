# Restoration Workflow

Follow this sequence for a DesignDraft restoration.

## Gate

1. Confirm `chrome-devtools-mcp` is connected
2. Run Capability Preflight and set `imageUnderstanding`
3. Resolve the design source tab using the user-description-first rule
4. Resolve the explicit target Vue file path
5. If the user did not specify a target Vue file, stop and ask which Vue page file should receive the restoration
6. Inspect the project stack, style mechanism, routes, and installed UI libraries
7. Identify the middle business design area
8. If the design source is CoDesign, discover target screen metadata from runtime state, acquire `meta_url`, and parse metadata JSON before blind canvas probing
9. Capture the candidate area or identify the first restoration node
10. Ask the user to confirm the starting point before writing implementation code

Do not start code changes before the starting point is confirmed.

Do not guess a target file from project context. Do not restore directly into `App.vue` unless the user explicitly names `App.vue`. For new pages, create or update a route page and route registration according to project conventions.

If the user did not explicitly request full-page restoration, the candidate area must exclude the product application's own navbar, sidebar, icon rail, and other persistent app shell. Use the remaining main content area as the default starting point.

## Restoration

1. Treat design-tool left and right editor chrome as inspection UI, not output content
2. Treat every visible business node inside the confirmed main content area as part of the restoration target
3. For CoDesign, use [codesign.md](codesign.md) to discover target screen metadata, acquire `meta_url`, open and parse metadata JSON, and build node indexes before large-scale canvas clicking
4. Split the confirmed area into restoration units
5. Lock one unit at a time
6. When image understanding is available, screenshot the locked unit and classify the unit's best implementation strategy before traversal
7. Use the region strategy classification to decide whether the unit is Pure DOM, Form/UI component, Chart/ECharts, Canvas, Asset/slice, or Mixed
8. For CoDesign, run metadata-driven node discovery inside the locked unit by matching name, content, fragments, parent path, rect, sibling, and child relationships
9. For CoDesign, extract layout and style from metadata fields such as `rect`, `realRect`, `css`, `fragments`, `fills`, `borders`, `effects`, `radius`, transforms, and `parent_id`
10. When image understanding is available, use the same screenshot to generate clickable node candidates and hotspots
11. Traverse the locked unit with metadata-driven traversal when available and click-driven depth-first search when panel validation, export controls, or ambiguity resolution are needed
12. Click each node when needed, validate the selected node, classify it, read its own right-side styles, and read its bounds
13. For layout-sensitive nodes, also read parent layout, parent bounds, sibling order, and adjacent sibling gaps
14. Implement the node's structure, styles, and placement in code
15. Resolve assets for the node before substituting or approximating them
16. Descend into the first child until the deepest visible child node is restored
17. Backtrack to the nearest unprocessed sibling, then descend again if that sibling has children
18. If no sibling exists, move up to the parent and look for the parent's next unprocessed sibling
19. Continue until every node in the locked unit has been discovered, inspected, and restored
20. Switch to the local page and verify the unit before continuing

If a CoDesign unit is rendered inside a canvas, attempt metadata JSON first. If metadata is unavailable or insufficient and image understanding is available, use the screenshot-derived click map before non-visual probing. Continue by clicking visible modules or child blocks inside the canvas. Do not treat a canvas root hit as proof that styles are unavailable. Wait for the right-side property/style popup or panel to appear or update before applying the downgrade path.

## Layout Coupling

Do not rely on a selected node's CSS block alone for final placement.

For layout-sensitive nodes, use a coupled layout dataset:

1. Node CSS declarations
2. Node design bounds: `x`, `y`, `width`, `height`
3. Parent CSS declarations and parent bounds
4. Previous and next visible sibling bounds
5. Sibling gaps and local unit origin

Layout-sensitive nodes include buttons below forms, upload/file rows, card footers, modal footers, table rows, list items, tabs, and toolbar actions.

Start with normal flow when the extracted parent layout fully explains the position. If local browser verification shows drift greater than 2 px, switch that unit or block to a deterministic placement strategy such as absolute positioning inside the local unit, CSS grid tracks, fixed flex-basis, or explicit spacers based on extracted bounds.

Keep coordinate locking local to the restoration unit. Do not use page-wide absolute positioning unless the selected design unit itself is page-level.

## Multimodal Fallback

Do not assume the active model can inspect images or screenshots.

Set `imageUnderstanding = available` only when the host explicitly supports image input and the current agent can safely reason over image attachments. If capability is unknown, set `imageUnderstanding = unavailable`.

Image understanding is optional. Never stop solely because screenshot/image understanding is unavailable. Never run a screenshot/image-understanding probe that may crash a text-only host.

When image understanding is available, use screenshots first to classify the confirmed area or locked unit's content type and best implementation strategy. Then use screenshots to find clickable candidates and hotspots inside that unit, and validate every candidate through Chrome MCP and the right-side panel. A screenshot can also help classify the selected node. When image understanding is unavailable, continue with right-side panel and these non-visual signals:

1. User-selected starting element
2. CoDesign metadata JSON when the platform is CoDesign
3. Selected layer name, layer hierarchy, and artboard or page name
4. Right-side property, style, inspect, or CSS panel values
5. Visible text from Chrome MCP snapshots, DOM text, and accessibility tree
6. Element role, component metadata, dimensions, constraints, and coordinates
7. User confirmation for ambiguous boundaries or strategy choices

If no element is selected and image understanding is unavailable, ask the user to select or confirm the target element instead of trying to infer it from a screenshot.

## Data Source Priority

Use this order for exact style values on general design platforms:

1. Right-side property, style, inspect, and CSS panel values readable through Chrome MCP
2. Selected layer name, layer hierarchy, artboard/page name, and design-tool metadata
3. DOM text, accessibility snapshot text, and visible text from Chrome MCP
4. Element dimensions, coordinates, constraints, and role metadata
5. Existing project component conventions and local implementation patterns
6. User confirmation for ambiguous boundaries, classifications, or asset matches
7. Screenshot understanding only when `imageUnderstanding = available`
8. Screenshot-based estimation only as a last resort and only when `imageUnderstanding = available`

For CoDesign, use this platform-specific order:

1. CoDesign metadata JSON from `design/screensMap` and `meta_url`, including screen data, node tree, `rect`, `realRect`, `css`, `fragments`, `fills`, `borders`, `effects`, `radius`, and parent-child structure
2. CoDesign right-side panel DOM/CSS extraction
3. Selected node metadata, DOM text, accessibility information, and layer signals
4. Canvas click validation
5. User confirmation
6. Visual estimation only as the last resort and only when `imageUnderstanding = available`

If metadata JSON gives an exact value, do not replace it with screenshot guessing. Metadata JSON and right-panel CSS are exact sources that can cross-check each other; visual understanding only assists discovery and confirmation.

For discovering clickable elements only, image understanding has higher priority when available on general design platforms. On CoDesign, attempt metadata JSON discovery first, then use screenshots to propose click targets when metadata is unavailable, ambiguous, or needs canvas validation. Use Chrome MCP selection and panel extraction to validate click targets.

For layout placement, a node's own CSS is not enough. Parent layout, selected bounds, sibling bounds, and sibling gaps must be collected and treated as first-class evidence.

## Ambiguity Recovery

Never terminate restoration solely because a visual, screenshot, OCR, or exact panel extraction step is unavailable.

When a required value, boundary, classification, or asset match is ambiguous:

1. For CoDesign, check metadata JSON by screen, node name, content, fragments, rect, path, and parent-child relationships
2. Retry with a more precise element click
3. Check the layer tree and selected-node metadata
4. Check right-side panel DOM text and accessibility text
5. Use `evaluate_script` to read script-accessible panel content or design-tool state
6. Ask one concise question or request one concrete action from the user
7. Mark the value as unresolved only after these attempts fail

Do not invent exact numeric values.

## Strategy

Choose implementation strategy from visual understanding when available. Do this at the region or locked-unit level before node traversal, then refine it per node. Also use visual understanding to generate click candidates before traversal. Without image understanding, choose from selected layer name, text, role, dimensions, component metadata, and right-side panel values.

Region strategy classes:

1. Pure DOM: ordinary layout, text, cards, lists, tables, simple tags, separators, simple icons, and static UI blocks; restore with HTML/CSS and exact panel extraction
2. Form/UI component: input, textarea, select, checkbox, radio, date picker, upload, switch, segmented control, and button workflows; prefer the current project's component library when it preserves fidelity
3. Chart/ECharts: axes, legends, series, KPI visualization, pie, bar, line, radar, map, and other data visualizations; prefer ECharts when available or authorized
4. Canvas: dense custom graphics, complex visual effects, highly irregular drawing, or regions where DOM/component restoration would be brittle; use canvas only for that region
5. Asset/slice: photos, logos, bitmap illustrations, complex icons, exported images, and image-like backgrounds; search slice/artifact sources before approximating
6. Mixed: split the region into smaller units and assign each subregion its own strategy

Do not add new dependencies without user approval unless the dependency already exists in the project or the user already authorized dependency changes.

## Traversal Example

```text
A
|- B
|  |- B1
|  `- B2
`- C
   |- C1
   `- C2
```

Required order:

```text
A -> B -> B1 -> B2 -> C -> C1 -> C2
```

Each node in the order must be clicked, classified, inspected, and restored before moving to the next node. When image input is supported, capture screenshots to identify clickable nodes and hotspots before traversal. Exact styles still come from panel extraction, not screenshot reading.

## Finish

1. Complete base draft restoration before implementing extra requested behavior
2. Re-check the whole restored page as far as the local app allows
3. Report files changed, units restored, asset usage, style extraction coverage, restoration ratio, and remaining gaps
