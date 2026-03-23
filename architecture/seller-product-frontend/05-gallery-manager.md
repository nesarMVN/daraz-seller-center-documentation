# Gallery Manager

## Overview

The gallery manager handles product images and videos. Sellers can add media from their Media Center (asset picker), reorder via drag and drop, set a primary image, and manage video entries with thumbnails. The gallery is part of the product create/edit form.

**Constraints:**
- Max 20 items per product (images + videos combined)
- Exactly 1 image must be marked as primary
- Videos require a thumbnail_url
- Videos cannot be marked as primary
- Each item has a position (integer, starting from 0)

**Image requirements** are enforced by the Media Center (asset picker modal).
The gallery manager itself only validates URL format and the 20-item limit.
See [media-center-frontend-architecture.md](../media-center-frontend-architecture.md)
for authoritative values. Typical requirements:
- Accepted formats: JPEG, PNG, WebP
- Max file size: 10 MB
- Min dimensions: 500 x 500 px

> Future: AI-powered image quality scoring and background removal — not in MVP.

---

## Gallery Section Layout

```
╔════════════════════════════════════════════════════════════════════════╗
║  GALLERY                                                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Product Images & Videos (5/20)                                        ║
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │                                                                  │ ║
║  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐│ ║
║  │  │ = drag   │ │ = drag   │ │ = drag   │ │ = drag   │ │ = drag ││ ║
║  │  │          │ │          │ │          │ │          │ │        ││ ║
║  │  │   IMG    │ │   IMG    │ │   IMG    │ │   VID    │ │  IMG   ││ ║
║  │  │          │ │          │ │          │ │  0:45    │ │        ││ ║
║  │  │          │ │          │ │          │ │          │ │        ││ ║
║  │  │ [Primary]│ │          │ │          │ │          │ │        ││ ║
║  │  │          │ │          │ │          │ │          │ │        ││ ║
║  │  │ [x]      │ │ [x]      │ │ [x]      │ │ [x]      │ │ [x]    ││ ║
║  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └────────┘│ ║
║  │   pos: 0       pos: 1       pos: 2       pos: 3       pos: 4  │ ║
║  │   Primary                                  Video               │ ║
║  │                                                                  │ ║
║  │  ┌──────────────┐  ┌──────────────┐                              │ ║
║  │  │              │  │              │                              │ ║
║  │  │   + Add      │  │   + Add      │                              │ ║
║  │  │   Image      │  │   Video      │                              │ ║
║  │  │              │  │              │                              │ ║
║  │  └──────────────┘  └──────────────┘                              │ ║
║  │                                                                  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Workflows

### Add Image from Media Center

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User clicks [+ Add Image]                                   ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Opens the Asset Picker modal from Media Center:                       ║
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Select Product Images                            [x]       ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  ┌──────────────────────────────────────────────────────┐   ║     ║
║  ║  │  Search...                        [Grid] [List]       │   ║     ║
║  ║  └──────────────────────────────────────────────────────┘   ║     ║
║  ║                                                              ║     ║
║  ║  Home > Products                                             ║     ║
║  ║                                                              ║     ║
║  ║  [ Images ]  [ Videos ]  [ All ]                             ║     ║
║  ║                                                              ║     ║
║  ║  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐          ║     ║
║  ║  │ [x] │ │     │ │ [x] │ │     │ │ [x] │ │     │          ║     ║
║  ║  │ IMG │ │ IMG │ │ IMG │ │ IMG │ │ IMG │ │ IMG │          ║     ║
║  ║  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘          ║     ║
║  ║                                                              ║     ║
║  ║  3 images selected                                           ║     ║
║  ║  - mouse-front.jpg (2.3 MB)  [x]                            ║     ║
║  ║  - mouse-side.jpg (1.8 MB)   [x]                            ║     ║
║  ║  - mouse-back.jpg (1.5 MB)   [x]                            ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                    [Cancel]  [Select 3 Images]              ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  Asset Picker props:                                                   ║
║  - mode: "multiple"                                                    ║
║  - fileType: "image"                                                   ║
║  - maxSelection: 20 - currentGalleryCount                              ║
║  - onSelect: receives array of asset objects                           ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                       User clicks [Select 3 Images]
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: Images added to gallery                                      ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  New images appended to gallery with sequential positions:             ║
║                                                                        ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  ║
║  │ = drag   │ │ = drag   │ │ = drag   │ │ = drag   │ │ = drag   │  ║
║  │          │ │          │ │          │ │          │ │          │  ║
║  │  (old)   │ │  (old)   │ │  (new)   │ │  (new)   │ │  (new)   │  ║
║  │ existing │ │ existing │ │ mouse-   │ │ mouse-   │ │ mouse-   │  ║
║  │ img 1    │ │ img 2    │ │ front    │ │ side     │ │ back     │  ║
║  │          │ │          │ │          │ │          │ │          │  ║
║  │ [Primary]│ │          │ │          │ │          │ │          │  ║
║  │ [x]      │ │ [x]      │ │ [x]      │ │ [x]      │ │ [x]      │  ║
║  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  ║
║   pos: 0       pos: 1       pos: 2       pos: 3       pos: 4       ║
║                                                                        ║
║  Each gallery item stores:                                             ║
║  {                                                                     ║
║    asset_id: "uuid-from-media-center",                                 ║
║    url: "https://cdn.../mouse-front.jpg",                              ║
║    type: "image",                                                      ║
║    thumbnail_url: null,                                                ║
║    duration: null,                                                     ║
║    position: 2,                                                        ║
║    is_primary: false                                                   ║
║  }                                                                     ║
║                                                                        ║
║  If this is the first image and no primary exists, auto-set            ║
║  is_primary=true on the first image.                                   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Image Addition Progress

```
╔════════════════════════════════════════════════════════════════════════╗
║  ADDING IMAGES -- VISUAL FEEDBACK                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  When user selects images from the asset picker and clicks             ║
║  [Select N Images], the following sequence plays:                      ║
║                                                                        ║
║  1. Brief overlay on gallery section:                                  ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │                                                                    │║
║  │  ┌────────────────────────────────┐                                │║
║  │  │  Adding 3 images...            │  <- centered overlay           │║
║  │  └────────────────────────────────┘                                │║
║  │                                                                    │║
║  │  ┌────┐ ┌────┐   (existing images dimmed, opacity 0.5)            │║
║  │  │IMG │ │IMG │                                                     │║
║  │  └────┘ └────┘                                                     │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
║  2. New images appear with fade-in animation:                          ║
║  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                                 ║
║  │IMG │ │IMG │ │new │ │new │ │new │  <- opacity 0 -> 1 transition    ║
║  │ ok │ │ ok │ │fade│ │fade│ │fade│    (200ms staggered per item)     ║
║  └────┘ └────┘ └────┘ └────┘ └────┘                                 ║
║                                                                        ║
║  3. Auto-scroll to newly added items if they overflow viewport.        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Add Video

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User clicks [+ Add Video]                                   ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Opens the Asset Picker modal with video filter:                       ║
║                                                                        ║
║  Asset Picker props:                                                   ║
║  - mode: "single"                                                      ║
║  - fileType: "video"                                                   ║
║  - onSelect: receives single video asset                               ║
║                                                                        ║
║  After selection, a thumbnail picker appears:                          ║
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Set Video Thumbnail                              [x]       ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  Video: promotional-video.mp4                                ║     ║
║  ║                                                              ║     ║
║  ║  Thumbnail is required for product videos.                   ║     ║
║  ║                                                              ║     ║
║  ║  ┌────────────────────────┐                                 ║     ║
║  ║  │                        │                                 ║     ║
║  ║  │     [Select from       │                                 ║     ║
║  ║  │      Media Center]     │                                 ║     ║
║  ║  │                        │                                 ║     ║
║  ║  └────────────────────────┘                                 ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                    [Cancel]  [Add Video]                    ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  Video gallery item stores:                                            ║
║  {                                                                     ║
║    asset_id: "video-asset-uuid",                                       ║
║    url: "https://cdn.../video.mp4",                                    ║
║    type: "video",                                                      ║
║    thumbnail_url: "https://cdn.../thumb.jpg",  <- required             ║
║    duration: 45,                                                       ║
║    position: 5,                                                        ║
║    is_primary: false  <- videos cannot be primary                      ║
║  }                                                                     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Drag and Drop Reorder

