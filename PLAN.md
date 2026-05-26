# แผนพัฒนา Souphattra Signage Studio — Modern Web App
**วันที่วิเคราะห์:** 26 พฤษภาคม 2026  
**ไฟล์หลัก:** `souphattra_sign_generator.html`  
**ภาพอ้างอิง:** 4 ภาพ (ป้ายงานเดี่ยว × 2, ป้าย 2 งาน, ป้าย 3 งาน)

---

## ส่วนที่ 1 — วิเคราะห์โครงสร้างปัจจุบัน (Current Structure Analysis)

### 1.1 โครงสร้าง HTML (Layout)

```
<body>
 ├── <aside>  — แถบควบคุมซ้าย (460px กว้างคงที่)
 │    ├── Logo Header (Souphattra brand bar)
 │    ├── Mode Selector (Single / Dual)
 │    ├── Section 1: Global Settings
 │    │    ├── Orientation (Portrait / Landscape)
 │    │    ├── Toggle Border
 │    │    └── Toggle Watermark
 │    ├── Section 2: Event 1 Config
 │    │    ├── Background Style (white / navy / dark)
 │    │    ├── Layout Style (stacked / columns)
 │    │    ├── Co-host Logos (EU / Gov / WB / AusAid)
 │    │    ├── Custom Logo Upload
 │    │    ├── Event Title (textarea + font-size slider)
 │    │    └── Floor / Room / Date / Time
 │    ├── Section 3: Event 2 Config (hidden unless Dual mode)
 │    │    └── [เหมือน Section 2 ทุกอย่าง]
 │    ├── Section 4: Decorative / Canvas Padding / Theme
 │    └── Print Button (sticky bottom)
 │
 └── <main>  — พื้นที่ Preview ขวา
      └── #sign-sign-frame (480×720px A4-ish canvas)
           ├── #border-inner (เส้นขอบทองบาง)
           ├── #watermark-container (SVG ลายน้ำ Souphattra)
           ├── <header> (โลโก้โรงแรม + ชื่อ)
           ├── #preview-workspace (inject Event Cards by JS)
           └── <footer> (❖ ❖ ❖ decor)
```

### 1.2 CSS Styles ที่สำคัญ

| Class / Variable | หน้าที่ |
|---|---|
| `--brand-gold: #c5a880` | สีทองหลักของ Souphattra |
| `--brand-dark: #1a1a1a` | สีเข้มหลัก |
| `--brand-bg: #FAF9F5` | พื้นหลังครีมนวล |
| `.hotel-title` | Cormorant Garamond, tracking กว้าง |
| `.serif-text` | Playfair Display สำหรับตัวเลขและรายละเอียด |
| `@media print` | ซ่อน `.no-print`, ขยาย canvas เต็ม A4 |
| `.sign-canvas` | ขนาด 480×720px ใน preview (A4 สัดส่วน) |

### 1.3 JavaScript Logic (State Machine)

```javascript
// State หลักของระบบ
state = {
  mode: 'single' | 'dual',
  orientation: 'portrait' | 'landscape',
  showBorder: boolean,
  showWatermark: boolean,
  canvasPadding: number,
  currentTheme: 'light' | 'dark' | 'sepia',
  event1: { title, titleSize, floor, room, date, time, bgStyle, layoutStyle, logos{}, customLogos[] },
  event2: { ...เหมือน event1... }
}
```

**ฟังก์ชันหลัก:**
- `renderPreview()` — วาด Event Card ทั้งหมดใน workspace ใหม่ทุกครั้งที่ state เปลี่ยน
- `createEventCardHTML(key, data, isSingle)` — สร้าง DOM element การ์ดงาน
- `getSponsorSVG(key)` — คืน SVG approximation ของโลโก้พาร์ทเนอร์
- `changeTheme(key)` — สลับโทนสีพื้นหลัง + ลายน้ำ
- `triggerPrint()` — เรียก `window.print()`

---

## ส่วนที่ 2 — วิเคราะห์ภาพอ้างอิง (Reference Image Analysis)

### 2.1 ภาพที่ 1 — ป้ายงานเดี่ยว (Single Event, 4 โลโก้)
**`WhatsApp Image 2026-05-26 at 04.17.57.jpeg`**

