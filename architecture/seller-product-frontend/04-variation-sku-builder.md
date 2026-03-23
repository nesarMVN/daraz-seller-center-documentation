# Variation & SKU Builder

## Overview

The variation/SKU builder lets sellers define product properties (variation axes like Color, Size) and their values, then manages the SKU matrix generated from all combinations. Each SKU gets its own price, stock, SKU code, and active toggle. Drag and drop is used to reorder property values and SKU rows.

**Constraints:**
- Max 3 properties per product
- Max 50 values per property
- Max 100 SKUs per product
- Property names must be unique
- Property values must be unique within their property
- Each SKU maps to exactly one value from each property
- No duplicate SKU combinations

---

## Empty State: No Properties

```
╔════════════════════════════════════════════════════════════════════════╗
║  VARIATIONS & SKUs (empty — no properties defined yet)                ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │                                                                  │ ║
║  │                     ┌──────────────┐                             │ ║
║  │                     │   ┌─┐ ┌─┐   │                             │ ║
║  │                     │   │ │ │ │   │  (grid/matrix illustration) │ ║
║  │                     │   └─┘ └─┘   │                             │ ║
║  │                     │   ┌─┐ ┌─┐   │                             │ ║
║  │                     │   │ │ │ │   │                             │ ║
║  │                     │   └─┘ └─┘   │                             │ ║
║  │                     └──────────────┘                             │ ║
║  │                                                                  │ ║
║  │         Define variations by adding properties                   │ ║
║  │         (e.g., Color, Size, Material)                            │ ║
║  │                                                                  │ ║
║  │                    [+ Add Property]                              │ ║
║  │                                                                  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  This empty state replaces the property definitions and SKU table     ║
║  when no properties have been added yet. Clicking [+ Add Property]    ║
║  opens the property dropdown (schema-driven) or free-text input.      ║
║                                                                        ║
║  If the product has no properties at all, it uses a single SKU        ║
║  (see "No Properties (Single SKU)" section below).                    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Section Layout

```
╔════════════════════════════════════════════════════════════════════════╗
║  VARIATIONS & SKUs                                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  PROPERTY DEFINITIONS                                                  ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │                                                                  │ ║
║  │  Property 1: Color *                                    [Remove] │ ║
║  │  ┌────────────────────────────────────────────────────────────┐  │ ║
║  │  │  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────────────────────┐ │  │ ║
║  │  │  │= Black│ │= White│ │= Red  │ │ + Add value...        │ │  │ ║
║  │  │  │ [img] │ │ [img] │ │ [img] │ │                       │ │  │ ║
║  │  │  │  [x]  │ │  [x]  │ │  [x]  │ └───────────────────────┘ │  │ ║
║  │  │  └───────┘ └───────┘ └───────┘                            │  │ ║
║  │  └────────────────────────────────────────────────────────────┘  │ ║
║  │                                                                  │ ║
║  │  Property 2: Size                                       [Remove] │ ║
║  │  ┌────────────────────────────────────────────────────────────┐  │ ║
║  │  │  ┌────┐ ┌────┐ ┌────┐ ┌───────────────────────┐          │  │ ║
║  │  │  │= S │ │= M │ │= L │ │ + Add value...        │          │  │ ║
║  │  │  │[x] │ │[x] │ │[x] │ └───────────────────────┘          │  │ ║
║  │  │  └────┘ └────┘ └────┘                                     │  │ ║
║  │  └────────────────────────────────────────────────────────────┘  │ ║
║  │                                                                  │ ║
║  │  [+ Add Property]  (max 3 properties)                            │ ║
║  │                                                                  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  SKU MATRIX (3 colors x 3 sizes = 9 SKUs)                             ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │                                                                  │ ║
║  │  ┌───┬───────┬──────┬─────────┬───────┬──────────┬────────────┐ │ ║
║  │  │   │ Color │ Size │ Price * │ Stock │ SKU Code │ Active     │ │ ║
║  │  ├───┼───────┼──────┼─────────┼───────┼──────────┼────────────┤ │ ║
║  │  │ = │ Black │ S    │ 24.99   │ 100   │ BK-S     │ [on]       │ │ ║
║  │  │ = │ Black │ M    │ 24.99   │ 100   │ BK-M     │ [on]       │ │ ║
║  │  │ = │ Black │ L    │ 26.99   │ 80    │ BK-L     │ [on]       │ │ ║
║  │  │ = │ White │ S    │ 24.99   │ 100   │ WH-S     │ [on]       │ │ ║
║  │  │ = │ White │ M    │ 24.99   │ 100   │ WH-M     │ [on]       │ │ ║
║  │  │ = │ White │ L    │ 26.99   │ 80    │ WH-L     │ [on]       │ │ ║
║  │  │ = │ Red   │ S    │ 24.99   │ 100   │ RD-S     │ [on]       │ │ ║
║  │  │ = │ Red   │ M    │ 24.99   │ 100   │ RD-M     │ [on]       │ │ ║
║  │  │ = │ Red   │ L    │ 26.99   │ 80    │ RD-L     │ [on]       │ │ ║
║  │  └───┴───────┴──────┴─────────┴───────┴──────────┴────────────┘ │ ║
║  │                                                                  │ ║
║  │  = drag handle for row reorder                                   │ ║
║  │  9 SKUs total (max 100)                                          │ ║
║  │                                                                  │ ║
║  │  Quick fill:  Price [_____] [Apply to all]                       │ ║
║  │               Stock [_____] [Apply to all]                       │ ║
║  │                                                                  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Property Management Workflows

