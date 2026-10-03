# 📘 LỘ TRÌNH THỰC HIỆN CHI TIẾT — DỰ ÁN 1
## Hệ thống kiểm soát cửa RFID & IoT tích hợp Desktop App
### ESP32 + Fastify + Electron

> **Tác giả:** NguyenHoangUy1305  
> **Repository:** `NguyenHoangUy1305/iot-rfid-access-control`  
> **Thời lượng:** 11 tuần triển khai chính + 6 ngày nghiệm thu/đóng gói  
> **Thời gian dự kiến:** 01/11/2026 – 15/01/2027  
> **Mục tiêu:** Hoàn thành MVP chạy ổn định trước; chỉ triển khai tính năng nâng cao sau khi MVP đã được nghiệm thu.  
> **Nguyên tắc:** Một tuần = một sản phẩm con chạy được + commit Git + ảnh/video + ghi chép kỹ thuật.

---

## Mục lục

1. [Phạm vi MVP và hướng nâng cao](#1-phạm-vi-mvp-và-hướng-nâng-cao)
2. [Lộ trình 11 tuần](#2-lộ-trình-11-tuần)
3. [Checklist nghiệm thu v1.0.0](#3-checklist-nghiệm-thu-v100)
4. [Bẫy phần cứng và phòng tránh](#4-bẫy-phần-cứng-và-phòng-tránh)
5. [Mã lỗi hệ thống](#5-mã-lỗi-hệ-thống)
6. [Câu hỏi phỏng vấn và bảo vệ kỹ thuật](#6-câu-hỏi-phỏng-vấn-và-bảo-vệ-kỹ-thuật)

---

## 1. Phạm vi MVP và hướng nâng cao

### MVP bắt buộc (100% Hoàn thành để nghiệm thu)

```text
ESP32 + RC522 đọc và chuẩn hóa UID
→ Server-side authorization qua Fastify API
→ Relay GPIO 26 + LED xanh GPIO 27 + LED đỏ GPIO 33 + Buzzer GPIO 25
→ Fastify + TypeScript + Prisma + SQLite
→ X-Device-Token xác thực thiết bị (kèm nonce/timestamp chống Replay)
→ Card lifecycle: ACTIVE / BLOCKED / REVOKED / EXPIRED
→ Quyền truy cập theo cửa (CardDoorPermission)
→ Access logs + rule-based anomaly alerts
→ Electron + React desktop app: xem log, CRUD cư dân/thẻ, cấp/khóa/thu hồi thẻ
→ Offline whitelist cache (NVS) + offline event queue (LittleFS) + idempotent sync
→ Test report, README, video demo, đóng gói Windows installer (.exe)
```

### Tính năng nâng cao (Chỉ triển khai sau khi MVP hoàn tất)

```text
- WebSocket live event stream (MVP dùng REST polling 3–5 giây ổn định)
- Export báo cáo log ra CSV/Excel
- Gửi thông báo cảnh báo qua Telegram Bot
- Counter transaction trong data block thẻ MIFARE Classic
- Remote unlock có xác nhận Admin hai bước, expiry và audit log
- Reed switch phát hiện cửa mở quá lâu (door-held-open)
- Tamper switch phát hiện tháo dỡ thiết bị
- Multi-door/multi-ESP32, PostgreSQL production, Docker deployment
- MQTT/OTA firmware upgrade/HTTPS production deployment
```

### Giới hạn bảo mật RFID (Tính trung thực học thuật)

```text
- UID chỉ là định danh tra cứu, không phải bí mật.
- MIFARE Classic và đầu đọc RC522 không được tuyên bố chống clone tuyệt đối.
- Bảo mật MVP dựa trên cơ chế phòng vệ nhiều lớp: device token, trạng thái thẻ
  phía server, quyền theo cửa, audit log và phát hiện bất thường.
- Counter trên thẻ là tính năng nâng cao, hỗ trợ phát hiện dữ liệu cũ/rollback
  trong một số tình huống; không chứng minh tuyệt đối thẻ là thẻ gốc.
```

---

## 2. Lộ trình 11 tuần

### Tuần 1 — 01/11–07/11: Môi trường và I/O cơ bản

**Kiến thức:** Git, PlatformIO, Serial Monitor, GPIO, nguồn 3.3 V/5 V/GND, boot-strapping pins ESP32.

**Thực hiện:**
- Cài đặt VS Code, Git, PlatformIO IDE, Node.js LTS, Postman.
- Khởi tạo cấu trúc repository: `firmware/`, `server/`, `desktop/`, `docs/`.
- Test LED on-board và Serial Monitor.
- Đấu breadboard các linh kiện ngoại vi an toàn:
  - LED xanh: GPIO 27, nối tiếp trở hạn dòng 220–330 Ω.
  - LED đỏ: GPIO 33, nối tiếp trở hạn dòng 220–330 Ω.
  - Active buzzer: GPIO 25; dùng transistor driver nếu dòng còi cao.
  - Relay input: GPIO 26; chỉ test tiếng click relay, chưa nối tải khóa 12V.
  - *Lưu ý khởi tạo Firmware:* Luôn đặt `digitalWrite(26, HIGH)` trước khi `pinMode(26, OUTPUT)` để triệt tiêu hiện tượng giật xung đóng relay Active-LOW khi bật nguồn.
- Tạo `docs/learning-log.md` ghi chép tiến độ hàng ngày.
- Tạo `.gitignore` từ đầu; tuyệt đối không commit password Wi-Fi, secret key hay token.

**Nghiệm thu:** Flash firmware nạp ổn định; Serial log hiển thị đúng; LED, buzzer và logic relay hoạt động chính xác.  
**Commit:** `week-01-hardware-io`.

---

### Tuần 2 — 08/11–14/11: SPI RC522 và UID

**Kiến thức:** Giao thức SPI (SCK, MOSI, MISO, SS), UID ISO/IEC 14443A, thư viện MFRC522, chuẩn hóa dữ liệu.

**Đấu nối RC522 (Chuẩn SPI an toàn):**

| RC522 | ESP32 | Ghi chú |
|---|---:|---|
| SS/SDA | GPIO 21 | SPI chip select (tránh GPIO 5 strapping) |
| SCK | GPIO 18 | SPI clock |
| MOSI | GPIO 23 | SPI MOSI |
| MISO | GPIO 19 | SPI MISO |
| RST | GPIO 22 | Reset RC522 |
| VCC | 3V3 | ⚠️ CẤM CẤP 5V (Hỏng IC MFRC522) |
| GND | GND | Mass chung toàn mạch |

**Thực hiện:**
- Cài đặt thư viện chuẩn `miguelbalboa/MFRC522`.
- Chạy ví dụ `DumpInfo` để xác nhận reader và thẻ RFID hoạt động tốt.
- Viết hàm chuẩn hóa chuỗi UID: chuyển `A4:3C:9B:10` thành `a43c9b10` (viết thường, loại bỏ dấu hai chấm).
- Quét tối thiểu 3 thẻ khác nhau; thực hiện 30 lần lặp đọc cho mỗi thẻ.
- Thêm thuật toán Card Repeat Suppression: bỏ qua cùng một UID trong vòng 2 giây để tránh gửi trùng lặp.
- Chụp ảnh mạch và quay clip đọc UID hiển thị trên Serial Monitor.

**Nghiệm thu:** Reader đọc UID ổn định ở cự ly thực tế (1–4 cm); không bị treo bus SPI; không có lỗi giao tiếp lặp lại.  
**Commit:** `week-02-rfid-read`.

---

### Tuần 3 — 15/11–21/11: Local Access Control và Tải cảm 12V

**Kiến thức:** Finite State Machine (FSM), debounce thời gian thực bằng `millis()`, điều khiển Relay, đặc tính tải cảm DC, hiện tượng Back-EMF và Flyback Diode.

**Thực hiện:**
- Khai báo mảng whitelist local tạm thời gồm 2 UID hợp lệ trong firmware.
- Logic quẹt thẻ:
  - UID hợp lệ: kích relay mở 3 giây, bật LED xanh, phát 2 tiếng beep ngắn.
  - UID không hợp lệ: bật LED đỏ 2 giây, phát 1 tiếng beep dài, giữ khóa.
- Đấu nút Exit Button tại GPIO 32, cấu hình `INPUT_PULLUP`, nút nối GND. Mắc song song tụ gốm 100 nF để lọc dội xung phần cứng (RC Debounce).
- Viết hàm debounce nút bấm non-blocking với ngưỡng thời gian 50 ms.
- Chạy thử nghiệm 50 lần quẹt valid, invalid và bấm exit button.
- Khi mạch logic hoàn toàn ổn định, tiến hành đấu nối tải khóa Solenoid 12V:
  - Dùng Adapter 12V DC riêng đủ dòng (tối thiểu 2A).
  - Relay COM nhận cực dương 12V; Relay NO nối vào cực dương khóa Solenoid (Fail-Secure).
  - Cực âm khóa Solenoid nối về cực âm (GND) adapter 12V.
  - Diode Flyback 1N4007 mắc song song ngược cực ngay tại 2 đầu dây Solenoid: Cathode (vạch trắng) nối cực dương, Anode nối cực âm.
- Đo thử nghiệm: Đảm bảo ESP32 không bị reset hay đơ do xung áp ngược khi relay đóng/ngắt.

**Nghiệm thu:** Mở/khóa cửa đúng logic; nút Exit hoạt động tức thì; ESP32 chạy ổn định không brownout/reset khi khóa đóng mở.  
**Commit:** `week-03-local-access`.

---

### Tuần 4 — 22/11–28/11: Wi-Fi, NTP và Kiến trúc Firmware

**Kiến thức:** Wi-Fi Station (STA), cơ chế Reconnect non-blocking, đồng bộ thời gian SNTP, cấu trúc module hướng đối tượng, FSM.

**Thực hiện:**
- Kết nối Wi-Fi bằng `WiFi.h`; log địa chỉ IP, RSSI và trạng thái kết nối.
- Thiết lập đồng bộ thời gian NTP: `configTime(7*3600, 0, "pool.ntp.org")` (Múi giờ GMT+7). Đặt cờ `timeSynced` theo dõi trạng thái đồng bộ giờ.
- Xây dựng cơ chế tự động kết nối lại Wi-Fi non-blocking mỗi 5 giây; tuyệt đối không dùng vòng lặp `while` chặn main loop.
- Tái cấu trúc firmware thành các module độc lập:
  - `RfidReader`: Quản lý SPI, đọc và chuẩn hóa UID.
  - `DoorController`: Điều khiển Relay, Solenoid, LED, Buzzer, FSM trạng thái cửa.
  - `NetworkManager`: Quản lý Wi-Fi, NTP và HTTP client.
  - `AppConfig`: Lưu trữ cấu hình thiết bị.
  - `OfflineStore`: Quản lý NVS cache và LittleFS queue.
- Định nghĩa các trạng thái cửa (Door States): `LOCKED`, `VERIFYING`, `UNLOCKED`, `DENIED`, `OFFLINE_CHECK`.
- Kiểm thử bật/tắt Wi-Fi router: nút Exit vẫn mở cửa bình thường; ESP32 tự động reconnect khi có mạng lại.
- Chạy burn-in test liên tục 4–8 giờ, ghi lại heap memory đầu/cuối và reset count.

**Nghiệm thu:** Không xảy ra rò rỉ bộ nhớ (memory leak); Wi-Fi tự phục hồi mượt mà; thời gian NTP được đồng bộ chính xác.  
**Commit:** `week-04-wifi-firmware-architecture`.

---

### Tuần 5 — 29/11–05/12: Backend Foundation (Fastify + Prisma + SQLite)

**Kiến thức:** Node.js, TypeScript, Fastify Framework, REST API, Zod schema validation, ORM Prisma, SQLite Database.

**Thực hiện:**
- Khởi tạo thư mục `server/`: Cấu hình Fastify, TypeScript, Prisma, SQLite, Zod, dotenv.
- Viết endpoint kiểm tra sức khỏe hệ thống: `GET /api/health` trả về trạng thái và server timestamp.
- Thiết kế mô hình cơ sở dữ liệu Prisma (`schema.prisma`):
  - `User`: Quản trị viên và nhân viên bảo vệ (`ADMIN`, `GUARD`).
  - `Resident`: Thông tin cư dân (họ tên, căn hộ, số điện thoại).
  - `Card`: Quản lý thẻ RFID (uid, status: ACTIVE/BLOCKED/REVOKED/EXPIRED, residentId).
  - `Door`: Thông tin cửa kiểm soát (deviceCode, name, location, active).
  - `Device`: Thiết bị đầu cuối (deviceCode, tokenHash, active, lastSeenAt).
  - `CardDoorPermission`: Phân quyền thẻ theo cửa (cardId, doorId, isAllowed, validFrom, validUntil).
  - `AccessLog`: Lịch sử quẹt thẻ (eventId, receivedAt, deviceEventAt, result, reasonCode).
  - `SecurityAlert`: Cảnh báo an ninh (ruleCode, severity, status, createdAt, resolvedAt).
- Thiết lập bảng `Device` chỉ lưu SHA-256 hash của raw token (`tokenHash`); raw token chỉ sinh ra và hiển thị 1 lần duy nhất khi tạo mới thiết bị.
- Thực hiện Prisma Migration và viết script Seed dữ liệu mẫu: 1 Admin, 3 Residents, 5 Cards, 1 Door, 1 Device.
- Chuẩn hóa định dạng lỗi trả về từ API: `{ success: false, errorCode, message }`.
- Tạo file mẫu `server/.env.example`; file cấu hình `.env` bắt buộc nằm trong `.gitignore`.

**Nghiệm thu:** Database migration và seed chạy trơn tru; `GET /api/health` trả HTTP 200; quan hệ phân quyền thẻ theo cửa được thiết lập chuẩn xác.  
**Commit:** `week-05-backend-foundation`.

---

### Tuần 6 — 06/12–12/12: Business Access API và Device Token

**Kiến thức:** HTTP Header Authentication, hashing token bằng SHA-256, kiểm tra chống tấn công Replay (timestamp window), status code, phân quyền truy cập.

**Thực hiện:**
- Xây dựng endpoint xác thực quẹt thẻ: `POST /api/device/access/verify`.
- Bắt buộc header `X-Device-Token`; backend băm token nhận được và so khớp với `tokenHash` trong database.
- Quy trình kiểm tra nghiệp vụ trên server:
  1. Thiết bị `Device` có tồn tại và đang kích hoạt không?
  2. Thẻ `Card` có tồn tại trong hệ sinh thái không?
  3. Trạng thái thẻ: có phải `ACTIVE` không? (Nếu BLOCKED/REVOKED/EXPIRED -> từ chối ngay).
  4. Quyền truy cập: thẻ có được cấp quyền mở tại `Door` này trong khung giờ hiện tại không?
- Trả về mã lý do chuẩn (`reasonCode`): `ACCESS_GRANTED`, `CARD_NOT_FOUND`, `CARD_BLOCKED`, `CARD_REVOKED`, `CARD_EXPIRED`, `ACCESS_DENIED`, `DEVICE_UNAUTHORIZED`.
- Tự động ghi bản ghi `AccessLog` cho mọi lượt quẹt hợp lệ về mặt thiết bị. Server gán mốc thời gian chính thức `receivedAt`.
- Kích hoạt cảnh báo an ninh (`SecurityAlert`) ngay lập tức nếu phát hiện thẻ bị khóa (`BLOCKED`) hoặc thẻ bị thu hồi (`REVOKED`) cố tình quẹt.
- Xây dựng bộ Postman Test Suite tự động (TC01 đến TC06).

**Nghiệm thu:** 100% test case trong Postman trả về đúng `allowed` và `reasonCode`. Đo benchmark 30 request nội bộ LAN, báo cáo median/avg/min/max latency.  
**Commit:** `week-06-access-api`.

---

### Tuần 7 — 13/12–19/12: ESP32 Tích hợp API và Đo kiểm Latency

**Kiến thức:** Thư viện `HTTPClient`, phân tích JSON bằng `ArduinoJson`, xử lý timeout, điều khiển chấp hành dựa trên kết quả API.

**Thực hiện:**
- ESP32 đóng gói request gửi tới backend: `deviceCode`, `uid`, `deviceEventAt`; kèm header `X-Device-Token`.
- Phân tích payload JSON phản hồi từ Fastify API:
  - `allowed == true`: kích relay GPIO 26 mở trong `unlockDurationMs` (3000 ms), bật LED xanh GPIO 27, phát 2 tiếng beep ngắn.
  - `allowed == false`: bật LED đỏ GPIO 33, phát 1 tiếng beep dài cảnh báo, giữ khóa.
- Xử lý khi mất mạng/API timeout: báo hiệu đèn đỏ nháy, ghi log lỗi kết nối; chưa cấp quyền offline nếu cache chưa sẵn sàng.
- In log chi tiết qua Serial: UID, HTTP Status Code, reasonCode, độ trễ truy cập (Round-Trip Time).
- Thử nghiệm thực tế: Đổi trạng thái thẻ trên database từ `ACTIVE` sang `BLOCKED`, quẹt lại thẻ trên ESP32 để kiểm chứng phản ứng thời gian thực.
- Đo kiểm Round-Trip Time ít nhất 30 lần trong mạng LAN, thống kê số liệu định lượng (min, max, median, average).

**Nghiệm thu:** Trạng thái trên server lập tức có hiệu lực ở lần quẹt tiếp theo; có bảng số liệu đo độ trễ thực tế.  
**Commit:** `week-07-esp32-api-integration`.

---

### Tuần 8 — 20/12–26/12: Electron Desktop MVP

**Kiến thức:** Kiến trúc Electron (Main Process, Preload Script, Renderer Process), Context Isolation, React, REST client.

**Thực hiện:**
- Khởi tạo dự án Desktop bằng Vite + Electron + React + TypeScript.
- Cấu hình bảo mật nghiêm ngặt trong `main.ts`:
  - `contextIsolation: true`
  - `nodeIntegration: false`
- Viết `preload.ts` chỉ expose các API an toàn qua `contextBridge`. Tuyệt đối không để lộ đối tượng Node.js nguyên bản cho Renderer.
- Thiết kế giao diện:
  - Màn hình đăng nhập dành cho Quản trị viên (Admin).
  - Trang Dashboard thống kê: tổng cư dân, tổng thẻ kích hoạt, lượt quẹt trong ngày, tỉ lệ chấp thuận/từ chối.
  - Bảng hiển thị nhật ký truy cập (Live Access Logs) tự động cập nhật qua cơ chế REST Polling (chu kỳ 3–5 giây).
  - Trang danh sách Cư dân (Residents) và Thẻ (Cards).

**Nghiệm thu:** Ứng dụng Desktop chạy mượt mà, đọc dữ liệu thời gian thực từ Fastify API; tuân thủ chuẩn bảo mật Electron; không bị cảnh báo console.  
**Commit:** `week-08-electron-mvp`.

---

### Tuần 9 — 27/12–02/01: CRUD, RBAC và Anomaly Alerts

**Kiến thức:** Mã hóa mật khẩu bằng bcrypt, phân quyền vai trò (RBAC), kiểm toán hệ thống (Audit Log), thuật toán phát hiện bất thường cửa sổ trượt (Sliding-window rule).

**Thực hiện:**
- Phân quyền người dùng: `ADMIN` (toàn quyền) và `GUARD` (chỉ xem log và xử lý alert).
- Xây dựng giao diện CRUD Cư dân và Thẻ RFID trên Desktop.
- Quy trình nạp thẻ (Card Enrollment): Nhập mã UID thủ công hoặc tự động bắt UID từ lần quẹt thẻ chưa gán gần nhất.
- Thao tác nhanh vòng đời thẻ: Kích hoạt (ACTIVE), Khóa tạm thời (BLOCKED), Thu hồi vĩnh viễn (REVOKED).
- Bộ lọc nhật ký nâng cao: Lọc theo khoảng ngày giờ, theo cư dân, theo cửa, theo trạng thái GRANTED/DENIED và theo reasonCode.
- Triển khai động cơ Anomaly Detection (Luật phát hiện tấn công):

| Quy tắc | Điều kiện kích hoạt | Cấp độ cảnh báo |
|---|---|---|
| `RULE_01` | 3 lần quét `CARD_NOT_FOUND` trong 60 giây tại cùng một cửa | Cảnh báo dò thẻ (Warning) |
| `RULE_02` | 5 lần bị `DENIED` liên tiếp trong 60 giây tại cùng một cửa | Báo động đột nhập (High Alert) |
| `RULE_03` | Quẹt thẻ có trạng thái `BLOCKED` hoặc `REVOKED` | Cảnh báo an ninh (Security Alert) |
| `RULE_04` | Token thiết bị gửi sai liên tiếp 3 lần trong 5 phút | Cảnh báo can thiệp thiết bị |

- Thiết kế trang Quản lý Cảnh báo (Alerts Management): hiển thị trạng thái `OPEN` / `RESOLVED`, cho phép bảo vệ bấm xác nhận xử lý và lưu audit vết.

**Nghiệm thu:** Quản lý vòng đời thẻ chính xác 100%; tự động sinh cảnh báo an ninh đúng theo các kịch bản thử nghiệm.  
**Commit:** `week-09-rbac-alerts`.

---

### Tuần 10 — 03/01–09/01: Offline Cache và Idempotent Log Sync

**Kiến thức:** Bộ nhớ phi bốc hơi NVS (Preferences), hệ thống tệp LittleFS, cấu trúc hàng đợi Queue, tính lũy kế (Idempotency), chiến lược Retry với Exponential Backoff.

**Thực hiện:**
- Xây dựng API tải danh sách trắng dự phòng: `GET /api/device/whitelist/cache` trả về danh sách thẻ `ACTIVE` được phép mở cửa, kèm theo version, `updatedAt` và mã kiểm tra tính toàn vẹn (CRC32).
- Phân định lưu trữ trên ESP32:
  - **NVS (Preferences):** Chỉ lưu cấu hình thiết bị và Whitelist Cache (mỗi bản ghi gồm UID 4-7 bytes + cờ quyền). Tuyệt đối không ghi log vào NVS để tránh chai hỏng bộ nhớ flash.
  - **LittleFS:** Lưu hàng đợi sự kiện ngoại tuyến (Offline Event Queue).
- Chính sách xử lý ngoại tuyến (Offline Policy):
  - Khi Wi-Fi hoặc API timeout: chuyển sang trạng thái `OFFLINE_CHECK`.
  - Nếu UID quẹt nằm trong NVS Whitelist Cache: cho phép mở cửa (Offline Grant), nháy LED xanh, đóng gói sự kiện ghi vào LittleFS.
  - Nếu UID không nằm trong Cache: từ chối (Offline Deny), nháy LED đỏ, ghi log từ chối vào LittleFS.
  - Khóa thẻ mới, phân quyền mới và mở cửa từ xa không hoạt động khi offline.
- Cấu trúc bản ghi sự kiện offline: `{ eventId, deviceCode, uid, deviceEventAt, result, timeSynced }`.
- Xây dựng endpoint đồng bộ: `POST /api/device/logs/sync`. Server tiến hành deduplication dựa trên `eventId` duy nhất (Idempotent Sync) để loại trừ triệt để việc ghi đúp log.
- Xử lý mốc thời gian: Nếu `timeSynced == false` (ESP32 mất mạng từ lúc khởi động chưa kịp sync NTP), server sẽ đánh dấu sự kiện có mốc thời gian ước lượng dựa trên `receivedAt`.
- Kịch bản kiểm thử: Ngắt Wi-Fi router, quẹt thẻ hợp lệ và thẻ lạ, bật lại router, xác nhận mọi sự kiện được đồng bộ đầy đủ và không bị trùng lặp.

**Nghiệm thu:** Chế độ mở cửa offline hoạt động tin cậy; quá trình đồng bộ log diễn ra tự động và đảm bảo tính lũy kế 100%.  
**Commit:** `week-10-offline-sync`.

---

### Tuần 11 — 10/01–15/01: Kiểm thử toàn diện, Đóng gói và Release

**Thực hiện:**
- Chạy toàn bộ bộ kiểm thử tích hợp End-to-End từ TC01 đến TC10.
- Đo kiểm định lượng tối thiểu 30 lần cho cả hai kịch bản Online và Offline: lập bảng thống kê Min, Max, Average, Median và độ lệch chuẩn của thời gian phản hồi.
- Đo thời gian tự động phục hồi kết nối (Wi-Fi Reconnect Time) và tỉ lệ đồng bộ thành công của hàng đợi ngoại tuyến.
- Chụp ảnh mô hình phần cứng thực tế; hoàn thiện các sơ đồ hệ thống: Architecture, Circuit Wiring, ERD Database, Flowchart, FSM.
- Hoàn thiện tài liệu `README.md`: tổng quan, sơ đồ chân pinout an toàn, hướng dẫn cài đặt từng bước, biến môi trường, ảnh chụp giao diện, bảng dữ liệu kiểm thử.
- Rà soát bảo mật mã nguồn: kiểm tra `.gitignore`, xóa sạch token/secret/password trong code commit.
- Đóng gói ứng dụng Electron Desktop thành bộ cài Windows installer (`.exe`) hoàn chỉnh bằng `electron-builder`.
- Quay video demo chất lượng cao dài 3–5 phút thể hiện trọn vẹn: quẹt thẻ hợp lệ/từ chối online, khóa thẻ trên phần mềm, quẹt thẻ khi ngắt mạng, tự động đồng bộ khi có mạng lại, cảnh báo đột nhập.
- Tạo bản phát hành chính thức trên GitHub với Git Tag `v1.0.0`.

**Nghiệm thu:** File `.exe` cài đặt và chạy mượt mà trên máy tính sạch; mã nguồn sạch sẽ; video demo và tài liệu hoàn chỉnh 100%.  
**Commit / Tag:** `v1.0.0`.

---

## 3. Checklist nghiệm thu v1.0.0

```text
[ ] Module RFID-RC522 chỉ cấp nguồn 3.3V; dây SPI ngắn, chống nhiễu tốt.
[ ] Relay kích mở ở GPIO 26; firmware đặt mức logic ngắt trước khi pinMode để không giật relay khi khởi động.
[ ] Nút Exit Button (GPIO 32) có tụ lọc RC 100 nF và xử lý debounce không blocking.
[ ] Buzzer (GPIO 25), LED xanh (GPIO 27), LED đỏ (GPIO 33) hoạt động đúng chức năng.
[ ] Tuyệt đối không commit password Wi-Fi, JWT secret, raw device token hay database thật lên Git.
[ ] Raw device token không lưu trữ dạng plaintext trong database (phải lưu tokenHash SHA-256).
[ ] Vòng đời thẻ ACTIVE / BLOCKED / REVOKED / EXPIRED được kiểm soát nghiêm ngặt.
[ ] Phân quyền thẻ theo từng cửa cụ thể được kiểm tra chính xác trên server.
[ ] 100% lượt quẹt thẻ (thành công hoặc thất bại) đều được ghi vào AccessLog.
[ ] Quy tắc phát hiện bất thường (Anomaly Rules) tự động sinh cảnh báo khi có tấn công dò thẻ.
[ ] Chế độ Offline chỉ mở cửa cho thẻ đã được lưu trong NVS Whitelist Cache hợp lệ.
[ ] Quá trình đồng bộ nhật ký ngoại tuyến (Offline Sync) đảm bảo tính lũy kế (Idempotent), không nhân bản log.
[ ] Electron Desktop App bật contextIsolation=true, tắt nodeIntegration=false, bảo mật IPC qua contextBridge.
[ ] Có bảng dữ liệu thực nghiệm tối thiểu 30 phép đo độ trễ cho cả chế độ Online và Offline.
[ ] Có đầy đủ README, sơ đồ nối dây, ảnh chụp mạch thật, video demo, file cài đặt .exe và Git Release Tag.
```

---

## 4. Bẫy phần cứng và phòng tránh

| Bẫy phần cứng | Hiện tượng gặp phải | Nguyên nhân gốc rễ | Giải pháp kỹ thuật chuẩn xác |
|---|---|---|---|
| **Boot Strapping Pins** | ESP32 không boot được, rơi vào chế độ nạp flash hoặc giật relay khi reset | Các chân GPIO 0, 2, 4, 12, 15 quy định cấu hình boot khi cấp nguồn | Chuyển toàn bộ tải sang chân an toàn: GPIO 26 (Relay), GPIO 25 (Buzzer), GPIO 27/33 (LEDs), GPIO 32 (Exit), GPIO 21 (RC522 SS). |
| **Relay Active-LOW giật xung lúc boot** | Cửa tự mở chớp nhoáng 1 cái khi vừa cắm nguồn ESP32 | Module relay kích mở ở mức LOW; hàm `pinMode(26, OUTPUT)` mặc định xuất mức LOW trước khi gán mức HIGH | Đặt mức logic ngắt trước: `digitalWrite(26, HIGH);` rồi mới gọi `pinMode(26, OUTPUT);`. |
| **Cấp nhầm 5V cho RC522** | Module MFRC522 nóng ran, hỏng chip hoặc lỗi giao tiếp SPI | Chip MFRC522 chỉ chịu được điện áp tối đa 3.3V | Luôn nối chân VCC của RC522 vào chân 3V3 của ESP32; dán nhãn cảnh báo cấm cắm 5V. |
| **Dây bus SPI quá dài** | Đọc thẻ chập chờn, lúc nhận lúc không | Điện dung ký sinh và nhiễu sóng cao tần làm méo xung clock 10 MHz | Dây SPI giữ dưới 15 cm; đi dây GND xen kẽ; giảm clock SPI xuống 4 MHz nếu cần. |
| **Relay 5V không nhận logic 3.3V** | Relay không đóng hoặc đóng chập chờn | Mạch kích trên module relay thiết kế cho mức logic 5V, không nhận diện được 3.3V từ ESP32 | Kiểm tra module thực tế; sử dụng module relay tương thích logic 3.3V hoặc đệm thêm transistor/driver. |
| **Back-EMF từ khóa Solenoid 12V** | Relay sinh tia lửa điện, ESP32 bị reset bất thường khi khóa ngắt | Khi ngắt dòng qua cuộn cảm solenoid, sinh ra sức điện động cảm ứng ngược cực lớn | Mắc song song Diode Flyback 1N4007 ngược cực ngay tại hai đầu cuộn dây khóa Solenoid. |
| **Hiểu lầm về cách ly quang Optocoupler** | Mạch vẫn bị nhiễu reset dù module có Opto PC817 | Nhiều module relay thương mại nối tắt GND logic và GND cuộn hút qua jumper | Muốn cách ly hoàn toàn: rút jumper VCC/JD-VCC, cấp nguồn 5V riêng cho cuộn hút, không chung GND với ESP32. |
| **Dội tiếp điểm nút bấm (Button Bounce)** | Bấm 1 lần nút Exit nhưng hệ thống sinh nhiều sự kiện | Tiếp điểm cơ khí dao động đàn hồi tạo chuỗi xung đóng ngắt nhanh | Kết hợp tụ lọc phần cứng 100 nF mắc song song nút bấm và thuật toán Debounce phần mềm 50 ms bằng `millis()`. |
| **Sụt áp nguồn (Brownout Reset)** | ESP32 tự reset khi kích hoạt Wi-Fi hoặc đóng relay | Nguồn cấp từ cổng USB máy tính không đủ dòng tức thời cho Wi-Fi (đỉnh 500mA) và cuộn hút | Dùng nguồn Adapter 12V 2A qua mạch Buck hạ áp 5V cấp cho ESP32 và Relay; thêm tụ hóa 470–1000 µF lọc nguồn. |

---

## 5. Mã lỗi hệ thống

| Mã lỗi | Tên định danh | Ý nghĩa kỹ thuật | Hướng xử lý |
|---|---|---|---|
| `ERR_101` | `RC522_INIT_FAILED` | Khởi tạo module MFRC522 thất bại | Kiểm tra chân cắm 3.3V, dây SPI (SS, SCK, MOSI, MISO, RST) |
| `ERR_102` | `CARD_READ_TIMEOUT` | Quá thời gian đọc thẻ hoặc thẻ rút ra quá nhanh | Đưa thẻ lại gần reader trong cự ly 1–4 cm, giữ thẻ ổn định |
| `ERR_103` | `CARD_AUTH_FAILED` | Xác thực khóa bảo mật Sector thất bại | Kiểm tra Key A/Key B và cấu hình Access Bits (tính năng nâng cao) |
| `ERR_201` | `WIFI_CONNECTION_LOST` | Mất kết nối mạng Wi-Fi | Tự động kích hoạt cơ chế reconnect non-blocking; chuyển sang offline policy |
| `ERR_202` | `SERVER_TIMEOUT` | Fastify backend không phản hồi sau 3000 ms | Kích hoạt retry có giãn cách; chuyển sang tra cứu NVS Whitelist Cache |
| `ERR_203` | `DEVICE_UNAUTHORIZED` | Header `X-Device-Token` bị sai hoặc thiết bị bị khóa | Kiểm tra lại cấu hình deviceCode và mã token trong firmware |
| `ERR_301` | `CARD_NOT_FOUND` | Mã UID không tồn tại trong hệ thống | Báo từ chối; ghi log; kiểm tra xem thẻ đã được cấp phát chưa |
| `ERR_302` | `CARD_BLOCKED` | Thẻ đang ở trạng thái bị khóa tạm thời | Báo từ chối; kích hoạt cảnh báo an ninh `RULE_03` |
| `ERR_303` | `CARD_REVOKED` | Thẻ đã bị thu hồi vĩnh viễn | Báo từ chối; kích hoạt cảnh báo an ninh `RULE_03` |
| `ERR_304` | `CARD_EXPIRED` | Thẻ đã quá thời hạn sử dụng cho phép | Báo từ chối; thông báo cư dân gia hạn thẻ |
| `ERR_305` | `ACCESS_DENIED` | Thẻ không được phân quyền mở tại cửa này | Báo từ chối; kiểm tra phân quyền trong bảng `CardDoorPermission` |
| `ERR_401` | `BRUTE_FORCE_ALERT` | Phát hiện hành vi quẹt thẻ lạ dồn dập | Kích hoạt cảnh báo mức cao trên Desktop; đánh dấu cửa cần theo dõi |
| `ERR_501` | `OFFLINE_CACHE_WRITE_FAILED` | Không thể ghi danh sách thẻ vào NVS Flash | Kiểm tra dung lượng phân vùng NVS; format lại namespace Preferences |
| `ERR_502` | `OFFLINE_QUEUE_STORAGE_FULL` | Phân vùng LittleFS lưu hàng đợi sự kiện bị đầy | Gửi cảnh báo đầy bộ nhớ; kích hoạt đồng bộ khẩn cấp về server |
| `ERR_503` | `OFFLINE_SYNC_FAILED` | Đồng bộ dữ liệu sự kiện offline về server thất bại | Giữ nguyên hàng đợi trong LittleFS; kích hoạt retry lũy tiến |

---

## 6. Câu hỏi phỏng vấn và bảo vệ kỹ thuật

### 1. Vì sao dự án chọn chân GPIO 26, 25, 27, 33, 32 và né các chân Boot Strapping?
**Trả lời:** Vi điều khiển ESP32 có các chân Boot Strapping nhạy cảm (GPIO 0, 2, 4, 12, 15). Các chân này kiểm tra mức điện áp logic khi bật nguồn để quyết định chế độ khởi động (Boot mode, SPI Voltage, UART download). Nếu nối Relay, Buzzer hoặc Nút nhấn vào các chân này, điện trở kéo hoặc tải ngoại vi sẽ vô tình ép chân về mức LOW/HIGH bất thường lúc khởi động, dẫn đến hiện tượng ESP32 bị treo ở Flash Download Mode hoặc giật đóng relay ngoài ý muốn. Do đó, dự án chuyển toàn bộ ngoại vi sang các GPIO hoàn toàn an toàn: GPIO 26 cho Relay, GPIO 25 cho Buzzer, GPIO 27/33 cho LEDs, GPIO 32 cho Exit Button và GPIO 21 cho chân SS của RC522.

### 2. Thẻ MIFARE Classic và đầu đọc RC522 có thể bị clone (sao chép) rất dễ dàng; hệ thống của bạn xử lý vấn đề này ra sao?
**Trả lời:** Dự án thẳng thắn thừa nhận giới hạn vật lý của chuẩn ISO/IEC 14443A giá rẻ: UID chỉ đóng vai trò là mã định danh tra cứu, không phải chìa khóa bí mật. Để bảo vệ hệ thống, chúng em áp dụng nguyên lý phòng vệ chiều sâu (Defense-in-Depth) ở tầng phần mềm:
- Mọi quyết định cấp quyền đều do Server thẩm định thời gian thực dựa trên trạng thái vòng đời của thẻ (`ACTIVE`, `BLOCKED`, `REVOKED`) và quyền theo từng cửa (`CardDoorPermission`).
- Bản thân thiết bị đầu cuối ESP32 phải xác thực với Server bằng `X-Device-Token` (được băm SHA-256).
- Mọi nỗ lực quẹt thẻ đều được ghi lại trong `AccessLog` không thể sửa đổi.
- Hệ thống tích hợp thuật toán phát hiện bất thường cửa sổ trượt (Sliding-window Anomaly Detection): nếu kẻ gian dùng thẻ clone dò quét tại nhiều cửa hoặc quẹt liên tục, hệ thống sẽ tự động kích hoạt `SecurityAlert` và cảnh báo ngay lập tức cho bảo vệ.

### 3. Tại sao bắt buộc phải mắc Diode Flyback song song với khóa Solenoid 12V?
**Trả lời:** Khóa Solenoid là một tải mang tính điện cảm lớn ($L$). Theo định luật tự cảm Faraday và định luật Lenz, điện áp cảm ứng xuất hiện trên cuộn cảm được tính theo công thức $V_L = -L rac{di}{dt}$. Khi Relay ngắt điện đột ngột, dòng điện $i$ giảm về 0 trong thời gian cực ngắn ($dt 	o 0$), làm cho đạo hàm $rac{di}{dt}$ tiến tới âm vô cùng, sinh ra một xung áp ngược cực lớn (Back-EMF, có thể lên tới hàng trăm Volts). Xung điện áp cao tần này sẽ phóng tia lửa điện phá hủy tiếp điểm Relay và cảm ứng sóng điện từ làm sụt áp/treo vi điều khiển ESP32. Việc mắc Diode Flyback (1N4007) ngược cực song song với cuộn dây tạo ra một mạch kín triệt tiêu: xung áp ngược sẽ mở thông diode, đưa dòng năng lượng từ trường tiêu tán an toàn tuần hoàn qua nội trở của chính cuộn dây.

### 4. Hệ thống hoạt động thế nào khi mất kết nối Wi-Fi hoặc Server gặp sự cố?
**Trả lời:** Khi phát hiện mất kết nối (sau khi hết thời gian Timeout), ESP32 tự động chuyển sang chế độ dự phòng ngoại tuyến (`OFFLINE_CHECK`). Thiết bị sẽ tra cứu mã UID vừa quẹt vào bộ nhớ `NVS Flash` (nơi đã lưu sẵn Whitelist Cache được mã hóa và kiểm tra toàn vẹn CRC32 từ lần đồng bộ gần nhất). Nếu thẻ có trong danh sách cache, cửa vẫn mở bình thường. Toàn bộ sự kiện quẹt thẻ offline được đóng gói kèm mã định danh duy nhất `eventId` và lưu tuần tự vào bộ nhớ tệp `LittleFS`. Khi mạng kết nối trở lại, ESP32 sẽ gửi toàn bộ hàng đợi này lên endpoint `/api/device/logs/sync`; Server dựa vào `eventId` để xử lý lũy kế (Idempotent Sync), cam kết không bao giờ tạo bản ghi trùng lặp.

### 5. Khi Server phát hiện tấn công dò thẻ (Brute-force Alert), Server có thể tự động khóa đầu đọc từ xa được không?
**Trả lời:** Trong kiến trúc MVP hiện tại, giao tiếp giữa ESP32 và Server hoạt động theo mô hình Request - Response của giao thức HTTP. Do đó, Server chỉ có thể gửi cờ hiệu cảnh báo kèm theo trong gói phản hồi của chính request gây vượt ngưỡng (`alarm: true` để ESP32 hú còi tại chỗ). Để Server có thể chủ động đẩy lệnh từ trên xuống (Push Command) khóa đầu đọc ngay tức khắc bất cứ lúc nào, hệ thống cần một kênh truyền thời gian thực hai chiều như MQTT Broker hoặc WebSocket — đây chính là tính năng nâng cao đã được quy hoạch rõ ràng trong lộ trình phát triển tiếp theo của dự án.
