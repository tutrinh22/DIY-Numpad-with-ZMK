# DIY Numpad - ZMK Firmware Documentation

---

## Changelog

### [v1.5.0] - 2026-04-17
- Thêm **Bluetooth multi-device support**: quản lý 5 profile BLE ngay trong Boot Mode
- Thêm `#include <dt-bindings/zmk/bt.h>` vào keymap
- Boot Mode có thêm các phím: `BT_SEL 0–4`, `BT_CLR`, `BT_CLR_ALL`

### [v1.4.0] - 2026-03-27
- Fix bàn phím không thức dậy khi đang chạy pin: thêm `wakeup-source` vào `kscan0` trong overlay

### [v1.3.0] - 2026-03-26
- Thêm **Boot Mode** (4-tap PAUSE): `NUM` 1 lần = reset, `NUM` 2 lần = UF2 bootloader
- Bật `CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y` — LED tắt sau 30s không dùng

### [v1.2.0] - 2026-03-26
- Thêm **LED Edit Mode** (3-tap PAUSE):
  - `*` → đổi màu (HUE+)
  - `+` 1 lần / 2 lần → tăng độ sáng / tăng tốc độ 5 bước
  - `-` 1 lần / 2 lần → giảm độ sáng / giảm tốc độ 5 bước
- Xóa các symbol Kconfig không hợp lệ: `ZMK_RGB_UNDERGLOW_EFF/SPD/BRT`

### [v1.1.0] - 2026-03-26
- Thêm WS2812B RGB underglow (24 LED, D1/P0.06, SPI3)
- Thêm tap-dance cho phím PAUSE:
  - 1 lần → `PAUSE_BREAK`
  - 2 lần → bật LED + chuyển effect tiếp theo
- Thêm macro `rgb_next` (RGB_ON + RGB_EFF) để đảm bảo LED bật trước khi đổi effect
- Bật `CONFIG_ZMK_RGB_UNDERGLOW_ON_START=y`
- Sửa `CONFIG_ZMK_IDLE_TIMEOUT=30000+` → `30000` (bỏ dấu `+` thừa)

### [v1.0.0] - 2026-03-25
- Hoàn thiện build ZMK firmware cho board SuperMini NRF52840
- Fix ROW5 từ D16 (chân NFC, P0.10) → **D6 (P1.00)**
- Disable NFC controller (`&nfct { status = "disabled"; }`)
- Fix vị trí phím ENTER: `RC(4,3)` → `RC(3,3)`
- Fix BLE reconnect: `CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START=n`
- Cấu hình sleep: idle 30s, deep sleep 18 phút
- Build thành công với Docker image `zmkfirmware/zmk-dev-arm:4.1-branch`

---


## Phần cứng

| Thành phần | Thông tin |
|------------|-----------|
| Vi điều khiển | SuperMini NRF52840 (nice!nano v2 compatible) |
| ZMK board ID | `nice_nano//zmk` |
| Shield | `diy_numpad` |
| LED | WS2812B, 24 LED, chân DATA = D1 (P0.06) |
| Số phím | 21 phím |

---

## Pinout

### Ma trận phím (6 hàng × 4 cột, col2row)

| | COL0 (D2 / P0.17) | COL1 (D3 / P0.20) | COL2 (D4 / P0.22) | COL3 (D5 / P0.24) |
|---|---|---|---|---|
| **ROW0** (D21 / P0.31) | PrtScr | Ins | ScrLk | Pause |
| **ROW1** (D20 / P0.29) | NumLk | / | * | - |
| **ROW2** (D19 / P0.02) | 7 | 8 | 9 | + ↕ |
| **ROW3** (D18 / P1.15) | 4 | 5 | 6 | Enter ↕ |
| **ROW4** (D15 / P1.13) | 1 | 2 | 3 | *(không có switch)* |
| **ROW5** (D6 / P1.00) | 0 ↔ | *(không có switch)* | Del | *(không có switch)* |

