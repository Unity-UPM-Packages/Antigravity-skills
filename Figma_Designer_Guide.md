# 🎨 Figma Designer Guide — Unity Automation Pipeline

> **Mục tiêu:** Làm đúng các bước này = hệ thống AI tự động import 100% vào Unity, không cần chỉnh tay.

---

## Nguyên tắc cốt lõi

- **Tên layer = ngôn ngữ giao tiếp với automation.** Đặt tên sai = import sai.
- **Component = Prefab trong Unity.** Dùng Component Figma đúng cách = Unity có Prefab tái sử dụng.
- **Constraints = Anchor trong Unity.** Không set Constraints = UI vỡ khi đổi thiết bị.

---

## Bước 1 — Setup Frame (Màn hình)

Mỗi màn hình = **1 Frame riêng**, đặt tên theo màn hình đó:

```
Frame: Screen_Home
Frame: Screen_Shop
Frame: Screen_Settings
```

Kích thước Frame phải khớp với `Reference Resolution` của project — hỏi Lead/Dev để xác nhận trước khi bắt đầu:

| Orientation | Kích thước chuẩn |
|---|---|
| Portrait | 1080 × 1920 |
| Landscape | 1920 × 1080 |

> ⚠️ Sai kích thước Frame = sai tỷ lệ toàn bộ UI khi import.

---

## Bước 2 — Đặt tên Layer (Prefix System)

Hệ thống đọc **prefix** ở đầu tên layer để tạo đúng component Unity tương ứng.

### Bảng Prefix

| Prefix | Unity tạo ra | Dùng khi nào |
|---|---|---|
| `Btn_` | `Button` + `Image` | Nút bấm có tương tác |
| `Img_` | `Image` | Hình ảnh, icon, avatar, banner |
| `Bg_` | `Image` (không tương tác) | Nền tĩnh của panel/frame |
| `Txt_` | `TextMeshProUGUI` | Mọi loại chữ |
| `Scroll_V_` | `ScrollRect` (dọc) | Vùng cuộn theo chiều dọc |
| `Scroll_H_` | `ScrollRect` (ngang) | Vùng cuộn theo chiều ngang |
| `Scroll_VH_` | `ScrollRect` (2 chiều) | Vùng cuộn cả dọc lẫn ngang |
| `Content_` | Content của ScrollRect | Container bên trong Scroll |
| `Scrollbar_` | `Scrollbar` | Thanh cuộn kéo tay |
| `Slider_` | `Slider` | Thanh kéo giá trị (volume, HP) |
| `Toggle_` | `Toggle` | Checkbox bật/tắt |
| `Mask_` | `Image` + `Mask` | Cắt/clip nội dung, không cần scroll |
| `Safe_` | `RectTransform` (không Image) | Vùng Safe Area (notch, home bar) |

> Layer **không có prefix hợp lệ** → hệ thống coi là `RectTransform` container thuần — không sinh Component.

### Quy tắc đặt tên

- **PascalCase** cho phần mô tả: `Btn_PlayNow` ✅ — `btn play now` ❌
- **Không dùng dấu cách** — dùng `_` thay thế
- **Không đặt tên trùng** trong cùng 1 Frame — gây conflict khi automation sinh code
- Không dùng tên mặc định của Figma: `Frame 1`, `Rectangle 5`, `Group 2`...

---

## Bước 3 — Component System (Tái sử dụng)

> **Quy tắc vàng:** Bất kỳ thứ gì xuất hiện ở nhiều hơn 1 màn hình → phải là **Figma Component**.

### Cách tạo Component

1. Thiết kế xong 1 cụm (ví dụ: `Header`, `Card_Item`, `Btn_Primary`)
2. Chọn toàn bộ cụm → chuột phải → **Create Component** (hoặc `Ctrl+Alt+K`)
3. Đặt **Master Component** vào trang `🧩 Components` (trang riêng, không phải trang design)
4. Khi dùng ở màn hình: `Ctrl+C` / `Ctrl+V` trên master → Figma tự tạo **Instance**

### Mapping sang Unity

```
Master Component (Figma)  →  Prefab (Unity)
Instance trên Screen_A    →  Prefab Instance với Overrides của màn hình A
Instance trên Screen_B    →  Prefab Instance với Overrides của màn hình B
```

> Hệ thống **đọc giá trị override trên từng instance** (text, ảnh, visibility) và apply vào đúng Prefab Instance trong Unity — Prefab gốc không bị thay đổi.

---

## Bước 4 — Variants (Trạng thái của Component)

