# Asset Rules

Resolve visual assets before substituting them.

## Asset Types

Assets include:

1. Images and photos
2. Logos
3. Icons
4. Background images
5. Exportable slices
6. Chart decorations and image-like complex regions

## Resolution Order

For each asset-bearing node:

1. Check whether the selected element exposes a downloadable or copyable slice
2. If a slice exists, download or export it when the host and permissions allow
3. If the selected element is likely slice-based and no slice is visible in the current view, open or locate the design draft named `切图`
4. Search the `切图` design draft for a matching slice before approximating the element
5. Download the matched slice to `src/assets/images`
6. Import and use the downloaded file from the Vue page or component
7. If the project already contains the same asset, reuse the existing file
8. If the asset cannot be obtained, create a same-size placeholder that preserves layout and report the gap

Do not invent branded or product-specific imagery when the source asset cannot be extracted.

Do not assume the active model can inspect images. If image understanding is unavailable, compare candidate slices by selected layer name, export name, dimensions, position, role, surrounding text, and artboard context. Ask the user to confirm when multiple candidate slices remain plausible. Never stop solely because screenshot/image understanding is unavailable.

Set `imageUnderstanding = available` only when the host explicitly supports image input and the current agent can safely reason over image attachments. If capability is unknown, set `imageUnderstanding = unavailable`.

When `imageUnderstanding = available`, visual comparison can help rank candidate slices, but it must not replace export metadata, layer names, dimensions, surrounding context, or user confirmation when those signals disagree.

## Implementation

1. Store downloaded slice assets in `src/assets/images` unless the user or project convention specifies another image directory
2. Import downloaded assets in Vue files instead of hardcoding absolute local paths
3. Keep image containers dimensioned so a future real asset can replace a placeholder without relayout
4. Preserve object fit, radius, opacity, shadow, border, and background behavior from extracted styles
5. Prefer existing icon systems when the project already uses one and the icon can be matched faithfully
6. If an icon cannot be matched faithfully and appears slice-based, search the `切图` design draft before using a placeholder
7. If an icon cannot be matched or downloaded, use a placeholder rather than a misleading approximate icon

## Slice Lookup In CoDesign

When working in CoDesign and a selected element likely represents a slice:

1. Use the left design draft list to click the `切图` draft
2. Compare visible slices against the selected element's appearance, size, role, and surrounding context when image understanding is available; otherwise compare names, sizes, export metadata, and surrounding context
3. Select the matching slice and check whether CoDesign exposes export or download controls
4. Download the matching slice into `src/assets/images`
5. Return to the original business draft and continue restoration

Do not skip this lookup when a slice is likely. Screenshot approximation is allowed only after the `切图` lookup fails or export is unavailable.

## Reporting

Report whether assets were:

1. Downloaded or exported from the design platform
2. Reused from the current project
3. Approximated with an existing component or icon
4. Replaced with placeholders
