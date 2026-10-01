# CẨM NANG TOÀN DIỆN: GIỎ HÀNG MUA SẮM VÀ LỘ TRÌNH TỪNG NGÀY / TỪNG TUẦN (06/10/2026 - 30/06/2027)

> **Mục tiêu tối thượng:** Đi từ con số 0 (chưa có phần cứng, chưa có code) đến hoàn thiện **3 Dự án chuẩn công nghiệp**, xây dựng Portfolio GitHub xịn, CV 1 trang và tự tin ứng tuyển vị trí **Junior / Fresher IoT & Embedded Engineer**.

---

## PHẦN 1: GIỎ HÀNG MUA SẮM TRỌN GÓI (ALL-IN-ONE HARDWARE SHOPPING LIST)

Để tiết kiệm chi phí vận chuyển và thời gian chờ đợi, bạn nên đặt mua **trọn gói một lần** trên các sàn TMĐT (Shopee, Lazada) hoặc các cửa hàng linh kiện điện tử (Hshop, Thegioiic, Nshop...). 

Tổng ngân sách cho cả 3 đồ án chỉ dao động từ **350.000đ – 450.000đ**.

### 1.1. Bảng danh mục linh kiện cần mua ngay (Đặt trước hoặc đúng ngày 06/10)

| STT | Tên linh kiện | Phục vụ dự án | Số lượng | Giá tham khảo (VND) | Lưu ý kỹ thuật khi chọn mua |
| :---: | :--- | :---: | :---: | :---: | :--- |
| 1 | **ESP32 DevKit V1** (30 chân hoặc 38 chân) | Cả 3 Project | 1 bo | 85.000 - 110.000 | Chọn loại cổng **Type-C** hoặc **MicroUSB** có sẵn chip nạp CP2102 hoặc CH340. |
| 2 | **Cáp kết nối USB** (Type-C hoặc Micro tùy bo) | Cả 3 Project | 1 sợi | 15.000 - 25.000 | **Quan trọng:** Phải là cáp truyền dữ liệu (Data Cable), không mua cáp chỉ sạc. |
| 3 | **Module RFID RC522 13.56MHz** | Project 1 | 1 bộ | 30.000 - 45.000 | Thường bán kèm sẵn 1 thẻ từ trắng và 1 móc chìa khóa RFID xanh. |
| 4 | **Thẻ trắng / Móc khóa RFID MIFARE 1K** | Project 1 | 3 - 5 cái | 20.000 - 30.000 | Dùng để test các kịch bản: Thẻ Admin, Thẻ Cư dân, Thẻ Bị Khóa, Thẻ Lạ. |
| 5 | **Module Relay 5V 1 kênh (Low Trigger)** | Project 1 | 1 cái | 15.000 - 20.000 | Có cách ly quang (optocoupler), kích mức thấp (Low level trigger). |
| 6 | **Active Buzzer 5V** (Còi chíp báo động) | Project 1, 3 | 1 cái | 5.000 - 10.000 | Mua loại Active (chỉ cần cấp nguồn 5V là tự kêu bíp bíp, không cần tạo xung). |
| 7 | **LED đơn 5mm (Đỏ + Xanh lá) + Trở 220Ω** | Cả 3 Project | 5 cặp | 10.000 | Dùng báo trạng thái Success / Error. |
| 8 | **Testboard / Breadboard MB-102** (830 lỗ) | Cả 3 Project | 1 cái | 25.000 - 35.000 | Bo cắm mạch thử nghiệm không cần hàn chì. |
| 9 | **Dây cắm Breadboard Dupont** | Cả 3 Project | 1 tệp | 35.000 - 45.000 | Mua tệp 40 sợi Đực - Cái và 1 tệp Đực - Đực (dài 20cm). |
| 10| **Màn hình OLED 0.96 inch I2C (SSD1306)** | Project 2, 3 | 1 cái | 45.000 - 60.000 | Giao tiếp I2C (chỉ cần 4 chân: VCC, GND, SCL, SDA). Màu xanh hoặc trắng. |
| 11| **Cảm biến nhiệt độ độ ẩm BME280 hoặc DHT22**| Project 2 | 1 cái | 40.000 - 75.000 | Ưu tiên BME280 (I2C, rất chuẩn công nghiệp) hoặc DHT22 / DHT11 rẻ hơn. |
| 12| **Nút nhấn 4 chân + Chiết áp xoay B10K** | Project 3 | 2 nút, 1 biến trở | 10.000 - 15.000 | Phục vụ đọc tín hiệu ngắt (Interrupt) và tín hiệu tương tự (ADC FreeRTOS). |