Dùng **Figma Variants** để định nghĩa các trạng thái khác nhau của 1 component.

### Khi nào cần Variants?

Khi component thay đổi **hình dạng / ảnh** theo trạng thái (không chỉ đổi màu):

```
Btn_Primary
├── State=Normal     ← ảnh nút bình thường
├── State=Pressed    ← ảnh nút khi bấm
└── State=Disabled   ← ảnh nút bị vô hiệu
```

### Cách tạo Variants

1. Tạo Master Component
2. Trong Component, nhấn **"Add variant"** trên panel bên phải
3. Đặt tên property: `State` — đặt tên từng variant: `Normal`, `Pressed`, `Disabled`
4. Thiết kế visual khác nhau cho từng variant

### Nếu không có Variants

→ Hệ thống dùng **ColorBlock** (Unity tự tint màu theo trạng thái). Designer không cần làm thêm gì.

### Quy tắc đặt tên Variant Property

| Property Name | Giá trị hợp lệ | Unity mapping |
|---|---|---|
| `State` | `Normal`, `Pressed`, `Disabled` | `SpriteState` của Button |
| `Checked` | `True`, `False` | `isOn` của Toggle |

> Chỉ dùng đúng tên property và giá trị như bảng trên. Đặt sai tên → hệ thống không map được.

---

## Bước 5 — Text (Không bake ảnh)

Mọi chữ **có thể thay đổi** → phải là layer `Txt_`, không được gộp vào ảnh.

| Đúng ✅ | Sai ❌ |
|---|---|
| Layer `Txt_Score` với text gõ thẳng trong Figma | Merge chữ + nền thành 1 ảnh PNG |
| Font, size, màu được set trong Figma | Export text thành bitmap |

> Hệ thống đọc: **font family, font weight, font size, màu chữ, letter spacing, line height, text alignment** — tất cả apply 1:1 vào `TextMeshProUGUI`.

### Về Font

- Ưu tiên dùng **Google Fonts** có trong thư viện Figma (Inter, Roboto, Noto Sans...) → hệ thống tự download và tạo TMP FontAsset.
- Nếu dùng **font thương mại** (không có trên Google Fonts) → phải cung cấp file `.ttf` / `.otf` cho dev để đặt thủ công vào `Assets/UI/Fonts/`. Hệ thống sẽ log warning khi không tìm thấy font.

---

## Bước 6 — 9-Slice (Object co dãn không vỡ góc)

Dùng khi object có thể **kéo dài / rộng hơn** mà không được vỡ góc (button nhiều kích thước, popup, panel...).

### Cách làm

1. Thiết kế component ở **kích thước nhỏ nhất** (vừa đủ chứa góc + 1px vùng giữa)
2. Thêm hậu tố `_9s` vào tên
3. Thêm annotation border: `_L[n]_R[n]_T[n]_B[n]` (đơn vị: pixel)

```
Btn_Play_9s_L12_R12_T8_B8
         ↑    ↑              
      9-slice  border: Left=12, Right=12, Top=8, Bottom=8
```

> Nếu không có border annotation → hệ thống fallback **25% mỗi cạnh** (có thể sai).

### Kéo ra kích thước thật

- Trong Figma, bạn **có thể preview** bằng cách kéo frame ra kích thước thật
- **Ảnh export vẫn là kích thước gốc nhỏ nhất** — Unity dùng `RectTransform` để scale
- **Không cần export ảnh kích thước to**

---

## Bước 7 — Auto Layout (Danh sách có thứ tự)

Mọi nhóm item **xếp hàng có thứ tự** → bật Auto Layout, không xếp tay.

| Figma Auto Layout | Unity Component |
|---|---|
| Horizontal (→) | `Horizontal Layout Group` |
| Vertical (↓) | `Vertical Layout Group` |
| Wrap (lưới) | `Grid Layout Group` |

Hệ thống copy 1:1: **Padding, Spacing, Alignment**.

### Resizing

| Figma Resizing | Unity sinh ra |
|---|---|
| **Fixed** | Kích thước tĩnh trên RectTransform |
| **Hug contents** | `Content Size Fitter` |
| **Fill container** | Stretch anchor |

> ⚠️ Không set cả Width lẫn Height đều **Hug contents** trên cùng 1 container → gây layout loop trong Unity.

> ⚠️ Không lồng Auto Layout quá **4 cấp** — ảnh hưởng hiệu năng Unity Layout system.

---

## Bước 8 — Scroll View (Vùng cuộn)