```
╔════════════════════════════════════════════════════════════════════════╗
║  User drags image from position 3 to position 0                       ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before:                                                               ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                 ║
║  │ = IMG A  │ │ = IMG B  │ │ = IMG C  │ │ = IMG D  │                 ║
║  │ [Primary]│ │          │ │          │ │          │                 ║
║  │  pos: 0  │ │  pos: 1  │ │  pos: 2  │ │  pos: 3  │                 ║
║  └──────────┘ └──────────┘ └──────────┘ └──────────┘                 ║
║                                                                        ║
║  User grabs IMG D by the drag handle (=), drags to first position:     ║
║                                                                        ║
║  During drag:                                                          ║
║  ┌ ─ ─ ─ ─ ─┐                                                        ║
║    = IMG D    <-- DragOverlay (floating preview)                      ║
║  └ ─ ─ ─ ─ ─┘                                                        ║
║                                                                        ║
║  ▼ drop zone                                                           ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐                              ║
║  │ = IMG A  │ │ = IMG B  │ │ = IMG C  │                              ║
║  └──────────┘ └──────────┘ └──────────┘                              ║
║                                                                        ║
║  After:                                                                ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                 ║
║  │ = IMG D  │ │ = IMG A  │ │ = IMG B  │ │ = IMG C  │                 ║
║  │          │ │ [Primary]│ │          │ │          │                 ║
║  │  pos: 0  │ │  pos: 1  │ │  pos: 2  │ │  pos: 3  │                 ║
║  └──────────┘ └──────────┘ └──────────┘ └──────────┘                 ║
║                                                                        ║
║  Note: is_primary stays on IMG A (does not follow position).           ║
║  Primary is a flag on the item, not tied to position 0.                ║
║                                                                        ║
║  Implementation:                                                       ║
║  - @dnd-kit DndContext + SortableContext                                ║
║  - Each gallery card is a useSortable item                             ║
║  - onDragEnd: splice item array, recalculate all positions             ║
║  - Positions are 0-indexed sequential integers                         ║
║                                                                        ║
║  For edit mode, generates patch ops:                                   ║
║  [                                                                     ║
║    { op: "replace", path: "/gallery/{D.id}/position", value: 0 },     ║
║    { op: "replace", path: "/gallery/{A.id}/position", value: 1 },     ║
║    { op: "replace", path: "/gallery/{B.id}/position", value: 2 },     ║
║    { op: "replace", path: "/gallery/{C.id}/position", value: 3 }      ║
║  ]                                                                     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Set Primary Image

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks an image to set it as primary                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before:                                                               ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐                              ║
║  │ IMG A    │ │ IMG B    │ │ IMG C    │                              ║
║  │ [Primary]│ │          │ │          │                              ║
║  └──────────┘ └──────────┘ └──────────┘                              ║
║                                                                        ║
║  User right-clicks IMG B -> "Set as Primary"                           ║
║  (or clicks a star icon overlay on the image)                          ║
║                                                                        ║
║  After:                                                                ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐                              ║
║  │ IMG A    │ │ IMG B    │ │ IMG C    │                              ║
║  │          │ │ [Primary]│ │          │                              ║
║  └──────────┘ └──────────┘ └──────────┘                              ║
║                                                                        ║
║  For create: update is_primary in form state.                          ║
║  For edit: generates patch ops:                                        ║
║  [                                                                     ║
║    { op: "replace", path: "/gallery/{A.id}/is_primary", value: false},║
║    { op: "replace", path: "/gallery/{B.id}/is_primary", value: true } ║
║  ]                                                                     ║
║                                                                        ║
║  Constraint: only images can be primary. Video items don't show        ║
║  the "Set as Primary" option.                                          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Remove Gallery Item

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [x] on a gallery item                                    ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  If the item is NOT primary:                                           ║
║  - Remove immediately from gallery array                               ║
║  - Recalculate positions                                               ║
║  - For edit: { op: "remove", path: "/gallery/{id}" }                   ║
║                                                                        ║
║  If the item IS primary:                                               ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Cannot Remove Primary Image                       [x]      ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  This is the primary product image. Set another image         ║     ║
║  ║  as primary before removing this one.                        ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                            [OK]             ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  Alternative: if only 1 image remains, removing it is allowed          ║
║  only if the product is a draft (no primary required for drafts).      ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Empty State

```
╔════════════════════════════════════════════════════════════════════════╗
║  GALLERY (0/20)                                                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │                                                                  │ ║
║  │                     No images or videos yet                      │ ║
║  │                                                                  │ ║
║  │          Add product images from your Media Center               │ ║
║  │                                                                  │ ║
║  │          ┌──────────────┐  ┌──────────────┐                      │ ║
║  │          │  + Add Image │  │  + Add Video │                      │ ║
║  │          └──────────────┘  └──────────────┘                      │ ║
║  │                                                                  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Gallery Limit States