### Add Property (with schema)

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User clicks [+ Add Property]                                 ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Category schema defines available properties:                         ║
║  variation_attributes: [                                               ║
║    { name: "Color", required: true, options: ["Black","White",...] },  ║
║    { name: "DPI", required: false, options: ["800","1600","3200"] }    ║
║  ]                                                                     ║
║                                                                        ║
║  Dropdown shows schema-defined property names:                         ║
║  ┌──────────────────────────────────┐                                 ║
║  │  Select property:                │                                 ║
║  │  ┌────────────────────────────┐  │                                 ║
║  │  │  Color * (required)        │  │                                 ║
║  │  │  DPI                       │  │                                 ║
║  │  └────────────────────────────┘  │                                 ║
║  │                                  │                                 ║
║  │  Already added properties are    │                                 ║
║  │  grayed out / disabled.          │                                 ║
║  └──────────────────────────────────┘                                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                          User selects "Color"
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: Property added with value picker                             ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Property: Color *                                             [Remove]║
║  ┌────────────────────────────────────────────────────────────────┐   ║
║  │  Add values from schema options:                               │   ║
║  │  ┌──────────────────────────────────────────────────────────┐ │   ║
║  │  │ Select colors...                                    [v]  │ │   ║
║  │  ├──────────────────────────────────────────────────────────┤ │   ║
║  │  │  [x] Black                                               │ │   ║
║  │  │  [x] White                                               │ │   ║
║  │  │  [ ] Red                                                 │ │   ║
║  │  │  [ ] Blue                                                │ │   ║
║  │  │  [ ] Green                                               │ │   ║
║  │  │  [ ] Yellow                                              │ │   ║
║  │  │  [ ] Gray                                                │ │   ║
║  │  │  [ ] Multicolor                                          │ │   ║
║  │  │  [ ] Custom                                              │ │   ║
║  │  └──────────────────────────────────────────────────────────┘ │   ║
║  │                                                                │   ║
║  │  Selected values (drag to reorder):                            │   ║
║  │  ┌───────┐ ┌───────┐                                          │   ║
║  │  │= Black│ │= White│                                          │   ║
║  │  │ [img] │ │ [img] │                                          │   ║
║  │  │  [x]  │ │  [x]  │                                          │   ║
║  │  └───────┘ └───────┘                                          │   ║
║  └────────────────────────────────────────────────────────────────┘   ║
║                                                                        ║
║  [img] = optional image for this value (via asset picker)              ║
║  [x] = remove this value                                              ║
║  = = drag handle for reorder                                           ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Add Property (without schema)

```
╔════════════════════════════════════════════════════════════════════════╗
║  When no category schema exists, property name is free-text            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌────────────────────────────────────────────────────────────────┐   ║
║  │  Property Name: (max 100 characters)                            │   ║
║  │  ┌──────────────────────────────────────────────┐              │   ║
║  │  │ Material_____________________________________│              │   ║
║  │  └──────────────────────────────────────────────┘              │   ║
║  │                                                                │   ║
║  │  Values (type and press Enter, max 100 chars each):            │   ║
║  │  ┌──────────────────────────────────────────────┐              │   ║
║  │  │ Add value...                                  │              │   ║
║  │  └──────────────────────────────────────────────┘              │   ║
║  │                                                                │   ║
║  │  ┌──────────┐ ┌──────────┐                                    │   ║
║  │  │= Leather │ │= Canvas  │                                    │   ║
║  │  │    [x]   │ │    [x]   │                                    │   ║
║  │  └──────────┘ └──────────┘                                    │   ║
║  └────────────────────────────────────────────────────────────────┘   ║
║                                                                        ║
║  User types value text and presses Enter to add.                       ║
║  Values shown as draggable tags.                                       ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Property Value Image Assignment

```
╔════════════════════════════════════════════════════════════════════════╗
║  Each property value can have an optional image                       ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌───────┐ ┌───────┐ ┌───────┐                                       ║
║  │= Black│ │= White│ │= Red  │                                       ║
║  │┌─────┐│ │┌─────┐│ │┌─────┐│                                       ║
║  ││ IMG ││ ││  +  ││ ││ IMG ││                                       ║
║  │└─────┘│ │└─────┘│ │└─────┘│                                       ║
║  │  [x]  │ │  [x]  │ │  [x]  │                                       ║
║  └───────┘ └───────┘ └───────┘                                       ║
║                                                                        ║
║  Click [+] or existing image to open Asset Picker modal:               ║
║  - mode: "single"                                                      ║
║  - fileType: "image"                                                   ║
║  - onSelect: sets image_url for this property value                    ║
║                                                                        ║
║  image_url constraints: valid URL, max 500 characters.                 ║
║  This image is shown on the storefront when buyers select a color.     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Property Value Tag States