- พื้นหลังขาว, เส้นขอบบางสี่เหลี่ยม
- **Header:** ลายโลโก้ geometric ของโรงแรม + `SOUPHATTRA HOTEL` + `VIENTIANE` (tracking กว้าง)
- **โลโก้แถว:** EU (แนวนอน), Gov Lao (วงกลม), THE WORLD BANK (จริงๆ ไม่ใช่ SVG เลียน), Australian Aid (จริง)
- **ชื่องาน:** ตัวอักษรลาวขนาดใหญ่ ตัวหนา กึ่งกลาง ≈ 24-28px
- **ตัวแยก:** เส้นแนวนอนบาง
- **"2nd Floor":** Serif ใหญ่มาก ≈ 60-72px
- **"Souphattra Room 1&2":** Serif ≈ 28-32px
- **วันที่/เวลา:** Sans ≈ 16-18px
- **Footer:** เส้น + สัญลักษณ์ ❖ กึ่งกลาง
- **ลายน้ำ:** SVG geometric จาง ≈ 3% opacity

### 2.2 ภาพที่ 2 — ป้ายงานเดี่ยว (Single Event, 2 โลโก้)
**`WhatsApp Image 2026-05-26 at 06.16.07.jpeg`**

- เหมือนภาพที่ 1 แต่ใช้ Gov Lao + AFD logo (รูปจริง ไม่ใช่ SVG)
- ชื่องาน: ภาษาลาว + ภาษาอังกฤษ (bilingual, 2 ย่อหน้า)
- โครงสร้างเหมือนกันทุกอย่าง
- **สังเกต:** โลโก้ AFD อยู่ด้านขวา, Gov อยู่ซ้าย → เหมือน left/right alignment ไม่ใช่แค่ center row

### 2.3 ภาพที่ 3 — ป้าย 3 งาน (Triple Event)
**`фывыфв.jpeg`**

> **⚠️ สำคัญมาก:** ภาพนี้แสดงโหมด **3 งานในป้ายเดียว** ซึ่งปัจจุบัน **ไม่มีใน code!**

- **การ์ดที่ 1 (บน):** Banner ภาพกว้างเต็มความกว้าง (banner-style ไม่ใช่โลโก้เล็ก), ชื่อลาว, "2nd Floor / Room 1&2", วันที่/เวลา
- **การ์ดที่ 2 (กลาง):** 4 โลโก้แนวนอน (FAO, Korea, Gov, Gov Lao), ชื่อ English ล้วน, "2nd Floor / Room 3"
- **การ์ดที่ 3 (ล่าง):** Gov + World Bank โลโก้, ชื่อลาว, "5th Floor / Room 4&5"
- แต่ละการ์ดมี padding น้อยกว่า single mode มาก (compact)
- ใช้ white background ทุกการ์ด

### 2.4 ภาพที่ 4 — ป้าย 2 งาน (Dual Event)
**`ывыффы.jpeg`**

- **การ์ดที่ 1 (บน):** CRS logo (ซ้าย) + Gov Lao (ขวา) + EU logo (กึ่งกลาง เล็ก), ชื่อลาว+อังกฤษ bilingual, white bg
- **การ์ดที่ 2 (ล่าง):** 2 โลโก้ซ้าย + The Asia Foundation ขวา, ชื่อลาว+อังกฤษ, "5th Floor / Room 4&5"
- ขนาดตัวอักษรเล็กกว่า single mode ประมาณ 70-75%

---

## ส่วนที่ 3 — Gap Analysis: โค้ดปัจจุบัน vs ภาพอ้างอิง

| ประเด็น | โค้ดปัจจุบัน | ภาพอ้างอิง | ความสำคัญ |
|---|---|---|---|
| **โหมดป้าย** | Single / Dual เท่านั้น | มี Triple (3 งาน) ด้วย | 🔴 สูง |
| **โลโก้พาร์ทเนอร์** | SVG เลียนแบบ (approximation) | ภาพจริง / อัปโหลดได้ | 🔴 สูง |
| **การจัดโลโก้** | Center row เท่านั้น | Left/Right/Center ได้ | 🟡 กลาง |
| **ขนาดโลโก้** | ปรับไม่ได้ | ต้องปรับ resize ได้ | 🔴 สูง |
| **โลโก้ banner แบบกว้าง** | ไม่มี | การ์ดแรกภาพที่ 3 เป็น banner | 🟡 กลาง |
| **Bilingual title** | ใส่เองใน textarea | ต้องการ field แยก EN/LA | 🟡 กลาง |
| **Triple Event mode** | ไม่มี | จำเป็น | 🔴 สูง |
| **Artwork drag/reposition** | ไม่มี | ต้องการ | 🔴 สูง |
| **Export PNG** | Print PDF เท่านั้น | ต้องการ export PNG ด้วย | 🟡 กลาง |
| **Before/After preview** | ไม่มี | ต้องการ | 🟢 ต่ำ |
| **Snap-to-center guide** | ไม่มี | ต้องการ | 🟢 ต่ำ |
| **Canvas padding slider** | มีแล้ว ✓ | ดีอยู่แล้ว | ✅ |
| **Theme (light/dark/sepia)** | มีแล้ว ✓ | ดีอยู่แล้ว | ✅ |