> **Tổng chi phí dự kiến:** ~ **380.000 VNĐ** (Chỉ bằng 1 bữa lẩu, nhưng dùng cho suốt 9 tháng học tập).

---

## PHẦN 2: LỘ TRÌNH HÀNH ĐỘNG CHI TIẾT TỪNG NGÀY TRONG THÁNG 10/2026 (KHỞI ĐỘNG TỪ SỐ 0)

Giai đoạn quan trọng nhất để vượt qua cảm giác "mông lung" là 2 tuần đầu tiên. Hãy làm chính xác từng ngày theo danh sách dưới đây:

### 📅 TUẦN 1: CÀI ĐẶT MÔI TRƯỜNG & SẴN SÀNG PHẦN CỨNG (06/10 - 12/10)

* **Thứ Hai (06/10/2026): Mua sắm & Khởi tạo Git**
  * Sáng: Lên Shopee/Cửa hàng đặt trọn bộ linh kiện ở Phần 1 (chọn shop cùng thành phố để giao nhanh trong 1-2 ngày).
  * Chiều: Cài đặt phần mềm vào máy tính:
    1. **VS Code** (Trình soạn thảo mã nguồn chính).
    2. **Git** (Quản lý phiên bản).
    3. **Node.js (LTS version)** (Nền tảng chạy backend).
    4. **Postman** (Công cụ test API).
  * Tối: Tạo tài khoản GitHub cá nhân. Tạo repository đầu tiên tên là `iot-rfid-access-control`. Viết vài dòng giới thiệu vào `README.md`, tập `git add`, `git commit`, `git push`.

* **Thứ Ba (07/10/2026): Cấu hình IDE lập trình ESP32 & Nắm cấu trúc dự án**
  * Sáng: Mở VS Code, cài đặt extension:
    * `PlatformIO IDE` (Khuyên dùng - chuẩn công nghiệp) HOẶC cài `Arduino IDE 2.x`.
    * Cài extension `Prettier`, `ESLint` cho TypeScript.
  * Chiều: Cài đặt driver cho máy tính nhận diện ESP32 (Driver CP210x hoặc CH34x tùy theo loại bo).
  * Tối: Tạo cấu trúc Monorepo trong thư mục dự án:
    ```text
    iot-rfid-access-control/
    ├── firmware/       # Nơi chứa code C++ ESP32
    ├── server/         # Nơi chứa API Node.js TypeScript
    ├── desktop/        # Nơi chứa ứng dụng Electron React
    └── docs/           # Lưu sơ đồ, tài liệu
    ```

* **Thứ Tư (08/10/2026): Làm quen C/C++ vi điều khiển cơ bản**
  * Học cấu trúc một chương trình vi điều khiển: hàm `setup()` chạy 1 lần và hàm `loop()` lặp vô tận.
  * Học các lệnh cốt lõi: `pinMode()`, `digitalWrite()`, `digitalRead()`, `delay()`.
  * Học cách giao tiếp máy tính qua cổng nối tiếp: `Serial.begin(115200)` và `Serial.println()`.
  * Ghi chú lại tài liệu vào thư mục `docs/notes_day3.md`.

* **Thứ Năm (09/10/2026): Nhập môn TypeScript cho Backend**
  * Hiểu TypeScript khác JavaScript ở điểm nào (Static typing, Type Safety).
  * Học 4 khái niệm nền tảng: `interface`, `type`, `async/await`, `Promise`.
  * Tạo thư mục `test-ts/`, chạy thử lệnh `npm init -y`, `npm install typescript ts-node` và in ra `Hello IoT TypeScript`.