```
╔════════════════════════════════════════════════════════════════════════╗
║  PROPERTY VALUE TAG INTERACTION STATES                                ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  DEFAULT:                                                              ║
║  ┌───────┐                                                            ║
║  │= Black│  (white bg, gray border #e5e7eb, = drag handle left)       ║
║  │ [img] │  (image thumbnail or + placeholder below name)             ║
║  │  [x]  │  ([x] remove, subtle gray)                                 ║
║  └───────┘                                                            ║
║                                                                        ║
║  HOVER:                                                                ║
║  ┌───────┐                                                            ║
║  │= Black│  (light gray bg #f3f4f6, [x] becomes prominent red)        ║
║  │ [img] │  (cursor grab on = handle area)                             ║
║  │  [x]  │                                                            ║
║  └───────┘                                                            ║
║                                                                        ║
║  DRAGGING:                                                             ║
║  ┌ ─ ─ ─ ┐                                                           ║
║    = Black   (DragOverlay: semi-transparent copy, slight shadow,       ║
║    [img]      follows cursor, original position shows empty gap)       ║
║     [x]                                                                ║
║  └ ─ ─ ─ ┘                                                           ║
║                                                                        ║
║  DROP TARGET (between two tags):                                       ║
║  ┌───────┐ │ ┌───────┐                                               ║
║  │= Black│ │ │= Red  │  (blue vertical line indicator between tags    ║
║  │ [img] │ │ │ [img] │   shows where the dragged item will land)      ║
║  │  [x]  │ │ │  [x]  │                                               ║
║  └───────┘ │ └───────┘                                               ║
║             ▲ blue drop indicator                                      ║
║                                                                        ║
║  Image slot states:                                                    ║
║                                                                        ║
║  NO IMAGE       HOVER          IMAGE LOADED                            ║
║  ┌─ ─ ─ ─ ┐   ┌──────────┐   ┌──────────┐                           ║
║  │   +    │   │    +     │   │  ┌────┐  │                           ║
║  │(dashed │   │ (darker  │   │  │IMG │  │                           ║
║  │ border)│   │  bg)     │   │  └────┘  │                           ║
║  └ ─ ─ ─ ┘   └──────────┘   │   🔄     │  ← replace icon on hover  ║
║                               └──────────┘                            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Drag and Drop Reorder Property Values

```
╔════════════════════════════════════════════════════════════════════════╗
║  User drags "Red" before "White"                                      ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before:                                                               ║
║  ┌───────┐ ┌───────┐ ┌───────┐                                       ║
║  │= Black│ │= White│ │= Red  │                                       ║
║  │ pos:0 │ │ pos:1 │ │ pos:2 │                                       ║
║  └───────┘ └───────┘ └───────┘                                       ║
║                                                                        ║
║  User grabs "Red" (=) drag handle, drags left:                         ║
║                                                                        ║
║  During drag:                                                          ║
║  ┌───────┐ ┌ ─ ─ ─┐ ┌───────┐                                       ║
║  │= Black│   Red    │= White│                                       ║
║  │ pos:0 │ └ ─ ─ ─┘ │ pos:2 │                                       ║
║  └───────┘ ▲ drop   └───────┘                                       ║
║              here                                                      ║
║                                                                        ║
║  After:                                                                ║
║  ┌───────┐ ┌───────┐ ┌───────┐                                       ║
║  │= Black│ │= Red  │ │= White│                                       ║
║  │ pos:0 │ │ pos:1 │ │ pos:2 │                                       ║
║  └───────┘ └───────┘ └───────┘                                       ║
║                                                                        ║
║  Implementation: @dnd-kit with SortableContext                         ║
║  - DndContext wraps the value list                                     ║
║  - Each value tag is a useSortable item                                ║
║  - onDragEnd: recalculate positions (0, 1, 2, ...)                     ║
║  - Position values used in both create body and edit patch ops         ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Remove Property Warning

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [Remove] on a property that has SKUs                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Remove "Color" Property?                          [x]      ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  This property is used by 9 SKUs. Removing it will:          ║     ║
║  ║                                                              ║     ║
║  ║  - Delete all 3 values (Black, White, Red)                   ║     ║
║  ║  - Remove the Color column from the SKU matrix               ║     ║
║  ║  - Merge duplicate SKU rows (same remaining properties)      ║     ║
║  ║                                                              ║     ║
║  ║  SKUs will be recalculated from: 9 -> 3 (Size only)         ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║               [Cancel]      [Remove Property]               ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## SKU Matrix Generation

