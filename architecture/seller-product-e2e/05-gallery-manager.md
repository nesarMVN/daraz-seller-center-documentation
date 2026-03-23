# Gallery Manager Component — E2E Test Architecture

## Page Object

### GalleryManagerComponent

```
File: e2e/pages/gallery-manager.component.ts
Class: GalleryManagerComponent
```

**Constructor parameter:** `page: Page`

**Locators:**

| Locator | Selector | Description |
|---------|----------|-------------|
| `section` | `page.getByRole('region', { name: /gallery/i })` | Gallery section container |
| `addImageButton` | `section.getByRole('button', { name: /add image/i })` | Opens asset picker for images |
| `addVideoButton` | `section.getByRole('button', { name: /add video/i })` | Opens asset picker for video |
| `galleryGrid` | `section.getByTestId('gallery-grid')` | Grid container holding gallery cards |
| `emptyState` | `section.getByText(/no images or videos/i)` | Empty gallery message |
| `itemCount` | `section.getByTestId('gallery-item-count')` | Shows "X of 20 items" count |
| `limitWarning` | `section.getByText(/maximum.*20/i)` | Limit warning message |
| `assetPickerModal` | `page.getByRole('dialog', { name: /select.*asset/i })` | Media center asset picker modal |
| `errorBanner` | `section.getByRole('alert')` | Error banner for gallery-level errors |

**Dynamic locators:**

| Method | Returns | Description |
|--------|---------|-------------|
| `galleryCard(index: number)` | `Locator` | Gallery card at position index |
| `primaryBadge(index: number)` | `Locator` | Primary badge on card at index |
| `removeButton(index: number)` | `Locator` | Remove (x) button on card at index |
| `setPrimaryButton(index: number)` | `Locator` | "Set as primary" button on card hover |
| `videoOverlay(index: number)` | `Locator` | Video play overlay on video cards |
| `durationBadge(index: number)` | `Locator` | Duration badge on video cards |

**Actions:**

| Method | Description |
|--------|-------------|
| `async addImages(count: number)` | Opens asset picker, selects `count` images, confirms |
| `async addVideo()` | Opens asset picker in single mode, selects a video, selects thumbnail |
| `async removeItem(index: number)` | Clicks remove button on item at index |
| `async setPrimary(index: number)` | Hovers card, clicks "Set as primary" |
| `async dragItem(fromIndex: number, toIndex: number)` | Drags gallery card from one position to another |
| `getItemCount(): Locator` | Returns item count text locator |
| `getAllCards(): Locator` | Returns locator for all gallery cards |

---

## Helpers

Uses media center asset picker interaction. Gallery tests depend on assets existing in the media center for the test seller account.

```
File: e2e/helpers/media-center.helper.ts

export const mediaCenterHelper = {
  /**
   * Uploads test images to media center via API before gallery tests.
   */
  async uploadTestAssets(request: APIRequestContext, count: number): Promise<string[]>

  /**
   * Selects assets in the asset picker modal.
   */
  async selectAssetsInPicker(page: Page, assetCount: number): Promise<void>
}
```

---

## Fixture Wiring

```typescript
galleryManagerComponent: async ({ page }, use) => {
  await use(new GalleryManagerComponent(page))
}
```

---

## Spec File Breakdown

| Spec File | Concern |
|-----------|---------|
| `product-create-gallery.spec.ts` | Gallery manager in create flow context |
| `product-edit-page.spec.ts` | Gallery editing in edit flow (subset) |

---

## Test Scenarios

### Empty State

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 1 | should display empty gallery state when no items added | Navigate to create page, select category | Observe gallery section | Empty state message visible: "No images or videos added." Add image and add video buttons visible |
| 2 | should show "0 of 20" item count in empty state | Navigate to create page with category selected | Observe item count | Item count displays "0 of 20 items" |

### Adding Images

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 3 | should open asset picker modal when add image button is clicked | Gallery section visible | Click "Add Image" | Asset picker modal opens in multiple selection mode |
| 4 | should add selected images to gallery grid | Open asset picker | Select 3 images, confirm | 3 image cards appear in gallery grid. First image auto-marked as primary. Item count updates to "3 of 20" |
| 5 | should auto-set first image as primary when gallery was empty | Gallery empty | Add first image | Image card shows primary badge. is_primary is true |
| 6 | should not change primary when adding additional images | Gallery has 1 primary image | Add 2 more images | New images added without primary badge. Original primary unchanged |

