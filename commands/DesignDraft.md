---
description: Restore a Mockplus or CoDesign design draft from Chrome into project files
argument-hint: [relative-file-path]
---

# /DesignDraft

The user invoked:

```text
/DesignDraft $ARGUMENTS
```

Use the installed `design-draft` skill behavior when available. If the host does not automatically load the skill package, follow the self-contained rules below.

## Command Rules

1. Treat the user's verbal description of the target design tab as the primary source of truth
2. If the user supplied a relative file path in `$ARGUMENTS`, use that as the restoration target
3. If the user did not specify a target Vue file, ask which Vue page file should receive the restoration before editing
4. Do not infer a target file or restore directly into `App.vue` unless the user explicitly named `App.vue`
5. If a new page is needed, create or update a route page and route registration according to project conventions
6. If the user did not specify a target tab, use the automatic fallback logic defined by the skill
7. Use `chrome-devtools-mcp` to inspect the design platform page and restore the design into workspace files
8. Default to the middle business design area and treat design-tool editor chrome as inspection UI, not restoration content
9. If the user did not explicitly request full-page restoration, also exclude the product application's own navbar, sidebar, icon rail, and persistent app shell from the default target
10. Before coding, capture the candidate main content area or identify the first restoration node and ask the user to confirm the starting point
11. After confirmation, split the approved area into restoration units
12. Before node traversal, use visual understanding when available to classify each confirmed unit's best implementation strategy: Pure DOM, Form/UI component, Chart/ECharts, Canvas, Asset/slice, or Mixed
13. Use click-driven depth-first traversal inside each unit: click every visible business node, classify it with visual understanding when available or non-visual design-tool signals when unavailable, read its own right-side styles and bounds, read parent layout and sibling gaps for layout-sensitive nodes, restore its own structure, styles, and placement, descend to children, then backtrack to unprocessed siblings
14. Verify each unit on the local page before continuing to the next unit
15. Choose the best implementation strategy from visual understanding when available; otherwise use selected layer name, hierarchy, visible text, role, dimensions, component metadata, and right-side panel values
16. Optimize code and style organization for the current project stack without reducing one-to-one visual fidelity
17. Default to style-only restoration and preserve existing business logic, data flow, routing, component communication, API calls, permissions, and validation behavior
18. Run capability preflight before restoration; if image capability is unknown, set `imageUnderstanding = unavailable`

## Self-Contained Execution Rules

Required capability:

1. `chrome-devtools-mcp` must be available
2. A supported design draft tab must be open in Chrome, or the user must identify a target draft tab
3. The workspace must be writable

Multimodal capability:

1. Do not assume the active model supports image understanding, screenshot reading, OCR, or visual comparison
2. Set `imageUnderstanding = available` only when the host explicitly supports image input and the current agent can safely reason over image attachments
3. If capability is unknown, set `imageUnderstanding = unavailable`
4. Never run a screenshot/image-understanding probe that may crash a text-only host
5. Never stop solely because screenshot/image understanding is unavailable
6. If image understanding is unavailable, continue with right-side panel data, selected layer name, hierarchy, visible text, DOM text, accessibility tree, dimensions, coordinates, element role, and component metadata
7. If no element is selected and image understanding is unavailable, ask the user to select or confirm the target element instead of guessing from a screenshot
8. If strategy or boundary remains ambiguous, ask the user to confirm rather than terminating the task

Data source priority:

1. Right-side property, style, inspect, and CSS panel values readable through Chrome MCP
2. Selected layer name, layer hierarchy, artboard/page name, and design-tool metadata
3. DOM text, accessibility snapshot text, and visible text from Chrome MCP
4. Element dimensions, coordinates, constraints, and role metadata
5. Existing project component conventions and local implementation patterns
6. User confirmation for ambiguous boundaries, classifications, or asset matches
7. Screenshot understanding only when `imageUnderstanding = available`
8. Screenshot-based estimation only as a last resort and only when `imageUnderstanding = available`

Right-side CSS extraction:

1. Extract right-side property, style, inspect, and CSS panel values from Chrome MCP-readable page content whenever possible
2. Use accessibility snapshot text, DOM text, selected node metadata, `evaluate_script` output, or copyable CSS text exposed by the design platform
3. Do not use image understanding, screenshot OCR, or visual estimation as the primary method for extracting CSS values from the right-side panel
4. If the panel visibly contains values but the first MCP snapshot does not expose them, inspect the right-panel DOM, accessibility tree, selected-node state, or script-readable text before declaring extraction unavailable
5. On CoDesign, all CSS style extraction must use the CoDesign DOM recipe: use `aside.screen-inspector.inspector.expanded` as the right panel root, locate the `section` whose class contains `node-box`, click the `CSS` span under `.css-node__codes--navs`, then extract each declaration from `mark.textContent.trim()` under the first `.css-node__codes .css-node__code--item`
6. On CoDesign, use generic right-panel fallback only after the CoDesign DOM recipe fails and after retrying element selection and CSS tab activation

Ambiguity handling and recovery:

1. Never terminate restoration solely because a visual, screenshot, OCR, or exact panel extraction step is unavailable
2. Retry ambiguous targets with a more precise element click
3. Check the layer tree, selected-node metadata, right-side panel DOM text, accessibility text, and script-accessible panel content
4. Ask one concise question or request one concrete user action when tool-readable information is insufficient
5. Allowed requests include asking the user to click/select the target element, confirm the restoration start, choose between slice candidates, or confirm a missing value after extraction fails
6. Mark values as unresolved only after these attempts fail

Evidence logging:

1. Track important layout and style decisions by source: `panel`, `layer`, `dom`, `script`, `project`, `user`, or `estimated`
2. Use `estimated` only for screenshot-based estimation when `imageUnderstanding = available`
3. Report unresolved or estimated values in the final output
4. For layout-sensitive nodes, track selected bounds, parent layout, parent bounds, sibling order, sibling gaps, and local unit origin
5. Mark placement evidence as `panel+layout` or `script+layout` when final position depends on both node CSS and parent/sibling geometry

Forbidden shortcuts:

1. Do not require image understanding for starting point detection
2. Do not require screenshot reading for node traversal
3. Do not use screenshot OCR as the main way to read right-side CSS
4. Do not use visual estimation when Chrome MCP-readable values exist
5. Do not guess a target Vue file when the user did not specify one
6. Do not restore product navbar/sidebar unless explicitly requested
7. Do not stop only because image understanding is unavailable
8. Do not invent exact CSS values that were not extracted, computed, or confirmed

Target tab selection:

1. `current tab`, `current open tab`, and `foreground tab` mean the real Chrome foreground tab, not the MCP-selected tab context
2. `first Mockplus tab`, `second Mockplus tab`, `first CoDesign tab`, and similar phrases are resolved in current Chrome window tab order
3. If the user clearly identifies a tab, use that tab and do not run automatic tab selection
4. If the user's description is ambiguous, ask for clarification before continuing
5. Only if the user does not specify a target tab at all, enumerate tabs through DevTools MCP and use automatic fallback selection
6. In fallback mode, if the current foreground tab is a supported Mockplus or CoDesign page, ask: `检测到你当前打开的是设计稿页面，是否直接使用当前页签中的设计稿？`

Platform detection:

1. Prefer URL and title signals for Mockplus, `摹客`, CoDesign, or Tencent design collaboration wording
2. Confirm the page has a central canvas or artboard and inspectable style/property UI
3. Prefer strong multi-signal matches over weak single-signal matches

Target file selection:

1. Use the relative file path supplied in `$ARGUMENTS` when present
2. If no target Vue file was supplied, stop and ask the user to choose or name the target Vue page file
3. Do not infer the target file from existing routes or feature files when the user has not specified one
4. Do not write into `App.vue` unless the user explicitly names `App.vue`
5. If the user wants a new page, create or update a route page and route registration according to project conventions

Restoration workflow:

1. Run capability preflight and set `imageUnderstanding`
2. Identify the middle business design area
3. Treat left and right design-tool editor chrome as inspection UI, not restoration content
4. Unless the user explicitly requested full-page restoration, filter out the product app shell such as navbar, sidebar, icon rail, menu rail, avatar/account areas, and global actions
5. Treat every visible business node inside the remaining main content area as part of the restoration target
6. Inspect the local project stack and installed UI libraries before writing code
7. Capture the candidate main content area or explicitly identify the first restoration node, then ask the user to confirm the starting point
8. Do not write implementation code until the user confirms the starting point
9. If the user narrows the scope to a specific element, immediately narrow the execution scope to that exact element
10. Split the confirmed area into restoration units such as toolbar, filter form, content card group, table, chart, modal, empty state, and footer
11. Traverse each unit with click-driven depth-first search
12. If image understanding is available, screenshot each unit and classify the unit's best implementation strategy before node traversal
13. For every visited node, click it, capture a screenshot of the node or closest bounding region only when image input is supported or when a confirmation artifact is useful, classify it visually when available or from non-visual design-tool signals when unavailable, read its own right-side style details and selected bounds, track evidence sources, create or update its code structure, and apply its exact styles
14. For layout-sensitive nodes, also read parent CSS, parent bounds, previous and next sibling bounds, sibling gaps, and the local unit origin before implementing placement
15. If the content is canvas-rendered, click visible modules or child blocks inside the canvas and wait for the right-side property/style popup or panel before deciding style extraction is unavailable
16. Choose implementation strategy from visual understanding when available; otherwise use layer names, hierarchy, visible text, role, dimensions, component metadata, and right-side panel values
17. Descend into the first child until the deepest visible node is restored
18. Backtrack to the nearest unprocessed sibling, then descend again if that sibling has children
19. If no sibling exists, move up to the parent and look for the parent's next unprocessed sibling
20. Continue until every visible business node in the unit has been clicked, inspected, and restored
21. Verify the current unit against the draft on the local page before continuing to the next unit

Strategy selection:

1. If image understanding is available, screenshot the confirmed area or locked unit and classify the whole region before traversing child nodes
2. Pure DOM: ordinary layout, text, cards, lists, tables, simple tags, separators, simple icons, and static UI blocks; use HTML/CSS with exact CSS extraction
3. Form/UI component: input, textarea, select, checkbox, radio, date picker, upload, switch, segmented control, and button workflows; use the current project's installed UI component library when possible
4. Chart/ECharts: axes, legends, series, KPI visualization, pie, bar, line, radar, map, and other data visualizations; prefer ECharts when available or authorized
5. Canvas: dense custom graphics, complex visual effects, highly irregular drawing, or regions where DOM/component restoration would be brittle; use canvas only for that region
6. Asset/slice: photos, logos, bitmap illustrations, complex icons, exported images, and image-like backgrounds; search slice/artifact sources before approximating
7. Mixed: split the region into smaller units and assign each subregion its own strategy
8. If image understanding is unavailable, choose from selected layer name, hierarchy, visible text, role, dimensions, component metadata, and right-side panel values
9. Do not add a new dependency for ECharts, canvas helper libraries, or UI components without asking unless the project already uses it or the user has authorized dependency changes

Style conversion:

1. Preserve exact numeric values whenever possible
2. Prefer extracted right-side panel values over screenshot estimation
3. Do not use screenshot estimation for canvas-rendered content until visible module clicks, precise hotspot retries, and layer-tree fallback fail to expose right-side style data
4. Treat Chrome MCP-readable right-side panel CSS as the source of truth when available
5. Use screenshot estimation only when image understanding is available; otherwise mark the value unresolved or ask for a more precise selection or confirmation
6. Do not treat a selected node's CSS block as sufficient for final placement; placement must also use selected bounds, parent layout, parent bounds, sibling bounds, and sibling gaps when available
7. Use normal flow only when it reproduces extracted bounds within 2 px; if it drifts, lock the key block inside the nearest restoration unit with absolute positioning, grid tracks, fixed flex-basis, or explicit spacers derived from extracted bounds
8. Keep coordinate locking local to the restoration unit and do not use page-wide absolute positioning unless the selected design unit itself is page-level
9. Convert CSS into the target project's existing style mechanism, such as scoped CSS, CSS modules, utility classes, inline style objects, or preprocessors
10. Optimize style organization for the current stack without changing the visual result

Expected result:

1. The design source tab used
2. The target file or files updated
3. A concise summary of what was restored
4. Whether `imageUnderstanding` was available or unavailable
5. Whether right-side panel styles were extracted through Chrome MCP-readable content
6. Which key values came from `panel`, `layer`, `dom`, `script`, `project`, `user`, or `estimated` sources
7. Whether assets were downloaded, reused, approximated, or replaced with placeholders
8. Estimated restoration ratio
9. Any unresolved visual gaps or regions that could not be fully inspected

Slice rule:

1. When a selected element is likely a slice, search the `切图` design draft for a matching slice before approximating it
2. Download matched slices into `src/assets/images`
3. Import downloaded slices from the target Vue file or component
4. If image understanding is unavailable, compare slice candidates by selected layer name, export name, dimensions, role, surrounding text, and artboard context; ask the user to confirm when multiple candidates remain plausible
