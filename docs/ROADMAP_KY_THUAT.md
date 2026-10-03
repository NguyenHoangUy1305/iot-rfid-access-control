# 📘 LỘ TRÌNH THỰC HIỆN CHI TIẾT — DỰ ÁN 1 (BẢN CHỐT v1.1)
## Hệ thống kiểm soát cửa RFID & IoT tích hợp Desktop App
### ESP32 + RC522 + Fastify + Electron

> **Tác giả:** NguyenHoangUy1305  
> **Repository:** `NguyenHoangUy1305/iot-rfid-access-control`  
> **Thời lượng:** 11 tuần triển khai chính + 6 ngày nghiệm thu/đóng gói  
> **Thời gian:** 01/11/2026–15/01/2027  
> **Mục tiêu:** Hoàn thành MVP ổn định, có dữ liệu đo và tài liệu tái tạo được trước khi làm Advanced Features.

---

## 1. Phạm vi MVP

```text
ESP32 + RC522 đọc/chuẩn hóa UID
→ Fastify server-side authorization
→ Relay GPIO26 + LED GPIO27/GPIO33 + Buzzer GPIO25 + Exit GPIO32
→ TypeScript + Prisma + SQLite
→ X-Device-Token; database chỉ lưu tokenHash
→ Card lifecycle: ACTIVE / BLOCKED / REVOKED / EXPIRED
→ CardDoorPermission
→ AccessLog + rule-based alert
→ Electron + React CRUD/log/alert
→ NVS whitelist cache + LittleFS offline queue + idempotent sync
→ Test report + README + video + Windows installer
```

### Không thuộc MVP

```text
WebSocket, Telegram, CSV/Excel, counter trong card, remote unlock,
reed switch, tamper switch, MQTT, OTA, Docker, PostgreSQL, multi-door.
```

### Giới hạn bảo mật

- UID chỉ là định danh tra cứu, không là bí mật.
- Không tuyên bố MIFARE Classic/RC522 chống clone tuyệt đối.
- MVP dựa vào device token, card lifecycle, door permission, audit log, offline policy và anomaly alerts.
- Counter trên card là Advanced Feature; không là bằng chứng tuyệt đối thẻ gốc.
- MVP không dùng nonce/timestamp replay protection nếu chưa có nonce cache, TTL, time window, retry policy và thiết kế HMAC hoàn chỉnh.

---

## 2. Quyết định kỹ thuật chốt

### 2.1 Pinout

| Thiết bị | GPIO ESP32 | Ghi chú |
|---|---:|---|
| RC522 SS/SDA | GPIO21 | SPI chip select |
| RC522 SCK/MOSI/MISO | GPIO18/GPIO23/GPIO19 | SPI |
| RC522 RST | GPIO22 | Reset |
| Relay IN | GPIO26 | Kiểm tra active-low/high và ngưỡng 3.3 V |
| Buzzer | GPIO25 | Driver transistor nếu dòng cao/5 V |
| LED xanh/đỏ | GPIO27/GPIO33 | Mỗi LED qua 220–330 ohm |
| Exit button | GPIO32 | `INPUT_PULLUP`, nút nối GND |

Không dùng GPIO0, 2, 4, 12, 15 cho tải ngoại vi trong bản này nếu chưa kiểm thử boot behavior trên module thật.

### 2.2 Relay và solenoid

Topology tài liệu giả định **khóa fail-secure**: cấp điện để nhả chốt; mất điện giữ khóa. Phải xác nhận loại khóa thật trước khi đấu.

```text
+12 V adapter → Relay COM → Relay NO → Solenoid (+)
Solenoid (−) → GND adapter
Flyback diode: cathode/vạch trắng → Solenoid (+); anode → Solenoid (−)
```

- Adapter 12 V phải chọn theo datasheet hoặc dòng đo thực của solenoid; không mặc định mọi khóa cần 2 A.
- Với relay active-low, đặt trạng thái OFF vào output latch trước khi đặt chân thành OUTPUT, sau đó kiểm tra lúc boot/reset trên module thật.
- Optocoupler trên relay module không mặc định là cách ly galvanic hoàn toàn.

### 2.3 Log, thời gian và cache

```text
receivedAt      = server time, nguồn audit chính thức
deviceEventAt   = device time, chỉ đối chiếu/debug
timeSynced      = cho biết ESP32 đã đồng bộ NTP hay chưa
CRC32           = phát hiện lỗi dữ liệu ngẫu nhiên/ghi dở; không là encryption/authentication
eventId unique  = idempotency khi offline sync
```

---

## 3. Timeline 11 tuần

### Tuần 1 — 01/11–07/11: Môi trường và I/O

