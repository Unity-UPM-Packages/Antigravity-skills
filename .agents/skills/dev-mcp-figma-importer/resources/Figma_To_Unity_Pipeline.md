# 🤖 Figma → Unity Pipeline — Quick Reference v3.0
> Tóm tắt kỹ thuật. Bản gốc: `Figma_To_Unity_Pipeline.md`

---

## 1. Prefix → Unity Component

| Prefix | Unity Component | Ghi chú |
|---|---|---|
| `Btn_` | `Button` + `Image` | Variants `State` → SpriteSwap; không có → ColorTint |
| `Img_` | `Image` | Hình ảnh tổng quát |
| `Bg_` | `Image` (non-interactive) | Không add `Button` |
| `Txt_` | `TextMeshProUGUI` | Không export ảnh |
| `Scroll_V_` / `Scroll_H_` / `Scroll_VH_` | `ScrollRect` + `RectMask2D` | vertical / horizontal / cả hai |
| `Content_` | child → `scrollRect.content` | Con trực tiếp của `Scroll_*` |
| `Scrollbar_` | `Scrollbar` | Con: `Img_Track` + `Img_Handle`; direction: height>width→BottomToTop |
| `Slider_` | `Slider` | Con: `Img_Background` + `Img_Fill`(fillRect) + `Img_Handle`(handleRect) |
| `Toggle_` | `Toggle` | Con: `Img_Background` + `Img_Checkmark`(graphic); visible=isOn |
| `Mask_` | `Image` + `Mask` | Clip, không scroll |
| `Safe_` | `RectTransform` only | Safe Area offset |
| *(không prefix)* | `RectTransform` thuần | Container rỗng |

**Naming:** PascalCase, dùng `_`, unique trong cùng Frame (trùng tên = conflict prefab). Trùng Frame khác = hợp lệ.

---

## 2. Canvas & Resolution

| Thiết lập | Giá trị |
|---|---|
| Reference Resolution | Portrait `1080×1920` / Landscape `1920×1080` (đọc từ project config) |
| Render Mode | `Screen Space — Overlay` |
| Canvas Scaler | `Scale With Screen Size` |
| Screen Match Mode | `Match Width Or Height` — Portrait `0.0` / Landscape `1.0` |

**Hierarchy:** Tìm Canvas đầu tiên trong scene → dùng lại; chưa có → tạo mới. Screen luôn là **con trực tiếp** của Canvas.

---

## 3. Component → Prefab & Overrides

- **Master Component** → `Assets/UI/Prefabs/Common/[Name].prefab` (bản gốc, không override)
- **Instance** → Instantiate Prefab + apply overrides (không sửa Prefab gốc)
- **Nested:** Component con = Prefab riêng → lồng vào Prefab cha; override apply đúng cấp

| Override | Áp dụng |
|---|---|
| Text | `TextMeshProUGUI.text = value` |
| Image | Download ảnh → `Image.sprite` |
| Visibility | `GameObject.SetActive(isVisible)` |
| Color | `Image.color` / `TMP.color` |
| Position/Size | Cập nhật `RectTransform` |

---

## 4. Smart Merge

| | Rule |
|---|---|
| **UPDATE** | RectTransform, Image.sprite/color, TMP.text, font props, SetActive, Layout Group, ContentSizeFitter, Prefab Overrides |
| **GIỮ NGUYÊN** | MonoBehaviour do dev thêm, SerializedField references, component ngoài scope |
| **Match** | Tên GameObject = tên layer (p1) → Figma Node ID trong metadata (p2) |
| **Không match** | Tạo mới — **không bao giờ xóa** GameObject còn script đang binding |
| **Log** | `[MERGE]` thay đổi · `[SKIP]` giữ script · `[WARN]` font thiếu |

---

## 5. Export & Download Sprites

| Loại | Format | Scale |
|---|---|---|
| Icon, UI element | PNG | @1× |
| Chi tiết mảnh (line < 2px) | PNG | @2× (`_x2` suffix) |
| Background toàn màn hình | JPG q90 | @1× |
| 9-Slice sprite | PNG | @1× (kích thước nhỏ nhất) |

> ⚠️ **CRITICAL:** `save_screenshots`/`get_screenshot` default `scale=2`. **Luôn truyền `scale=1` tường minh.**

**Thuật toán download đúng:**
1. Export **đúng node ID của layer ảnh** (leaf node `Img_`, `Bg_`, `Btn_`...) — **không export frame/group chứa nó** (sẽ bị tính thêm padding → sai kích thước)
2. Gọi `save_screenshots` với `scale=1` tường minh
3. Kích thước file PNG = kích thước layer trong Figma (1:1) — dùng để verify

**Folder routing:**
- Layer thuộc `Screen_*` → `Assets/UI/Sprites/[ScreenName]/`
- Layer thuộc trang `Components` → `Assets/UI/Sprites/Common/`

**Tên file:** bỏ prefix + suffix automation (`_9s`, `_fh`, `_x2`…) → lowercase_underscore.

