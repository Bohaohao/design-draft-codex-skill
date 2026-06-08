---
name: design-draft
description: Use this skill when the user asks to restore a Mockplus, CoDesign, or similar web design draft into project files, mentions `/DesignDraft`, asks to read the current design tab, or wants pixel-accurate restoration through chrome-devtools-mcp.
---

# DesignDraft

Restore a browser-based design draft into implementation files in the current workspace with high visual fidelity.

This skill is intended for Claude, Codex, OpenCode, and similar coding agents that can read and edit local files and access `chrome-devtools-mcp`.

## Core Command

Primary command:

```text
/DesignDraft
```

Optional target path form:

```text
/DesignDraft <relative-file-path>
```

## Start Conditions

Do not proceed unless all of the following are true:

1. `chrome-devtools-mcp` is available
2. A supported design draft tab is open in Chrome, or the user identifies a target draft tab
3. The workspace is writable

If a required condition is missing, stop and report the missing condition.

## Required References

Read these references when executing a restoration:

1. [references/restoration-workflow.md](references/restoration-workflow.md)
2. [references/content-region.md](references/content-region.md)
3. [references/pixel-restore-rules.md](references/pixel-restore-rules.md)
4. [references/style-conversion.md](references/style-conversion.md)
5. [references/asset-rules.md](references/asset-rules.md)
6. [references/unit-splitting.md](references/unit-splitting.md)
7. [references/platform-detection.md](references/platform-detection.md)
8. Platform-specific reference when applicable:
   - Mockplus: [references/mockplus.md](references/mockplus.md)
   - CoDesign: [references/codesign.md](references/codesign.md)
9. Before final output, read [references/reporting.md](references/reporting.md)

## Target Tab Rule

The user's verbal description of the target tab is the primary source of truth.

Interpret references using these defaults:

1. `current tab`, `current open tab`, and `foreground tab` mean the real Chrome foreground tab, not the MCP-selected tab context
2. `first Mockplus tab`, `second Mockplus tab`, `first CoDesign tab`, and similar phrases are resolved in current Chrome window tab order
3. If the user clearly identifies a tab, use that tab and do not run automatic tab selection
4. If the user's description is ambiguous, ask for clarification before continuing
5. Only if the user does not specify a target tab at all may you use automatic tab selection as a fallback

Fallback selection:

1. Enumerate tabs in the current Chrome window
2. If the current foreground tab is a supported Mockplus or CoDesign page, ask: `检测到你当前打开的是设计稿页面，是否直接使用当前页签中的设计稿？`
3. If not confirmed, choose the first matching supported design draft tab
4. If no matching design draft tab exists, stop and report that no supported design draft page is open

## Supported Platform Detection

Use layered detection in this order:

1. URL pattern
2. Page title
3. Visible interface text
4. Stable editor structure

Strong Mockplus signals:

1. URL or title includes `mockplus`, `Mockplus`, or `摹客`
2. The interface exposes a central canvas with design-tool chrome
3. Selecting an element reveals property or CSS information on the right side

Strong CoDesign signals:

1. URL or title includes `codesign`, `co-design`, `CoDesign`, or Tencent design collaboration wording
2. The interface exposes a central canvas with design-tool chrome
3. Selecting an element reveals property or CSS information on the right side

Generic fallback signals:

1. There is a central artboard or canvas area
2. The page is an editor, not just a published preview
3. The right-side property panel can reveal inspectable styles after element selection

## Target File Rule

Select the output file in this order:

1. If the user provided a relative file path, use that path
2. If the user did not provide a target Vue file, stop and ask which Vue page file should receive the restoration
3. Do not guess a target file from project context when the user has not specified one
4. Do not restore directly into `App.vue` unless the user explicitly names `App.vue` as the target
5. For new pages, create or update a route page and its route registration according to the current project conventions instead of putting the restored page directly in `App.vue`

When the user chooses an existing or new route page, prefer:

1. Existing route page files
2. Existing feature entry files
3. Existing component directories
4. Existing naming and UI-library conventions

## Restoration Area Rule

By default, restore only the middle business design area.