* **Thứ Sáu (10/10/2026): Nhận linh kiện & "Hello World" ESP32**
  * Nhận kiện hàng linh kiện, mở kiểm tra số lượng theo bảng check ở Phần 1.
  * Cắm dây cáp kết nối ESP32 vào cổng USB máy tính.
  * Mở PlatformIO/Arduino IDE, chọn Board `DOIT ESP32 DEVKIT V1`, chọn đúng cổng COM.
  * Nạp chương trình **Blink LED** mẫu. 
  * *Kết quả đạt được:* Đèn LED nhỏ màu xanh trên thân bo ESP32 nhấp nháy 1 giây/lần.

* **Thứ Bảy (11/10/2026): Lắp ráp mạch IO cơ bản (LED + Còi Buzzer + Relay)**
  * Cắm ESP32 lên Breadboard MB-102.
  * Nối LED đỏ (chân GPIO 14) và LED xanh (chân GPIO 27) kèm điện trở 220Ω xuống GND.
  * Nối Buzzer (chân tín hiệu vào GPIO 12).
  * Nối Relay 5V (chân IN vào GPIO 26, VCC vào 5V/VIN, GND vào GND).
  * Viết code C++: Mỗi 3 giây kêu còi bíp 1 cái, bật LED xanh, kích relay nhảy "tách" một cái; sau đó đổi sang LED đỏ.
  * *Kết quả đạt được:* Nắm vững cách xuất tín hiệu điều khiển phần cứng.

* **Chủ Nhật (12/10/2026): Giao tiếp đầu đọc RC522 & Đọc thẻ RFID đầu tiên**
  * Nối dây RC522 với ESP32 theo chuẩn SPI:
    * `SDA` -> GPIO 5 | `SCK` -> GPIO 18 | `MOSI` -> GPIO 23 | `MISO` -> GPIO 19 | `RST` -> GPIO 22 | `3.3V` -> 3V3 ESP32 | `GND` -> GND.
  * Cài thư viện `rfid` của *Miguel Balboa* (trên Arduino/PlatformIO).
  * Nạp chương trình mẫu `DumpInfo`. Mở Serial Monitor (tốc độ 115200 baud).
  * Đưa thẻ trắng và móc khóa RFID lại gần đầu đọc.
  * *Kết quả đạt được:* Màn hình Serial hiển thị mã UID dạng hex (ví dụ: `Card UID: 8A 3B 21 F0`). Quay 1 video ngắn 10 giây lưu lại kỷ niệm thành công đầu tiên!

---

### 📅 TUẦN 2: THIẾT KẾ CƠ SỞ DỮ LIỆU & LẬP TRÌNH API SERVER (13/10 - 19/10)
* **Mục tiêu tuần:** Có REST API chạy trên máy tính, có Database SQLite lưu danh sách thẻ.
* **Thứ 2 - Thứ 3:** Cài Fastify + TypeScript + Prisma. Khởi tạo database SQLite `dev.db`.
* **Thứ 4 - Thứ 5:** Tạo bảng `Resident`, `Card`, `Door`, `AccessLog`. Viết API thêm cư dân, cấp thẻ (`POST /api/cards`), khóa thẻ (`PATCH /api/cards/:id/block`).
* **Thứ 6:** Dùng Postman test toàn bộ các API, seed dữ liệu mẫu (3 thẻ: 1 thẻ Active, 1 thẻ Blocked, 1 thẻ Revoked).
* **Thứ 7 - CN:** Viết tài liệu quy ước API vào file `docs/API_DOCUMENTATION.md` và push code lên GitHub.

### 📅 TUẦN 3: KẾT NỐI ESP32 VỚI API SERVER QUA WI-FI (20/10 - 26/10)
* **Mục tiêu tuần:** Quẹt thẻ ngoài đời thực -> ESP32 gửi Wi-Fi lên Server -> Cửa mở hoặc từ chối.
* **Thứ 2 - Thứ 3:** Viết code ESP32 kết nối Wi-Fi nhà bạn (`WiFi.begin(ssid, password)`), in ra địa chỉ IP cục bộ.
* **Thứ 4 - Thứ 5:** Dùng thư viện `HTTPClient` và `ArduinoJson` trên ESP32. Khi quẹt thẻ, gửi POST request:
  `{ "deviceCode": "DOOR_01", "uid": "8A3B21F0" }` tới `http://<IP_MÁY_TÍNH>:3000/api/access/verify`.