### Adding Videos

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 7 | should add a video with thumbnail to gallery | Gallery has images | Click "Add Video", select video, select thumbnail | Video card appears with play overlay and duration badge. Thumbnail visible on card |
| 8 | should not allow video to be set as primary | Gallery has a video | Attempt to set video as primary | "Set as primary" button not available on video cards. Only images can be primary |

### Drag-and-Drop Reorder

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 9 | should reorder gallery items via drag and drop | Gallery has 4 items | Drag item from position 3 to position 0 | Items reordered. Position values updated. is_primary stays on the same item (not position-based) |
| 10 | should preserve primary indicator on the same item after reorder | Primary image at position 0 | Drag primary image to position 2 | Primary badge moves with the item to position 2. No other item gains primary |
| 11 | should support keyboard-based reorder | Focus on a gallery card | Use keyboard shortcuts for drag | Item moves to new position. Order updates |

### Set Primary Image

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 12 | should set a different image as primary | Gallery with 3 images, first is primary | Hover second image, click "Set as primary" | Second image gets primary badge. First image loses primary badge |
| 13 | should not allow setting video as primary | Gallery with images and a video | Hover video card | No "Set as primary" option available for video cards |

### Remove Gallery Item

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 14 | should remove a non-primary gallery item | Gallery with 3 images | Click remove on second image | Second image removed. Item count decreases. Primary unchanged |
| 15 | should not allow removing primary image when other images exist | Gallery with 3 images, first is primary | Attempt to remove primary image | Remove button disabled or action blocked with message "Reassign primary before removing" (unless it is the last image in draft) |
| 16 | should allow removing last remaining gallery item | Gallery with 1 image (primary) | Click remove | Image removed. Gallery returns to empty state |
| 17 | should update item count after removal | Gallery with 5 items showing "5 of 20" | Remove one item | Count updates to "4 of 20" |

### Limit Enforcement

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 18 | should show warning when approaching gallery limit | Gallery has 18 items | Observe | Warning text visible: "2 items remaining" or similar near-limit indicator |
| 19 | should disable add buttons when gallery has 20 items | Gallery has 20 items | Observe add buttons | "Add Image" and "Add Video" buttons are disabled. Limit message visible: "Maximum 20 items reached" |
| 20 | should prevent adding images beyond 20 limit via asset picker | Gallery has 18 items | Open asset picker, try to select 5 images | Asset picker limits selection to 2 (remaining capacity). Or: all 5 selected but only 2 added with warning toast |
| 21 | should show server 422 error when gallery exceeds 20 on submit | Manipulate form to have 21 items (edge case) | Submit form | Error: "The gallery must not have more than 20 items" |

### Primary Image Validation

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 22 | should show validation error when no primary image is set on submit | Gallery has images but none marked primary (edge case) | Submit for review | Error: "Exactly one gallery image must be marked as primary" |
| 23 | should show validation error when primary is set on a video | Edge case: is_primary true on video type | Submit | Error: "Only images can be marked as primary" |

### Video Validation

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 24 | should require thumbnail_url for video gallery items | Add a video without selecting thumbnail | Submit | Error: "Gallery video must include a thumbnail_url" |

### Gallery Card States

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 25 | should show hover state with action buttons on image card | Gallery has images | Hover over an image card | Card shows hover overlay with "Set as primary" and "Remove" action buttons |
| 26 | should show primary badge on the primary image card | Gallery has a primary image | Observe primary card | Primary badge (star or "Primary" label) visible on the card |
| 27 | should show play overlay on video cards | Gallery has a video | Observe video card | Play icon overlay visible. Duration badge shows video length |
| 28 | should show dragging state during drag operation | Start dragging a card | Observe | Dragged card shows drag overlay style. Drop placeholder visible at target position |

### Error Recovery

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 29 | should handle broken asset gracefully | Gallery has an item with broken URL | Observe gallery | Broken image shows placeholder/error icon. Gallery remains functional |
| 30 | should show error when asset_id is not found on submit | Gallery references a deleted asset | Submit form | Error: "The asset '{asset_id}' was not found" |

### Edit Context

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 31 | should display existing gallery items when editing a product | Navigate to edit page for product with gallery | Observe gallery section | All gallery items displayed in correct order. Primary badge on correct item. Videos show thumbnails and duration |
| 32 | should track gallery changes for JSON Patch generation | In edit mode, reorder items and add one | Save changes | PATCH includes position replace ops for reordered items and add op for new item |