Treat these as design-tool editor chrome, not restoration content:

1. Left page tree, layer tree, asset tree, and workspace navigation
2. Right property, inspect, style, and CSS panels
3. Design-tool navbars, toolbars, comments, and floating assistants

Use editor chrome only when it is needed for inspection or style extraction. Do not recreate editor chrome as part of the output UI.

If the user does not explicitly request full-page restoration, also exclude the product application's own persistent shell from the default restoration target:

1. Product navbar, header, or top app bar
2. Product sidebar, icon rail, menu rail, or fixed navigation
3. Account, tenant, notification, avatar, and global action areas
4. Other repeated app-shell regions that are not the requested page's main content

The default restoration target is the remaining main content area after excluding design-tool chrome and product app shell. Restore product navbar/sidebar only when the user explicitly asks to restore the full page or names those shell regions as part of the target.

Inside the selected main content area, every visible business node is part of the restoration target, including containers, groups, cards, controls, text, images, icons, toolbars, filters, forms, tables, charts, lists, modals, and empty states. If it is unclear whether a region belongs to product shell or page content, ask the user before restoring that region.

## Starting Point Confirmation

Do not write implementation code until the starting point is confirmed.

Before restoration starts:

1. Understand the user's requested scope
2. Identify the effective business content area that corresponds to that scope, excluding product navbar/sidebar unless full-page restoration was explicitly requested
3. Capture a screenshot of the candidate area or explicitly identify the first restoration node
4. Ask the user to confirm whether the starting point is correct
5. Start code changes only after confirmation

If the user narrows the scope to a specific element, immediately narrow the execution scope to that element. Do not continue restoring a parent container or whole page unless the user expands the scope again.

## Safety Boundary

Default to style-only restoration.

Do not change existing business logic, data flow, routing, component communication, API calls, permissions, or validation behavior unless the user explicitly requests it. Minimal wrapper structure changes are allowed only when the current DOM cannot support the draft layout.

## Component Mapping Rule

Before writing code for a clicked design element:

1. Understand the element's semantic role
2. Inspect the current project for installed UI libraries and conventions
3. Use the most suitable existing project-native or UI-library primitive when appropriate
4. Create class hooks first so exact styles can be applied afterward

Examples in an Element Plus project:

1. Prefer `el-button` for button-like controls
2. Prefer `el-input` for input-like controls
3. Prefer `el-card` for card-like shells when the result stays faithful
4. Prefer `el-menu` for menu-like navigation when that matches the draft
5. Fall back to custom wrappers when stock components cannot preserve fidelity

## Capability Preflight

Before restoration starts, determine the execution mode.

Do not assume the active model supports image understanding, screenshot reading, OCR, or visual comparison.

Set `imageUnderstanding = available` only when the host explicitly supports image input and the current agent can safely reason over image attachments. If capability is unknown, set `imageUnderstanding = unavailable`.

Never run a screenshot/image-understanding probe that may crash a text-only host.

When `imageUnderstanding = unavailable`, skip screenshot-dependent classification, screenshot OCR, visual comparison, and screenshot-based estimation. Continue with Chrome MCP-readable content, selected design-tool metadata, project conventions, and user confirmation.

When `imageUnderstanding = available`, use it as an enhancement for classification, boundary confirmation, complex visual comparison, and final visual verification. Do not let image understanding replace exact panel extraction, DOM-readable data, or user-confirmed boundaries.

## Data Source Priority

Use this priority for exact style values on general design platforms:

1. Right-side property, style, inspect, and CSS panel values readable through Chrome MCP
2. Selected layer name, layer hierarchy, artboard/page name, and design-tool metadata
3. DOM text, accessibility snapshot text, and visible text from Chrome MCP
4. Element dimensions, coordinates, constraints, and role metadata
5. Existing project component conventions and local implementation patterns
6. User confirmation for ambiguous boundaries, classifications, or asset matches
7. Screenshot understanding only when `imageUnderstanding = available`
8. Screenshot-based estimation only as a last resort and only when `imageUnderstanding = available`

Use this CoDesign-specific priority before blind canvas probing:

1. CoDesign metadata JSON from `design/screensMap` and `meta_url`: screen metadata, node tree, `rect`, `realRect`, `css`, `fragments`, `fills`, `borders`, `effects`, `radius`, and parent-child structure
2. CoDesign right-side panel DOM/CSS extraction through the recipe in [references/codesign.md](references/codesign.md)
3. Selected node metadata, DOM text, accessibility information, and layer signals
4. Canvas click validation
5. User confirmation
6. Visual estimation only as the last resort and only when `imageUnderstanding = available`

If CoDesign metadata JSON gives an exact value, do not replace it with screenshot guessing. Treat metadata JSON and the right-side panel as exact sources that can cross-check each other. Screenshot or visual understanding may help discover and confirm candidates, but it must not override exact metadata or panel values.

## CoDesign Metadata-First Rule

When the design source is CoDesign, use metadata-first restoration as a standard path, not as an ad-hoc fallback.

Before large-scale canvas clicking:

1. Read [references/codesign.md](references/codesign.md)
2. Discover the target screen from runtime state such as `window.$nuxt?.$store?.getters['design/screensMap']` or related `design/screen` state
3. Match the screen by user-specified screen name, artboard name, selected module, or current screen context
4. Record `id`, `name`, `meta_url`, `frame`, `preview_path`, `object_id`, and `version`
5. If `meta_url` exists, acquire the metadata JSON; if in-page `fetch(meta_url)` is blocked, open `meta_url` in a separate Chrome MCP page and parse `document.body.innerText`
6. Traverse metadata node fields such as `layers`, `children`, and `nodes`
7. Index node identifiers including `id`, `object_id`, `master_id`, and `node.id`, and preserve `parent_id` relationships
8. Use metadata for screen location, node discovery, exact layout/style values, text fragments, and parent-child layout reconstruction
9. Fall back to canvas clicking, right-panel probing, and user confirmation only when metadata is missing, blocked, ambiguous, or insufficient

Metadata-first does not weaken the CoDesign right-panel CSS rule. Continue to use the deterministic right-panel DOM recipe for selected-node CSS when panel verification is needed or when metadata and implementation require cross-checking.

Use a different priority only for discovering clickable design elements and hotspot candidates:

1. On CoDesign, run metadata JSON discovery first when available; use visual click maps after metadata is missing, ambiguous, or needs canvas validation.
2. If `imageUnderstanding = available`, capture the confirmed artboard or locked unit and use image understanding first to identify visible modules, child nodes, likely groups, repeated items, and safe click hotspots.
3. Convert visual candidates into a click queue with names, approximate bounding boxes, and center or semantic click points.
4. Validate every candidate by clicking it through Chrome MCP and checking selected-node metadata, layer name, right-side panel updates, or CSS extraction results.
5. Keep candidates only when the click selects the intended design node or reveals a useful right-side panel. Retry with a more precise hotspot when validation fails.
6. If `imageUnderstanding = unavailable`, discover click targets from metadata JSON, selected layer data, layer tree, DOM text, accessibility text, right-side panel metadata, and user-confirmed points.

This clickable-target priority does not lower the priority of panel-extracted CSS. Visual understanding may choose where to click, but it must not be treated as the source of exact CSS values.

## Multimodal Capability Rule

Do not assume the active model can inspect images or screenshots.

Image understanding is optional. Use it when the host model supports image input. Never stop solely because screenshot/image understanding is unavailable.

If image understanding is unavailable, continue with right-side panel and non-visual inspection signals:

1. User-selected starting element
2. CoDesign metadata JSON when the platform is CoDesign
3. Selected layer name, layer hierarchy, and artboard or page name
4. Right-side property, style, inspect, or CSS panel values
5. Visible text from Chrome MCP snapshots, DOM text, and accessibility tree
6. Element role, component metadata, dimensions, constraints, and coordinates
7. Browser computed styles when inspecting an implemented local page
8. User confirmation for ambiguous boundaries or strategy choices

For starting point detection:

1. If the user clicked or selected a start element in the design tool, use the selected layer name, hierarchy, coordinates, and right-side panel as the starting point candidate
2. Ask the user to confirm the selected node when the business boundary is ambiguous
3. If no element is selected and image understanding is unavailable, ask the user to select or confirm the target element instead of guessing from a screenshot

For node classification without image understanding:

1. Form, input, select, checkbox, radio, date, search, and button names or metadata indicate project component-library implementation
2. Table, list, card, toolbar, filter, modal, drawer, tab, and layout names or text indicate ordinary HTML/CSS or existing UI-library components
3. Axis, legend, series, chart, pie, line, bar, and metric labels indicate ECharts when available or authorized
4. Image, bitmap, export, slice, logo, icon, and background metadata indicate asset resolution
5. If the strategy remains unclear, ask the user a concise confirmation question rather than terminating the task

## Right Panel Extraction Rule

Right-side property, style, inspect, and CSS panel values must be extracted from Chrome MCP-readable page content whenever possible.

Accepted extraction sources:

1. Accessibility snapshot text from the right-side panel
2. DOM text from the right-side panel
3. Selected node metadata exposed by the design tool page
4. `evaluate_script` results that read panel text, attributes, computed data, or design-tool state
5. Copyable CSS text exposed by the design platform

Do not use image understanding, screenshot OCR, or visual estimation as the primary method for extracting CSS values from the right-side panel.

Screenshots may support human confirmation or element classification, but right-side CSS values must come from Chrome MCP-readable content when that content exists. If panel values are visible but not directly readable through the first snapshot, probe DOM text, accessibility tree output, selected-node metadata, and evaluated script output before using any downgrade path.

## Ambiguity Handling

Never terminate restoration solely because a visual, screenshot, OCR, or exact panel extraction step is unavailable.

When a required value, boundary, classification, or asset match is ambiguous:

1. For CoDesign, check metadata JSON by screen, node name, content, fragments, rect, path, and parent-child relationships
2. Retry with a more precise element click
3. Check the layer tree and selected-node metadata
4. Check right-side panel DOM text and accessibility text
5. Use `evaluate_script` to read script-accessible panel content or design-tool state
6. Ask the user to select or confirm the target element, boundary, value, or candidate
7. Mark the value as unresolved only after these attempts fail

Do not invent exact numeric values.

## User-Assisted Recovery

When tool-readable information is insufficient, ask one concise question or request one concrete action.

Allowed recovery requests:

1. Ask the user to click or select the target element in the design draft
2. Ask the user to confirm whether the selected node is the restoration start
3. Ask the user to choose between multiple matching slice candidates
4. Ask the user to provide or confirm a missing value only after panel, DOM, metadata, and script extraction fail

Do not ask broad questions when a specific click, selection, or confirmation would resolve the ambiguity.

## Evidence Logging

For important layout and style decisions, track the source:

1. `metadata`: extracted from CoDesign metadata JSON, including screen data, node tree, exact geometry, style fields, text fragments, and parent-child relationships
2. `panel`: extracted from right-side property, style, inspect, or CSS panel
3. `layer`: inferred from selected layer name or hierarchy
4. `dom`: read from DOM text or accessibility snapshot
5. `script`: read through evaluated script output that is not specifically CoDesign metadata JSON
6. `project`: matched from existing project conventions
7. `user`: confirmed by the user
8. `estimated`: estimated from screenshot, only when `imageUnderstanding = available`

`metadata` is an exact machine-readable source. It is not screenshot OCR and must not be reported as `estimated`. It may be combined with `panel` evidence when CoDesign metadata JSON and right-panel CSS confirm the same value.

Report unresolved or estimated values in the final output.

For layout decisions, evidence must include more than the clicked node's own CSS when the node participates in a group, form, card, list, toolbar, table, or modal:

1. Record the selected node bounds from the design panel or selected-layer metadata: `x`, `y`, `width`, and `height`
2. Record the parent container's layout CSS, including display, flex direction, grid tracks, padding, gap, alignment, border box, and fixed dimensions when available
3. Record sibling order and sibling-to-sibling gaps for vertical or horizontal stacks
4. Record the nearest positioned ancestor or restoration unit coordinate system
5. Mark a value as `metadata+layout`, `panel+layout`, or `script+layout` when the final placement depends on both node CSS and parent/sibling geometry