---

## ส่วนที่ 4 — สถาปัตยกรรมใหม่ (New Architecture)

### 4.1 State Structure ใหม่

```javascript
const appState = {
  // ---- การตั้งค่า Sign หลัก ----
  mode: 'single' | 'dual' | 'triple',   // เพิ่ม triple
  orientation: 'portrait' | 'landscape',
  theme: 'light' | 'dark' | 'sepia',
  showBorder: true,
  showWatermark: true,
  canvasPadding: 24,

  // ---- Event Array (รองรับ 1-3 งาน) ----
  events: [
    {
      id: 'event-1',
      title: { lao: '', english: '' },    // แยก bilingual
      titleSize: 20,
      floor: '',
      floorSize: 48,
      room: '',
      roomSize: 20,
      date: '',
      time: '',
      bgStyle: 'white',        // 'white' | 'navy' | 'dark'
      layoutStyle: 'stacked',  // 'stacked' | 'columns'
      
      // Artwork Management (โลโก้ทั้งหมด)
      artworks: [
        {
          id: 'art-uuid',
          type: 'preset' | 'upload',
          presetKey: 'eu' | 'gov' | 'wb' | 'aus',  // สำหรับ preset
          src: 'base64...',                          // สำหรับ upload
          name: 'logo.png',
          
          // Artwork Adjustment System
          position: { x: 0, y: 0 },     // drag position (px offset)
          size: { w: 80, h: 40 },        // explicit size หลัง resize
          scale: 1.0,                    // zoom factor
          rotation: 0,                   // degrees
          cropRect: null,                // { x, y, w, h } หรือ null
          maintainAspectRatio: true,
          alignment: 'left' | 'center' | 'right',
          isBannerMode: false,           // ขยายเต็มความกว้าง
        }
      ],
    }
  ],

  // ---- Artwork Editor State ----
  artworkEditor: {
    isOpen: false,
    activeArtworkId: null,
    activeEventId: null,
    isDragging: false,
    dragStart: { x: 0, y: 0 },
    showCropTool: false,
    showGuides: true,           // snap-to-center guide
  }
};
```

### 4.2 Component Architecture

```
App
 ├── Sidebar (no-print)
 │    ├── BrandHeader
 │    ├── ModeSelector (Single / Dual / Triple) ← ใหม่
 │    ├── GlobalSettings
 │    │    ├── OrientationPicker
 │    │    ├── ThemePicker
 │    │    ├── BorderToggle
 │    │    └── PaddingSlider
 │    ├── EventTabBar (tabs for Event 1 / 2 / 3) ← ใหม่ pattern
 │    │    └── EventConfigPanel (active tab)
 │    │         ├── BgStylePicker
 │    │         ├── LayoutStylePicker
 │    │         ├── ArtworkManager ← NEW COMPONENT
 │    │         │    ├── PresetLogoGrid
 │    │         │    ├── UploadZone (drag & drop)
 │    │         │    └── ArtworkList (thumbnails + edit button)
 │    │         ├── TitleInput (lao + english fields)
 │    │         ├── FontSizeSlider
 │    │         └── EventDetails (floor / room / date / time)
 │    └── PrintExportBar (sticky bottom)
 │         ├── PrintButton
 │         └── ExportPNGButton ← ใหม่
 │
 ├── PreviewCanvas (print-area)
 │    └── SignFrame (A4)
 │         ├── WatermarkLayer
 │         ├── BorderFrame
 │         ├── HotelHeader
 │         ├── EventCards (1-3 cards)
 │         │    └── EventCard
 │         │         ├── ArtworkRow (draggable logos)
 │         │         │    └── ArtworkItem ← พร้อม drag handle
 │         │         ├── TitleArea
 │         │         └── DetailsSection (stacked / columns)
 │         └── SignFooter
 │
 └── ArtworkEditorModal ← NEW OVERLAY
      ├── ImagePreview (ดูภาพจริง)
      ├── ResizeControls (w/h sliders + lock aspect ratio)
      ├── PositionControls (x/y offset)
      ├── RotationControl (0-360 slider)
      ├── CropTool (กรอบครอป interactive)
      ├── AlignmentButtons (left / center / right)
      ├── BannerModeToggle
      ├── ResetButton
      └── ApplyButton
```

