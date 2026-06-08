# Reporting

Read this before final output.

## Required Final Report

Report these fields:

1. Design source: platform, tab title or identifier, and selected business area
2. Files changed: created or updated files
3. Restored units: unit names and short scope descriptions
4. Style extraction: full, partial, or unavailable; mention whether right-side panel styles were used
5. CoDesign metadata JSON: whether it was used, which screens or units it covered, and whether `meta_url` was opened through the current page or a separate page
6. Metadata coverage: which exact values came from metadata, such as screen info, node ids, `rect`, `realRect`, `css`, `fragments`, `fills`, `borders`, `effects`, `radius`, and parent-child relationships
7. Source split: which values came from metadata, right-side panel, user confirmation, project conventions, or remain unresolved
8. Assets: downloaded/exported, reused, approximated, placeholder, or none
9. Restoration ratio: estimated percentage and basis for the estimate
10. Remaining differences: visual gaps, approximate values, missing assets, unverified states, or unresolved ambiguities
11. Extra behavior: clearly separate base draft restoration from any extra user-requested functionality
12. Capability mode: whether `imageUnderstanding` was available or unavailable
13. Evidence sources: which key values came from `metadata`, `panel`, `layer`, `dom`, `script`, `project`, `user`, or `estimated`

## Restoration Ratio

Use a practical estimate based on:

1. Coverage of visible business nodes restored
2. Percentage of key styles extracted from real panel values
3. Asset completeness
4. Verification against the local page

Do not claim 100% if any visible asset, state, typography, spacing, or layout detail remains approximate or unverified.

Use `estimated` only when screenshot-based estimation was used and only when `imageUnderstanding = available`. If image understanding was unavailable, report unresolved values instead of visual estimates.

Do not label CoDesign metadata JSON values as `estimated`. Metadata JSON is exact machine-readable evidence. If metadata was used together with right-panel CSS, report it as `metadata+panel`. If metadata was attempted but unavailable, state the failed step briefly, such as no `screensMap`, missing `meta_url`, blocked separate page, parse failure, or ambiguous node match.

## Concision

Keep the final answer concise. Include enough detail for the user to understand what changed and what remains, but do not paste long extracted CSS or full file contents.