## Forbidden Shortcuts

Do not:

1. Require image understanding for starting point detection
2. Require screenshot reading for node traversal when the host lacks image understanding
3. Use screenshot OCR as the main way to read right-side CSS
4. Use visual estimation when Chrome MCP-readable values exist
5. Blind-click CoDesign canvas pages before attempting available metadata JSON discovery
6. Guess a target Vue file when the user did not specify one
7. Restore product navbar/sidebar unless explicitly requested
8. Stop only because image understanding is unavailable
9. Invent exact CSS values that were not extracted, computed, or confirmed

## Visual Understanding Strategy

After selecting a design element, capture a screenshot of the selected element or the smallest available bounding region around it only when the host model supports image input or when the screenshot is useful as a human-facing confirmation artifact.

When `imageUnderstanding = available`, use visual understanding at the region level before node traversal. Screenshot the confirmed restoration area or locked unit, then classify the visible content and choose the best implementation strategy for that unit before implementing child nodes.

Region strategy classes:

1. Pure DOM: ordinary layout, text, cards, lists, tables, simple tags, separators, simple icons, and static UI blocks; restore with HTML/CSS and exact panel extraction
2. Form/UI component: input, textarea, select, checkbox, radio, date picker, upload, switch, segmented control, and button workflows; prefer the current project's component library when it preserves fidelity
3. Chart: axes, legends, series, KPI visualization, pie, bar, line, radar, map, and other data visualizations; prefer ECharts when available or authorized, then map visible chart type, axes, labels, colors, and layout
4. Canvas: dense custom graphics, complex visual effects, highly irregular drawing, or regions where DOM/component restoration would be brittle; use canvas only for that region
5. Asset/slice: photos, logos, bitmap illustrations, complex icons, exported images, and image-like backgrounds; search the design draft slice/artifact source before approximating
6. Mixed: split the region into smaller units and assign each subregion its own strategy

The region classification controls the restoration plan, traversal units, and implementation primitives. It does not replace exact CSS extraction, selected bounds extraction, parent layout extraction, or asset lookup.

When image understanding is available, use visual understanding first to discover clickable candidates inside the confirmed restoration area or locked unit. Build a candidate map of visible modules, child controls, text blocks, icons, cards, images, tables, and repeated items. For each candidate, record the visible role, approximate bounds, and a click hotspot. Then click candidates with Chrome MCP and validate selection through right-side panel or selected-layer signals before extracting styles.

When image understanding is available, use visual understanding to classify the element before choosing an implementation strategy. When image understanding is unavailable, classify the element from right-side panel values, selected layer name, hierarchy, text, role, dimensions, coordinates, and component metadata.

Visual understanding is allowed to guide click order, hotspot selection, classification, and implementation strategy. It is not allowed to replace exact right-side style extraction. After every successful click, read styles from the platform panel or DOM-readable metadata.

Classification and implementation strategy:

1. Pure DOM: restore with click-driven depth-first node traversal, exact style extraction, and ordinary HTML/CSS
2. Form/UI component: use the current project's component library when possible, such as Element Plus, Ant Design, or an equivalent installed UI library
3. Chart: prefer ECharts and map the visible chart type, axes, legend, labels, colors, and layout as closely as possible
4. Canvas: use a canvas-based implementation only for the region that is too complex for reliable DOM, UI components, or ECharts
5. Asset/slice: resolve image-like nodes through slice lookup, download, and project asset import before drawing approximations
6. Mixed: split into smaller units and apply the matching strategy to each subregion

Strategy constraints:

1. Visual understanding guides the implementation strategy, but it must not replace right-side style extraction when exact style values are available
2. Component-library, ECharts, and canvas choices must still preserve visual fidelity as the top priority
3. Follow the current project's installed dependencies and conventions before introducing a new dependency
4. If ECharts or a canvas helper is needed but the project does not already include it, ask before adding the dependency unless the user already authorized dependency changes