> **Lưu ý quan trọng:**
> - ROW5 dùng **D6 (P1.00)** thay vì D16 (P0.10) vì D16 là chân NFC
> - NFC controller bị disable trong overlay để giải phóng chân GPIO
> - `+` là phím 2U tall (switch tại ROW2-COL3, keycap vật lý trải dài ROW2+ROW3)
> - `Enter` là phím 2U tall (switch tại ROW3-COL3, keycap vật lý trải dài ROW3+ROW4)
> - `0` là phím 2U wide (switch tại ROW5-COL0, keycap vật lý trải rộng COL0+COL1)

### WS2812B LED

| | |
|---|---|
| Chân DATA | D1 (P0.06) |
| Giao tiếp | SPI3 (bắt buộc dùng spi3 với nRF52) |
| Số LED | 24 |
| Màu thứ tự | GRB (Green-Red-Blue) |
| VCC | Nối vào **3.3V** (KHÔNG dùng 5V/RAW) |

---

## Layout phím

```
┌────────┬──────┬────────┬─────────┐
│ PRTSCR │ INS  │ SCRLK  │  PAUSE* │
├────────┼──────┼────────┼─────────┤
│  NUM   │  /   │   *    │    -    │
├────────┼──────┼────────┼─────────┤
│   7    │  8   │   9    │    +    │
├────────┼──────┼────────┼─────────┤
│   4    │  5   │   6    │  ENTER  │
├────────┼──────┼────────┼─────────┘
│   1    │  2   │   3    │         
├────────┼──────┼────────┤
│   0    │      │  DEL   │  
└────────┴──────┴────────┘
```

\* Phím PAUSE có chức năng đặc biệt (xem bên dưới)

---

## Chức năng phím PAUSE (Tap-Dance)

| Thao tác | Kết quả |
|----------|---------|
| **1 lần nhấn** | Gửi phím `PAUSE_BREAK` |
| **2 lần nhấn** | Bật LED + chuyển sang effect tiếp theo |
| **3 lần nhấn** | Vào / ra **LED Edit Mode** |
| **4 lần nhấn** | Vào / ra **Boot Mode** |

---

## LED Edit Mode

Khi vào LED Edit Mode, các phím sau trên numpad thay đổi chức năng:

| Phím | 1 lần nhấn | 2 lần nhấn |
|------|------------|------------|
| **PAUSE** | Thoát LED Edit Mode | — |
| **`*`** | Đổi màu (HUE+) | — |
| **`+`** | Tăng độ sáng +5 | Tăng tốc độ +5 |
| **`-`** | Giảm độ sáng -5 | Giảm tốc độ -5 |

> Các phím còn lại (`trans`) vẫn hoạt động bình thường.

---

## Boot Mode

Vào bằng **4-tap PAUSE**. Thoát bằng **1-tap PAUSE**.

```
┌──────────┬──────────┬──────────┬──────────┐
│  [pass]  │ BT_SEL 1 │ BT_SEL 2 │   EXIT   │
├──────────┼──────────┼──────────┼──────────┤
│   NUM*   │ BT_SEL 0 │ BT_SEL 3 │ BT_SEL 4 │
├──────────┼──────────┼──────────┼──────────┤
│  [none]  │  [none]  │  [none]  │ BT_CLR   │
├──────────┼──────────┼──────────┼──────────┤
│  [none]  │  [none]  │  [none]  │BT_CLR_ALL│
├──────────┼──────────┼──────────┤          │
│  [none]  │  [none]  │  [none]  │          │
├──────────┴──────────┼──────────┤          │
│       [none]        │  [none]  │          │
└─────────────────────┴──────────┴──────────┘
```

