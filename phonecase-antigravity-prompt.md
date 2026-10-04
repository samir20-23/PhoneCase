# PHONECASE — Build Prompt

You are a senior full-stack engineer and product designer. Build a complete, working web application called **PHONECASE** inside an existing Next.js project.

## 0. Rules of engagement

- The Next.js project already exists. Its folder name is `phonecase`. **Do not recreate it, do not scaffold a new app, do not replace its structure.**
- Firebase is already created. A `.env` file already exists with the Firebase variables. **Do not overwrite or print `.env`.** Read the existing variable names and use them. If a variable is missing, list exactly which one at the end instead of inventing values.
- Do not write code comments anywhere.
- Do not ask me design questions. All UI/UX decisions are specified below. Make the remaining small decisions yourself.
- Do not add features that are not in this prompt. Do not implement payments (no Stripe, PayPal, or card forms).
- Install only the packages listed in section 3. If you need another package, state why in your phase summary before using it.
- Inspect first: `package.json`, `next.config.*`, Tailwind version and config, router type (App Router expected), TypeScript config, and any existing Firebase setup. Reuse what exists.
- Work in the phases of section 2. After each phase, run typecheck, lint, and build, fix errors, then continue to the next phase.
- Everything must really work: real editor, real Firebase writes, real admin lookups. No static mockups, no hardcoded-array dashboards.

## 1. Product summary

PHONECASE is an online **phone-case design studio**. The designer is the product; the store is secondary.

Customer flow:

1. Choose phone brand, then exact model.
2. Choose case type.
3. Open the Design Studio: upload an image, drag/resize/rotate it freely on the case, pick wallpapers from the app's own library, add text and shapes, manage layers.
4. Live preview on the exact case shape, including the camera cutout.
5. Review the final product.
6. Fill in name, phone number, location (optional note).
7. Press **Place Order**: the order is saved to Firebase with a unique order ID.
8. Success page shows the order ID and a **Send order on WhatsApp** button that opens WhatsApp to the store owner with a prefilled message.

Admin flow:

1. Log in with Firebase Authentication.
2. Type an order ID in a prominent search box.
3. See everything: customer name, phone, location, product, background, uploaded image, final preview, print file, design data, status.
4. Update the order status.

**Firebase is the source of truth, not WhatsApp.** If the customer deletes the message text and sends only the order ID, the admin must still find the full order by that ID.

There is **no online payment in V1.** The price is stored in the order for the admin.

## 2. Phases

**Phase 1 — Foundation**
Project inspection, dependencies, design system (tokens, fonts, base components), Firebase client setup, data types, Firestore and Storage rules files, seed loader, phone and case data layer.

**Phase 2 — Customer selection and Design Studio**
Homepage, brand/model selection, case selection, the full editor (desktop and mobile), image upload with quality check, backgrounds, text, shapes, layers, camera cutout and safe area, draft persistence.

**Phase 3 — Order flow**
Final preview page, order form, image uploads to Storage, preview and print export, order creation with unique ID, success page, WhatsApp link.

**Phase 4 — Admin**
Login, route protection, dashboard, quick ID search, orders table, order details, status update.

**Phase 5 — Admin managers and polish**
Phone model manager, case type manager, background manager, template library, empty/error/loading states, animation polish, accessibility pass, final testing against section 15.

## 3. Technology

- Next.js App Router, React, TypeScript (strict), Tailwind CSS
- Firebase: Auth, Firestore, Storage (client SDK only; no admin credentials in the browser)
- Editor: **Konva with react-konva** (touch support is strong). The editor must be a client-only component (dynamic import with SSR disabled).
- Allowed extra packages: `konva`, `react-konva`, `framer-motion`, `lucide-react`, `idb-keyval`, `zustand` (editor state only), `zod`, `clsx`
- Fonts via `next/font/google`: Sora (headings), Inter (UI), plus text-tool fonts: Playfair Display, Bebas Neue, Pacifico, Orbitron, Cairo, Amiri

## 4. Design system (customer side)

The customer side is a premium creative tool, never a CRUD app. Inspiration: Apple product pages, Canva, Figma, Printful, Zakeke. Do not copy any branding.

### Colors

