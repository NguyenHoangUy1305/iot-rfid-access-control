# 🚪 HỆ THỐNG KIỂM SOÁT CỬA RFID & IOT TÍCH HỢP PHẦN MỀM DESKTOP

[![CI](https://github.com/NguyenHoangUy1305/iot-rfid-access-control/actions/workflows/ci.yml/badge.svg)](https://github.com/NguyenHoangUy1305/iot-rfid-access-control/actions/workflows/ci.yml)

> **Tên đề tài:** Xây dựng hệ thống kiểm soát truy cập cửa ứng dụng RFID và IoT, tích hợp phần mềm quản lý desktop và cơ chế phát hiện truy cập bất thường  
> **English Title:** Development of an IoT-Based RFID Door Access Control System with Desktop Management Software and Abnormal Access Detection  
> **Thời gian:** Tháng 10/2026 - Tháng 02/2027  
> **Trạng thái:** Đang phát triển (Giai đoạn khởi động: 06/10/2026)

---

> 📘 **SỔ TAY KỸ THUẬT & LỘ TRÌNH 10 TUẦN CHI TIẾT:** Xem toàn bộ lý thuyết, bẫy phần cứng, sơ đồ gói tin và checklist tại [`docs/ROADMAP_KY_THUAT.md`](./docs/ROADMAP_KY_THUAT.md)


## 1. CẤU TRÚC THƯ MỤC DỰ ÁN
```text
01-rfid-access-control/
├── firmware/       # Mã nguồn C++ cho ESP32 (PlatformIO / Arduino Framework)
│   ├── src/        # File main.cpp, wifi_service, rfid_service, offline_cache
│   └── include/    # File cấu hình chân pinout, config
├── server/         # Backend REST API (Node.js + TypeScript + Fastify + Prisma)
│   ├── prisma/     # schema.prisma, migrations, seed.ts
│   └── src/        # controllers, routes, services, middleware
├── desktop/        # Ứng dụng Desktop quản trị (Electron + React + TypeScript)
│   ├── electron/   # main process, preload script
│   └── src/        # renderer UI (Dashboard, Residents, Cards, Logs, Alerts)
├── docs/           # Sơ đồ khối, sơ đồ nguyên lý mạch, bảng mã API
└── README.md       # Tài liệu đặc tả dự án
```

---

## 2. THÀNH PHẦN PHẦN CỨNG & SƠ ĐỒ ĐẤU DÂY (PINOUT)

| Module | Chân trên Module | Chân kết nối ESP32 DevKit V1 | Chức năng |
| :--- | :--- | :--- | :--- |
| **RFID RC522** | SDA (SS) | **GPIO 5** | SPI Slave Select |
| | SCK | **GPIO 18** | SPI Clock |
| | MOSI | **GPIO 23** | SPI Master Out Slave In |
| | MISO | **GPIO 19** | SPI Master In Slave Out |
| | RST | **GPIO 22** | Reset chân đầu đọc |
| | 3.3V | **3V3** | Nguồn cấp 3.3V ổn định |
| | GND | **GND** | Nối đất chung |
| **Relay 5V 1 Kênh**| IN | **GPIO 26** | Điều khiển đóng/mở chốt khóa (Low Trigger) |
| | VCC / GND | VIN (5V) / GND | Nguồn nuôi cuộn hút relay |
| **Active Buzzer** | (+) Signal | **GPIO 12** | Còi báo động khi thẻ sai hoặc bị khóa |
| **LED Xanh lá** | Anode (+) qua trở 220Ω | **GPIO 27** | Báo mở cửa thành công |
| **LED Đỏ** | Anode (+) qua trở 220Ω | **GPIO 14** | Báo truy cập bị từ chối |

---

## 3. CÔNG NGHỆ PHẦN MỀM CHỐT
* **Firmware:** C++ trên PlatformIO, thư viện `MFRC522`, `HTTPClient`, `ArduinoJson`, `LittleFS` / `NVS`.
* **Backend API:** Node.js (LTS), TypeScript, Fastify framework (tối ưu tốc độ cao).
* **Cơ sở dữ liệu:** SQLite quản lý qua Prisma ORM (gọn nhẹ, lưu file cục bộ, dễ sao lưu và bảo vệ đồ án).
* **Desktop App:** Electron + React (Vite) + TypeScript + Ant Design + Recharts.
* **Giao tiếp:** RESTful API qua giao thức HTTP/JSON trên nền mạng Wi-Fi nội bộ.

---

## 4. QUY TẮC BẢO MẬT & MÃ TRẠNG THÁI PHẢN HỒI

Hệ thống **không cam kết "chống clone thẻ tuyệt đối"** với phần cứng MIFARE Classic 1K, mà tập trung vào **"Phát hiện và giảm thiểu truy cập bất thường"**:

| Mã phản hồi API | Quyết định Relay | Phản hồi phần cứng | Ý nghĩa nghiệp vụ |
| :--- | :---: | :--- | :--- |
| `ACCESS_GRANTED` | **MỞ (3s)** | LED Xanh sáng | Thẻ hợp lệ, còn hạn, đúng cửa |
| `CARD_NOT_FOUND` | ĐÓNG | LED Đỏ, Buzzer kêu 1 lần | Thẻ chưa từng đăng ký |
| `CARD_BLOCKED` | ĐÓNG | LED Đỏ, Buzzer kêu 2s | Thẻ bị tạm khóa (cư dân báo mất) |
| `CARD_REVOKED` | ĐÓNG | LED Đỏ, Buzzer kêu 2s | Thẻ đã thu hồi hoàn toàn |
| `ACCESS_NOT_ALLOWED`| ĐÓNG | LED Đỏ, Buzzer bíp 2 lần| Thẻ không được phép vào cửa này |
| `COUNTER_MISMATCH` | ĐÓNG | Còi hú báo động liên tục| Phát hiện counter trên thẻ nhỏ hơn counter server |
| `DEVICE_NOT_AUTHORIZED`| ĐÓNG | Báo lỗi hệ thống | ESP32 gửi sai API Key |
| `OFFLINE_ACCESS_GRANTED`| **MỞ (3s)** | LED Xanh nhấp nháy | Mất Wi-Fi nhưng thẻ có trong cache Flash |

---

## 5. BỘ KỊCH BẢN KIỂM THỬ (10 TEST CASES)
- [ ] **TC01:** Thẻ Active, có quyền tại cửa -> Relay mở, LED xanh, Log `ACCESS_GRANTED`.
- [ ] **TC02:** UID chưa đăng ký -> Cửa đóng, LED đỏ, Log `CARD_NOT_FOUND`.
- [ ] **TC03:** Thẻ bị khóa (`BLOCKED`) -> Cửa đóng, Buzzer báo động, Desktop hiện cảnh báo.
- [ ] **TC04:** Thẻ bị thu hồi (`REVOKED`) -> Cửa đóng, Log `CARD_REVOKED`.
- [ ] **TC05:** Thẻ không có quyền ở cửa này -> Cửa đóng, Log `ACCESS_NOT_ALLOWED`.
- [ ] **TC06:** Dữ liệu counter không khớp -> Cửa đóng, sinh `SecurityAlert` mức Critical.
- [ ] **TC07:** Mất Wi-Fi, quẹt thẻ đã lưu cache -> Mở cửa theo chính sách offline, lưu log vào Flash.
- [ ] **TC08:** Mất Wi-Fi, quẹt thẻ không có trong cache -> Từ chối an toàn, lưu log từ chối tạm.
- [ ] **TC09:** Wi-Fi có trở lại -> ESP32 tự động sync toàn bộ log dồn về Server.
- [ ] **TC10:** Quẹt thẻ lạ liên tục 4 lần trong 60s -> Phát hiện dò mã Brute-force, Desktop cảnh báo đỏ.

---

## 6. HƯỚNG DẪN CHẠY DỰ ÁN (GETTING STARTED)
*(Sẽ được cập nhật chi tiết mã lệnh theo tiến độ từ ngày 06/10/2026)*