---

## ส่วนที่ 5 — แผนงานพัฒนา (Implementation Roadmap)

### Phase 1 — โครงสร้างพื้นฐานใหม่ (HTML Skeleton Rebuild)
**เป้าหมาย:** ล้างโค้ดเก่าที่ซ้ำซ้อน + วางโครงสร้าง component ใหม่

#### งานที่ต้องทำ:
- [ ] ลบ Event 2 sidebar section แยก → เปลี่ยนเป็น Tab system
- [ ] เพิ่ม Mode Selector ปุ่มที่ 3 (Triple)
- [ ] ย้าย Global Settings ขึ้นก่อน Event Tab
- [ ] สร้าง `<div id="artwork-editor-modal">` overlay structure
- [ ] ปรับ Preview canvas ให้ scroll ได้ถ้า landscape + triple
- [ ] เพิ่ม Export PNG button ข้าง Print button

#### ไฟล์ที่แก้:
- `souphattra_sign_generator.html` — โครงสร้าง HTML ทั้งหมด

---

### Phase 2 — Artwork Management System (ระบบจัดการ Artwork)
**เป้าหมาย:** ให้ผู้ใช้อัปโหลด, ปรับขนาด, จัดตำแหน่ง, crop โลโก้ได้

#### 2.1 Artwork Upload & Storage
```javascript
// แทนที่ customLogos[] array เดิม → เป็น artworks[] ที่มี metadata ครบ
function addArtwork(eventId, file) {
  // อ่านไฟล์ → Base64
  // สร้าง artwork object พร้อม default size จาก image natural dimensions
  // push เข้า state.events[id].artworks
  // renderPreview()
}
```

#### 2.2 Artwork Drag-and-Drop Reorder
```javascript
// ใน ArtworkRow → แต่ละ ArtworkItem มี drag handle
// ใช้ HTML5 Drag and Drop API หรือ pointer events
// drag-over reorder ตำแหน่งใน artworks[]
function handleArtworkDragReorder(eventId, fromIndex, toIndex) { ... }
```

#### 2.3 Artwork Resize (ใน Modal)
```javascript
// Resize Controls: width/height input + lock aspect ratio checkbox
// Range slider สำหรับ scale (0.5x - 3.0x)
function resizeArtwork(artworkId, newW, newH, maintainAspect) {
  if (maintainAspect) newH = newW * (original.h / original.w);
  updateArtworkProp(artworkId, 'size', { w: newW, h: newH });
}
```

#### 2.4 Artwork Drag-to-Position (ใน Preview)
```javascript
// ใน preview canvas: pointer events สำหรับ drag artwork
// เมื่อกด artwork item ใน preview → enter "edit mode"
// แสดง guides (เส้นกึ่งกลางแนวนอน + แนวตั้ง)
// snap เข้า center เมื่อใกล้กว่า 8px

function initArtworkDrag(artworkId, eventId) {
  const el = document.querySelector(`[data-artwork-id="${artworkId}"]`);
  el.addEventListener('pointerdown', startDrag);
  // ...
}
```

#### 2.5 Artwork Rotation
```javascript
// Range slider 0-360 degree ใน modal
// CSS transform: rotate(${deg}deg)
function rotateArtwork(artworkId, degrees) {
  updateArtworkProp(artworkId, 'rotation', degrees);
}
```

#### 2.6 Artwork Crop
```javascript
// Crop overlay สี่เหลี่ยมปรับขนาดได้ในขณะ preview
// ใช้ clip-path: inset() หรือ canvas 2D context
// บันทึก cropRect เป็น percentage ของ image
function applyCrop(artworkId, cropRect) {
  // cropRect = { x: 0.1, y: 0.05, w: 0.8, h: 0.9 } (normalized 0-1)
  updateArtworkProp(artworkId, 'cropRect', cropRect);
}
```