| Token | Value |
|---|---|
| bg | `#0A0A0F` |
| surface | `#12121A` |
| surface-2 | `#1A1A25` |
| surface-3 | `#232333` |
| border | `rgba(255,255,255,0.08)` |
| border-strong | `rgba(255,255,255,0.16)` |
| text | `#F5F5F7` |
| text-muted | `#9A9AAE` |
| accent-from | `#6366F1` |
| accent-to | `#8B5CF6` |
| cyan | `#22D3EE` (small highlights only) |
| success | `#34D399` |
| warning | `#FBBF24` |
| danger | `#F87171` |

Primary gradient: `linear-gradient(135deg, #6366F1, #8B5CF6)`. Background of landing and studio: `bg` with one or two very soft radial glows (indigo at 12% opacity, cyan at 6%) placed off-center. Never a flat pure black everywhere, never rainbow gradients.

### Shape, spacing, depth

- 8px spacing grid. Generous whitespace.
- Radius: 12px inputs and small controls, 16px cards, 24px large panels and sheets, full for chips and avatars.
- Borders: 1px `border`. Hover raises to `border-strong`.
- Shadows: one soft layered shadow for elevated panels only (`0 10px 30px rgba(0,0,0,0.35)`). No giant shadows on every card.
- Glass: use `backdrop-blur-xl` with `rgba(18,18,26,0.7)` only for the studio header, bottom tab bar, and bottom sheets. Nowhere else.

### Typography

- Headings: Sora, weight 600, tight letter-spacing (-0.02em). Hero 56–72px desktop, 36–44px mobile.
- Body: Inter 14–16px. Muted text uses `text-muted`.
- No oversized text outside the hero and section titles.

### Components

- **Primary button:** 48px tall (56px for hero CTA), radius 14px, primary gradient, white text weight 600, hover brightens and `scale(1.02)`, active `scale(0.98)`, visible focus ring in cyan. Loading state shows a small spinner and disables the button.
- **Secondary button:** transparent, 1px `border-strong`, hover fills `surface-2`.
- **Inputs:** 48px tall, `surface-2` background, 1px border, radius 12px, floating or top label, focus ring in accent, clear inline error text in `danger`. Minimum 16px font on mobile to prevent iOS zoom.
- **Chips / category pills:** 36px tall, radius full, selected state uses accent gradient border and subtle fill.
- **Cards:** `surface` background, radius 16px, 1px border, hover translateY(-2px) with border-strong. Image cards crop to a consistent aspect ratio.
- **Thumbnails (wallpapers):** compact squares (aspect 1/1 or 3/4), radius 12px, 3 columns on mobile sheet, 3–4 on desktop panel. Hover scale 1.03. Selected: 2px accent ring plus a small check badge.
- **Toasts:** bottom-center on mobile, top-right on desktop, 4s auto-dismiss.
- **Skeletons:** use shimmering `surface-2` blocks while data loads. Never show a blank screen.

### Motion

Use Framer Motion sparingly: page fade/slide (200–300ms ease-out), card hover, panel open/close, bottom sheet spring, upload progress bar, success checkmark draw. No bouncing, spinning, or looping decoration except the very subtle floating motion of hero phones (translateY ±6px, 6s ease-in-out). Respect `prefers-reduced-motion`.

## 5. Customer screens

### 5.1 Homepage `/`