- Cài VS Code, Git, PlatformIO, Node.js LTS, Postman.
- Tạo `firmware/`, `server/`, `desktop/`, `docs/`; tạo `.gitignore`.
- Test LED onboard/Serial Monitor.
- Test LED GPIO27/GPIO33, buzzer GPIO25, relay input GPIO26; chưa nối solenoid.
- Với active-low relay, kiểm tra không có xung mở relay khi boot/reset.
- Viết `docs/learning-log.md`.

**Nghiệm thu:** Firmware flash ổn; I/O hoạt động đúng; secrets không nằm trong Git.  
**Commit:** `week-01-hardware-io`.

### Tuần 2 — 08/11–14/11: RC522 và UID

- Đấu RC522: SS21, SCK18, MOSI23, MISO19, RST22, VCC 3V3, GND.
- Cài `miguelbalboa/MFRC522`; chạy `DumpInfo`.
- Chuẩn hóa UID: `A4:3C:9B:10` → `a43c9b10`.
- Đọc 3 thẻ, mỗi thẻ ít nhất 30 lần.
- Suppress UID lặp trong 2 giây.
- Ghi lại khoảng cách đọc đo được và điều kiện test; không cam kết số cố định.

**Nghiệm thu:** Đọc UID ổn định trên hardware thật; không có lỗi SPI lặp liên tục.  
**Commit:** `week-02-rfid-read`.

### Tuần 3 — 15/11–21/11: Local access control

- Whitelist firmware gồm 2 UID hợp lệ.
- Valid: relay 3 giây + LED xanh + 2 beep; invalid: LED đỏ + beep dài.
- Exit button GPIO32, debounce `millis()` 30–50 ms; tụ 100 nF là hỗ trợ lọc nhiễu, không thay debounce software.
- Test 50 lần valid/invalid/exit.
- Sau khi logic ổn định mới đấu solenoid DC theo topology đã chốt; test brownout/reset.

**Nghiệm thu:** Hành vi valid/invalid/exit đúng; ESP32 không reset bất thường khi đóng/ngắt tải.  
**Commit:** `week-03-local-access`.

### Tuần 4 — 22/11–28/11: Wi-Fi, NTP, firmware architecture

- Wi-Fi STA, log IP/RSSI/status; reconnect non-blocking mỗi 5 giây.
- Đồng bộ SNTP/NTP khi Internet sẵn sàng; dùng `timeSynced` flag.
- Modules: `RfidReader`, `DoorController`, `NetworkManager`, `AppConfig`, `OfflineStore`.
- States: `LOCKED`, `VERIFYING`, `UNLOCKED`, `DENIED`, `OFFLINE_CHECK`.
- Test tắt/bật router; exit button phải vẫn chạy.
- Burn-in 4–8 giờ: ghi free heap đầu/cuối, reconnect count, reset count.

**Nghiệm thu:** Không reset bất thường trong test; free heap không giảm liên tục đáng kể; reconnect/NTP hoạt động khi có mạng.  
**Commit:** `week-04-wifi-firmware-architecture`.

### Tuần 5 — 29/11–05/12: Backend foundation

- Fastify + TypeScript + Prisma + SQLite + Zod + dotenv.
- `GET /api/health` trả status và server timestamp.
- Schema: `User`, `Resident`, `Card`, `Door`, `Device`, `CardDoorPermission`, `AccessLog`, `SecurityAlert`.
- `Device` lưu `tokenHash`, không lưu raw token; raw token chỉ hiển thị một lần lúc provisioning.
- Migration, seed: Admin, residents, cards, door, device.
- `.env.example`; `.env` nằm trong `.gitignore`.

**Nghiệm thu:** Migration/seed/health chạy; quyền card theo cửa tồn tại trong schema.  
**Commit:** `week-05-backend-foundation`.

### Tuần 6 — 06/12–12/12: Business Access API

- `POST /api/device/access/verify` với `X-Device-Token`.
- Hash token nhận được, so với `tokenHash`.
- Check device active, card exists, status, expiry, `CardDoorPermission`.
- Reason codes: `ACCESS_GRANTED`, `CARD_NOT_FOUND`, `CARD_BLOCKED`, `CARD_REVOKED`, `CARD_EXPIRED`, `ACCESS_DENIED`, `DEVICE_UNAUTHORIZED`.
- Ghi `AccessLog` cho mọi attempt từ device xác thực; server gán `receivedAt`.
- Alert thẻ blocked/revoked; Postman TC01–TC06.

**Nghiệm thu:** Test case đúng; benchmark 30 request LAN, báo cáo min/max/average/median.  
**Commit:** `week-06-access-api`.

### Tuần 7 — 13/12–19/12: ESP32 gọi API

