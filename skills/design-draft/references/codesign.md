# CoDesign Notes

Use these notes when the selected design source appears to be Tencent CoDesign.

## Metadata-First Strategy

For CoDesign canvas pages, prefer metadata-driven restoration before blind canvas clicking.

The standard CoDesign path is:

1. Discover the target screen from runtime state
2. Read the screen's `meta_url`
3. Open and parse the metadata JSON
4. Traverse and index the node tree
5. Locate target modules from name, text, fragments, rects, and parent-child relationships
6. Extract layout and style from metadata fields
7. Cross-check selected nodes with the right-side CSS panel when needed
8. Fall back to canvas clicks only when metadata is unavailable, ambiguous, or insufficient

Metadata JSON is an exact machine-readable source. It is not screenshot OCR and it is not `estimated`.

## Detection

Strong signals:

1. URL, title, or visible product text includes `codesign`, `co-design`, `CoDesign`, or Tencent design collaboration wording
2. A central canvas or artboard is visible
3. Left panels expose layers, pages, assets, or components
4. Selecting an element updates the right-side inspect, property, style, or CSS panel

## Discover Target Screen Metadata

Use Chrome MCP `evaluate_script` on the CoDesign page to inspect runtime state before probing the canvas.

Prefer `design/screensMap`, then related `design/screen` state:

```js
() => {
  const store = window.$nuxt?.$store;
  const getters = store?.getters || {};
  const state = store?.state || {};

  const screensMap =
    getters['design/screensMap'] ||
    state.design?.screensMap ||
    state.design?.screens?.screensMap ||
    state.design?.screen?.screensMap ||
    null;

  const screens = screensMap
    ? Object.values(screensMap)
    : [
        ...(Array.isArray(state.design?.screens) ? state.design.screens : []),
        ...(Array.isArray(state.design?.screen?.screens) ? state.design.screen.screens : []),
      ];

  return screens
    .filter(Boolean)
    .map((screen) => ({
      id: screen.id,
      name: screen.name,
      meta_url: screen.meta_url,
      frame: screen.frame,
      preview_path: screen.preview_path,
      object_id: screen.object_id,
      version: screen.version,
    }));
}
```

Match the target screen in this order:

1. Exact user-specified screen or artboard name
2. Current selected or visible screen name
3. Module-owned screen from selected-node or layer metadata
4. Case-insensitive partial name match
5. `frame` or visible artboard rect match
6. Ask the user to confirm when multiple screens remain plausible

Record `id`, `name`, `meta_url`, `frame`, `preview_path`, `object_id`, and `version` as `metadata` evidence.

## Acquire Metadata JSON

If the target screen has `meta_url`, treat it as a primary CoDesign data source.

Try in-page fetch first:

```js
async (metaUrl) => {
  try {
    const response = await fetch(metaUrl, { credentials: 'include' });
    const text = await response.text();
    return { ok: response.ok, status: response.status, text };
  } catch (error) {
    return { ok: false, reason: String(error?.message || error) };
  }
}
```

If `fetch(meta_url)` is blocked, rejected, redirected, or returns unusable text, do not give up. Open `meta_url` in a separate page through chrome-devtools-mcp:

1. Call `new_page` with `url: meta_url`
2. Wait for the page to load
3. Evaluate `document.body.innerText`
4. Parse the raw text as JSON with the recipe below
5. Return to the CoDesign page after extraction

This separate-page path is a standard CoDesign workflow, not a last-ditch trick.

## Parse Raw Metadata Page

Do not use screenshots or OCR to read metadata JSON.

Use `document.body.innerText` and parse the first complete JSON object:

```js
() => {
  const raw = document.body?.innerText || '';
  const start = raw.indexOf('{');
  const end = raw.lastIndexOf('}');
  if (start < 0 || end <= start) {
    return { ok: false, reason: 'No JSON object found in meta_url page text', rawHead: raw.slice(0, 300) };
  }

  const jsonText = raw.slice(start, end + 1);
  try {
    return { ok: true, metadata: JSON.parse(jsonText) };
  } catch (error) {
    return { ok: false, reason: String(error?.message || error), rawHead: jsonText.slice(0, 300) };
  }
}
```

If the browser exposes the page as a native JSON document, direct parsing of the readable document text is also allowed. The source remains `metadata`.

## Traverse And Index Nodes

CoDesign metadata can use different tree fields. Traverse all of these child containers:

1. `layers`
2. `children`
3. `nodes`

Index all useful node id forms:

1. `id`
2. `object_id`
3. `master_id`
4. `node.id`

Preserve `parent_id` and also compute parent-child links from traversal.