### Auto-generation on Value Changes

```
╔════════════════════════════════════════════════════════════════════════╗
║  SKU MATRIX AUTO-GENERATION                                           ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Given properties:                                                     ║
║  - Color: [Black, White]                                               ║
║  - Size: [S, M, L]                                                     ║
║                                                                        ║
║  Cartesian product generates 2 x 3 = 6 SKUs:                          ║
║                                                                        ║
║  ┌───────┬──────┬──────────────────┐                                  ║
║  │ Color │ Size │ property_values  │                                  ║
║  ├───────┼──────┼──────────────────┤                                  ║
║  │ Black │ S    │ [[0,0], [1,0]]   │                                  ║
║  │ Black │ M    │ [[0,0], [1,1]]   │                                  ║
║  │ Black │ L    │ [[0,0], [1,2]]   │                                  ║
║  │ White │ S    │ [[0,1], [1,0]]   │                                  ║
║  │ White │ M    │ [[0,1], [1,1]]   │                                  ║
║  │ White │ L    │ [[0,1], [1,2]]   │                                  ║
║  └───────┴──────┴──────────────────┘                                  ║
║                                                                        ║
║  When user adds "Red" to Color:                                        ║
║  - 3 new SKUs added: Red/S, Red/M, Red/L                              ║
║  - New SKUs get default price=0.01, stock=0, is_active=true             ║
║    (price min is 0.01 -- server rejects price=0)                       ║
║  - Existing SKU data preserved                                         ║
║                                                                        ║
║  When user removes "L" from Size:                                      ║
║  - 3 SKUs removed: Black/L, White/L, Red/L                            ║
║  - Confirmation dialog shown (data will be lost)                       ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### SKU Explosion Pre-emptive Warning

```
╔════════════════════════════════════════════════════════════════════════╗
║  SKU COUNT GUARD BEFORE VALUE ADDITION                                ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before adding a new value, the UI calculates the resulting cartesian  ║
║  product count. If it would exceed 100, the addition is blocked.       ║
║                                                                        ║
║  Live counter (always visible below property definitions):             ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  Projected SKUs: 75 / 100                                        │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (gray text, updates in real-time as values are added/removed)        ║
║                                                                        ║
║  When projected count is 90-100 (approaching limit):                  ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  Projected SKUs: 96 / 100  ⚠️ Approaching limit                  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (orange text, warning icon)                                          ║
║                                                                        ║
║  When adding a value would exceed 100:                                ║
║                                                                        ║
║  User tries to add "Purple" to Color (currently 4 colors x 5 sizes   ║
║  x 5 materials = 100 SKUs. Adding Purple = 5 x 5 x 5 = 125).        ║
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Too Many SKUs                                       [x]    ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  Adding "Purple" to Color would create 125 SKUs,             ║     ║
║  ║  exceeding the 100 maximum.                                  ║     ║
║  ║                                                              ║     ║
║  ║  Current: 4 colors x 5 sizes x 5 materials = 100 SKUs       ║     ║
║  ║  After:   5 colors x 5 sizes x 5 materials = 125 SKUs       ║     ║
║  ║                                                              ║     ║
║  ║  Remove some values from other properties first.             ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                        [OK, Got It]          ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  The [Add] button / Enter key is disabled when projected count > 100.  ║
║  Tooltip on disabled button: "Would exceed 100 SKU limit."            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Value Addition Workflow

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User adds "Red" to Color property                           ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before: 6 SKUs (Black x S/M/L + White x S/M/L)                       ║
║                                                                        ║
║  ┌───┬───────┬──────┬─────────┬───────┬──────────┬──────┐            ║
║  │   │ Color │ Size │ Price   │ Stock │ SKU Code │ Act  │            ║
║  ├───┼───────┼──────┼─────────┼───────┼──────────┼──────┤            ║
║  │ = │ Black │ S    │ 24.99   │ 100   │ BK-S     │ [on] │            ║
║  │ = │ Black │ M    │ 24.99   │ 100   │ BK-M     │ [on] │            ║
║  │ = │ Black │ L    │ 26.99   │ 80    │ BK-L     │ [on] │            ║
║  │ = │ White │ S    │ 24.99   │ 100   │ WH-S     │ [on] │            ║
║  │ = │ White │ M    │ 24.99   │ 100   │ WH-M     │ [on] │            ║
║  │ = │ White │ L    │ 26.99   │ 80    │ WH-L     │ [on] │            ║
║  └───┴───────┴──────┴─────────┴───────┴──────────┴──────┘            ║
║                                                                        ║
║  User selects "Red" from the color dropdown...                         ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: New SKU rows appear                                          ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  After: 9 SKUs (3 colors x 3 sizes)                                   ║
║                                                                        ║
║  ┌───┬───────┬──────┬─────────┬───────┬──────────┬──────┐            ║
║  │   │ Color │ Size │ Price * │ Stock │ SKU Code │ Act  │            ║
║  ├───┼───────┼──────┼─────────┼───────┼──────────┼──────┤            ║
║  │ = │ Black │ S    │ 24.99   │ 100   │ BK-S     │ [on] │            ║
║  │ = │ Black │ M    │ 24.99   │ 100   │ BK-M     │ [on] │            ║
║  │ = │ Black │ L    │ 26.99   │ 80    │ BK-L     │ [on] │            ║
║  │ = │ White │ S    │ 24.99   │ 100   │ WH-S     │ [on] │            ║
║  │ = │ White │ M    │ 24.99   │ 100   │ WH-M     │ [on] │            ║
║  │ = │ White │ L    │ 26.99   │ 80    │ WH-L     │ [on] │            ║
║  │ = │ Red   │ S    │ [0.01]  │ [0]   │ [______] │ [on] │  NEW      ║
║  │ = │ Red   │ M    │ [0.01]  │ [0]   │ [______] │ [on] │  NEW      ║
║  │ = │ Red   │ L    │ [0.01]  │ [0]   │ [______] │ [on] │  NEW      ║
║  └───┴───────┴──────┴─────────┴───────┴──────────┴──────┘            ║
║                                                                        ║
║  New rows highlighted briefly to draw attention.                       ║
║  User needs to fill in price, stock, and SKU code for new rows.        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Value Removal Confirmation

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [x] on "White" value                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Remove Value "White"?                             [x]      ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  This will remove 3 SKUs:                                    ║     ║
║  ║  - White / S (price: $24.99, stock: 100)                     ║     ║
║  ║  - White / M (price: $24.99, stock: 100)                     ║     ║
║  ║  - White / L (price: $26.99, stock: 80)                      ║     ║
║  ║                                                              ║     ║
║  ║  SKU data for these combinations will be lost.               ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║              [Cancel]          [Remove Value]               ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Cascade Behavior (Edit Mode)