- Gửi `deviceCode`, UID, optional `deviceEventAt`; gửi token qua header.
- Parse `allowed`, `reasonCode`, `unlockDurationMs`, `alarm`.
- Allowed: relay/LED xanh/beep; denied: LED đỏ/beep.
- Timeout API: chỉ báo lỗi; chưa offline grant nếu cache chưa có.
- Serial log UID, HTTP code, reasonCode, round-trip time.
- Đổi trạng thái ACTIVE → BLOCKED trong DB, quẹt lại để test.

**Nghiệm thu:** Quyền server có hiệu lực lần quẹt sau; có dữ liệu latency 30 lần.  
**Commit:** `week-07-esp32-api-integration`.

### Tuần 8 — 20/12–26/12: Electron MVP

- Electron + Vite + React + TypeScript.
- `contextIsolation: true`, `nodeIntegration: false`.
- `preload/contextBridge` chỉ expose API cần thiết.
- Login, dashboard, residents, cards, recent logs.
- REST polling log 3–5 giây.

**Nghiệm thu:** App đọc dữ liệu Fastify thật; renderer không có Node API không cần thiết.  
**Commit:** `week-08-electron-mvp`.

### Tuần 9 — 27/12–02/01: CRUD, RBAC, alerts

- Roles: `ADMIN`, `GUARD`.
- CRUD resident/card, enrollment, block/revoke, filters logs.
- Alert rules:

| Rule | Điều kiện | Kết quả |
|---|---|---|
| RULE_01 | 3 `CARD_NOT_FOUND` trong 60 giây/cùng cửa | Warning |
| RULE_02 | 5 `DENIED` tổng cộng trong 60 giây/cùng cửa | High |
| RULE_03 | Card BLOCKED/REVOKED được quẹt | Security alert |
| RULE_04 | Token sai 3 lần trong 5 phút | Device security alert |

- Trang alerts open/resolved và audit người resolve.
- Server chỉ có thể trả `alarm:true` trên response hiện tại; push command là Advanced.

**Nghiệm thu:** CRUD/card lifecycle/rules đúng trong kịch bản test.  
**Commit:** `week-09-rbac-alerts`.

### Tuần 10 — 03/01–09/01: Offline cache và sync

- `GET /api/device/whitelist/cache`: cache version, updatedAt, CRC32, cards allowed tại cửa.
- NVS: config + cache nhỏ; LittleFS: offline event queue.
- Offline grant chỉ cho UID cached/valid tại lần sync gần nhất.
- UID lạ không cached: deny offline.
- Event: `eventId`, `deviceCode`, `uid`, `deviceEventAt`, `timeSynced`, `result`.
- `POST /api/device/logs/sync`; unique `eventId` để deduplicate.
- Nếu `timeSynced=false`, server đánh dấu device time là ước lượng và dùng `receivedAt` làm thời gian chính.
- Test mất mạng → grant/deny → hồi mạng → sync không duplicate.

**Nghiệm thu:** Cache/queue/sync chạy đúng trong lab; không trùng event sau retry.  
**Commit:** `week-10-offline-sync`.

### Tuần 11 — 10/01–15/01: Test, release, portfolio

- Chạy TC01–TC10.
- 30 phép đo online và offline; báo cáo min/max/average/median/standard deviation.
- Đo reconnect time, offline sync success rate.
- Hoàn thiện README, wiring, architecture, ERD, FSM, flowchart, ảnh mạch.
- Review secrets, `.gitignore`, `.env.example`.
- Build Windows installer bằng `electron-builder`.
- Video 3–5 phút: online grant/deny, block card, offline cache/sync, alert.
- Git tag/release `v1.0.0`.

**Nghiệm thu:** Installer, source, documentation, test report, video và release hoàn chỉnh.  
**Commit/tag:** `v1.0.0`.

---

## 4. Checklist release v1.0.0

```text
[ ] RC522 chạy 3.3 V; SPI wiring ngắn và ổn định.
[ ] Relay GPIO26 không tự kích khi ESP32 boot/reset.
[ ] Relay module nhận ổn định logic 3.3 V hoặc có driver phù hợp.
[ ] Loại khóa fail-secure/fail-safe đã được xác nhận trước khi đấu relay.
[ ] Adapter được chọn theo dòng tải thực/datasheet.
[ ] Không commit Wi-Fi password, raw token, JWT secret, database thật.
[ ] Raw token không lưu plaintext trong database.
[ ] Card lifecycle và CardDoorPermission chạy đúng.
[ ] AccessLog ghi mọi attempt phù hợp; không tuyên bố immutable nếu chưa có append-only control.
[ ] Alert rules đúng kịch bản test.
[ ] CRC32 chỉ dùng integrity check, không gọi là encryption/authentication.
[ ] Offline chỉ grant card cache hợp lệ; eventId sync không duplicate.
[ ] Electron: contextIsolation true, nodeIntegration false, contextBridge tối thiểu.
[ ] Có 30 phép đo online/offline cùng bảng kết quả.
[ ] Có README, hình mạch, sơ đồ, video, installer và Git release.
```