#### 2.7 Snap-to-Center Guide
```javascript
// เมื่อ drag artwork ใน preview
// ตรวจสอบ center ของ artwork vs center ของ container
// ถ้าใกล้กว่า threshold → snap + แสดงเส้น guide สีฟ้า
const SNAP_THRESHOLD = 8; // px
function checkSnap(artworkCenter, containerCenter) {
  if (Math.abs(artworkCenter.x - containerCenter.x) < SNAP_THRESHOLD) {
    return { snapX: containerCenter.x, showGuideX: true };
  }
}
```

---

### Phase 3 — Triple Event Mode

#### 3.1 Mode Selector Update
```javascript
// เพิ่ม btn-mode-triple
// state.mode = 'triple'
// render 3 Event Cards แทน 2
// ปรับ font size multiplier สำหรับ triple (เล็กลงอีก 85%)

const SIZE_MULTIPLIERS = {
  single: 1.0,
  dual:   0.80,
  triple: 0.65,
};
```

#### 3.2 Event Tab Navigation
```javascript
// แทนที่ sidebar sections แยก → Tab bar มี 1-3 tabs
// Tab ที่ active แสดง config panel ของ event นั้น
// Tab สีเปลี่ยนตามธีม event (amber/purple/blue)

function setActiveEventTab(index) {
  state.ui.activeEventTab = index;
  renderSidebarPanel();
}
```

---

### Phase 4 — Bilingual Title Support

#### 4.1 Title Fields
```html
<!-- แทนที่ textarea เดียว → 2 fields -->
<div class="space-y-2">
  <div>
    <label>ชื่องาน (ลาว/ไทย)</label>
    <textarea id="inp-e1-title-lao" rows="2" 
      placeholder="ຊື່ກອງປະຊຸມ ພາສາລາວ"></textarea>
    <input type="range" id="size-e1-title-lao" ...>
  </div>
  <div>
    <label>Event Name (English)</label>
    <textarea id="inp-e1-title-en" rows="2"
      placeholder="English event name"></textarea>
    <input type="range" id="size-e1-title-en" ...>
  </div>
</div>
```

#### 4.2 Render Bilingual
```javascript
// ถ้า title.english ไม่ว่าง → แสดง 2 บรรทัด (lao + smaller english)
const titleHTML = `
  <h3 style="font-size:${titleSize}px">${data.title.lao}</h3>
  ${data.title.english 
    ? `<p class="serif-text" style="font-size:${titleSize * 0.7}px">${data.title.english}</p>` 
    : ''}
`;
```

---

### Phase 5 — Export PNG Feature

#### 5.1 ใช้ html2canvas library
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
```

```javascript
async function exportAsPNG() {
  const canvas = document.getElementById('sign-sign-frame');
  
  // ซ่อน UI controls ชั่วคราว
  document.querySelectorAll('.no-print').forEach(el => el.style.display = 'none');
  
  const snapshot = await html2canvas(canvas, {
    scale: 2,              // 2x resolution สำหรับคุณภาพสูง
    useCORS: true,
    backgroundColor: null,
    logging: false,
  });
  
  // คืนค่า UI controls
  document.querySelectorAll('.no-print').forEach(el => el.style.display = '');
  
  // Download PNG
  const link = document.createElement('a');
  link.download = `Souphattra_Sign_${Date.now()}.png`;
  link.href = snapshot.toDataURL('image/png');
  link.click();
  
  showToast('บันทึกเป็น PNG สำเร็จ!');
}
```

---

### Phase 6 — Modern UI/UX Redesign

#### 6.1 Layout Changes
```
เดิม: aside (460px fixed) + main (flex-1)
ใหม่: 
  - Desktop: aside (380px) + main (flex-1 with centered preview)
  - Tablet:  Sidebar collapse → drawer (hamburger trigger)
  - Mobile:  Full-width panels + bottom preview toggle
