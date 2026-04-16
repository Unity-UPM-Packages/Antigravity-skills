# 🤖 Figma → Unity Automation Pipeline — Technical Specification v3.0

> **Đối tượng:** AI Agent / Developer xây dựng hệ thống automation.  
> Tài liệu này định nghĩa **toàn bộ quy tắc mapping** từ dữ liệu Figma sang Unity uGUI.  
> Tuân thủ đúng spec này = import 1:1 không cần chỉnh tay.

---

## 1. Naming Conventions

### 1.1 Prefix Table

Hệ thống đọc **prefix** đầu tên layer để quyết định Unity Component sinh ra.

| Prefix | Unity Component | Ghi chú |
|---|---|---|
| `Btn_` | `Button` + `Image` | Xem Section 5 — Variants/ColorBlock |
| `Img_` | `Image` | Hình ảnh tổng quát |
| `Bg_` | `Image` (non-interactive) | Không add `Button` component |
| `Txt_` | `TextMeshProUGUI` | Không export ảnh. Đọc font data từ Figma |
| `Scroll_V_` | `ScrollRect` (vertical=true, horizontal=false) | Bắt buộc có `RectMask2D` |
| `Scroll_H_` | `ScrollRect` (horizontal=true, vertical=false) | Bắt buộc có `RectMask2D` |
| `Scroll_VH_` | `ScrollRect` (vertical=true, horizontal=true) | Bắt buộc có `RectMask2D` |
| `Content_` | Child gán vào `scrollRect.content` | Xem Section 8 |
| `Scrollbar_` | `Scrollbar` | Xem Section 9 |
| `Slider_` | `Slider` | Xem Section 10 |
| `Toggle_` | `Toggle` | Xem Section 11 |
| `Mask_` | `Image` + `Mask` | Clip content, không scroll |
| `Safe_` | `RectTransform` only | Không sinh `Image`. Dùng để offset layout theo Safe Area |

> Layer **không có prefix hợp lệ** → sinh `RectTransform` thuần (container), không sinh Component.

### 1.2 Quy tắc đặt tên

- PascalCase cho phần mô tả: `Btn_PlayNow`, `Txt_PlayerName`
- Không dùng dấu cách — dùng `_` ngăn cách
- **Tên layer phải unique trong phạm vi 1 Frame** (màn hình). Trùng tên = conflict khi sinh prefab
- Layer trùng tên ở Frame khác nhau = hợp lệ (khác namespace)

---

## 2. Resolution & Canvas

### 2.1 Figma Frame

- Tên Frame = tên màn hình: `Screen_Home`, `Screen_Shop`...
- Kích thước Frame phải khớp với `Reference Resolution` đã thống nhất với project

| Orientation | Reference Resolution |
|---|---|
| Portrait | 1080 × 1920 |
| Landscape | 1920 × 1080 |

> Kích thước là per-project, không cứng. Đọc từ project config trước khi import.

### 2.2 Unity Canvas Settings

| Thiết lập | Giá trị |
|---|---|
| Render Mode | `Screen Space — Overlay` |
| Canvas Scaler Mode | `Scale With Screen Size` |
| Reference Resolution | Khớp với Figma Frame của project |
| Screen Match Mode | `Match Width Or Height` |
| Match (Portrait) | `0.0` — ưu tiên Width |
| Match (Landscape) | `1.0` — ưu tiên Height |

---

## 3. Component System — Prefab Architecture

> Đây là phần quan trọng nhất của pipeline. Figma Component = Unity Prefab.

### 3.1 Master Component → Prefab

Khi automation gặp 1 **Master Component** trong Figma:
1. Parse toàn bộ cấu trúc layer của Master Component
2. Tạo **1 Prefab** tương ứng trong `Assets/UI/Prefabs/[ComponentName].prefab`
3. Prefab này là bản gốc — không chứa dữ liệu override

```
Master Component "Header"  →  Assets/UI/Prefabs/Header.prefab
Master Component "Card_Item"  →  Assets/UI/Prefabs/Card_Item.prefab
```

### 3.2 Component Instance → Prefab Instance + Overrides

Khi automation gặp 1 **Instance** của component trên 1 Frame:
1. Xác định Master Component tương ứng
2. Instantiate Prefab tương ứng trong hierarchy của màn hình đó
3. **Đọc tất cả Instance Overrides** từ Figma
4. Apply overrides lên instance (không sửa Prefab gốc):