**Import Settings (áp dụng cho MỌI sprite download về):**
```
TextureType    = Sprite (2D and UI)
SpriteMeshType = FullRect
FilterMode     = Bilinear
Compression    = Lossless (PNG) / Normal (JPG)
```

---

## 6. 9-Slice

**Detection:** suffix `_9s` → bắt buộc có đúng 9 child dạng lưới (log warning nếu khác).

**Tính Border tự động:** Parse tọa độ 9 mảnh con:
- `Left` = Width góc Top-Left · `Top` = Height góc Top-Left
- `Right` = Width góc Bottom-Right · `Bottom` = Height góc Bottom-Right

> ⚠️ **Vector4 order:** `sprite.border = new Vector4(left, **bottom**, right, **top**)` — **KHÁC** annotation `L→R→T→B`.

**Import:** Dùng chung Import Settings ở §5 + thêm `border = Vector4(L, B, R, T)`.

---

## 7. Filled Image

**Detection:** suffix `_fh/_fv/_fr90/_fr180/_fr360` (không apply cho `Img_Fill` con của `Slider_`).

| Suffix | `fillMethod` | Suffix | `fillMethod` |
|---|---|---|---|
| `_fh` | `Horizontal` | `_fr90` | `Radial90` |
| `_fv` | `Vertical` | `_fr180` | `Radial180` |
| | | `_fr360` | `Radial360` |

`image.type = Filled; image.fillAmount = 1.0f;` (runtime tự điều chỉnh).

---

## 8. Auto Layout → Layout Group

| Figma | Unity | Resizing | Unity |
|---|---|---|---|
| Horizontal → | `HorizontalLayoutGroup` | Hug Width | `horizontalFit = PreferredSize` |
| Vertical ↓ | `VerticalLayoutGroup` | Hug Height | `verticalFit = PreferredSize` |
| Wrap Grid | `GridLayoutGroup` (CellSize=first child) | Fixed | Không tạo CSF |

**Properties 1:1:** Padding TRBL · `spacing` · `childAlignment` · `reverseArrangement`.

> ⚠️ Không set cả Width lẫn Height đều Hug — gây layout loop.

---

## 9. Constraints → Anchors

| Figma | anchorMin | anchorMax | Figma | anchorMin | anchorMax |
|---|---|---|---|---|---|
| Left+Top | (0,1) | (0,1) | Right+Top | (1,1) | (1,1) |
| Left+Bottom | (0,0) | (0,0) | Right+Bottom | (1,0) | (1,0) |
| Center+Top | (0.5,1) | (0.5,1) | Center+Bottom | (0.5,0) | (0.5,0) |
| Center+Center | (0.5,0.5) | (0.5,0.5) | Full Stretch | (0,0) | (1,1) |
| Stretch H | (0,Y) | (1,Y) | Stretch V | (X,0) | (X,1) |
| Scale | pivot | pivot | | | |

- **Point anchor** (min==max): `anchoredPosition` + `sizeDelta`
- **Stretch anchor**: `offsetMin=(left, bottom)` · `offsetMax=(−right, −top)`

---

## 10. Font Pipeline (Figma → TMP)

| Figma | TMP | Figma | TMP |
|---|---|---|---|
| `fontFamily` | Download → SDF Asset | `letterSpacing` | `characterSpacing` |
| `fontWeight` | Regular/Bold… | `lineHeight` | `lineSpacing` |
| `fontSize` | `fontSize` | `textAlignH/V` | `alignment` |
| `fills[0].color` | `color` | `textDecoration` | `fontStyle` (underline/strike) |

**Pipeline:** Google Fonts → `.ttf` → `Assets/UI/Fonts/` → TMP Font Asset Creator → cache. **Fallback:** `defaultFontAsset` + `[WARN]`, tiếp tục apply size/color/spacing.

---

## 11. Visibility & Z-Order

- `visible=true` → `SetActive(true)` · `visible=false` → `SetActive(false)` · `opacity<100%` → `CanvasGroup.alpha`
- Override trên Instance không ảnh hưởng Prefab gốc

> ⚠️ **Z-Order REVERSE bắt buộc:** Figma layer[0] = render trên; Unity child[last] = render trên.  
> **Rule:** `children[] từ Figma API → Reverse array → tạo GameObject theo thứ tự đã reverse.`

---

## 12. Validation Checklist (auto, throw error nếu vi phạm)

- Frame tên bắt đầu `Screen_`; kích thước khớp Reference Resolution
- Không layer trùng tên trong cùng Frame; không tên mặc định Figma
- `Scroll_*`: `Clip Content=true`, có con `Content_` trực tiếp
- `Scrollbar_`: đúng 2 con `Img_Track`, `Img_Handle`
- `Slider_`: đúng 3 con `Img_Background`, `Img_Fill`, `Img_Handle`
- `Toggle_`: đúng 2 con `Img_Background`, `Img_Checkmark`
- `Btn_` Variants: property name = `State`, values hợp lệ
- Auto Layout không lồng quá 4 cấp