* **Thứ 6:** Server Fastify nhận UID, tra cứu trong SQLite:
  * Nếu thẻ Active: trả về `{ "allowed": true, "reason": "ACCESS_GRANTED" }`.
  * Nếu thẻ Blocked: trả về `{ "allowed": false, "reason": "CARD_BLOCKED" }`.
* **Thứ 7 - CN:** ESP32 đọc JSON trả về:
  * `allowed == true`: Kích Relay mở 3 giây, bật LED xanh.
  * `allowed == false`: Bật còi Buzzer kêu báo động, bật LED đỏ.
  * *Milestone:* Hệ thống IoT đóng/mở cửa đầu tiên hoạt động hoàn chỉnh!

### 📅 TUẦN 4: GHI NHẬN LOG & XỬ LÝ ANOMALY (27/10 - 02/11)
* **Mục tiêu tuần:** Mọi lượt quẹt đều lưu log; phát hiện thẻ lạ quẹt liên tục.
* **Thứ 2 - Thứ 4:** Server tự động ghi dữ liệu vào bảng `AccessLog` (thời gian, cửa nào, UID, kết quả).
* **Thứ 5 - Thứ 6:** Viết logic phát hiện bất thường: Nếu 1 cửa có thẻ lạ quét lỗi 3 lần trong vòng 60 giây -> Ghi một bản ghi vào bảng `SecurityAlert` (Loại: `BRUTE_FORCE_ATTEMPT`).
* **Thứ 7 - CN:** Tổng hợp toàn bộ code Tháng 10, viết báo cáo tháng đầu tiên, quay video demo tổng thể.

---

## PHẦN 3: LỘ TRÌNH 9 THÁNG CHI TIẾT TỪNG TUẦN (10/2026 - 06/2027)

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ DỰ ÁN 1: RFID Access Control & Desktop Management (Tháng 10/2026 - Tháng 02/2027)      │
│ → Kỹ năng: C++ Firmware, Fastify, Prisma, SQLite, Electron, React, Offline Cache       │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ DỰ ÁN 2: MQTT Industrial Device Monitoring Agent (Tháng 03/2027 - Tháng 04/2027)       │
│ → Kỹ năng: MQTT/MQTTS, Mosquitto Broker, LittleFS Queue, Store-and-Forward, Docker, OTA│
├────────────────────────────────────────────────────────────────────────────────────────┤
│ DỰ ÁN 3: Multi-tasking FreeRTOS Sensor Hub (Tháng 05/2027)                             │
│ → Kỹ năng: ESP-IDF / FreeRTOS, Task Scheduling, Queue, Mutex, Watchdog Timer           │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ GIAI ĐOẠN VỀ ĐÍCH: Portfolio, GitHub Polish, CV & Phỏng Vấn (Tháng 06/2027)            │
│ → Kỹ năng: Kỹ năng mềm, Pitching 3 phút, Trả lời kiến trúc, Hoàn thiện CV              │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### 🔹 THÁNG 11/2026: XÂY DỰNG TOÀN DIỆN PHẦN MỀM DESKTOP (ELECTRON + REACT)
* **Tuần 5 (03/11 - 09/11):** Khởi tạo project Electron + React + Vite + TypeScript. Cấu hình IPC Main-Renderer. Dựng khung giao diện Layout chuẩn với Sidebar và Header (dùng Ant Design hoặc MUI).
* **Tuần 6 (10/11 - 16/11):** Xây dựng trang **Dashboard**: Card hiển thị tổng số cư dân, số thẻ đang hoạt động, số lượt quẹt thành công/thất bại trong ngày, biểu đồ số lượt ra vào theo giờ (Recharts).
* **Tuần 7 (17/11 - 23/11):** Xây dựng màn hình **Quản lý Cư dân & Thẻ**: Bảng danh sách phân trang, tìm kiếm theo tên/phòng, modal thêm cư dân mới, nút gạt (Switch) Khóa / Mở thẻ tức thì.
* **Tuần 8 (24/11 - 30/11):** Xây dựng màn hình **Lịch sử truy cập & Cảnh báo an ninh**: Bộ lọc tìm kiếm log theo ngày/kết quả, hộp cảnh báo đỏ nhấp nháy khi có sự kiện bất thường.