| Override type | Apply như thế nào |
|---|---|
| Text content | `TextMeshProUGUI.text = overrideValue` |
| Image fill | Download ảnh override, gán vào `Image.sprite` |
| Layer visibility | `GameObject.SetActive(isVisible)` |
| Color | `Image.color = overrideColor` hoặc `TextMeshProUGUI.color` |
| Position/Size | Cập nhật `RectTransform` của instance |

### 3.3 Xử lý Nested Component

Khi 1 Component chứa Component khác bên trong:
- Component con cũng được tạo Prefab riêng
- Prefab cha tham chiếu Prefab con (Nested Prefab)
- Override trên instance của component con được apply đúng cấp

### 3.4 Thư mục Prefab

```
Assets/
└── UI/
    ├── Prefabs/
    │   ├── Common/        ← Prefab dùng chung (Header, Footer, Card_Item...)
    │   └── [ScreenName]/  ← Prefab riêng của màn hình đó (nếu không phải component)
    ├── Sprites/
    │   ├── Common/
    │   ├── [ScreenName]/
    │   └── 9Slice/
    └── Fonts/
```

---

## 4. Smart Merge — Chiến lược Update

Khi automation chạy lại trên màn hình đã được import:

### 4.1 Những gì được UPDATE

- `RectTransform`: position, size, anchors, pivot
- Visual: `Image.sprite`, `Image.color`, `TextMeshProUGUI.text`, font properties
- `GameObject.SetActive` (visibility)
- Layout Group: padding, spacing, alignment
- `Content Size Fitter` settings
- Prefab Overrides (text, ảnh, visibility trên instance)

### 4.2 Những gì được GIỮ NGUYÊN

- Các **script component** do developer thêm vào (không import từ Figma)
- **SerializedField references** (binding giữa script và GameObject)
- **Component tùy chỉnh** không thuộc scope automation
- Bất kỳ `MonoBehaviour` nào không có trong danh sách component automation tạo ra

### 4.3 Quy tắc Match (nhận diện object để merge)

Hệ thống match GameObject với Figma layer bằng:
1. **Tên GameObject = tên layer Figma** (priority 1)
2. **Figma Node ID** lưu trong metadata component (priority 2, nếu có)

> Nếu không match được → tạo mới. Không bao giờ tự xóa GameObject có script đang binding.

### 4.4 Conflict log

Mọi thay đổi trong quá trình merge phải được log ra:
```
[MERGE] Screen_Home > Header > Txt_Title: "HOME" → "Home Screen"
[MERGE] Screen_Home > Btn_Play: position (0, -200) → (0, -180)
[SKIP]  Screen_Home > PlayerController: custom script — preserved
[WARN]  Font "CustomBrand" not found on Google Fonts → fallback to default
```

---

## 5. Button States (Variants → SpriteSwap)

### 5.1 Detection Logic

```
Figma layer có prefix Btn_
        ↓
Có Figma Component Variants với property "State"?
        ├── YES → SpriteSwap
        └── NO  → ColorBlock (default)
```

### 5.2 SpriteSwap — Variant Mapping

Automation đọc từng variant và export sprite riêng:

| Variant `State=` | Unity `SpriteState` field | Tên sprite export |
|---|---|---|
| `Normal` | `Button.image.sprite` (sprite chính) | `btn_[name]_normal.png` |
| `Pressed` | `spriteState.pressedSprite` | `btn_[name]_pressed.png` |
| `Disabled` | `spriteState.disabledSprite` | `btn_[name]_disabled.png` |
| `Highlighted` | `spriteState.highlightedSprite` | `btn_[name]_highlighted.png` |

Sau đó set `button.transition = Selectable.Transition.SpriteSwap`.

### 5.3 ColorBlock — Fallback

Khi không có Variants:
- `button.transition = Selectable.Transition.ColorTint`
- ColorBlock dùng giá trị mặc định Unity (có thể override qua project config)

---

## 6. Export Sprites

### 6.1 Format & Scale

| Loại | Format | Scale | Ghi chú |
|---|---|---|---|
| Icon, UI element | PNG | @1× | Đã ở 1080p |
| Chi tiết mảnh (line < 2px) | PNG | @2× | Thêm `_x2` vào tên layer |
| Background toàn màn hình | JPG (quality 90) | @1× | Giảm VRAM |
| 9-Slice sprite | PNG | @1× | Kích thước nhỏ nhất |