```
╔════════════════════════════════════════════════════════════════════════╗
║  PROPERTY / VALUE REMOVAL CASCADE IN EDIT MODE                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Remove a property (via PATCH):                                        ║
║  { "op": "remove", "path": "/properties/{id}" }                       ║
║                                                                        ║
║  Server cascade:                                                       ║
║  1. All values under the property are deleted (FK CASCADE)             ║
║  2. All SKU associations referencing those values are removed           ║
║  3. Post-operation check: if any SKU now has incomplete associations   ║
║     (doesn't reference one value per remaining property), server       ║
║     returns 422:                                                       ║
║     "SKU '{id}' has incomplete property associations after             ║
║      value removal."                                                   ║
║                                                                        ║
║  Remove a single value:                                                ║
║  { "op": "remove", "path": "/properties/{id}/values/{id}" }           ║
║                                                                        ║
║  Server cascade:                                                       ║
║  1. The value is deleted (FK CASCADE removes SKU associations)         ║
║  2. SKUs that referenced ONLY this value from that property become     ║
║     orphaned — same 422 error as above                                 ║
║                                                                        ║
║  Frontend must calculate affected SKU count before showing             ║
║  the confirmation dialog. Count = number of SKUs whose associations    ║
║  include a reference to the value(s) being removed.                    ║
║                                                                        ║
║  Unique constraints (enforced per-product by DB):                      ║
║  - (product_id, name) on properties — no duplicate axis names          ║
║  - (property_id, value) on values — no duplicate values per axis       ║
║  - (product_id, sku_code) on SKUs — no duplicate codes (where not null)║
║  - (sku_id, property_value_id) on associations — no duplicate assoc    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## SKU Table Interactions

### Inline Editing

```
╔════════════════════════════════════════════════════════════════════════╗
║  Each cell in the SKU table is directly editable                      ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Price cell: Click to edit                                             ║
║  ┌─────────┐      ┌─────────┐                                        ║
║  │  24.99  │  ->  │ [29.99] │  Type new value, press Tab/Enter        ║
║  └─────────┘      └─────────┘                                        ║
║  Validation: numeric, >= 0.01, <= 9999999999.99                        ║
║                                                                        ║
║  Stock cell: Click to edit                                             ║
║  ┌─────────┐      ┌─────────┐                                        ║
║  │   100   │  ->  │ [150__] │  Type new value, press Tab/Enter        ║
║  └─────────┘      └─────────┘                                        ║
║  Validation: integer, >= 0                                             ║
║                                                                        ║
║  SKU Code cell: Click to edit                                          ║
║  ┌──────────┐      ┌──────────┐                                      ║
║  │  BK-S    │  ->  │ [BLK-SM] │  Type new value                       ║
║  └──────────┘      └──────────┘                                      ║
║  Validation: string, max 50, unique across product                     ║
║                                                                        ║
║  Active toggle: Click to toggle                                        ║
║  ┌──────┐      ┌──────┐                                              ║
║  │ [on] │  ->  │ [off]│  Inactive SKUs are grayed out                 ║
║  └──────┘      └──────┘                                              ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### SKU Cell Edit States