Use this generic traversal recipe:

```js
(metadata) => {
  const byId = new Map();
  const byParent = new Map();
  const all = [];

  const getIds = (item) => [
    item?.id,
    item?.object_id,
    item?.master_id,
    item?.node?.id,
  ].filter(Boolean).map(String);

  const childLists = (item) => [
    item?.layers,
    item?.children,
    item?.nodes,
  ].filter(Array.isArray);

  const visit = (item, parent = null, path = []) => {
    if (!item || typeof item !== 'object') return;

    const ids = getIds(item);
    const primaryId = ids[0] || null;
    const parentId = item.parent_id || parent?.id || parent?.object_id || null;
    const entry = { item, ids, primaryId, parentId, path };

    all.push(entry);
    for (const id of ids) byId.set(id, entry);
    if (parentId) {
      const key = String(parentId);
      if (!byParent.has(key)) byParent.set(key, []);
      byParent.get(key).push(entry);
    }

    for (const list of childLists(item)) {
      for (const child of list) {
        visit(child, item, [...path, item.name || primaryId || item.type || 'node']);
      }
    }
  };

  visit(metadata);

  return {
    count: all.length,
    nodes: all,
    byId: Object.fromEntries(byId),
    byParent: Object.fromEntries([...byParent].map(([key, value]) => [key, value.map((entry) => entry.primaryId)])),
  };
}
```

Use real `Map` objects in local reasoning when possible; serialize only summaries through MCP if the full tree is large.

## Metadata Fields To Use

Capture these fields when present:

1. Identity and structure: `type`, `name`, `content`, `parent_id`
2. Geometry: `rect`, `realRect`, `absoluteTransform`, `relativeTransform`
3. Style: `css`, `fills`, `borders`, `shadows`, `effects`, `radius`, `opacity`, `rotation`
4. Text: `fragments`, `content`, `css`, color, font size, font face, font weight, line height, and text alignment when present

For text nodes, prefer:

1. `fragments`
2. `css`
3. Fragment-level color, `fontSize`, `fontFace`, `fontWeight`, `lineHeight`, and `textAlign`
4. `content` only when fragments are unavailable or only plain text is needed

For shape, line, vector, group, and frame nodes, prefer:

1. `rect` and `realRect`
2. `css`
3. `fills`
4. `borders`
5. `radius`
6. `effects` or `shadows`
7. `opacity`
8. `rotation`

## Locate Modules From Metadata

Search candidate nodes in this order:

1. Exact `name` match
2. `content` match
3. Text found inside `fragments`
4. Parent-child path or semantic path match
5. `rect` or `realRect` match against the user-confirmed region
6. Shared parent, sibling, or child relationship match

Use module-level filtering before choosing a final node:

```js
(nodes, query) => {
  const q = String(query.text || '').trim();
  const region = query.region || null;

  const textOf = (item) => [
    item.name,
    item.content,
    ...(Array.isArray(item.fragments) ? item.fragments.map((fragment) => fragment.content || fragment.text || '') : []),
  ].filter(Boolean).join(' ');

  const intersects = (rect) => {
    if (!region || !rect) return true;
    const x = rect.x ?? rect.left ?? 0;
    const y = rect.y ?? rect.top ?? 0;
    const w = rect.width ?? rect.w ?? 0;
    const h = rect.height ?? rect.h ?? 0;
    return x < region.x + region.width &&
      x + w > region.x &&
      y < region.y + region.height &&
      y + h > region.y;
  };

  return nodes
    .filter((entry) => {
      const item = entry.item;
      return (!q || textOf(item).includes(q)) &&
        (intersects(item.realRect) || intersects(item.rect));
    })
    .map((entry) => ({
      id: entry.primaryId,
      parentId: entry.parentId,
      name: entry.item.name,
      type: entry.item.type,
      rect: entry.item.rect,
      realRect: entry.item.realRect,
      path: entry.path,
      text: textOf(entry.item),
    }));
}
```

When multiple children clearly belong to the requested module but the parent group is not obvious:

1. Find their shared `parent_id`
2. Inspect the shared parent node
3. If the parent is not useful or absent, derive a local container from the union of child `rect` or `realRect`
4. Preserve child offsets relative to that local container

This is metadata-driven parent-child reconstruction, not visual estimation.

## Reconstruct Layout From Metadata

Use metadata to recover structure before writing CSS:

1. Use `parent_id` and traversal parents to establish nesting
2. Use shared parent ids to group related children
3. Use `rect` or `realRect` to compute local x/y offsets, width, height, and sibling gaps
4. Use `relativeTransform` and `absoluteTransform` to confirm coordinate systems or transforms
5. Use sibling rect order to infer vertical stacks, horizontal rows, grid-like groups, and overlapping layers
6. Use the union bounds of related children to derive a module box when the group node is missing or unhelpful

Prefer normal flow only when metadata-derived parent padding, gaps, and child sizes reproduce the design within 2 px. Otherwise lock the local unit with absolute positioning, CSS grid tracks, fixed flex basis, or explicit spacers derived from metadata bounds.

Evidence examples:

1. `metadata`: node width from `realRect.width`
2. `metadata+layout`: button position from child `rect`, shared `parent_id`, and sibling gaps
3. `metadata+panel`: text CSS confirmed by both metadata `css` and CoDesign right-panel CSS

## Convert Metadata Styles

Map metadata directly into project styles:

1. `css` declarations can be copied or normalized into the project's style system
2. `rect` or `realRect` maps to width, height, and local placement
3. `fills` maps to background color, gradients, or image backgrounds
4. `borders` maps to border width, color, style, and side-specific borders
5. `radius` maps to `border-radius`
6. `effects` and `shadows` map to `box-shadow`, filter, blur, or opacity where applicable
7. `fragments` maps to text content spans when typography varies inside one text node
8. `opacity` maps to CSS opacity
9. `rotation` or transforms map to CSS transform only when visible in the target design

For line nodes, use `borders`, stroke width, stroke color, dashes, and geometry before approximating with a generic divider.

## Combine Metadata With Right Panel CSS

Metadata JSON and right-panel CSS are both exact sources.

Use metadata first for:

1. Screen-level location
2. Node discovery
3. Parent-child structure
4. Bulk rect and style extraction
5. Text fragments and mixed typography
6. Sibling gaps and local layout reconstruction

Use the right-side panel DOM recipe for:

1. Verifying a selected node's emitted CSS
2. Resolving metadata fields that are absent or hard to map
3. Confirming exact CSS declarations for a node before implementation
4. Inspecting export or slice controls

If metadata and panel disagree:

1. Confirm the selected panel node matches the metadata node id, name, rect, or text
2. Prefer the source tied to the confirmed target node
3. Record both sources and the resolution
4. Ask the user only if the mismatch changes the visible result and cannot be resolved by node identity

## Clickable Target Priority

When image understanding is available, use it first for discovering clickable elements in CoDesign:

1. Screenshot the confirmed artboard or locked unit.
2. Visually identify modules, child nodes, repeated items, icons, text blocks, form controls, cards, and safe click hotspots.
3. Click those hotspots through Chrome MCP when panel validation or export controls are needed.
4. Validate each click through selected-node name, selected bounds, panel updates, or CSS extraction results.
5. Retry with a more precise visual hotspot if the click selects the wrong node or only selects the canvas root.

When image understanding is unavailable, discover candidates from metadata JSON, selected-layer data, layer tree, DOM text, accessibility text, right-side panel metadata, and user-confirmed points.

This priority only applies to finding where to click. CoDesign CSS values still must come from metadata JSON or the right-panel DOM recipe below whenever available. Use visual click maps to validate candidates, not to replace exact metadata or panel values.

## Inspection Approach

1. Start from the middle business artboard, not the CoDesign application shell
2. Discover the target screen metadata and `meta_url`
3. Parse metadata JSON and locate target nodes before blind canvas clicking
4. Click the candidate element directly on the canvas when panel validation, export controls, or ambiguity resolution is needed
5. If the artboard content is canvas-rendered, click the visible module, block, or child element inside the canvas to trigger the right-side property/style popup or panel
6. Wait for the right-side inspect, property, style, or popup panel to appear or update before deciding extraction failed
7. Use the layer tree only when metadata and direct canvas module selection are insufficient
8. Read exact values from metadata JSON and the right inspect or style panel whenever available
9. For image-like or icon-like nodes, check whether export or slice controls are available
10. If the node is likely slice-based, locate the slice draft from the left design draft list and search for a matching slice before approximating it

## Right Panel CSS Extraction Recipe

Use this deterministic DOM recipe after selecting a CoDesign element.

All CSS style extraction in CoDesign must use this recipe. Do not use generic right-panel fallback until this CoDesign DOM recipe fails after retrying element selection and CSS tab activation. Do not use screenshot OCR or image-based estimation to read CoDesign CSS panel values.

1. Find the design configuration panel root:
   - tag: `aside`
   - classes: `screen-inspector inspector expanded`
   - selector: `aside.screen-inspector.inspector.expanded`
2. Inside that root, find the selected-node style section:
   - tag: `section`
   - class contains `node-box`