### 6.2 Đặt tên file export

Tên file = tên layer **bỏ prefix**, lowercase, underscore, bỏ suffix `_9s`:

```
Layer name         →  File export           →  Thư mục
Btn_Play_9s_L12_R12_T8_B8  →  btn_play.png  →  Assets/UI/Sprites/9Slice/
Bg_HomeMain        →  bg_home_main.png       →  Assets/UI/Sprites/Screen_Home/
Img_Avatar         →  img_avatar.png         →  Assets/UI/Sprites/Screen_Home/
```

> Quy tắc: bỏ prefix (`Btn_`, `Img_`...) và bỏ toàn bộ suffix automation (`_9s`, `_L12`...). Chỉ giữ phần mô tả.

---

## 7. 9-Slice

### 7.1 Detection

Layer có hậu tố `_9s` → xử lý 9-Slice.

### 7.2 Parse Border Values

Đọc annotation từ tên layer: `_9s_L[n]_R[n]_T[n]_B[n]`

```
Btn_Play_9s_L12_R12_T8_B8
→ Left=12, Right=12, Top=8, Bottom=8
```

Fallback nếu không có annotation: `25%` mỗi cạnh (dựa trên kích thước sprite).

### 7.3 Apply vào Unity

```csharp
// Sprite.border nhận Vector4 theo thứ tự: Left, Bottom, Right, Top
// Annotation _L_R_T_B → cần re-map:
sprite.border = new Vector4(left, bottom, right, top);
//                          ↑     ↑       ↑     ↑
//                          L     B       R     T   (từ annotation)
```

> ⚠️ **Thứ tự Vector4 KHÁC thứ tự annotation.** Annotation là L→R→T→B, nhưng Unity nhận L→**B**→R→**T**.

### 7.4 Sprite Import Settings

```
TextureType = Sprite
SpriteMeshType = FullRect
FilterMode = Bilinear
Compression = Lossless (PNG) / Normal (JPG)
border = Vector4(L, B, R, T)
```

---

## 8. Auto Layout → Layout Group

### 8.1 Mapping

| Figma | Unity Component |
|---|---|
| Auto Layout Horizontal (→) | `Horizontal Layout Group` |
| Auto Layout Vertical (↓) | `Vertical Layout Group` |
| Auto Layout Wrap (Grid) | `Grid Layout Group` |

### 8.2 Properties Copy 1:1

| Figma Property | Unity Property |
|---|---|
| Padding Top/Bottom/Left/Right | `padding.top/bottom/left/right` |
| Item Spacing | `spacing` |
| Alignment | `childAlignment` |
| Reverse Order | `reverseArrangement` |

### 8.3 Resizing → Content Size Fitter

| Figma Resizing | Unity |
|---|---|
| Hug contents (Width) | `ContentSizeFitter.horizontalFit = PreferredSize` |
| Hug contents (Height) | `ContentSizeFitter.verticalFit = PreferredSize` |
| Fixed | Không tạo `ContentSizeFitter` |
| Fill container | Stretch anchor (xem Section 12) |

> ⚠️ **Không** set cả Width lẫn Height đều `Hug contents` → gây layout loop trong Unity.

### 8.4 Grid Layout Group

Khi Figma dùng Auto Layout Wrap:
- `CellSize` = kích thước item con đầu tiên
- `Spacing` = item spacing của Auto Layout
- `Constraint` = `FixedColumnCount` hoặc `FixedRowCount` dựa trên hướng wrap

---

## 9. Scroll View

### 9.1 Cấu trúc bắt buộc

```
Scroll_V_Inventory          → ScrollRect (vertical), RectMask2D
└── Content_Inventory        → RectTransform, Vertical Layout Group, ContentSizeFitter(Height=Preferred)
    ├── Item_1
    ├── Item_2
    └── Item_3
```

### 9.2 Quy tắc mapping

- Frame `Scroll_*` có `Clip Content = true` → sinh `RectMask2D`
- `Content_` là **con trực tiếp** của `Scroll_*` → gán vào `scrollRect.content`
- Kích thước của `Scroll_*` frame = kích thước viewport (vùng nhìn thấy)
- `Content_` tự mở rộng qua `ContentSizeFitter` (Height = Preferred cho dọc, Width = Preferred cho ngang)

### 9.3 Scroll 2 chiều