### 🔹 THÁNG 12/2026: BẢO MẬT & XỬ LÝ OFFLINE CACHE (NVS / FLASH)
* **Tuần 9 (01/12 - 07/12):** Bảo mật hệ thống: Mã hóa mật khẩu Admin bằng `bcrypt`, cấp phát `JWT` đăng nhập trên Desktop App; Cấp phát `Device Token (API Key)` cho từng ESP32 để chặn thiết bị giả mạo.
* **Tuần 10 (08/12 - 14/12):** Lập trình Offline Cache trên ESP32: Lưu danh sách 50 UID hợp lệ vào bộ nhớ Flash (NVS / LittleFS). Khi mất kết nối Wi-Fi, kiểm tra cục bộ để vẫn cho phép cư dân mở cửa (`OFFLINE_ACCESS_GRANTED`).
* **Tuần 11 (15/12 - 21/12):** Hàng đợi log ngoại tuyến: Khi mất mạng, lưu lượt quẹt vào bộ nhớ tạm. Khi mạng khôi phục, tự động đồng bộ (sync) toàn bộ log dồn về Server.
* **Tuần 12 (22/12 - 28/12):** Chạy kiểm thử ma trận 10 kịch bản test (TC01 - TC10). Đo đạc thời gian phản hồi (Latency) khi quẹt thẻ online vs offline.

### 🔹 THÁNG 01/2027: HOÀN THIỆN ĐỒ ÁN 1 & ĐÓNG GÓI SẢN PHẨM
* **Tuần 13 (29/12 - 04/01):** Đóng gói phần mềm Desktop ra file cài đặt `.exe` bằng Electron Builder. Cấu hình khởi động cùng Windows.
* **Tuần 14 (05/01 - 11/01):** Viết file `README.md` chuyên nghiệp cho Dự án 1: Có sơ đồ kiến trúc hệ thống (Mermaid), bảng mã lỗi, ảnh chụp mạch phần cứng, ảnh chụp màn hình ứng dụng desktop.
* **Tuần 15 (12/01 - 18/01):** Quay video demo 3 phút: Giới thiệu hệ thống, demo quẹt thẻ mở cửa, demo khóa thẻ trên desktop và quẹt lại bị từ chối, demo rút dây mạng vẫn mở cửa offline. Đăng lên YouTube ở chế độ Unlisted.
* **Tuần 16 (19/01 - 25/01):** Dọn dẹp code, xóa bỏ token/mật khẩu nhạy cảm, tạo release v1.0.0 trên GitHub. Nghỉ Tết / Tự thưởng bản thân vì đã hoàn thành 1/3 chặng đường xuất sắc!

---

### 🔹 THÁNG 02/2027 & THÁNG 03/2027: DỰ ÁN 2 - MQTT DEVICE MONITORING AGENT
> **Mục tiêu:** Nắm vững giao thức tiêu chuẩn công nghiệp (MQTT), cảm biến môi trường I2C, cơ chế lưu trữ bền vững (Store-and-Forward) và giám sát từ xa.

* **Tuần 17 (16/02 - 22/02): Nhập môn MQTT & Cấu hình Broker**
  * Hiểu cơ chế Publish / Subscribe, Topic, Quality of Service (QoS 0, 1, 2), Retained Message, Last Will and Testament (LWT).
  * Cài đặt Mosquitto MQTT Broker trên máy tính hoặc chạy qua Docker. Dùng MQTTX để test gửi/nhận message.
* **Tuần 18 (23/02 - 01/03): ESP32 + Cảm biến BME280/DHT22 + OLED**
  * Nối cảm biến nhiệt độ, độ ẩm và màn hình OLED qua bus I2C.
  * Đọc dữ liệu môi trường mỗi 2 giây, hiển thị trực quan lên màn hình OLED.
