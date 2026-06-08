# Mockplus Notes

Use these notes when the selected design source appears to be Mockplus or 摹客.

## Detection

Strong signals:

1. URL, title, or visible product text includes `mockplus`, `Mockplus`, or `摹客`
2. A central canvas or artboard is visible
3. Left panels expose pages, layers, assets, or slices
4. Selecting an element updates the right-side inspect, property, style, or CSS panel

## Inspection Approach

1. Start from the middle business artboard, not the Mockplus application shell
2. Click the candidate element directly on the canvas first
3. Use the left layer tree only when direct clicking cannot reliably select the intended node
4. Read size, position, typography, fill, border, radius, shadow, opacity, and export information from the right panel when visible
5. If a downloadable slice is available, prefer it over rebuilding complex image-like content

## Downgrade

If the selected element does not reveal stable style data:

1. Retry a more precise click on the visible node
2. Use layer-tree selection only to locate the smallest target node
3. Use computed styles or screenshot estimation only after panel extraction fails
4. Report any estimated values in the final gaps
