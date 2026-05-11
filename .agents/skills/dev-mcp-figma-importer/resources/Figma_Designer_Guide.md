# 🎨 Figma Designer Guide — Quick Reference

> **Nguyên tắc cốt lõi:** Tên layer = ngôn ngữ giao tiếp với automation. Đặt sai = import sai.

---

## 1. Frame (Màn hình)

- Mỗi màn hình = **1 Frame**, đặt tên: `Screen_Home`, `Screen_Shop`, `Screen_Settings`
- Kích thước Frame **phải khớp** Reference Resolution (hỏi Lead/Dev trước):

| Orientation | Kích thước |
|---|---|
| Portrait | 1080 × 1920 |
| Landscape | 1920 × 1080 |

> ⚠️ Sai kích thước = sai tỷ lệ toàn bộ UI khi import.

---

## 2. Prefix Table

| Prefix | Unity Component | Ghi chú |
|---|---|---|
| `Btn_` | `Button` + `Image` | Nút bấm có tương tác |
| `Img_` | `Image` | Hình ảnh, icon, avatar, banner |
| `Bg_` | `Image` (không tương tác) | Nền tĩnh của panel/frame |
| `Txt_` | `TextMeshProUGUI` | Mọi loại chữ — không bake vào ảnh |
| `Scroll_V_` / `Scroll_H_` / `Scroll_VH_` | `ScrollRect` + `RectMask2D` | vertical / horizontal / cả hai |
| `Content_` | child → `scrollRect.content` | Con trực tiếp của `Scroll_*`, bật Auto Layout |
| `Scrollbar_` | `Scrollbar` | Con: `Img_Track` + `Img_Handle` |
| `Slider_` | `Slider` | Con: `Img_Background` + `Img_Fill` + `Img_Handle` |
| `Toggle_` | `Toggle` | Con: `Img_Background` + `Img_Checkmark` |
| `Mask_` | `Image` + `Mask` | Clip nội dung, không scroll |
| `Safe_` | `RectTransform` only | Safe Area (notch, home bar) |
| *(không prefix)* | `RectTransform` thuần | Container rỗng |

**Naming:** PascalCase · dùng `_` · không dấu cách · không tên trùng trong cùng Frame · không tên mặc định Figma (`Frame 1`, `Rectangle 5`…)

---

## 3. Component & Variants

**Component:** Bất kỳ thứ gì xuất hiện > 1 màn hình → phải là Figma Component.
- Master Component đặt ở trang `Components` (trang riêng)
- Instance tại màn hình → automation đọc override (text/ảnh/visibility) → apply vào Prefab Unity
- Prefab gốc không bị thay đổi khi override

**Variants — khi nào cần:** Component thay đổi hình/ảnh theo state (không chỉ đổi màu):

| Property Name | Giá trị hợp lệ | Unity mapping |
|---|---|---|
| `State` | `Normal` / `Pressed` / `Disabled` / `Highlighted` | Button `SpriteState` |
| `Checked` | `True` / `False` | Toggle `isOn` |

> ⚠️ Dùng đúng tên property và giá trị như bảng. Sai tên → hệ thống không map được.  
> Nếu không có Variants → dùng ColorBlock (Unity tự tint màu, không cần làm gì thêm).

---

## 4. Text & Font

- Layer `Txt_` = text thuần trong Figma. **Không merge chữ vào ảnh PNG.**
- Hệ thống đọc 1:1: font family, weight, size, màu, letter spacing, line height, alignment → `TextMeshProUGUI`
- **Font:** Ưu tiên Google Fonts (Inter, Roboto, Noto Sans…) → hệ thống tự tải và tạo TMP FontAsset
- **Font thương mại:** cung cấp `.ttf`/`.otf` cho dev thủ công → hệ thống log `[WARN]` khi không tìm thấy

---

## 5. 9-Slice (co dãn không vỡ góc)

Dùng khi object kéo to/nhỏ mà không vỡ góc (button nhiều size, popup, panel).

1. Thiết kế ở kích thước chuẩn → dùng **Figma 9-Slice plugin** để tách thành 9 mảnh
2. Thêm suffix **`_9s`** vào tên Group/Component: `Btn_Play_9s`
3. **Không cần gõ thông số border** — hệ thống tự trích xuất từ tọa độ 9 mảnh con

> Export vẫn là kích thước gốc nhỏ nhất — Unity dùng `RectTransform` để scale. Không xuất ảnh kích thước to.

---

## 6. Filled Image (tiến trình)

Dùng cho HP bar, loading bar, countdown ring — tạo **layer tĩnh mới** (không dùng 9-Slice).