- Sticky top bar: logo (wordmark "PHONECASE" with small gradient dot), links (Designs, How it works), primary button "Start Designing".
- **Hero:** left column text, right column a composition of 3 phone-case mockups at slight angles with different designs (rendered from the app's own template shapes and gradient wallpapers; no external images). Headline: "Design a phone case that is actually yours." Subheading: "Choose your phone, add your image, create your design and preview it instantly." Primary CTA "Start Designing" (to `/design`), secondary CTA "Explore Designs" (scrolls to designs).
- Sections in order: popular brands (logo-less text cards), popular designs (grid of wallpaper-based case mockups from Firestore), How it works (4 steps: Choose your phone, Create your design, Preview your case, Order), categories (chips linking into the studio), final CTA band.
- Footer: minimal, WhatsApp contact link.

### 5.2 Phone selection `/design`

- Step indicator at the top: Phone → Case → Design.
- Brand cards: Apple, Samsung, Xiaomi, Huawei, Google, Other. Large, simple, selected state clear.
- After brand: model grid (searchable input at top). Each model card shows a small case silhouette generated from the model's template config and the model name. Models grouped by series for Apple and Samsung.
- All data comes from Firestore. Show skeletons while loading.

### 5.3 Case selection `/design/[model]`

- Header with chosen phone and a "Change" link.
- Case type cards (Clear, Matte, Tough, MagSafe): name, short description, price in MAD, availability badge. Selecting continues to the studio with `model` and `caseType` in the URL.

### 5.4 Design Studio `/design/[model]/studio` (the most important screen)

The case preview is the hero. Controls must not overwhelm it.

**Desktop (≥1024px)**

```
┌──────────────────────────────────────────────────────────────┐
│ ‹  PHONECASE   Galaxy A71 · Clear      Undo Redo  Preview [Continue] │ 64px glass header
├────┬───────────────┬──────────────────────────┬──────────────┤
│Rail│ Tool panel    │                          │ Properties / │
│72px│ 320px         │      CASE CANVAS         │ Layers       │
│    │ (Background,  │      (centered, large)   │ 300px        │
│    │  Upload,      │                          │              │
│    │  Text, ...)   │                          │              │
├────┴───────────────┴──────────────────────────┴──────────────┤
│        − 100% +    Fit    Reset    Guides ◉                  │ 48px bottom bar
└──────────────────────────────────────────────────────────────┘
```

- Left rail: icon + label buttons: Background, Upload, Text, Shapes, Templates, Layers. Active tool has accent indicator bar on the left.
- Tool panel slides in next to the rail and shows the active tool's content.
- Center: dark canvas area with a faint dot grid. The case is displayed large with a soft shadow beneath it, vertically centered.
- Right panel: contextual. Nothing selected → case info and guides toggle. Image selected → position, scale, rotation, flip, opacity, crop, image quality indicator. Text selected → text, font, size, weight, color, alignment, opacity, letter spacing, rotation. Shape selected → fill, opacity, rotation, size. Layers list at the bottom of the right panel at all times.
- Keyboard shortcuts: Delete removes selection, Ctrl/Cmd+Z undo, Ctrl/Cmd+Shift+Z redo, Ctrl/Cmd+D duplicate, arrow keys nudge by 1px (10px with Shift).

**Mobile (<1024px)** — purpose-built, not a shrunken desktop.

```
┌────────────────────────┐
│ ‹  Galaxy A71     Save │ 56px header
├────────────────────────┤
│                        │
│      CASE CANVAS       │ fills remaining height
│                        │
│  [contextual toolbar]  │ appears when an object is selected
├────────────────────────┤
│ Bg  Upload Text Tpl More│ 64px bottom tab bar, glass
└────────────────────────┘
```

- Tapping a tab opens a **bottom sheet** (glass, radius 24px top corners, drag handle). Two snap points: 45% and 85% height. The canvas stays visible above the 45% sheet. Sheet content: Background → search + category chips + wallpaper grid; Upload → upload panel; Text → text list + add button + style controls; Templates → template grid; More → Shapes, Layers, Guides, Undo/Redo.
- Contextual floating toolbar under the canvas when an object is selected: duplicate, flip, layer up, layer down, delete.
- Pinch to zoom, two-finger rotate on a selected object, one-finger drag to move. Page scroll must not interfere with canvas gestures (`touch-action: none` on the canvas).
- "Continue" lives in the header on tablet and as a floating pill button on phones.

### 5.5 Image upload panel

Not just a file input. Layout:

1. Large dashed drop zone, radius 16px: "Upload your image" with sub-text "Drag and drop, or tap to choose".
2. Info box (always visible, `surface-2`, small info icon):
   **"Best quality: use a high-resolution image. Recommended: 4K, or at least 2000 px on the shortest side. Low-resolution images may look blurry when printed."**
3. Accepted formats: JPG, PNG, WebP. Max 20 MB. Reject others with a friendly message.
4. After selecting a file: upload progress bar, then thumbnail, file name, pixel dimensions, and the **quality indicator**.
5. Buttons: "Use Another Image" (keeps the previous one available in the panel until replaced) and "Remove".

**Quality logic** (do not blindly reject non-4K images; judge by print resolution):

- Each phone template stores `printWidthMm` and `printHeightMm`.
- Effective resolution of an image layer = `naturalWidthPx / (displayedWidthMm / 25.4)` (pixels per inch at its current scale on the case). Recompute live when the user resizes the image.
- Excellent: ≥ 250 PPI. Good: 150–249. Low: 100–149 (warning, allowed). Too small: < 100 (strong warning, still allowed but needs confirmation before order).
- Show a colored badge: Excellent ✓ (success), Good (cyan), Low quality ⚠ (warning), Too small ⚠ (danger).
- Message example: "This image is 900 × 900 px. It may appear blurry on a large print area." plus a "Use Another Image" button.
- Also show the quality of the layer in the right panel and in the final order review.

### 5.6 Image manipulation

Uploaded image becomes a Konva layer with a transformer: drag, resize with corner handles, rotate handle, flip horizontal/vertical, opacity, and **crop**. Crop mode: a rectangle overlay over the image with draggable edges; applying it stores crop values (`cropX`, `cropY`, `cropWidth`, `cropHeight`) on the layer, non-destructively. The original file is never altered or deleted.

### 5.7 Backgrounds (wallpapers)

- Panel: search input, horizontal category chips (All, Anime, Football, Cars, Gaming, Abstract, Minimal, Luxury, Nature, Space, Arabic, Patterns, Dark, Purple, Black, White), thumbnail grid.
- Data from Firestore `backgrounds`, categories from `backgroundCategories`. Paginate in pages of 24 with "Load more" or infinite scroll. Lazy-load thumbnails. Use small thumbnail URLs for the grid and the full image only when selected.
- Selecting a wallpaper sets it as the single background layer (locked at the bottom, always `cover`-fitted, with a scale/position adjust in the right panel). The user can also choose a solid color or gradient background.
- The selected background's `id` and `name` are saved in the order.

### 5.8 Text, shapes, layers, templates

- **Text:** real Konva text layer. Controls: content, font (the fonts in section 3, grouped Latin/Arabic), size, weight, color, alignment, opacity, letter spacing, rotation. Double-click or double-tap to edit in place.
- **Shapes:** circle, square, rounded rectangle, heart, star, diamond, line. Controls: fill color, opacity, rotation, size.
- **Layers panel:** list with type icon, name, visibility toggle, lock toggle, delete, and drag-to-reorder. The selected layer has an accent border and tinted row. The background layer is listed at the bottom and cannot be reordered above others.
- **Templates:** Firestore `templates` collection. Each template is a set of **editable layers** (background reference, placeholder text, shapes, a placeholder image slot), not a flattened picture. Categories include Anime, Football, Cars, Gaming, Minimal, Luxury, Names, Arabic Calligraphy, Couple, Kids, Black & Purple, Space. Applying a template replaces the current design after a confirmation if the design is not empty.

### 5.9 Case shape, camera cutout, safe area

Each phone model has a template config (section 8). The case is **rendered procedurally from config**, not from a PNG:

- Case outline: rounded rectangle from `widthMm`, `heightMm`, `cornerRadiusMm`.
- Camera cutout: shape from `camera` (normalized coordinates 0–1 of the case; type `rounded-rect`, `circle`, or `multi-circle`).
- Safe area: inset from the case edge defined in config.
- Layer stack (bottom to top): design layers → clip to case outline → case rim/gloss overlay → camera cutout drawn as a dark cutout with a subtle inner shadow → guides layer (editor only).
- Guides (toggle, on by default in studio, always off in exports): dashed safe-area outline, camera area highlight with label "Camera area", and a faint warning tint if text or a face-sized region overlaps the camera zone.
- If the model template is flagged `verified: false`, show a small "Dimensions are approximate" note in the studio. Do not present placeholder dimensions as manufacturing-ready.

### 5.10 Draft persistence

Autosave the design (layer JSON plus uploaded image blobs) to IndexedDB (`idb-keyval`) every few seconds and on route change. Restore on reload with a toast "Draft restored". Key the draft by model and case type. Clear the draft after a successful order.

### 5.11 Final preview `/preview`

- Large case render showing exactly what will be ordered: background, uploaded image in its exact position, text, shapes, camera cutout.
- Side card: phone brand, model, case type, selected background name, price, and the image quality badge for each uploaded image.
- If any image is "Too small", show a warning and require a checkbox "I understand this may print blurry".
- Required checkbox: "I have the right to use these images." The store reviews designs before printing.
- Buttons: "Back to editor", "Order this case".

### 5.12 Order form `/order`

- Left: form. Right (sticky): order summary with small case preview, phone model, case type, background name, price.
- Fields: **Full name** (required), **Phone number** (required; accept 06/07/05 + 8 digits, or +212 format; normalize and validate), **Location** (required; city and area), **Note** (optional). Large touch targets, inline validation with friendly messages (zod).
- Button: "Place Order" with loading text states: "Uploading your images…" → "Saving your design…" → "Creating your order…". Disable the form during submission.
- No payment fields of any kind.

### 5.13 Success page `/order/success`

- Animated checkmark, heading "Your order is ready".
- Large order ID with a copy button, case preview, phone model, case type, customer name, location.
- Text: "Your order has been saved. Send the order ID through WhatsApp to confirm the order."
- Button: **Send order on WhatsApp**.
- Render from data stored in `localStorage` at order time (order summary and a preview data URL) so the page works after reload and does not need to read Firestore (public reads of orders are not allowed). If that data is missing, show only the ID input state with a WhatsApp button that sends the ID only.

## 6. WhatsApp

- Store owner number is stored in one constant. Local format `0708104761`; **international format for the link is `212708104761`**. URL: `https://wa.me/212708104761?text=<encoded message>`.
- Message template (built from the saved order values):

```
Hi, I want to order this phone case.

Order ID: {orderId}
Name: {customerName}
Phone: {customerPhone}
Location: {customerLocation}
Phone model: {brand} {model}
Case: {caseType}
Price: {price} {currency}
```

- Open with `window.open(url, "_blank")`, with a fallback link visible on the page.
- The button is only enabled once the order has been confirmed as saved. The message is generated from the saved values, never from user-edited content.

## 7. Order ID

- Format: `PC-YYYYMMDD-XXXXXX` where `XXXXXX` is 6 characters from the alphabet `ABCDEFGHJKLMNPQRSTUVWXYZ23456789` (no 0, O, 1, I), generated with `crypto.getRandomValues`.
- The ID is the Firestore document ID in `orders`. Creation must fail if the document exists; on conflict, regenerate and retry up to 5 times.
- The ID is generated **before** uploads begin, because uploaded files are stored under `orders/{orderId}/...`.

## 8. Data model (Firestore)

Collections: `phoneModels`, `caseTypes`, `backgrounds`, `backgroundCategories`, `templates`, `orders`, `admins`.

```ts
type PhoneModel = {
  id: string
  brand: string
  model: string
  series?: string
  active: boolean
  sortOrder: number
  caseTypeIds: string[]
  template: {
    widthMm: number
    heightMm: number
    cornerRadiusMm: number
    printWidthMm: number
    printHeightMm: number
    bleedMm: number
    canvasWidth: number
    canvasHeight: number
    camera: { type: "rounded-rect" | "circle" | "multi-circle"; x: number; y: number; w: number; h: number; radius?: number; circles?: { cx: number; cy: number; r: number }[] }
    safeAreaInsetMm: number
    verified: boolean
  }
  previewImageUrl?: string
}

type CaseType = {
  id: string
  name: string
  description: string
  price: number
  currency: "MAD"
  available: boolean
  imageUrl?: string
}

type Background = {
  id: string
  title: string
  categoryId: string
  thumbUrl: string
  imageUrl: string
  width: number
  height: number
  active: boolean
  createdAt: Timestamp
}

type OrderStatus = "pending" | "confirmed" | "in_production" | "ready" | "delivered" | "cancelled"

type Order = {
  orderId: string
  customer: { name: string; phone: string; location: string; note: string }
  product: { phoneModelId: string; brand: string; model: string; caseTypeId: string; caseType: string; price: number; currency: "MAD" }
  design: {
    backgroundId: string | null
    backgroundName: string | null
    previewUrl: string
    printUrl: string
    originalImageUrls: { url: string; widthPx: number; heightPx: number; quality: string }[]
    editableData: { canvasWidth: number; canvasHeight: number; background: unknown; objects: unknown[] }
  }
  search: { orderId: string; name: string; phone: string; model: string }
  whatsappPhone: string
  status: OrderStatus
  createdAt: Timestamp
  updatedAt: Timestamp
}
```

Save **both** the final preview image and the full editable design data so the design can be reconstructed. Large images never go inside Firestore documents; only URLs and metadata do.

### Storage paths

- `orders/{orderId}/original/{filename}`
- `orders/{orderId}/preview/final.png`
- `orders/{orderId}/print/final.png`
- `backgrounds/{categoryId}/{filename}` and `backgrounds/{categoryId}/thumbs/{filename}`
- `products/{brand}/{model}/{filename}`

## 9. Exports

- **Preview image:** Konva stage export with the case rim, camera cutout, and gloss included, guides hidden. Used on the preview, success page, and admin.
- **Print file:** a separate export at 300 DPI from `printWidthMm × printHeightMm` (plus `bleedMm`), design layers only, clipped to the case outline, **camera area cleared to transparent**, no guides, no rim or gloss. Cap the longest side at 4096 px to protect mobile browsers. PNG.
- Do not confuse the UI preview with the print file.
- **CORS:** images from Firebase Storage taint the canvas unless the bucket allows CORS. Load every remote image with `crossOrigin="anonymous"`. Create a `cors.json` in the project root allowing GET from the site's origins and `localhost`, and give me the exact `gsutil cors set` command to run. Customer-uploaded images are local blobs and are not affected.

## 10. Security rules

Provide `firestore.rules` and `storage.rules` files and explain where to paste them.

**Firestore**

- `isAdmin()` = signed in and a document exists at `admins/{uid}`.
- `phoneModels`, `caseTypes`, `backgrounds`, `backgroundCategories`, `templates`: public read, admin write.
- `orders`: **public create only**, validated: document ID matches `^PC-[0-9]{8}-[A-Z0-9]{6}$` and equals `orderId`; `status == "pending"`; only the expected keys; string fields present with length limits; `price` is a number; `createdAt`/`updatedAt` are server timestamps. **No public read, update, or delete.** Admin can read, update (status and notes), and delete.
- `admins`: no public access.
- Price is stored as a snapshot from the product the customer chose; the admin confirms it when processing the order.

**Storage**

- `orders/{orderId}/**`: public **create** only, image content types only, size limit (20 MB originals, 15 MB exports), orderId path segment must match the ID regex. No public list, update, or delete. Admin read and delete.
- `backgrounds/**` and `products/**`: public read, admin write.
- The admin sees order images through the download URLs saved in the order document.

## 11. Admin (separate visual style)

The admin is a calm, professional operations dashboard, **visually different from the customer side**: light theme, `#F7F7F9` background, white cards, 1px `#E7E7EE` borders, indigo `#4F46E5` accent, Inter only, 14px base text, compact tables, radius 12px. No glass, no glows.

- **`/admin/login`:** centered card, email and password (Firebase Auth), friendly errors. Non-admin accounts get "You do not have access".
- **Route protection:** all `/admin/*` routes require an authenticated user with a document in `admins`. Redirect to login otherwise. Never render admin data before the check completes. Show a loading state.
- **Layout:** left sidebar (Dashboard, Orders, Products, Backgrounds, Phone models), top bar with the user and sign out.
- **`/admin`:** the **Quick Search** is the first thing on the page: a large input "Enter Order ID" (autofocus). It accepts IDs in any case, with spaces, with or without the `PC-` prefix; Enter opens `/admin/orders/[id]`. If not found, show "No order found for this ID". Below: stat cards (Total, Pending, Confirmed, In Production, Ready, Delivered) and a Recent Orders table.
- **`/admin/orders`:** table with Order ID, Customer, Phone, Phone model, Case, Location, Status badge, Date. Search by order ID, name, phone, or model (use the `search` fields). Status filter tabs. Pagination.
- **`/admin/orders/[id]`:**
  - Header: order ID with copy button, date, status dropdown (updates `status` and `updatedAt`, with a toast).
  - Customer card: name, phone, location, note. Button "Message customer on WhatsApp" (normalizes Moroccan numbers to international format).
  - Product card: brand, model, case type, price.
  - Design card: large final preview, selected background (thumbnail and name), template if any, uploaded original images with "Download", the **print file** with "Download print file", and a collapsible read-only view of the design data (objects list).
  - Image quality shown for each uploaded image.
- **Status badges:** Pending (amber), Confirmed (blue), In Production (violet), Ready (teal), Delivered (green), Cancelled (red/gray). Small rounded pills with a dot.
- **`/admin/products`, `/admin/phone-models`:** list, add, edit, enable/disable phone models and case types, prices, and all template config fields (dimensions, print area, camera cutout, safe area, `verified` flag). Include a live case preview that updates as the numbers change.
- **`/admin/backgrounds`:** upload (with a generated thumbnail), title, category, enable/disable, delete. Manage categories.
- **Seed:** an admin-only "Load sample data" button on `/admin/products` writes starter phone models, case types, categories, and generated backgrounds using the signed-in admin's client session (no service account keys).

## 12. Seed data

- Brands: Apple, Samsung, Xiaomi, Huawei, Google, Other.
- Apple: iPhone 13, 13 Pro, 13 Pro Max, 14, 14 Pro, 14 Pro Max, 15, 15 Pro, 15 Pro Max.
- Samsung: Galaxy A71, S23, S23 Ultra, S24, S24 Ultra.
- Add a few Xiaomi, Huawei, and Google models.
- Dimensions and camera positions are **placeholders** with `verified: false`. Do not invent precision.
- Case types: Clear (89 MAD), Matte (99 MAD), Tough (119 MAD), MagSafe (129 MAD) as editable starter prices.
- Backgrounds: generate about 20 abstract, gradient, pattern, dark, purple, black, white, minimal, and space-style wallpapers locally (SVG or canvas-rendered). Do **not** download copyrighted images (anime, football, cars, games). I will upload licensed wallpapers for those categories through `/admin/backgrounds`.

## 13. States, errors, accessibility, performance

- Every async action has a loading state with specific text ("Uploading image…", "Saving design…", "Creating order…", "Loading phone models…", "Loading backgrounds…").
- Friendly errors, never raw exceptions: "Upload failed", "This image is too small for a sharp print", "We couldn't save your order. Please try again.", "Please enter your phone number", "This phone model is not available". Offer retry. If order creation fails after uploads, keep the draft and allow retry without re-uploading.
- Empty states with an icon, one sentence, and one action.
- Accessibility: labeled inputs, visible focus states, keyboard-reachable tools and sheets, alt text, contrast AA, `aria-live` for upload and order status, reduced-motion support.
- Performance: Next.js image optimization for static assets, lazy-load wallpaper thumbnails and model images, dynamic import the editor, debounce autosave, avoid re-rendering the whole stage on every pointer move.

## 14. Code structure

Reusable components, no giant page files. Suggested names: `PhoneSelector`, `PhoneModelCard`, `CaseSelector`, `CasePreview`, `DesignCanvas`, `EditorToolbar`, `EditorHeader`, `MobileBottomSheet`, `UploadPanel`, `ImageQualityBadge`, `BackgroundLibrary`, `TemplateLibrary`, `TextTool`, `ShapeTool`, `LayerPanel`, `PropertiesPanel`, `OrderForm`, `OrderSummary`, `OrderSuccess`, `AdminSidebar`, `OrdersTable`, `OrderDetails`, `SearchOrder`, `ProductManager`, `BackgroundManager`, `StatusBadge`.

Organize `lib/` for firebase client, types, order-id, whatsapp, image-quality, case-geometry, and export. Editor state in one Zustand store with undo/redo history. Everything typed. Keep V1 focused; structure the code so payments, accounts, saved designs, 3D previews, and AI tools can be added later, but do not build them.

## 15. Final testing checklist

Run through each of these and report results honestly (what passed, what could not be verified):

**Customer:** choose phone, choose case, upload a low-resolution image (warning shows), upload a high-resolution image (Excellent), drag, resize, rotate, flip, crop, choose wallpaper, add text, add shapes, reorder layers, camera cutout visible, guides toggle, undo/redo, draft restore after refresh, preview matches editor, form validation, order saved to Firestore, files saved to Storage, order ID unique, success page, WhatsApp URL is `https://wa.me/212708104761?text=...` with correct decoded values.

**Admin:** login, non-admin blocked, dashboard loads, search by order ID (any case, with and without prefix), order opens, customer data complete, preview shown, background shown, original image downloadable, print file downloadable and has transparent camera area, status update persists.

**Responsive:** test at 375px, 768px, 1024px, 1440px. The studio on mobile uses the bottom tab bar and bottom sheet and touch gestures work.

**Security:** confirm that an anonymous user cannot read, update, or delete any order, and cannot write outside `orders/{id}` in Storage.

At the end, give me: the list of files created, the exact manual steps I must do (add the admin user and its `admins/{uid}` document, paste rules, run the CORS command, any missing `.env` variables), and a short list of known limitations.

## 16. Never do this

- No generic bootstrap-style or admin-looking customer UI.
- No default browser inputs, plain gray buttons, or huge text everywhere.
- No fake editor with static images.
- No payments.
- No secrets in client code. No printing of `.env`.
- No code comments.
- No changes outside the scope above.

Start by inspecting the existing project, then begin with Phase 1.