`Scroll_VH_` → `ScrollRect` với `horizontal = true` và `vertical = true`

---

## 10. Scrollbar

### 10.1 Cấu trúc

```
Scrollbar_[Tên]
├── Img_Track      → Image (background rãnh)
└── Img_Handle     → Image (con trượt) → gán vào scrollbar.handleRect
```

### 10.2 Direction detection

| Điều kiện | Kết quả |
|---|---|
| `height > width` | `Scrollbar.direction = BottomToTop` |
| `width > height` | `Scrollbar.direction = LeftToRight` |
| `width == height` | Fallback: `BottomToTop`. Log warning |

---

## 11. Slider

### 11.1 Cấu trúc

```
Slider_[Tên]
├── Img_Background  → Image → không gán
├── Img_Fill        → Image → gán vào slider.fillRect
└── Img_Handle      → Image → gán vào slider.handleRect
```

### 11.2 Direction detection

Giống Scrollbar — so sánh Width vs Height của Slider container.

---

## 12. Toggle

### 12.1 Cấu trúc

```
Toggle_[Tên]
├── Img_Background  → Image (background của toggle)
└── Img_Checkmark   → Image → gán vào toggle.graphic
```

### 12.2 Default state

- `Img_Checkmark` visible trong Figma → `toggle.isOn = true`
- `Img_Checkmark` hidden trong Figma → `toggle.isOn = false`

### 12.3 Variants cho Toggle

| Variant Property | Giá trị | Unity |
|---|---|---|
| `Checked` | `True` | `toggle.isOn = true` |
| `Checked` | `False` | `toggle.isOn = false` |

---

## 13. Font Pipeline (Figma → TMP FontAsset)

### 13.1 Đọc font data từ Figma

Với mỗi `Txt_` layer, automation đọc:

| Figma Property | Unity / TMP Property |
|---|---|
| `fontFamily` | Tên font để download |
| `fontWeight` (400, 700...) | Font style (Regular, Bold...) |
| `fontSize` | `TextMeshProUGUI.fontSize` |
| `fills[0].color` | `TextMeshProUGUI.color` |
| `letterSpacing` | `TextMeshProUGUI.characterSpacing` |
| `lineHeight` | `TextMeshProUGUI.lineSpacing` |
| `textAlignHorizontal` | `TextMeshProUGUI.alignment` (horizontal) |
| `textAlignVertical` | `TextMeshProUGUI.alignment` (vertical) |
| `textDecoration` | `fontStyle` flags (underline, strikethrough) |
| `textCase` | `TextMeshProUGUI.fontStyle` (uppercase, lowercase) |

### 13.2 Font download pipeline

```
1. Đọc fontFamily từ Figma API
2. Query Google Fonts API: GET https://fonts.googleapis.com/css2?family={fontFamily}
3. Parse CSS response → lấy URL file font (.ttf/.woff2)
4. Download font file → Assets/UI/Fonts/{FontFamily}-{Weight}.ttf
5. Trigger Unity TMP Font Asset Creator (EditorScript) → tạo {FontFamily}-{Weight} SDF.asset
6. Cache mapping: fontFamily+weight → FontAsset path
```

### 13.3 Fallback

Nếu font **không tìm thấy** trên Google Fonts:
1. Dùng `defaultFontAsset` được config trong project settings
2. Log warning: `[WARN] Font "{name}" not found on Google Fonts → fallback to default`
3. Tiếp tục apply các thuộc tính khác (size, color, spacing) bình thường

---

## 14. Constraints → Anchors

### 14.1 Mapping bảng đầy đủ

| Figma Constraints | anchorMin | anchorMax | Ghi chú |
|---|---|---|---|
| Left + Top | (0, 1) | (0, 1) | Góc trên-trái |
| Right + Top | (1, 1) | (1, 1) | Góc trên-phải |
| Left + Bottom | (0, 0) | (0, 0) | Góc dưới-trái |
| Right + Bottom | (1, 0) | (1, 0) | Góc dưới-phải |
| Center + Top | (0.5, 1) | (0.5, 1) | Căn giữa ngang, ghim trên |
| Center + Bottom | (0.5, 0) | (0.5, 0) | Căn giữa ngang, ghim dưới |
| Center + Center | (0.5, 0.5) | (0.5, 0.5) | Căn giữa màn hình |
| Left + Right (Stretch H) | (0, Y) | (1, Y) | Y = tọa độ vertical anchor |
| Top + Bottom (Stretch V) | (X, 0) | (X, 1) | X = tọa độ horizontal anchor |
| Left + Right + Top + Bottom | (0, 0) | (1, 1) | Full stretch |
| Scale | pivot | pivot | anchorMin = anchorMax = pivot point |