| Suffix | Unity `fillMethod` | Dùng khi nào |
|---|---|---|
| `_fh` | `Horizontal` | HP bar, loading bar ngang |
| `_fv` | `Vertical` | Progress bar dọc |
| `_fr360` | `Radial360` | Loading ring tròn hoàn toàn |
| `_fr180` | `Radial180` | Nửa vòng tròn |
| `_fr90` | `Radial90` | Góc phần tư |

> ⚠️ **Không dùng `_9s` và `_fh/_fv/_fr*` trên cùng 1 layer.**  
> `Img_Fill` bên trong `Slider_` **không cần** suffix này — hệ thống tự xử lý.

---

## 7. Auto Layout

| Figma | Unity | Resizing | Unity |
|---|---|---|---|
| Horizontal → | `HorizontalLayoutGroup` | Fixed | Kích thước tĩnh |
| Vertical ↓ | `VerticalLayoutGroup` | Hug contents | `ContentSizeFitter` |
| Wrap (lưới) | `GridLayoutGroup` | Fill container | Stretch anchor |

Hệ thống copy 1:1: Padding · Spacing · Alignment.

> ⚠️ Không set cả Width lẫn Height đều **Hug contents** trên cùng container → layout loop.  
> ⚠️ Không lồng Auto Layout quá **4 cấp** → ảnh hưởng hiệu năng Unity.

---

## 8. Scroll View / Scrollbar / Slider / Toggle

| Component | Cấu trúc & Yêu cầu |
|---|---|
| `Scroll_*` | Frame ngoài = vùng nhìn thấy · **bật Clip Content** · con `Content_` trực tiếp bật Auto Layout + Hug height |
| `Scrollbar_` | Đúng 2 con: `Img_Track` + `Img_Handle` · **không vuông** (ratio tối thiểu 1:3) · direction tự detect |
| `Slider_` | Đúng 3 con: `Img_Background` + `Img_Fill` + `Img_Handle` |
| `Toggle_` | Đúng 2 con: `Img_Background` + `Img_Checkmark` · visible=ON / hidden=OFF |

---

## 9. Constraints (bắt buộc mọi layer)

> ⚠️ Không set Constraints = Anchor sai = UI vỡ khi đổi thiết bị.

| Mục tiêu | Constraints |
|---|---|
| Ghim góc trên-trái / trên-phải | Left+Top / Right+Top |
| Ghim góc dưới-trái / dưới-phải | Left+Bottom / Right+Bottom |
| Căn giữa màn hình | Center + Center |
| Căn giữa ngang, ghim trên/dưới | Center + Top / Center + Bottom |
| Kéo dài full ngang / full dọc | Left+Right / Top+Bottom |
| Full màn hình | Left + Right + Top + Bottom |

---

## 10. Export Ảnh (Download về Unity)

**Visibility:** `visible=true` → `SetActive(true)` · ẩn → `SetActive(false)` · `opacity<100%` → `CanvasGroup.alpha`

**Quy tắc download:**
1. Export đúng **node ID của layer lá** (`Img_`, `Bg_`, `Btn_`…) — **không export frame/group chứa nó** (sẽ bị tính thêm padding → sai kích thước)
2. Luôn chọn **`1x` (1:1)** khi export thủ công từ Figma
3. Kích thước file = kích thước layer Figma (1:1) — dùng để verify

**Folder routing:**
- `Screen_*` layer → `Assets/UI/Sprites/[ScreenName]/`
- Trang `Components` layer → `Assets/UI/Sprites/Common/`

**Import Settings (bắt buộc cho MỌI sprite):**

| Tham số | Giá trị |
|---|---|
| Texture Type | `Sprite (2D and UI)` |
| Sprite Mode | `Single` (icon) / `Multiple` (sprite sheet) |
| Filter Mode | `Bilinear` |
| Compression | Lossless (PNG) / Normal (JPG) |

---

## ✅ Checklist Bàn Giao

- [ ] Frame đúng kích thước resolution · đặt tên `Screen_[Tên]`
- [ ] Mọi layer có prefix hợp lệ · không tên trùng trong Frame · không tên mặc định Figma
- [ ] `Txt_` là text thuần, không bake vào ảnh
- [ ] Component tái sử dụng → trang `Components`; Variants đặt đúng tên property `State`/`Checked`
- [ ] Object co dãn → hậu tố `_9s`, đã cắt 9 mảnh bằng plugin
- [ ] Image tiến trình → hậu tố `_fh/_fv/_fr*` · **không** kết hợp với `_9s`
- [ ] Auto Layout đúng hướng · không lồng quá 4 cấp
- [ ] Scroll: `Clip Content=true` · `Content_` là con trực tiếp · Hug height
- [ ] Scrollbar/Slider/Toggle đúng cấu trúc layer con
- [ ] Mọi layer đã set Constraints
- [ ] Font từ Google Fonts hoặc đã báo dev file font thương mại