* **Tuần 19 (02/03 - 08/03): Gửi Telemetry & Nhận Command qua MQTT**
  * ESP32 kết nối Wi-Fi, publish dữ liệu định dạng JSON lên topic `devices/ESP32_01/telemetry`.
  * Subscribe topic `devices/ESP32_01/commands` để nhận lệnh từ xa (ví dụ: bật tắt LED, khởi động lại thiết bị).
  * Cấu hình Last Will để server nhận biết ngay lập tức khi thiết bị mất nguồn đột ngột.
* **Tuần 20 (09/03 - 15/03): Độ tin cậy cao - Store and Forward (LittleFS Queue)**
  * Khi mất kết nối Wi-Fi hoặc mất kết nối tới MQTT Broker: ESP32 không vứt bỏ dữ liệu mà ghi gói tin JSON vào file hàng đợi trong bộ nhớ Flash (LittleFS).
  * Khi có kết nối lại: Đọc tuần tự các gói tin cũ gửi bù lên server kèm mốc thời gian gốc.
* **Tuần 21 (16/03 - 22/03): Dashboard Giám sát & Nâng cấp OTA**
  * Dựng Dashboard theo dõi thông số nhiệt độ/độ ẩm thời gian thực bằng Grafana hoặc Node-RED (chạy Docker Compose).
  * Lập trình tính năng **OTA (Over-The-Air Update)**: Nạp firmware mới cho ESP32 qua Wi-Fi mà không cần cắm dây cáp.
* **Tuần 22 (23/03 - 29/03): Hoàn thiện Project 2**
  * Viết README, vẽ sơ đồ luồng dữ liệu Store-and-Forward, quay video demo tính năng rút mạng vẫn lưu log.

---

### 🔹 THÁNG 04/2027 & THÁNG 05/2027: DỰ ÁN 3 - EMBEDDED SYSTEM VỚI FREERTOS / ESP-IDF
> **Mục tiêu:** Khẳng định năng lực Firmware Engineer thực thụ; hiểu sâu về bộ nhớ, đa nhiệm vi điều khiển và xử lý lỗi phần cứng.

* **Tuần 23 - 24 (30/03 - 12/04): Chuyển dịch từ Arduino sang ESP-IDF & FreeRTOS cơ bản**
  * Hiểu khái niệm RTOS (Real-Time Operating System), tại sao Arduino `delay()` là tối kỵ trong công nghiệp.
  * Cài đặt ESP-IDF extension trên VS Code. Tạo project FreeRTOS đầu tiên.
  * Học cách tạo Task độc lập: `xTaskCreatePinnedToCore()` phân bổ chạy trên Core 0 và Core 1 của ESP32.
* **Tuần 25 - 26 (13/04 - 26/04): Đồng bộ hóa & Giao tiếp giữa các Task**
  * **FreeRTOS Queue:** Tạo Task 1 chuyên đọc cảm biến, đẩy dữ liệu vào Queue; Task 2 lấy dữ liệu từ Queue hiển thị lên màn hình.
  * **Mutex / Semaphore:** Dùng Mutex để bảo vệ tài nguyên dùng chung (tránh xung đột khi 2 task cùng in ra Serial hoặc cùng ghi I2C).
  * **Event Groups:** Đồng bộ trạng thái: chỉ bắt đầu gửi tin khi cả Wi-Fi và cảm biến đã sẵn sàng.
* **Tuần 27 - 28 (27/04 - 10/05): Giám sát hệ thống & Tự phục hồi lỗi (Watchdog Timer)**
  * Cấu hình **Task Watchdog Timer (TWDT)**: Nếu một task bị treo (deadlock), watchdog tự động phát hiện và reset vi điều khiển.
  * Quản lý bộ nhớ: Giám sát Stack High Water Mark để phát hiện tràn bộ nhớ (Stack Overflow).
* **Tuần 29 (11/05 - 17/05): Hoàn thiện Project 3 (FreeRTOS Sensor Hub)**
  * Đóng gói code chuẩn mực C/C++, có header guard, struct rõ ràng, README giải thích chi tiết kiến trúc đa tác vụ.