### 14.2 Tính toán position sau khi set Anchor

- Với **Point anchor** (anchorMin == anchorMax): dùng `anchoredPosition` và `sizeDelta`
- Với **Stretch anchor**: dùng `offsetMin` và `offsetMax`
  ```
  offsetMin.x = left margin từ anchor
  offsetMax.x = -(right margin từ anchor)
  offsetMin.y = bottom margin từ anchor  
  offsetMax.y = -(top margin từ anchor)
  ```

---

## 15. Layer Visibility

| Figma | Unity |
|---|---|
| Layer visible | `GameObject.SetActive(true)` |
| Layer hidden | `GameObject.SetActive(false)` |
| Opacity < 100% | Thêm `CanvasGroup`, set `alpha = opacity/100` |

> Override visibility trên Instance được apply lên instance trong Scene, không ảnh hưởng Prefab gốc.

---

## 16. Hierarchy Structure — Cấu trúc Cha Con

### 16.1 Nguyên tắc 1:1

Cấu trúc cha-con và tên trong Figma được giữ nguyên 1:1 sang Unity:

```
Figma                           Unity Hierarchy
─────────────────────           ─────────────────────
Frame: Screen_Home         →    GameObject: Screen_Home
├── Bg_Main                →    ├── Bg_Main
├── Header (Component)     →    ├── Header (Prefab Instance)
│   ├── Bg_Header          →    │   ├── Bg_Header
│   └── Txt_Title          →    │   └── Txt_Title
└── Btn_Play               →    └── Btn_Play
```

- **Tên GameObject = tên layer Figma** (giữ nguyên, bao gồm cả prefix)
- **Cấp lồng nhau giữ nguyên** — con trong Figma = con trong Unity
- **Không flatten, không gộp** bất kỳ layer nào trừ khi layer đó là leaf node dạng ảnh thuần

### 16.2 Sibling Order — Đảo ngược để giữ đúng z-order

> ⚠️ **Quy tắc bắt buộc:** Thứ tự sibling phải được **đảo ngược** khi tạo GameObject.

**Lý do:** Figma và Unity có chiều z-order ngược nhau:

| | Render trên cùng (z cao nhất) |
|---|---|
| **Figma** | Layer **đầu tiên** (trên cùng trong panel) |
| **Unity uGUI** | Child **cuối cùng** (sibling index cao nhất) |

**Ví dụ:**

```
Figma Layer Panel (top = on top):    Unity Hierarchy (last = on top):
───────────────────────────────      ─────────────────────────────────
[0] Btn_Play    ← render trên    →   [0] Bg_Main     ← render sau (behind)
[1] Img_Banner                   →   [1] Img_Banner
[2] Bg_Main     ← render sau     →   [2] Btn_Play    ← render trên (on top)
```

**Quy tắc:** Khi create GameObject từ danh sách children của 1 node Figma:
```
Lấy children[] từ Figma API  →  Reverse array  →  Tạo GameObject theo thứ tự đã reverse
```

Kết quả visual trong Unity sẽ **khớp hoàn toàn** với Figma.

---

## 17. Checklist Validation (trước khi import)

Automation tự validate trước khi chạy, throw error nếu vi phạm:

- [ ] Frame có tên bắt đầu `Screen_`
- [ ] Kích thước Frame khớp Reference Resolution của project
- [ ] Không có layer trùng tên trong cùng 1 Frame
- [ ] Không có layer tên mặc định Figma
- [ ] Mọi `Scroll_*` frame: `Clip Content = true` và có con `Content_`
- [ ] Mọi `Content_`: là con trực tiếp của `Scroll_*`
- [ ] Mọi `Scrollbar_`: có đúng 2 con `Img_Track`, `Img_Handle`
- [ ] Mọi `Slider_`: có đúng 3 con `Img_Background`, `Img_Fill`, `Img_Handle`
- [ ] Mọi `Toggle_`: có đúng 2 con `Img_Background`, `Img_Checkmark`
- [ ] Btn_ với Variants chỉ dùng property name `State` và values hợp lệ
- [ ] Auto Layout không lồng quá 4 cấp
