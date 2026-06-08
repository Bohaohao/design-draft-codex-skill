# Platform Detection

Use layered signals in this order:

1. URL pattern
2. Page title
3. Visible interface text
4. Stable editor structure

Prefer strong multi-signal matches over weak single-signal matches.

## Mockplus Signals

1. URL or title includes `mockplus`, `Mockplus`, or `摹客`
2. The interface has a central design canvas or artboard
3. The page has design-tool chrome such as page/layer panels or style panels
4. Selecting an element reveals style, property, or CSS details on the right side

## CoDesign Signals

1. URL or title includes `codesign`, `co-design`, `CoDesign`, or Tencent design collaboration wording
2. The interface has a central design canvas or artboard
3. The page has design-tool chrome such as layer, asset, inspect, or style panels
4. Selecting an element reveals style, property, or CSS details on the right side

## Generic Design-Tool Signals

1. A central canvas-like region is visible
2. The page is an editor, not just a published static preview
3. Element selection changes the visible inspector or property panel
4. A right-side inspect or property panel can reveal styles after selection

## Selection Caution

Do not confuse the MCP-selected page context with the real Chrome foreground tab. If the user says `current tab`, `current open tab`, or `foreground tab`, resolve the real Chrome foreground tab.

If a user-provided title or URL fragment matches multiple tabs, stop and ask the user to disambiguate.