```
╔════════════════════════════════════════════════════════════════════════╗
║  SKU CELL INTERACTION DETAIL                                          ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  PRICE CELL lifecycle:                                                 ║
║                                                                        ║
║  VIEW         HOVER        EDIT          ERROR         SAVED           ║
║  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐   ┌────────┐        ║
║  │ $24.99 │  │ $24.99 │  │ [29.99]│  │ [0.00] │   │ $29.99 │        ║
║  │(right) │  │(gray bg)│  │(blue   │  │(red    │   │ ✓ (grn)│        ║
║  │        │  │        │  │ border)│  │ border)│   │        │        ║
║  └────────┘  └────────┘  └────────┘  └────────┘   └────────┘        ║
║                                        ⚠️ Min 0.01  (green flash     ║
║                                                      500ms)           ║
║                                                                        ║
║  STOCK CELL with warnings:                                             ║
║                                                                        ║
║  NORMAL       LOW STOCK        OUT OF STOCK                            ║
║  ┌────────┐  ┌────────┐      ┌────────┐                              ║
║  │  100   │  │   8    │      │   0    │                              ║
║  │        │  │ ⚠️ Low  │      │ ⛔ Out  │                              ║
║  └────────┘  └────────┘      └────────┘                              ║
║               (orange ⚠️      (red ⛔ badge                           ║
║                when < 10)      when = 0)                               ║
║                                                                        ║
║  SKU CODE CELL:                                                        ║
║                                                                        ║
║  EMPTY             FILLED           DUPLICATE                          ║
║  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                  ║
║  │ Enter SKU... │ │ BK-S-001     │ │ BK-S-001     │                  ║
║  │ (gray text)  │ │              │ │ (red border)  │                  ║
║  └──────────────┘ └──────────────┘ └──────────────┘                  ║
║                                     ⚠️ Already used                    ║
║                                        by another SKU                  ║
║                                                                        ║
║  ACTIVE TOGGLE states:                                                 ║
║                                                                        ║
║  ON              OFF               DISABLED                            ║
║  ┌────────────┐ ┌────────────┐   ┌────────────┐                      ║
║  │ [===on===] │ │ [off======]│   │ [off======]│                      ║
║  │ (green)    │ │ (gray, row │   │ (gray, cur-│                      ║
║  │            │ │  dimmed)   │   │  sor help) │                      ║
║  └────────────┘ └────────────┘   └────────────┘                      ║
║                  Row opacity 0.5  Tooltip: "SKU has                    ║
║                                   incomplete                           ║
║                                   associations"                        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Quick Fill (Batch Apply)

```
╔════════════════════════════════════════════════════════════════════════╗
║  User wants to set the same price for all SKUs                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Quick fill bar below the SKU table:                                   ║
║                                                                        ║
║  ┌────────────────────────────────────────────────────────────────┐   ║
║  │  Set all prices: [24.99__]  [Apply to All]                     │   ║
║  │  Set all stocks:  [100____]  [Apply to All]                     │   ║
║  └────────────────────────────────────────────────────────────────┘   ║
║                                                                        ║
║  Clicking [Apply to All] fills the value into every SKU row.           ║
║  Only affects the specific column (price or stock).                    ║
║  Existing per-SKU values are overwritten.                              ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Quick Fill Feedback States