```
╔════════════════════════════════════════════════════════════════════════╗
║  GALLERY LIMIT INDICATORS                                             ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  NEAR LIMIT (18/20):                                                   ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  Product Images & Videos (18/20)                                  │║
║  │  Warning: 2 slots remaining                                       │║
║  │  (orange warning text)                                            │║
║  │                                                                    │║
║  │  ┌────┐ ┌────┐ ┌────┐  ... (18 items)  ┌────────┐ ┌────────┐   │║
║  │  │IMG │ │IMG │ │IMG │                   │+ Image │ │+ Video │   │║
║  │  └────┘ └────┘ └────┘                   └────────┘ └────────┘   │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
║  AT LIMIT (20/20):                                                     ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  Product Images & Videos (20/20)                                  │║
║  │  Warning: Maximum reached -- remove items to add more             │║
║  │  (orange warning text)                                            │║
║  │                                                                    │║
║  │  ┌────┐ ┌────┐ ┌────┐  ... (20 items)                            │║
║  │  │IMG │ │IMG │ │IMG │                                             │║
║  │  └────┘ └────┘ └────┘                                            │║
║  │                                                                    │║
║  │  [+ Add Image]  [+ Add Video]  <- both disabled (gray),           │║
║  │                                  cursor not-allowed                │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
║  OVER LIMIT (server returned 422 -- should not happen client-side):    ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  Maximum 20 images/videos allowed. Remove items before saving.    │║
║  │  (red banner, red-50 bg above gallery grid)                       │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Gallery Item Detail View

```
┌──────────────────────┐
│ = drag handle        │
│ ┌──────────────────┐ │
│ │                  │ │
│ │    IMAGE         │ │
│ │    PREVIEW       │ │
│ │                  │ │
│ └──────────────────┘ │
│                      │
│ [Primary] badge      │  <- only on primary image
│ [VID 0:45] badge     │  <- only on videos (with duration)
│                      │
│ mouse-front.jpg      │  <- file name (truncated)
│ 2.3 MB               │  <- file size
│                      │
│ [x] Remove           │
└──────────────────────┘
```

### Gallery Card Interaction States

```
╔════════════════════════════════════════════════════════════════════════╗
║  GALLERY CARD STATES                                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  DEFAULT (white border):                                               ║
║  ┌──────────────────────┐                                             ║
║  │ = drag               │                                             ║
║  │ ┌──────────────────┐ │                                             ║
║  │ │                  │ │  (1px gray border, no overlay)               ║
║  │ │    IMAGE         │ │                                             ║
║  │ │                  │ │                                             ║
║  │ └──────────────────┘ │                                             ║
║  │ [x] Remove           │                                             ║
║  └──────────────────────┘                                             ║
║                                                                        ║
║  HOVER (action overlay appears):                                       ║
║  ┌──────────────────────┐                                             ║
║  │ = drag               │                                             ║
║  │ ┌──────────────────┐ │                                             ║
║  │ │ ┌──────────────┐ │ │  (2px blue border, dark overlay)            ║
║  │ │ │ View Set Rm   │ │ │  (action icons centered on overlay:        ║
║  │ │ │              │ │ │   View, Set Primary, Remove)                ║
║  │ │ └──────────────┘ │ │                                             ║
║  │ └──────────────────┘ │                                             ║
║  └──────────────────────┘                                             ║
║                                                                        ║
║  PRIMARY (gold badge, gold border):                                    ║
║  ┌──────────────────────┐                                             ║
║  │ * Primary             │                                             ║
║  │ ┌──────────────────┐ │  (2px gold border #eab308,                  ║
║  │ │                  │ │   star badge top-left corner)                ║
║  │ │    IMAGE         │ │                                             ║
║  │ │                  │ │                                             ║
║  │ └──────────────────┘ │                                             ║
║  └──────────────────────┘                                             ║
║                                                                        ║
║  DRAGGING:                                                             ║
║  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┐                                             ║
║    = drag                  (DragOverlay: semi-transparent copy,        ║
║    ┌──────────────────┐     slight shadow, follows cursor)             ║
║    │    IMAGE         │                                               ║
║    └──────────────────┘                                               ║
║  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┘                                             ║
║                                                                        ║
║  DROP ZONE (between cards):                                            ║
║  ┌──────────┐ ┃ ┌──────────┐                                         ║
║  │  IMG A   │ ┃ │  IMG B   │  (blue dashed border indicator           ║
║  └──────────┘ ┃ └──────────┘   between cards shows drop position)     ║
║                                                                        ║
║  VIDEO CARD (with play overlay):                                       ║
║  ┌──────────────────────┐                                             ║
║  │ = drag               │                                             ║
║  │ ┌──────────────────┐ │                                             ║
║  │ │      >           │ │  (play > overlay centered on thumbnail)     ║
║  │ │   THUMBNAIL      │ │                                             ║
║  │ │           [0:45] │ │  (duration badge bottom-right)              ║
║  │ └──────────────────┘ │                                             ║
║  │ [VID] [x] Remove     │                                             ║
║  └──────────────────────┘                                             ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Error Recovery