| Phím | Chức năng |
|------|-----------|
| **NUM** (1-tap) | Reset thiết bị |
| **NUM** (2-tap) | Vào UF2 bootloader để flash firmware |
| **INS** → `BT_SEL 1` | Chuyển sang profile 1 (điện thoại) |
| **SCRLK** → `BT_SEL 2` | Chuyển sang profile 2 |
| **/** → `BT_SEL 0` | Chuyển về profile 0 (PC) |
| **`*`** → `BT_SEL 3` | Chuyển sang profile 3 |
| **`-`** → `BT_SEL 4` | Chuyển sang profile 4 |
| **`+`** → `BT_CLR` | Xóa bond của profile hiện tại |
| **ENTER** → `BT_CLR_ALL` | Xóa **tất cả** bond BLE |

### Cách kết nối thiết bị mới

1. Vào Boot Mode (4-tap PAUSE)
2. Chọn profile trống (ví dụ: `BT_SEL 1` cho điện thoại)
3. Thoát Boot Mode (1-tap PAUSE)
4. Bật Bluetooth trên thiết bị → tìm **"Min Numpad"** → kết nối

---

## Danh sách hiệu ứng LED

Nhấn **2 lần PAUSE** để cycle qua các hiệu ứng:

| Số | Tên hiệu ứng |
|----|--------------|
| 0 | Solid (màu đơn sắc) |
| 1 | Breathe (nhấp nháy nhịp thở) |
| 2 | Rainbow (cầu vồng) |
| 3 | Snake (ánh sáng chạy dọc) |
| 4 | Knight Rider (quét qua lại) |
| 5 | Rainbow Swirl (xoáy cầu vồng) |
| 6 | Twinkle (lấp lánh ngẫu nhiên) |

---

## Cấu hình Bluetooth (BLE)

| Tùy chọn | Giá trị | Mô tả |
|----------|---------|-------|
| `CONFIG_ZMK_BLE` | `y` | Bật Bluetooth |
| `CONFIG_BT_DEVICE_NAME` | `"Min Numpad"` | Tên hiển thị BLE |
| `CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START` | `n` | Không xóa bond khi khởi động |
| `CONFIG_ZMK_USB` | `y` | Hỗ trợ USB đồng thời với BLE |

### Kết nối nhiều thiết bị (Multi-Profile)

ZMK hỗ trợ **5 profile BLE** (0–4). Quản lý qua **Boot Mode**:

| Profile | Thiết bị gợi ý |
|---------|----------------|
| 0 | PC (mặc định) |
| 1 | Điện thoại |
| 2–4 | Thiết bị khác |

> **Lưu ý:** ZMK chỉ quảng bá BLE trên 1 profile tại một thời điểm. Khi USB đang cắm, BLE không quảng bá.

---

## Cấu hình nguồn / Sleep

| Tùy chọn | Giá trị | Mô tả |
|----------|---------|-------|
| `CONFIG_ZMK_SLEEP` | `y` | Bật chế độ ngủ |
| `CONFIG_ZMK_IDLE_TIMEOUT` | `30000` (30s) | Sau 30s không dùng → idle |
| `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` | `1080000` (18 phút) | Sau 18 phút idle → ngủ sâu |

---

## Cấu hình LED RGB

| Tùy chọn | Giá trị | Mô tả |
|----------|---------|-------|
| `CONFIG_ZMK_RGB_UNDERGLOW` | `y` | Bật LED underglow |
| `CONFIG_ZMK_RGB_UNDERGLOW_ON_START` | `y` | LED bật ngay khi khởi động |
| `CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE` | `n` | Không tắt LED khi idle |
| `CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_USB` | `n` | Không tắt LED khi USB |
| `CONFIG_NFCT_PINS_AS_GPIOS` | `y` | Giải phóng chân NFC làm GPIO |

---

## Lệnh build firmware

```bash
docker run --rm -v "D:\Arduino\DIY-numpad:/zmk-workspace" zmkfirmware/zmk-dev-arm:4.1-branch /bin/bash -c "cd /zmk-workspace && west build -p -s zmk/app -b nice_nano//zmk -- -DSHIELD=diy_numpad -DZMK_CONFIG=/zmk-workspace/config -DCMAKE_PREFIX_PATH=/zmk-workspace/zephyr 2>&1"
```

File firmware output: `build/zephyr/zmk.uf2`

---

## Cấu trúc file

```
config/
├── diy_numpad.conf          # Kconfig options (BLE, USB, RGB, Sleep)
├── diy_numpad.keymap        # Keymap, behaviors, macros
└── boards/shields/diy_numpad/
    ├── diy_numpad.overlay   # Hardware config (matrix, SPI, LED)
    ├── diy_numpad.conf      # Shield-level config
    └── diy_numpad.zmk.yml   # Shield metadata
```