## Style Fidelity Rule

The goal is one-to-one restoration.

Restoration priority:

1. Layout hierarchy
2. Positioning and spacing
3. Width and height
4. Typography
5. Colors and backgrounds
6. Borders, radius, and shadows
7. Visible text and imagery

If the design platform reveals element CSS or style details after clicking an element:

1. Click the element
2. Read the right-side style panel
3. Extract exact style values whenever possible
4. Convert them into the target project's supported style format
5. Reproduce them with minimal loss of fidelity

When the platform is CoDesign, all CSS style extraction must use the platform-specific right panel CSS extraction recipe in [references/codesign.md](references/codesign.md). The CoDesign recipe uses `aside.screen-inspector.inspector.expanded`, `.node-box`, `.css-node__codes--navs`, the `CSS` tab, `.css-node__codes .css-node__code--item`, and `mark.textContent` to extract exact declarations from the DOM. Use generic right-panel fallback only after this CoDesign DOM recipe fails and after retrying selection and CSS tab activation.

This applies to every visited node, not just leaf nodes. Container, group, layout, text, image, icon, and atomic control nodes all need their own clicked style inspection and restoration.

Canvas-rendered drafts are still inspectable targets. If business content appears inside a canvas, do not skip style extraction and do not jump directly to screenshot approximation. Click the visible module, block, or child element inside the canvas to trigger the design platform's right-side property/style popup or panel, then read the available values. Use screenshot-based approximation only after direct canvas module clicks, more precise hotspot retries, and layer-tree fallback fail to expose usable style data.

## Layout Coupling Rule

Do not treat a node's own CSS block as sufficient for pixel placement.

Many design platforms expose CSS for the currently selected node only. Browser output is also affected by parent layout, sibling gaps, text line boxes, font fallback, and natural document flow. For layout-sensitive nodes, restore placement from a coupled layout dataset:

1. The node's own CSS declarations
2. The node's design bounds: `x`, `y`, `width`, and `height`
3. The parent container's CSS and bounds
4. The previous and next visible sibling bounds
5. The measured gap between adjacent siblings
6. The restoration unit's coordinate origin

Layout-sensitive nodes include buttons below forms, form fields, textarea blocks, upload/file rows, card footers, tabs, segmented controls, table headers, list rows, toolbar actions, modal footers, and any node whose visual position depends on surrounding content.

When implementing:

1. Prefer normal flow only when the parent layout CSS, fixed dimensions, padding, gaps, and sibling sizes reproduce the extracted bounds within 2 px
2. If normal flow produces visible drift, lock the key block inside the nearest restoration unit using absolute positioning, CSS grid tracks, or fixed flex basis derived from design bounds
3. Keep the locked coordinate system local to the current restoration unit; do not make page-wide absolute positioning unless the selected unit itself is page-level
4. Preserve the extracted node CSS after choosing the placement strategy
5. Re-measure the local page with Chrome MCP and compare implemented bounding boxes against the design bounds before moving on
6. Report any remaining layout drift greater than 2 px as a remaining difference

Do not "fix" spacing by arbitrary visual nudging when panel or selected-layer bounds are available. Use extracted parent/child coordinates and sibling gaps as the source of truth.

## Traversal Rule

Traverse each approved restoration unit with a click-driven depth-first search.

Each node must be discovered, classified, inspected, implemented, and styled before traversal moves on. For CoDesign, metadata JSON may provide the first traversal tree and parent-child structure; click and right-panel validation are still used when a node needs panel CSS verification, asset export, or ambiguity resolution. When image understanding is available, use it first to propose the node traversal candidates and hotspots inside the locked unit. Classification may use visual understanding when available, or non-visual design-tool signals when image understanding is unavailable.

For each target element:

1. If the platform is CoDesign and metadata JSON is available, find the next candidate in the metadata tree by name, content, fragments, parent path, rect, or sibling relationship
2. If image understanding is available, screenshot the locked unit and generate a visual click map before clicking child nodes
3. Select the next candidate element in the draft by clicking its validated hotspot when panel validation or export is needed
4. Confirm the click selected the intended node or revealed useful panel information
5. Capture a screenshot of the selected element or its closest available bounding region when image input is supported or when a confirmation artifact is useful
6. Classify the selected element with visual understanding when available; otherwise use selected layer name, hierarchy, right-side panel data, visible text, element role, dimensions, and metadata
7. Choose the implementation strategy: project component library, ordinary HTML/CSS restoration, ECharts, or canvas
8. Understand its semantic role
9. Read exact style details from CoDesign metadata JSON, Chrome MCP-readable panel, or other platform metadata
10. Read selected bounds, parent layout, sibling order, and sibling gaps when the node is layout-sensitive
11. Create the implementation structure in code
12. Apply styles and placement in project-supported format
13. Measure the local implemented node and compare it with design bounds when practical
14. Optimize code and style organization without reducing visual fidelity
15. Traverse into the first visible child element when children exist
16. Continue descending until the deepest visible child node is reached
17. Return to the nearest unprocessed sibling
18. Move up to the parent only when no sibling remains at the current level
19. Continue until all nodes in the selected restoration area have been discovered, inspected, and restored

When the visible node is rendered on canvas, `Select the element` means clicking the corresponding visible module in the canvas, not selecting only the canvas root element.

Required traversal order example:

```text
A -> B -> B1 -> B2 -> C -> C1 -> C2
```

## Unit Restoration Rule

Split the approved design area into restoration units before coding. Typical units are header, sidebar, toolbar, filter form, content card group, table, chart, modal, empty state, and footer.

For each unit:

1. State the currently locked target node or unit
2. Confirm the selected node or boundary when needed
3. Inspect real element styles in the design tool
4. Extract parent-child layout relationships, node bounds, and sibling gaps for layout-sensitive parts
5. Track evidence sources for key layout and style decisions
6. Resolve asset usage
7. Implement only that unit with the current project's conventions
8. Switch to the local page and verify the result before moving to the next unit

Do not continue to the next unit if the current unit has not been checked against the draft as far as the browser and project allow.

## Standard Execution Flow

Follow this sequence:

1. Confirm `chrome-devtools-mcp` is connected
2. Run Capability Preflight and set `imageUnderstanding`
3. Resolve the target design tab using the user-description-first rule
4. Resolve the target Vue file path from explicit user input; if it is missing, ask before continuing
5. If the target platform is CoDesign, discover target screen metadata, acquire `meta_url`, open and parse metadata JSON, and use metadata-driven node discovery before canvas probing
6. Identify the middle design area and treat editor chrome as inspection UI, not restoration content
7. Capture the candidate business area or identify the first restoration node
8. Ask the user to confirm the starting point
9. Treat every visible business node inside the confirmed area as part of the restoration target
10. Read the current project context to determine framework, style mechanism, and installed UI library
11. Split the confirmed area into restoration units
12. For each unit, use metadata-driven traversal when available and click-driven depth-first traversal when panel validation is needed; classify, inspect, and restore every node at every level; use screenshots and visual understanding only when supported
13. Write or update the target file incrementally
14. Verify each unit against the draft on the local page before continuing
15. Once base draft restoration is complete, handle any extra user-requested behavior
16. Report files changed, asset usage, evidence sources, style extraction coverage, restoration ratio, and remaining gaps

## Final Output Rule

At the end of execution, report:

1. Which tab was used as the design source
2. Which file or files were created or updated
3. Whether `imageUnderstanding` was available or unavailable
4. Whether styles were extracted from the right-side panel through Chrome MCP-readable content
5. Whether CoDesign metadata JSON was used, which units it covered, and which exact values came from it
6. Which key values came from `metadata`, `panel`, `layer`, `dom`, `script`, `project`, `user`, or `estimated` sources
7. Whether assets were downloaded, reused, approximated, or replaced with placeholders
8. Which units were restored
9. Estimated restoration ratio
10. Any regions that could not be fully inspected or restored
11. Any remaining differences or unresolved ambiguities

When a selected element is likely a slice, search the `切图` design draft for a matching slice before approximating it. Download matched slices into `src/assets/images` and import them from the target Vue file.