---

### 🔹 THÁNG 06/2027: XÂY DỰNG PORTFOLIO, CV & CHINH PHỤC NHÀ TUYỂN DỤNG
* **Tuần 30 (18/05 - 24/05): Đánh bóng GitHub Profile**
  * Ghim (Pin) 3 repository lên trang cá nhân GitHub:
    1. `iot-rfid-access-control` (Full-stack IoT System).
    2. `esp32-mqtt-monitoring-agent` (Reliable Industrial IoT).
    3. `esp32-freertos-sensor-hub` (Advanced Firmware Engineering).
  * Viết file `README.md` cá nhân có gắn badge công nghệ, link LinkedIn, mô tả định hướng nghề nghiệp.
* **Tuần 31 (25/05 - 31/05): Viết CV chuẩn kỹ thuật 1 trang**
  * Định dạng PDF 1 trang duy nhất, không dùng icon màu mè hoa lá.
  * Nêu bật 3 project: Mỗi project có 3 gạch đầu dòng (Vấn đề giải quyết - Công nghệ sử dụng - Kết quả định lượng được).
  * Đính kèm link GitHub và link Video demo của từng project.
* **Tuần 32 (01/06 - 07/06): Luyện tập phỏng vấn kỹ thuật (Technical Interview Prep)**
  * Tự trả lời trôi chảy các câu hỏi kinh điển:
    * *Tại sao chọn HTTP cho Project 1 mà lại chọn MQTT cho Project 2?*
    * *Cơ chế Store-and-Forward hoạt động thế nào khi đầy bộ nhớ Flash?*
    * *Sự khác nhau giữa Mutex và Binary Semaphore trong FreeRTOS là gì?*
    * *Tại sao không nên tin tưởng tuyệt đối vào UID của thẻ RFID MIFARE 1K?*
    * *Cách bạn debug khi ESP32 bị crash reset liên tục (Guru Meditation Error)?*
* **Tuần 33 - 34 (08/06 - 30/06): Rải CV & Phỏng vấn ứng tuyển**
  * Nộp hồ sơ vào các vị trí: *IoT Embedded Engineer, Firmware Engineer, IoT Platform Developer, Smart Home / Smart Building Engineer*.
  * Tự tin bước vào phỏng vấn với vị thế một ứng viên có sản phẩm thật, chạy thật, hiểu sâu bản chất kỹ thuật.

---

## PHẦN 4: 5 NGUYÊN TẮC VÀNG ĐỂ ĐẢM BẢO 100% THÀNH CÔNG

1. **Nguyên tắc "Tự tay gõ lại":** Tuyệt đối không copy paste mù quáng code do AI sinh ra. Hãy yêu cầu AI giải thích từng dòng, tự tay gõ lại vào bàn phím để não bộ ghi nhớ syntax và luồng logic.
2. **Nguyên tắc "Commit hàng ngày":** Dù chỉ sửa 1 dòng comment hay đọc xong 1 chương tài liệu, hãy commit lên GitHub. Biểu đồ commit màu xanh lá cây đều đặn suốt 9 tháng là minh chứng thuyết phục nhất với mọi nhà tuyển dụng.
3. **Nguyên tắc "Tách biệt và cô lập lỗi":** Khi gặp lỗi (ví dụ quẹt thẻ không mở cửa), hãy kiểm tra theo từng lớp: Phần cứng có điện không? -> RC522 có đọc ra mã không? -> ESP32 có vào Wi-Fi không? -> API Server có nhận request không? -> Database có trả về kết quả không? Không bao giờ đoán mò!
4. **Bảo mật thông tin:** Không bao giờ đưa mật khẩu Wi-Fi nhà bạn, mật khẩu database hoặc secret token lên GitHub công khai. Luôn dùng file `.env` hoặc file cấu hình mẫu `config.example.h`.
5. **Giữ gìn năng lượng:** Mỗi ngày chỉ cần dành **2 - 3 tiếng tập trung cao độ**, không cần thức trắng đêm. Sự kiên trì tích lũy qua 270 ngày sẽ tạo nên một bước nhảy vọt phi thường cho sự nghiệp của bạn.