```
╔════════════════════════════════════════════════════════════════════════╗
║  GALLERY ERROR SCENARIOS                                              ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  1. Asset deleted from Media Center after being added to gallery:      ║
║                                                                        ║
║     ┌──────────┐ ┌──────────┐ ┌──────────┐                           ║
║     │ = IMG A  │ │ = ⚠ ERR │ │ = IMG C  │                           ║
║     │ [Primary]│ │ Broken   │ │          │                           ║
║     │          │ │ asset    │ │          │                           ║
║     │ [x]      │ │ [x]      │ │ [x]      │                           ║
║     └──────────┘ └──────────┘ └──────────┘                           ║
║                                                                        ║
║     Show broken placeholder with warning icon.                        ║
║     User must remove and re-add from Media Center.                    ║
║     Submit blocked until broken references resolved.                  ║
║                                                                        ║
║  2. Gallery exceeds 20 items during asset picker selection:            ║
║                                                                        ║
║     Asset picker enforces maxSelection = 20 - currentGalleryCount.    ║
║     If gallery has 18 items, picker allows selecting max 2.           ║
║     Selecting more shows inline warning in the picker.                ║
║                                                                        ║
║  3. Server 422 on gallery validation:                                  ║
║                                                                        ║
║     Error: "Exactly one gallery image must be marked as primary."     ║
║     -> Show banner error above gallery section                         ║
║     -> Highlight gallery section header in red                         ║
║     -> Scroll to gallery section                                       ║
║                                                                        ║
║  4. Video without thumbnail submitted:                                 ║
║                                                                        ║
║     ┌──────────┐                                                      ║
║     │ = VID    │                                                      ║
║     │ ⚠ No    │  <- red border + warning icon                       ║
║     │ thumbnail│                                                      ║
║     │ [x]      │                                                      ║
║     └──────────┘                                                      ║
║     "Video requires a thumbnail image" inline error.                  ║
║     Click the video card to open thumbnail picker.                    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Validation Summary

```
┌───────────────────────────────────┬──────────────────────────────────────┐
│ Rule                              │ Error Message                        │
├───────────────────────────────────┼──────────────────────────────────────┤
│ Max 20 gallery items              │ "Maximum 20 images/videos allowed"   │
│ Exactly 1 primary image           │ "Set one image as primary"           │
│ Video without thumbnail           │ "Video requires a thumbnail image"   │
│ is_primary on video               │ "Only images can be primary"         │
│ Asset not found (stale reference) │ "Asset no longer exists"             │
│ Gallery URL invalid               │ "Invalid image URL"                  │
└───────────────────────────────────┴──────────────────────────────────────┘
```

---

## Video Gallery Cards

```
╔════════════════════════════════════════════════════════════════════════╗
║  VIDEO CARD DISPLAY                                                    ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Video gallery cards show:                                             ║
║                                                                        ║
║  ┌──────────────────┐                                                 ║
║  │ ┌──────────────┐ │                                                 ║
║  │ │              │ │                                                 ║
║  │ │    ▶ PLAY    │ │  ← play button overlay (centered, semi-         ║
║  │ │              │ │    transparent circle with triangle)             ║
║  │ │              │ │                                                 ║
║  │ │         0:32 │ │  ← duration badge (bottom-right, dark bg        ║
║  │ └──────────────┘ │    with white text, format: "m:ss")             ║
║  │                  │                                                  ║
║  │  product-demo.mp4│                                                 ║
║  │  [Set Primary]   │  ← disabled for videos (tooltip: "Only          ║
║  │  [Remove]        │    images can be primary")                      ║
║  └──────────────────┘                                                 ║
║                                                                        ║
║  Thumbnail source:                                                     ║
║  - Server-generated thumbnail from Media Center (preferred).          ║
║  - Falls back to first frame if no thumbnail available.               ║
║  - thumbnail_url is required for videos (server validates).           ║
║                                                                        ║
║  Duration:                                                             ║
║  - Sourced from Media Center asset metadata (duration field).         ║
║  - Stored as integer (seconds) in gallery item.                        ║
║  - Display format: "m:ss" (e.g., "0:32", "2:15").                    ║
║  - Duration is optional; badge hidden if null.                         ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Accessibility

```
╔════════════════════════════════════════════════════════════════════════╗
║  ACCESSIBILITY REQUIREMENTS                                           ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Keyboard navigation:                                                  ║
║  - Tab: move focus between gallery cards                              ║
║  - Enter: open detail view / asset picker for focused card            ║
║  - Delete: remove focused card (with confirmation dialog)             ║
║  - Space: toggle primary image on focused card (if image type)        ║
║                                                                        ║
║  ARIA attributes:                                                      ║
║  - role="list" on the gallery grid container                          ║
║  - role="listitem" on each gallery card                               ║
║  - aria-label on action buttons:                                       ║
║    "Set as primary image", "Remove {filename}", "Replace image"       ║
║  - alt text sourced from Media Center asset data (asset.alt_text       ║
║    or asset.filename as fallback)                                      ║
║                                                                        ║
║  Drag-and-drop keyboard support (via @dnd-kit keyboard sensor):        ║
║  - Space: grab / drop the focused card                                ║
║  - Arrow keys: move the grabbed card left / right                     ║
║  - Escape: cancel drag operation                                       ║
║  - Screen reader announcement: "Picked up {filename}. Use arrow       ║
║    keys to move. Press Space to drop."                                 ║
║                                                                        ║
║  Focus management:                                                     ║
║  - After removing a card: focus moves to the next card (or previous   ║
║    if last card was removed).                                          ║
║  - After adding a card: focus moves to the newly added card.          ║
║  - After reordering: focus remains on the moved card in its new       ║
║    position.                                                           ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```