```
╔════════════════════════════════════════════════════════════════════════╗
║  QUICK FILL INTERACTION LIFECYCLE                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  DEFAULT (empty input):                                                ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │  Set all prices: [__________]  [Apply to All]                  │  ║
║  │                                  (gray, disabled)              │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                        ║
║  VALID (input has value):                                              ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │  Set all prices: [24.99_____]  [Apply to All]                  │  ║
║  │                                  (blue, enabled)               │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                        ║
║  HOVER (button hovered):                                               ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │  Set all prices: [24.99_____]  [Apply to All]                  │  ║
║  │                                  (darker blue)                 │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                        ║
║  APPLYING (mutation in progress):                                      ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │  Set all prices: [24.99_____]  [⏳ Applying...]                │  ║
║  │                   (disabled)    (spinner, disabled)             │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                        ║
║  SUCCESS (brief confirmation, resets after 2s):                        ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │  Set all prices: [__________]  ✓ Applied to 9 SKUs            │  ║
║  │                                  (green text, fades after 2s)  │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### SKU Table Search and Filter

```
╔════════════════════════════════════════════════════════════════════════╗
║  SKU TABLE FILTER BAR                                                 ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  When the SKU matrix grows large, a filter bar appears above the       ║
║  table to help sellers find specific SKUs:                             ║
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  Color: [All ▼]  Size: [All ▼]  SKU: [Search code...________]  │ ║
║  │                                                                  │ ║
║  │  Show: [All] [Active Only] [Inactive Only] [Missing Price]      │ ║
║  │                                                                  │ ║
║  │  Showing 12 of 100 SKUs                                          │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  Property value dropdowns:                                             ║
║  - One dropdown per property (e.g., "Color: [All ▼]", "Size: [All ▼]")║
║  - Options: "All" + each value in that property                       ║
║  - Selecting a value filters table to only SKUs with that value       ║
║  - Multiple dropdowns combine with AND logic                          ║
║                                                                        ║
║  Text search on SKU code:                                              ║
║  - Filters by substring match on sku_code field                       ║
║  - Case-insensitive                                                    ║
║                                                                        ║
║  Quick filter toggles:                                                 ║
║  - [All]: show all SKUs (default)                                     ║
║  - [Active Only]: is_active = true                                     ║
║  - [Inactive Only]: is_active = false                                  ║
║  - [Missing Price]: price = 0.01 (default/unfilled)                    ║
║                                                                        ║
║  All filtering is client-side (no API calls).                          ║
║  Filter bar only shown when total SKU count > 10.                     ║
║  Filtered count: "Showing {visible} of {total} SKUs".                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Drag and Drop SKU Row Reorder

```
╔════════════════════════════════════════════════════════════════════════╗
║  User drags SKU row "Red/S" from position 6 to position 0            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Each SKU row has a drag handle (=) on the left.                       ║
║  Reordering changes the display order only (no business logic).        ║
║                                                                        ║
║  SKU order is persisted via array position in the request body.        ║
║  There is NO position field on the SKU model. The order of items       ║
║  in the skus array in the POST body (or the order of add ops in       ║
║  the PATCH) determines display order. See 07 SellerProductSku          ║
║  interface note.                                                       ║
║                                                                        ║
║  Implementation: @dnd-kit SortableContext on the table body            ║
║  - Each <tr> is a sortable item keyed by SKU combination               ║
║  - onDragEnd recalculates array indices                                ║
║  - SKU order in the create/edit request body reflects the new order    ║
║                                                                        ║
║  Before:                                                               ║
║  ┌───┬───────┬──────┐                                                 ║
║  │ = │ Black │ S    │  pos 0                                          ║
║  │ = │ Black │ M    │  pos 1                                          ║
║  │ = │ Black │ L    │  pos 2                                          ║
║  │ = │ White │ S    │  pos 3                                          ║
║  │ = │ White │ M    │  pos 4                                          ║
║  │ = │ White │ L    │  pos 5                                          ║
║  │ = │ Red   │ S    │  pos 6  <-- drag this                           ║
║  └───┴───────┴──────┘                                                 ║
║                                                                        ║
║  During drag: visual feedback with DragOverlay                         ║
║  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─┐                                               ║
║    = │ Red   │ S      <- floating row following cursor                ║
║  └ ─ ─ ─ ─ ─ ─ ─ ─ ─┘                                               ║
║                                                                        ║
║  After:                                                                ║
║  ┌───┬───────┬──────┐                                                 ║
║  │ = │ Red   │ S    │  pos 0  <-- moved here                          ║
║  │ = │ Black │ S    │  pos 1                                          ║
║  │ = │ Black │ M    │  pos 2                                          ║
║  │ = │ ...   │ ...  │                                                  ║
║  └───┴───────┴──────┘                                                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## No Properties (Single SKU)

When a product has no variation properties, it gets a single SKU:

```
╔════════════════════════════════════════════════════════════════════════╗
║  VARIATIONS & SKUs                                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  No properties defined. This product has a single SKU.                 ║
║  [+ Add Property] to create variations.                                ║
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  SINGLE SKU                                                  │      ║
║  │                                                              │      ║
║  │  Price *   ┌──────────────┐                                  │      ║
║  │            │ 24.99________│                                  │      ║
║  │            └──────────────┘                                  │      ║
║  │                                                              │      ║
║  │  Stock     ┌──────────────┐                                  │      ║
║  │            │ 100__________│                                  │      ║
║  │            └──────────────┘                                  │      ║
║  │                                                              │      ║
║  │  SKU Code  ┌──────────────┐                                  │      ║
║  │            │ MOUSE-001____│                                  │      ║
║  │            └──────────────┘                                  │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Validation Summary

