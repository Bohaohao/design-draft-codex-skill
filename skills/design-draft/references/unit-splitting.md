# Unit Splitting

Split the confirmed design area into units before implementation. Units keep restoration reviewable and reduce the risk of broad unrelated edits.

## Typical Units

Use natural visual and semantic boundaries:

1. Header or top navigation
2. Sidebar or product navigation
3. Toolbar or action row
4. Filter/search form
5. Content card group
6. Table or data list
7. Chart or dashboard panel
8. Detail panel
9. Modal or drawer
10. Empty state
11. Footer

If the user requested one specific element, that element is the only unit unless the user expands scope.

## Per-Unit Workflow

For each unit:

1. State the locked unit before reading styles or editing files
2. Click and inspect the unit container first
3. Traverse children depth-first
4. Resolve assets used by the unit
5. Implement only the code and styles needed for that unit
6. Switch to the local page and compare against the draft
7. Record unresolved differences before moving on

Do not begin another unit until the current unit has been inspected and checked.

## When To Merge Units

Merge units only when separating them would create false boundaries or duplicated wrappers, such as a tab bar and its directly attached panel when the draft treats them as one component.

## When To Split Further

Split a unit further when it contains multiple complex subregions, such as a dashboard card that contains filters, charts, and a table.