```

#### 6.2 Design System (CSS Variables)
```css
:root {
  /* Brand */
  --gold: #c5a880;
  --gold-light: #dcc8a0;
  --gold-dark: #a8895e;
  
  /* Surface (Dark UI) */
  --surface-0: #0a0a0a;   /* darkest bg */
  --surface-1: #111111;   /* sidebar bg */
  --surface-2: #1a1a1a;   /* card bg */
  --surface-3: #232323;   /* input bg */
  --surface-4: #2d2d2d;   /* hover state */
  
  /* Text */
  --text-primary: #f5f5f5;
  --text-secondary: #a0a0a0;
  --text-muted: #666;
  
  /* Accent */
  --accent-amber: #f59e0b;
  --accent-purple: #a855f7;
  --accent-blue: #3b82f6;
  
  /* Spacing */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
  
  /* Radius */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 16px;
  
  /* Shadow */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.4);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.5);
  --shadow-lg: 0 8px 32px rgba(0,0,0,0.6);
}
```

#### 6.3 Sidebar Improvements
- ใช้ **Tab system** แทน Accordion (Event 1 / 2 / 3 เป็น tabs)
- ทุก control group มี **section header** ชัดเจน
- Input fields มี **focus ring** สีทอง
- Sliders ทั้งหมดแสดง **live value** ที่อ่านง่าย
- ปุ่ม Mode มี **icon SVG** ชัดเจน (ไม่ใช่ emoji)
- **Upload zone** รองรับ drag & drop พร้อม hover state

#### 6.4 Preview Area Improvements
- Preview canvas อยู่ใน **scrollable container** พร้อม zoom control
- แสดง **canvas size info** (e.g., "A4 Portrait · 210×297mm")
- เส้น **guide อัตโนมัติ** เมื่อ drag artwork
- Artwork ที่เลือกมี **selection ring** + resize handles
- Animation `transition: all 0.2s ease` สำหรับทุก state change

#### 6.5 Artwork Editor Modal Design
```
┌─────────────────────────────────────────────────┐
│ แก้ไข Artwork                              [✕]   │
├─────────────────┬───────────────────────────────┤
│                 │  ● ขนาด (Size)                 │
│  [IMAGE        ]│    กว้าง: [80]px  สูง: [40]px  │
│  [PREVIEW      ]│    🔒 รักษาสัดส่วน              │
│  [AREA        ]│                                 │
│                 │  ● ขยาย (Scale)                │
│                 │    [━━━●━━━━━━━] 1.2×           │
│                 │                                 │
│                 │  ● หมุน (Rotation)              │
│                 │    [━━━━━━●━━━] 0°             │
│                 │                                 │
│                 │  ● ตำแหน่ง (Alignment)          │
│                 │    [←] [↔] [→]                 │
│                 │                                 │
│                 │  ● โหมด Banner                  │
│                 │    [ ] ขยายเต็มความกว้าง        │
├─────────────────┴───────────────────────────────┤
│  [รีเซ็ต]                  [ยกเลิก]  [ใช้งาน]    │
└─────────────────────────────────────────────────┘
```

---

## ส่วนที่ 6 — รายละเอียดโค้ดที่ต้องแก้/เพิ่ม

### 6.1 ฟังก์ชันใหม่ทั้งหมด

```javascript
// Artwork Management
addArtwork(eventId, source, type)
removeArtwork(eventId, artworkId)
openArtworkEditor(eventId, artworkId)
closeArtworkEditor()
updateArtworkProp(artworkId, prop, value)
resetArtwork(artworkId)
reorderArtworks(eventId, fromIdx, toIdx)

// Artwork Adjustment
resizeArtwork(artworkId, w, h)
scaleArtwork(artworkId, factor)
rotateArtwork(artworkId, degrees)
setCropRect(artworkId, rect)
setArtworkAlignment(artworkId, alignment)
setArtworkPosition(artworkId, x, y)
toggleBannerMode(artworkId, isBanner)
toggleAspectLock(artworkId, locked)

// Snap Guide
checkSnapGuides(artworkEl, containerEl)
showGuideLines(axis, position)
hideGuideLines()

// Event Management  
setSignMode(mode)  // ปรับใหม่ รองรับ 'triple'
setActiveEventTab(index)
addEvent()  // เพิ่ม event ใหม่
removeEvent(index)

// Export
exportAsPNG()
exportAsPDF()  // wrapper รอบ window.print()

// Bilingual
syncEventTitle(eventId, lang, value)  // 'lao' หรือ 'en'

// UI
renderPreview()  // ปรับใหม่ทั้งหมด
renderSidebarPanel()
renderEventCard(eventData, mode)
renderArtworkRow(artworks)
renderArtworkItem(artwork)
showToast(message, type)  // เพิ่ม type: 'success' | 'error' | 'info'
```

### 6.2 เหตุการณ์ Event Listeners ใหม่

```javascript
// Drag-to-reorder artworks
artworkRow.addEventListener('dragstart', ...)
artworkRow.addEventListener('dragover', ...)
artworkRow.addEventListener('drop', ...)