```
Client-side validation (prevent submission):
┌──────────────────────────────────┬──────────────────────────────────┐
│ Rule                             │ Client Error                     │
├──────────────────────────────────┼──────────────────────────────────┤
│ Max 3 properties                 │ "Maximum 3 properties allowed"   │
│ Max 50 values per property       │ "Max 50 values per property"     │
│ Max 100 SKUs                     │ "Max 100 SKUs allowed"           │
│ Duplicate property name          │ "Property name already exists"   │
│ Duplicate value in property      │ "Value already exists"           │
│ Property name empty              │ "Property name is required"      │
│ Property name > 100 chars        │ "Max 100 characters"             │
│ Property value > 100 chars       │ "Max 100 characters"             │
│ Property has no values           │ "Add at least one value"         │
│ SKU price < 0.01                 │ "Price must be at least 0.01"    │
│ SKU price > 9999999999.99        │ "Price exceeds maximum"          │
│ SKU stock < 0                    │ "Stock cannot be negative"       │
│ SKU code > 50 chars              │ "Max 50 characters"              │
│ Duplicate SKU code               │ "SKU code already used"          │
│ Duplicate SKU combination        │ "Duplicate SKU combination"      │
└──────────────────────────────────┴──────────────────────────────────┘

Server error messages (422 responses -- use for mapServerErrors):
┌─────────────────────────────────────────────────────────────────────┐
│ Create:                                                             │
│  "The properties must not have more than 3 items."                  │
│  "The property '{name}' must not have more than 50 values."         │
│  "The skus must not have more than 100 items."                      │
│  "Duplicate property name: '{name}'."                               │
│  "Duplicate value '{value}' in property '{name}'."                  │
│  "The property '{name}' is not defined in the category schema."     │
│  "The value '{value}' is not a valid option for '{name}'."          │
│  "The required property '{name}' is missing."                       │
│  "Each SKU must reference exactly one value from each property."    │
│  "Duplicate SKU: two SKUs reference the same combination of        │
│   property values."                                                 │
│  "The sku_code '{code}' is already used by another SKU in this     │
│   product."                                                         │
│  "The skus.{i}.price must be at least 0.01."                       │
│                                                                     │
│ PATCH (post-operation constraints):                                 │
│  "Product cannot have more than 3 properties. Current: {n}."       │
│  "Property '{name}' cannot have more than 50 values. Current: {n}."│
│  "Product cannot have more than 100 SKUs. Current: {n}."           │
│  "SKU '{id}' has incomplete property associations after value      │
│   removal."                                                         │
│  "SKU price must be at least 0.01."                                │
│  "SKU stock must be a non-negative integer."                        │
│  "Property must include name, position, and values."                │
│  "SKU must include price, stock, and property_value_ids."           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow: Properties -> SKU Matrix

```
         Properties (axes)
              │
              │  Color: [Black, White, Red]
              │  Size:  [S, M, L]
              │
              ▼
     ┌────────────────────┐
     │ Cartesian Product  │
     │ Generator          │
     └────────┬───────────┘
              │
              │  3 x 3 = 9 combinations
              │
              ▼
     ┌────────────────────┐
     │ Merge with existing│
     │ SKU data           │
     └────────┬───────────┘
              │
              │  Existing SKUs: preserve price/stock/code
              │  New SKUs: price=0.01, stock=0, code=null
              │  Removed combos: drop from array
              │
              ▼
     ┌────────────────────┐
     │ Render SKU table   │
     │ (editable)         │
     └────────────────────┘

On save (create):
  SKU property_values are positional [axisIndex, valueIndex]
  Each entry is a 2-element integer array: [propertyIndex, valueIndex]
  Example: [[0, 2], [1, 0]] = 3rd value of 1st property + 1st value of 2nd property

On save (edit — existing SKUs):
  SKU property_value_ids are BIGINT database IDs from associations.data[]
  The GET response includes per-SKU associations:
    skus[].associations.data[] = [
      { property_id: 101, property_value_id: 201, property: {...}, property_value: {...} }
    ]
  The frontend maps these associations back to the property/value UI.

On save (edit — new SKUs added in same patch):
  property_value_ids: [existingDbId, ...]   for existing values
  property_value_refs: ["temp-red-001", ...]  for values being created in same patch
  Server resolves _tempId -> real BIGINT ID during patch processing.

Regional pricing:
  Each SKU can also have regional_prices (see 06-regional-pricing.md).
  Regional prices are NOT part of the SKU builder UI -- they appear
  in the separate Regional Pricing section below the SKU table.
```