---

## 5. Mã lỗi

| Code | Ý nghĩa | Hành động |
|---|---|---|
| ERR_101 | RC522_INIT_FAILED | Kiểm tra 3V3/SPI/RST |
| ERR_102 | CARD_READ_TIMEOUT | Kiểm tra thẻ, khoảng cách, wiring |
| ERR_201 | WIFI_CONNECTION_LOST | Reconnect non-blocking, xét offline policy |
| ERR_202 | SERVER_TIMEOUT | Retry/backoff, xét offline cache |
| ERR_203 | DEVICE_UNAUTHORIZED | Kiểm tra deviceCode/token |
| ERR_301 | CARD_NOT_FOUND | Deny + log |
| ERR_302 | CARD_BLOCKED | Deny + alert |
| ERR_303 | CARD_REVOKED | Deny + alert |
| ERR_304 | CARD_EXPIRED | Deny + log |
| ERR_305 | ACCESS_DENIED | Kiểm tra CardDoorPermission |
| ERR_401 | BRUTE_FORCE_ALERT | High alert |
| ERR_501 | OFFLINE_CACHE_WRITE_FAILED | Kiểm tra NVS/cache payload |
| ERR_502 | OFFLINE_QUEUE_STORAGE_FULL | Sync/maintenance alert |
| ERR_503 | OFFLINE_SYNC_FAILED | Giữ queue, retry backoff |

---

## 6. Ghi chú bảo vệ đồ án

- Không nói “100% chống clone”, “100% không memory leak”, “latency chắc chắn dưới X ms” nếu chưa có số liệu thực nghiệm.
- Không nói CRC32 là mã hóa.
- Không nói optocoupler relay luôn cách ly hoàn toàn.
- Không nói access log bất biến nếu database/app chưa có append-only policy.
- Nêu rõ: HTTP request-response không đủ cho server push command; MQTT/WebSocket/command polling là hướng mở rộng.
- Khi có dữ liệu thật, bảo vệ bằng bảng test case, ảnh hardware, log serial, Postman collection và biểu đồ latency.

---

## 7. Phân tích đối chiếu mô hình doanh nghiệp (Enterprise Gap Analysis)

| Tiêu chuẩn Doanh nghiệp (Enterprise Standard) | Thực tế MVP Đồ án (v1.1) | Giải trình kỹ thuật & Hướng nâng cấp Production |
| :--- | :--- | :--- |
| **Kết nối mạng:** Ethernet PoE (IEEE 802.3af) hoặc RS-485 / OSDP v2.2. Chống nhiễu và chống rớt gói. | Wi-Fi STA 2.4 GHz qua vi điều khiển ESP32. | Phù hợp chi phí lab sinh viên. Hướng mở rộng: dùng module Ethernet W5500 PoE để cấp nguồn và tín hiệu qua 1 sợi cáp LAN duy nhất. |
| **Bảo mật thẻ:** MIFARE DESFire EV2/EV3 mã hóa cứng AES-128, xác thực tương hỗ (Mutual Auth). | Chuẩn UID ISO/IEC 14443A (MIFARE Classic). | Thừa nhận giới hạn chống clone phần cứng; dời toàn bộ logic bảo mật lên server (Server-side validation + Anomaly rules). |
| **Liên động PCCC (Fire Alarm Interface):** Rơ-le cơ ngắt nguồn khóa khẩn cấp độc lập với vi điều khiển. | Điều khiển đóng ngắt khóa hoàn toàn bằng firmware ESP32. | Mô hình lab minh họa giải thuật; thực tế tòa nhà khóa Maglock được đấu trực tiếp qua tiếp điểm NC của tủ trung tâm báo cháy PCCC. |
| **Chống quay vòng thẻ (Anti-Passback):** Ngăn chặn quẹt thẻ vào rồi chuyền thẻ ra ngoài cho người khác. | Chỉ kiểm tra phân quyền hợp lệ tại 1 đầu đọc độc lập. | Quy hoạch vào Advanced Features: Quản lý cụm Reader Vào/Ra theo cặp (In/Out Reader Pair). |
| **Kiểm toán nhật ký (Audit Trail):** Nhật ký bất biến WORM (Write-Once-Read-Many) hoặc Hash-Chain SHA-256. | Bảng AccessLog trên SQLite / PostgreSQL. | Đạt yêu cầu truy vết hệ thống SMB; có thể bổ sung Hash-Chain (block-by-block digest) khi triển khai Enterprise. |