Cần đúng **2 lớp** theo cấu trúc sau:

```
Scroll_V_Inventory          ← Frame, kích thước = vùng nhìn thấy
│   [Bật: Clip Content ✅]
└── Content_Inventory        ← Auto Layout (dọc), chứa tất cả item
    ├── Item_1
    ├── Item_2
    └── Item_3
```

**Checklist Scroll View:**
- [ ] Frame ngoài dùng prefix `Scroll_V_`, `Scroll_H_`, hoặc `Scroll_VH_`
- [ ] **Bật Clip Content** trên Frame ngoài
- [ ] Kích thước Frame = **vùng nhìn thấy**, không phải toàn bộ nội dung
- [ ] `Content_` là **con trực tiếp** của frame cuộn
- [ ] `Content_` bật **Auto Layout**
- [ ] `Content_` set **Height = Hug contents** (để tự mở rộng khi thêm item)

---

## Bước 9 — Scrollbar

Vẽ tay theo cấu trúc cố định, chỉ được có **đúng 2 layer con**:

```
Scrollbar_Inventory
├── Img_Track      ← nền mờ (rãnh)
└── Img_Handle     ← nút kéo (con trượt)
```

> Hệ thống nhận diện dọc/ngang tự động: **chiều cao > chiều rộng = dọc**, **chiều rộng > chiều cao = ngang**.
> Scrollbar **không được vuông** — ratio tối thiểu 1:3.

---

## Bước 10 — Slider

```
Slider_Volume
├── Img_Background    ← nền rãnh đầy đủ
├── Img_Fill          ← phần đã fill (bên trái / bên dưới)
└── Img_Handle        ← nút kéo
```

---

## Bước 11 — Toggle

```
Toggle_Sound
├── Img_Background    ← khung ngoài
└── Img_Checkmark     ← dấu check (visible = đang bật)
```

> Layer `Img_Checkmark` **visible** trong Figma → Toggle mặc định = **ON**.  
> Layer `Img_Checkmark` **ẩn** trong Figma → Toggle mặc định = **OFF**.

---

## Bước 12 — Constraints (Neo góc)

> ⚠️ **Bắt buộc set cho MỌI layer.** Không set Constraints = Anchor sai = UI vỡ khi đổi thiết bị.

| Tôi muốn layer này... | Constraints cần set |
|---|---|
| Ghim góc trên-trái | Left + Top |
| Ghim góc trên-phải | Right + Top |
| Ghim góc dưới-trái | Left + Bottom |
| Ghim góc dưới-phải | Right + Bottom |
| Căn giữa màn hình | Center + Center |
| Căn giữa ngang, ghim trên | Center + Top |
| Kéo dài full ngang | Left + Right |
| Kéo dài full dọc | Top + Bottom |
| Full màn hình | Left + Right + Top + Bottom |

**Cách set:** Chọn layer → Design panel bên phải → mục **Constraints**.

---

## Bước 13 — Visibility & States

| Figma | Unity nhận |
|---|---|
| Layer visible | `GameObject.SetActive(true)` |
| Layer ẩn (Hidden) | `GameObject.SetActive(false)` |
| Opacity < 100% | `CanvasGroup.alpha = [opacity/100]` |

> Override visibility trên Instance được hệ thống đọc và apply riêng cho từng màn hình.

---

## ✅ Checklist trước khi bàn giao

- [ ] Frame đúng kích thước resolution của project
- [ ] Frame đặt tên đúng: `Screen_[TênMànHình]`
- [ ] Mọi layer có **prefix hợp lệ**
- [ ] Không có **tên layer trùng** trong cùng 1 Frame
- [ ] Không có tên layer mặc định Figma (`Frame 1`, `Rectangle 5`...)
- [ ] Mọi `Txt_` layer là **text thuần**, không bake vào ảnh
- [ ] Component tái sử dụng đã được tạo đúng cách và để ở trang `🧩 Components`
- [ ] Button có thay đổi hình dạng theo state → đã tạo **Variants** với đúng tên property
- [ ] Object co dãn có hậu tố **`_9s`** kèm border annotation
- [ ] Auto Layout dùng cho mọi danh sách có thứ tự, không lồng quá 4 cấp
- [ ] Scroll View: đúng cấu trúc `Scroll_` + `Content_`, bật **Clip Content**
- [ ] Scrollbar/Slider/Toggle: đúng cấu trúc layer con
- [ ] Mọi layer đã set **Constraints**
- [ ] Font dùng từ Google Fonts hoặc đã thông báo dev về file font thương mại
