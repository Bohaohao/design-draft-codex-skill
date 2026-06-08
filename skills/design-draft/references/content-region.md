# Content Region

Use this guide before any implementation work.

## Effective Business Area

The effective business area is the real design content inside the draft. It is usually the middle artboard, frame, canvas, or selected page content.

Exclude design-tool editor chrome:

1. Left page tree, layer tree, asset tree, and workspace navigation
2. Right property, inspect, style, and CSS panels
3. Top design-tool navbar and toolbar
4. Comments, cursors, floating assistants, zoom controls, rulers, and selection handles

Use editor chrome only to inspect, locate, or extract styles. Do not recreate it in the target application.

## Product Shell Boundary

If the user does not explicitly request full-page restoration, exclude the designed product's own persistent shell before choosing the starting point:

1. Product navbar, header, or top app bar
2. Product sidebar, icon rail, menu rail, or fixed navigation
3. Account, tenant, notification, avatar, and global action areas
4. Other repeated app-shell regions shared across multiple pages

Default to the page's main content area after excluding design-tool editor chrome and product app shell.

Restore product navbar/sidebar only when the user explicitly requests full-page restoration or directly names those shell regions as the target.

## Business Content Inside The Main Area

Restore every visible business node inside the confirmed main content area:

1. Containers and groups
2. Cards and panels
3. Page-local tabs, segmented controls, and navigation that belong to the content itself
4. Toolbars, filters, forms, tables, charts, lists, modals, and empty states
5. Text, icons, images, backgrounds, and visible controls

If it is unclear whether a region is product shell or page content, ask before restoring it.

## Starting Point Confirmation

Before coding:

1. Identify the requested scope
2. Locate the matching main content area or first restoration node, excluding product shell unless full-page restoration was requested
3. Capture a screenshot of that candidate area, or explicitly name the first node if a screenshot is not practical
4. Ask the user to confirm whether the starting point is correct

Do not write implementation code before confirmation.

If the user confirms only a smaller element, narrow the scope to that element immediately.