3. Inside the root or selected-node section, find `.css-node__codes--navs`
4. Traverse every `span` under `.css-node__codes--navs`
5. Find the `span` whose class contains `css-node__code` and whose trimmed text is `CSS`
6. Click that `CSS` span to switch the panel to CSS mode
7. Under `.css-node__codes`, get the first `.css-node__code--item`
8. Read every `mark` under that `.css-node__code--item`
9. Each `mark` is one CSS declaration line. Use `mark.textContent.trim()` so punctuation spans and plain text nodes, such as `inline-flex`, are preserved in DOM order.
10. Join all non-empty `mark` text values with newlines. The result is the exact CSS style block for the current selected design element.

Expected CoDesign markup shape:

```html
<mark><span class="token property">display</span><span class="token punctuation">:</span> inline-flex<span class="token punctuation">;</span></mark>
```

The extracted declaration for that `mark` should be:

```css
display: inline-flex;
```

Use a Chrome MCP `evaluate_script` equivalent to this recipe when the panel is script-readable:

```js
async () => {
  const panel = document.querySelector('aside.screen-inspector.inspector.expanded');
  if (!panel) {
    return { ok: false, reason: 'CoDesign inspector panel not found' };
  }

  const nodeBox = [...panel.querySelectorAll('section')]
    .find((section) => String(section.className || '').includes('node-box'));
  if (!nodeBox) {
    return { ok: false, reason: 'CoDesign node-box section not found' };
  }

  const navRoot = panel.querySelector('.css-node__codes--navs');
  if (!navRoot) {
    return { ok: false, reason: 'CoDesign CSS nav root not found' };
  }

  const cssTab = [...navRoot.querySelectorAll('span')]
    .find((span) =>
      String(span.className || '').includes('css-node__code') &&
      span.textContent.trim() === 'CSS'
    );
  if (!cssTab) {
    return { ok: false, reason: 'CoDesign CSS tab not found' };
  }

  cssTab.click();
  await new Promise((resolve) => requestAnimationFrame(() => requestAnimationFrame(resolve)));

  const codeItem = panel.querySelector('.css-node__codes .css-node__code--item');
  if (!codeItem) {
    return { ok: false, reason: 'CoDesign CSS code item not found' };
  }

  const declarations = [...codeItem.querySelectorAll('mark')]
    .map((mark) => mark.textContent.trim())
    .filter(Boolean);

  return {
    ok: declarations.length > 0,
    source: 'panel',
    platform: 'CoDesign',
    declarations,
    cssText: declarations.join('\n'),
  };
}
```

If this recipe returns no declarations, do not switch to screenshot OCR. Retry selection, verify that the right `aside.screen-inspector.inspector.expanded` panel is open, verify the `CSS` tab is selected, and then use the generic right-panel extraction fallback from `pixel-restore-rules.md`.

## Fallback To Canvas Clicks

Use canvas clicks after metadata-first steps when:

1. `design/screensMap` or related runtime state is unavailable
2. No `meta_url` exists for the matched screen
3. `meta_url` cannot be opened or parsed even in a separate page
4. The target module cannot be distinguished by name, content, fragments, rect, path, or relationships
5. Export controls or right-panel CSS are required for a specific selected node
6. The user explicitly selects a node and asks to use that selected boundary

When falling back, stay inside the smallest user-confirmed scope and still use the right-panel DOM CSS recipe for exact selected-node declarations.

## Evidence Logging

When metadata JSON is used, report it explicitly:

1. Screen metadata source: `metadata`
2. Node discovery source: `metadata`
3. Geometry source: `metadata` for `rect`, `realRect`, transforms, and sibling gaps
4. Style source: `metadata` for `css`, `fills`, `borders`, `effects`, `radius`, `opacity`, and `rotation`
5. Text source: `metadata` for `fragments` and `content`
6. Cross-check source: `metadata+panel` when right-panel CSS confirms metadata
7. User disambiguation source: `user` only for user choices or confirmations

Do not label metadata-derived values as `estimated`.

## Downgrade

If direct canvas interaction does not expose stable information:

1. Confirm metadata JSON was attempted or explain why it was unavailable
2. Confirm you clicked the visible module inside the canvas, not just the canvas root
3. Retry at a more precise target point
4. Wait for the right-side property/style popup or panel to appear or update
5. Use the layer tree as a locating aid, not as permission to widen scope
6. Keep probing within the smallest user-confirmed node
7. For likely slice-based elements, search the slice draft and attempt export before using a placeholder
8. Use screenshot estimation only as a last resort and report the approximation