// Drag artwork in preview
artworkEl.addEventListener('pointerdown', startArtworkDrag)
document.addEventListener('pointermove', onArtworkDrag)
document.addEventListener('pointerup', endArtworkDrag)

// Artwork upload drop zone
uploadZone.addEventListener('dragover', highlightZone)
uploadZone.addEventListener('drop', handleFileDrop)

// Keyboard shortcuts
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') closeArtworkEditor();
  if (e.key === 'r' && e.ctrlKey) resetActiveArtwork();
})
```

---

## ส่วนที่ 7 — เก็บรักษา Functionality เดิม (Preserve Existing)

ฟีเจอร์เหล่านี้ทำงานได้ดีแล้ว → **อย่าแตะต้อง logic core**:

| ฟีเจอร์ | สถานะ | action |
|---|---|---|
| `changeTheme()` (light/dark/sepia) | ✅ ดี | คงไว้, อัปเดต CSS vars เท่านั้น |
| `toggleBorder()` | ✅ ดี | คงไว้ |
| `toggleWatermark()` | ✅ ดี | คงไว้ |
| `adjustCanvasPadding()` | ✅ ดี | คงไว้ |
| `setOrientation()` | ✅ ดี | คงไว้ |
| `triggerPrint()` / `@media print` | ✅ ดี | คงไว้ + เพิ่ม PNG export |
| `getSponsorSVG()` | ⚠️ เปลี่ยน | เปลี่ยน: Preset logos เป็น real image URLs หรือ inline base64 สวยขึ้น |
| `createEventCardHTML()` | 🔄 Refactor | แยกเป็น sub-functions ย่อย |
| `showToast()` | ✅ ดี | เพิ่ม type/icon support |

---

## ส่วนที่ 8 — ลำดับความสำคัญในการพัฒนา (Priority Order)

### 🔴 ต้องทำก่อน (Critical)
1. **Triple Event Mode** — ภาพอ้างอิงแสดงชัดว่าจำเป็น
2. **Artwork Upload แบบ Drag & Drop** — UX สำคัญมาก
3. **Artwork Resize ใน Modal** — แก้ปัญหาโลโก้ขนาดไม่พอดี
4. **Event Tab System** — แทนที่ sidebar ซ้ำซ้อน
5. **Bilingual Title Fields** — ภาพอ้างอิงทุกใบมีสองภาษา

### 🟡 ทำในรอบที่ 2 (Important)
6. **Artwork Drag-to-Reorder** — ผู้ใช้ต้องการเรียงโลโก้เอง
7. **Export PNG** — พิมพ์ PDF ไม่พอ, ต้องการ PNG ด้วย
8. **Artwork Drag-in-Preview** — ปรับตำแหน่งโดยตรงบน preview
9. **Snap-to-Center Guide** — UX ที่ดี
10. **Artwork Alignment Buttons** — Left/Center/Right

### 🟢 ทำเพิ่มเติม (Enhancement)
11. **Rotation Control** — ไม่ค่อยได้ใช้แต่ต้องมี
12. **Crop Tool** — ซับซ้อน implement แต่มีค่า
13. **Before/After Comparison** — nice-to-have
14. **Keyboard Shortcuts** — ช่วย power users

---

## ส่วนที่ 9 — UI ใหม่ Wireframe (โครงร่าง)

### Desktop Layout (≥1024px)
```
┌────────────────────────────────────────────────────────────┐
│ [S] Souphattra Signage Studio                    PRO v3.0  │ ← header
├──────────────────────┬─────────────────────────────────────┤
│ CONTROLS (380px)     │  PREVIEW AREA                       │
│                      │                                     │
│ ┌──────────────────┐ │    ┌───────────────────────────┐    │
│ │ Mode: 1 · 2 · 3  │ │    │  ╔═════════════════════╗  │    │
│ └──────────────────┘ │    │  ║  SOUPHATTRA HOTEL   ║  │    │
│                      │    │  ║       VIENTIANE     ║  │    │
│ [Event 1][Event 2]   │    │  ╠═════════════════════╣  │    │
│                      │    │  ║ [EU] [GOV] [WB] [AU]║  │    │
│ ─── Artwork ─────   │    │  ║                     ║  │    │
│ [+ Add Logo]         │    │  ║   ชื่องานประชุม...  ║  │    │
│ [eu.svg][gov.svg]   │    │  ║                     ║  │    │
│                      │    │  ║    2nd Floor        ║  │    │
│ ─── Title ────────  │    │  ║  Souphattra Room 1  ║  │    │
│ [textarea lao]       │    │  ║  26 May 2026        ║  │    │
│ [textarea english]   │    │  ╚═════════════════════╝  │    │
│                      │    │                           │    │
│ ─── Details ──────  │    │  A4 Portrait · 210×297mm  │    │
│ Floor / Room         │    │  [🔍-] [50%] [🔍+]       │    │
│ Date / Time          │    └───────────────────────────┘    │
│                      │                                     │
├──────────────────────┤                                     │
│ [🖨 Print PDF]       │                                     │
│ [📸 Export PNG]      │                                     │
└──────────────────────┴─────────────────────────────────────┘
```

### Mobile Layout (≤768px)
```
┌─────────────────────┐
│ [☰] Souphattra  [⚙] │ ← mobile header
├─────────────────────┤
│                     │
│   [SIGN PREVIEW]    │ ← ย่อลงมา 60%
│                     │
├─────────────────────┤
│ [Controls ▲]        │ ← bottom drawer
│ ─────────────────── │
│ Mode: [1] [2] [3]   │
│ Event: [1▼]         │
│ [+ Add Logo]        │
│ Title: [          ] │
│ Floor: [    ]       │
│ ...                 │
└─────────────────────┘
```

---

## ส่วนที่ 10 — ข้อสังเกตพิเศษจากภาพอ้างอิง

1. **โลโก้จริง vs SVG:** ภาพอ้างอิงทุกใบใช้โลโก้จริง (ภาพ PNG/JPG จริง) ไม่ใช่ SVG เลียนแบบ → ต้องให้ผู้ใช้อัปโหลดโลโก้จริงเสมอ ไม่ควรพึ่งพา preset SVG อีกต่อไป หรือถ้าจะ preset ก็ควรเป็น PNG จริง

2. **"Banner" Logo Mode:** การ์ดแรกในภาพ `фывыфв.jpeg` มีภาพ banner เต็มความกว้างแทนโลโก้เล็กๆ → ต้องเพิ่ม toggle "Banner Mode" ให้โลโก้ขยายเต็มความกว้างของการ์ด

3. **การ์ดสีขาวล้วน:** ในภาพ triple event ทุกการ์ดเป็นสีขาว — ปัจจุบัน navy/dark option ควรยังคงอยู่แต่ default ควรเป็น white

4. **Font Weight ใน Triple Mode:** ชื่องานในป้าย 3 งานตัวเล็กมาก (≈13-14px) แต่ยังอ่านได้ → ระบบต้องปรับ font size อัตโนมัติตาม mode

5. **เส้นคั่นหน้าต่างการ์ด:** ระหว่างการ์ดในโหมด dual/triple ไม่มี margin มาก — เส้นขอบของการ์ดติดกันแทบพอดี

6. **Logo Row อาจมีทั้ง ซ้าย-ขวา pattern:** ภาพที่ 4 การ์ด 1 (CRS ซ้าย + Gov ขวา) → ต้องรองรับ "flank" alignment ไม่ใช่แค่ center row

---

## ส่วนที่ 11 — สรุปการเปลี่ยนแปลงรวม

| หมวด | รายการ | จำนวน |
|---|---|---|
| ฟีเจอร์ใหม่ | Triple mode, Artwork editor, Bilingual, PNG export, Drag&Drop | 5 หลัก |
| Refactor | EventCard factory, Sidebar → Tabs, State structure | 3 หลัก |
| คงเดิม | Theme, Border, Watermark, Padding, Print | 5 รายการ |
| Design | CSS Variables, Responsive mobile layout | 2 หลัก |
| Library เพิ่ม | html2canvas (PNG export) | 1 |

**เวลาประมาณการ (ถ้าทำทีละ Phase):**
- Phase 1 (Skeleton): ~2-3 ชั่วโมง
- Phase 2 (Artwork System): ~5-8 ชั่วโมง ← ส่วนที่ใช้เวลามากสุด
- Phase 3 (Triple Mode): ~1-2 ชั่วโมง
- Phase 4 (Bilingual): ~1 ชั่วโมง
- Phase 5 (Export PNG): ~1-2 ชั่วโมง
- Phase 6 (UI Redesign): ~3-5 ชั่วโมง

**รวมประมาณ: 13-21 ชั่วโมง**

---

*จัดทำโดย Claude Code — Souphattra Hotel Signage Studio Upgrade Plan v1.0*
