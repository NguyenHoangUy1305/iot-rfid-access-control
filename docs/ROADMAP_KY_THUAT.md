# 📘 LỘ TRÌNH THỰC HIỆN CHI TIẾT — DỰ ÁN 1 (BẢN FINAL CHUẨN)
## Hệ thống kiểm soát cửa RFID & IoT tích hợp Desktop App
### ESP32 + RC522 + Fastify + Electron

> **Tác giả:** NguyenHoangUy1305  
> **Repository:** `NguyenHoangUy1305/iot-rfid-access-control`  
> **Thời lượng:** 11 tuần triển khai chính + 6 ngày nghiệm thu/đóng gói  
> **Thời gian:** 01/11/2026–15/01/2027  
> **Mục tiêu:** Hoàn thành MVP ổn định, đo được, tái tạo được trước khi triển khai tính năng nâng cao.

---

## 1. Phạm vi MVP

```text
ESP32 + RC522 đọc/chuẩn hóa UID
→ Fastify server-side authorization
→ Relay GPIO26 + LED GPIO27/GPIO33 + buzzer GPIO25 + exit GPIO32
→ TypeScript + Prisma + SQLite
→ X-Device-Token; database chỉ lưu tokenHash
→ Card lifecycle: ACTIVE / BLOCKED / REVOKED / EXPIRED
→ CardDoorPermission
→ AccessLog + rule-based alerts
→ Electron + React: login, CRUD, logs, alerts
→ NVS whitelist cache + LittleFS offline queue + idempotent sync
→ Test report + README + video + Windows installer
```

### Không thuộc MVP

```text
WebSocket, Telegram, CSV/Excel, card counter, remote unlock, reed switch,
tamper switch, MQTT, OTA, Docker, PostgreSQL, multi-door, HMAC/nonce replay protection.
```

### Giới hạn bảo mật

- UID chỉ là định danh tra cứu, không phải bí mật.
- Không tuyên bố MIFARE Classic/RC522 chống clone tuyệt đối.
- MVP dựa vào device token, card lifecycle, door permission, audit logging, offline policy và anomaly alerts.
- Counter card là Advanced Feature, không chứng minh tuyệt đối card là thẻ gốc.
- Raw token chỉ lưu trên device secret/config; database chỉ lưu `tokenHash`.
- AccessLog là audit record để truy vết; không gọi immutable/non-repudiation nếu chưa có append-only control, chữ ký hoặc hash chain được bảo vệ.

---

## 2. Quyết định kỹ thuật chốt

### 2.1 Pinout

| Thiết bị | GPIO ESP32 | Ghi chú |
|---|---:|---|
| RC522 SS/SDA | GPIO21 | SPI chip select |
| RC522 SCK/MOSI/MISO | GPIO18/GPIO23/GPIO19 | SPI |
| RC522 RST | GPIO22 | Reset |
| Relay IN | GPIO26 | Kiểm tra active-low/high và ngưỡng 3.3 V |
| Buzzer | GPIO25 | Dùng transistor/MOSFET nếu dòng cao hoặc 5 V |
| LED xanh/đỏ | GPIO27/GPIO33 | Mỗi LED qua 220–330 ohm |
| Exit button | GPIO32 | `INPUT_PULLUP`, nút nối GND |

Không dùng GPIO0, GPIO2, GPIO4, GPIO5, GPIO12 và GPIO15 cho relay/buzzer/button nếu chưa kiểm thử boot behavior trên board/module thật.

### 2.2 Relay và solenoid

Tài liệu giả định khóa **fail-secure**: cấp điện để nhả chốt, mất điện giữ khóa. Phải xác nhận loại khóa thực trước khi đấu. Maglock fail-safe có wiring/policy khác.

```text
+12 V adapter → Relay COM → Relay NO → Solenoid (+)
Solenoid (−) → GND adapter
Flyback diode: cathode/vạch trắng → Solenoid (+); anode → Solenoid (−)
```

- Adapter 12 V chọn theo datasheet hoặc dòng đo thực của solenoid, có dự phòng phù hợp.
- Relay active-low: đặt output latch ở trạng thái OFF trước khi `pinMode(..., OUTPUT)`, sau đó test boot/reset thực tế.
- Relay module phải nhận ổn định logic 3.3 V, hoặc dùng driver phù hợp.
- Optocoupler relay không mặc định cách ly galvanic hoàn toàn.

### 2.3 Constants firmware

```text
CARD_REPEAT_SUPPRESSION_MS = 2000
EXIT_BUTTON_DEBOUNCE_MS    = 50
DEFAULT_UNLOCK_DURATION_MS = 3000
WIFI_RECONNECT_INTERVAL_MS = 5000
HTTP_CONNECT_TIMEOUT_MS    = 1000
HTTP_RESPONSE_TIMEOUT_MS   = 2000
OFFLINE_FALLBACK_AFTER_MS  = 2000
```

Các giá trị này là giá trị khởi đầu; phải tinh chỉnh sau khi đo thực tế.

### 2.4 Log, time và cache

```text
receivedAt      = server time, nguồn audit chính thức
deviceEventAt   = device time, chỉ đối chiếu/debug
timeSynced      = ESP32 đã đồng bộ NTP hay chưa
CRC32           = kiểm tra lỗi dữ liệu ngẫu nhiên/ghi dở; không là encryption/authentication
eventId unique  = idempotency khi sync offline queue
```

---

## 3. Timeline 11 tuần

### Tuần 1 — 01/11–07/11: Môi trường và I/O

- Cài VS Code, Git, PlatformIO, Node.js LTS, Postman.
- Tạo `firmware/`, `server/`, `desktop/`, `docs/`; tạo `.gitignore`.
- Test LED onboard/Serial Monitor.
- Test LED GPIO27/GPIO33, buzzer GPIO25, relay GPIO26; chưa nối solenoid.
- Kiểm tra relay active-low không tạo xung mở khi boot/reset.
- Viết `docs/learning-log.md`.

**Nghiệm thu:** Firmware flash ổn; I/O chạy đúng; secret không nằm trong Git.  
**Commit:** `week-01-hardware-io`.

### Tuần 2 — 08/11–14/11: RC522 và UID

- Đấu RC522: SS21, SCK18, MOSI23, MISO19, RST22, VCC 3V3, GND.
- Cài `miguelbalboa/MFRC522`, chạy `DumpInfo`.
- Chuẩn hóa UID: `A4:3C:9B:10` → `a43c9b10`.
- Đọc tối thiểu 3 thẻ, mỗi thẻ 30 lần.
- Suppress UID lặp 2 giây.
- Ghi khoảng cách đọc thực đo được và điều kiện test.

**Nghiệm thu:** UID ổn định trên hardware thật; không có lỗi SPI lặp liên tục.  
**Commit:** `week-02-rfid-read`.

### Tuần 3 — 15/11–21/11: Local access control

- Whitelist firmware gồm 2 UID hợp lệ.
- Valid: relay 3 giây, LED xanh, 2 beep; invalid: LED đỏ, beep dài.
- Exit GPIO32, debounce `millis()` 30–50 ms; tụ 100 nF chỉ hỗ trợ lọc nhiễu.
- Test 50 lần valid/invalid/exit.
- Sau khi logic ổn, đấu solenoid DC theo topology chốt; test reset/brownout.

**Nghiệm thu:** Valid/invalid/exit đúng; không reset bất thường khi đóng/ngắt tải.  
**Commit:** `week-03-local-access`.

### Tuần 4 — 22/11–28/11: Wi-Fi, NTP, firmware architecture

- Wi-Fi STA, log IP/RSSI/status, reconnect non-blocking mỗi 5 giây.
- Đồng bộ SNTP/NTP khi có Internet; dùng `timeSynced` flag.
- Modules: `RfidReader`, `DoorController`, `NetworkManager`, `AppConfig`, `OfflineStore`.
- States: `LOCKED`, `VERIFYING`, `UNLOCKED`, `DENIED`, `OFFLINE_CHECK`.
- Test router tắt/bật; exit button vẫn hoạt động.
- Burn-in 4–8 giờ: free heap đầu/cuối, reconnect count, reset count.

**Nghiệm thu:** Không reset bất thường trong test; free heap không giảm liên tục đáng kể; reconnect/NTP hoạt động khi có mạng.  
**Commit:** `week-04-wifi-firmware-architecture`.

### Tuần 5 — 29/11–05/12: Backend foundation

- Fastify + TypeScript + Prisma + SQLite + Zod + dotenv.
- `GET /api/health` trả server status/timestamp.
- Schema: `User`, `Resident`, `Card`, `Door`, `Device`, `CardDoorPermission`, `AccessLog`, `SecurityAlert`.
- `Device`: `deviceCode`, `doorId`, `tokenHash`, `active`, `lastSeenAt`, `firmwareVersion`.
- Migration, seed: Admin, residents, cards, door, device.
- `.env.example`; `.env` trong `.gitignore`.

**Nghiệm thu:** Migration/seed/health chạy; Device liên kết Door; permission theo cửa tồn tại.  
**Commit:** `week-05-backend-foundation`.

### Tuần 6 — 06/12–12/12: Auth, RBAC và Business Access API

- `POST /api/auth/login`; password băm bằng bcrypt hoặc Argon2.
- Chọn JWT access token thời hạn ngắn hoặc session strategy phù hợp Electron.
- Middleware `requireAuth`, `requireRole(ADMIN/GUARD)`.
- ADMIN: CRUD/card lifecycle; GUARD: xem logs và resolve alerts.
- `POST /api/device/access/verify` với `X-Device-Token`.
- Hash token nhận được, so với `tokenHash`; check Device, Card, status, expiry, permission.
- Reason codes: `ACCESS_GRANTED`, `CARD_NOT_FOUND`, `CARD_BLOCKED`, `CARD_REVOKED`, `CARD_EXPIRED`, `ACCESS_DENIED`, `DEVICE_UNAUTHORIZED`.
- Ghi AccessLog cho attempt từ device xác thực; server gán `receivedAt`.
- Postman TC01–TC07.

**Nghiệm thu:** Login/RBAC/API đúng; benchmark 30 request LAN với min/max/average/median.  
**Commit:** `week-06-auth-access-api`.

### Tuần 7 — 13/12–19/12: ESP32 gọi API

- ESP32 gửi `deviceCode`, UID, optional `deviceEventAt`, `timeSynced`; token qua header.
- Parse `allowed`, `reasonCode`, `unlockDurationMs`, `alarm`.
- Allowed: relay/LED xanh/beep; denied: LED đỏ/beep.
- Timeout: chỉ báo lỗi; chưa offline grant nếu cache chưa có.
- Serial log UID, HTTP code, reasonCode, RTT.
- Đổi ACTIVE → BLOCKED trong DB rồi quẹt lại.

**Nghiệm thu:** Quyền server áp dụng ở lần quẹt kế; có 30 phép đo latency.  
**Commit:** `week-07-esp32-api-integration`.

### Tuần 8 — 20/12–26/12: Electron MVP

- Electron + Vite + React + TypeScript.
- `contextIsolation: true`, `nodeIntegration: false`.
- Preload/contextBridge chỉ expose API tối thiểu.
- Login backend thật, dashboard, residents, cards, recent logs.
- REST polling logs 3–5 giây.

**Nghiệm thu:** Renderer không có Node API không cần thiết; protected routes/API theo role hoạt động.  
**Commit:** `week-08-electron-mvp`.

### Tuần 9 — 27/12–02/01: CRUD và anomaly alerts

- CRUD resident/card, enrollment, block/revoke, log filters.
- Alerts open/resolved và audit người resolve.

| Rule | Điều kiện | Kết quả |
|---|---|---|
| RULE_01 | 3 `CARD_NOT_FOUND` trong 60 giây/cùng cửa | Warning |
| RULE_02 | 5 `DENIED` tổng cộng trong 60 giây/cùng cửa | High |
| RULE_03 | Card BLOCKED/REVOKED được quẹt | Security alert |
| RULE_04 | Token sai 3 lần trong 5 phút | Device security alert |

- Server có thể trả `alarm:true` trong response hiện tại; server push command là Advanced.

**Nghiệm thu:** CRUD/card lifecycle/rules đúng trong test.  
**Commit:** `week-09-rbac-alerts`.

### Tuần 10 — 03/01–09/01: Offline cache và sync

- `GET /api/device/whitelist/cache`: version, updatedAt, CRC32, cards allowed theo Door.
- NVS: config + cache nhỏ; LittleFS: offline event queue.
- Offline grant chỉ cho UID cached/valid tại lần sync gần nhất.
- Event: `eventId`, `deviceCode`, `uid`, `deviceEventAt`, `timeSynced`, `result`.
- `POST /api/device/logs/sync`; unique `eventId` deduplicate.
- Nếu `timeSynced=false`, server coi device time là ước lượng và dùng `receivedAt` làm audit time.
- Test mất mạng → grant/deny → hồi mạng → sync không duplicate.

**Nghiệm thu:** Cache/queue/sync chạy đúng trong lab.  
**Commit:** `week-10-offline-sync`.

### Tuần 11 — 10/01–15/01: Test, release, portfolio

- Chạy TC01–TC10.
- 30 phép đo online/offline: min/max/average/median/standard deviation.
- Đo reconnect time, sync success rate.
- Hoàn thiện README, wiring, architecture, ERD, FSM, flowchart, ảnh mạch.
- Review secrets, `.gitignore`, `.env.example`.
- Build Windows installer bằng `electron-builder`.
- Video 3–5 phút: online grant/deny, block card, offline sync, alert.
- Git tag/release `v1.0.0`.

**Nghiệm thu:** Installer, source, test report, video và release hoàn chỉnh.  
**Commit/tag:** `v1.0.0`.

---

## 4. Test matrix TC01–TC10

| ID | Kịch bản | Kết quả mong đợi |
|---|---|---|
| TC01 | Card ACTIVE có quyền | GRANTED, relay mở, AccessLog |
| TC02 | UID chưa đăng ký | DENIED, `CARD_NOT_FOUND` |
| TC03 | Card BLOCKED | DENIED, security alert |
| TC04 | Card REVOKED | DENIED, security alert |
| TC05 | Card EXPIRED | DENIED, `CARD_EXPIRED` |
| TC06 | Không có quyền cửa | DENIED, `ACCESS_DENIED` |
| TC07 | Device token sai | `DEVICE_UNAUTHORIZED`, device alert theo rule |
| TC08 | Offline, card cached | Offline grant, LittleFS event queue |
| TC09 | Offline, UID không cache | Offline deny, event queue |
| TC10 | Mạng phục hồi | Queue sync thành công, không duplicate theo `eventId` |

---

## 5. Checklist release v1.0.0

```text
[ ] RC522 chạy 3.3 V; SPI wiring ngắn và ổn định.
[ ] Relay GPIO26 không tự kích khi boot/reset.
[ ] Relay nhận logic 3.3 V hoặc có driver phù hợp.
[ ] Xác nhận khóa fail-secure/fail-safe trước wiring.
[ ] Adapter chọn theo dòng tải thực/datasheet.
[ ] Không commit Wi-Fi password, raw token, JWT secret, database thật.
[ ] Raw token không lưu plaintext trong database.
[ ] Login backend, password hash, RBAC và protected routes hoạt động.
[ ] Card lifecycle/CardDoorPermission chạy đúng.
[ ] AccessLog ghi attempt phù hợp; không tuyên bố immutable nếu chưa có control.
[ ] CRC32 chỉ integrity check, không gọi encryption/authentication.
[ ] Offline cache valid; sync eventId không duplicate.
[ ] Electron contextIsolation true, nodeIntegration false, contextBridge tối thiểu.
[ ] Có 30 phép đo online/offline và bảng số liệu.
[ ] Có README, sơ đồ, ảnh mạch, video, installer, Git release.
```

---

## 6. Mã lỗi

| Code | Ý nghĩa | Hành động |
|---|---|---|
| ERR_101 | RC522_INIT_FAILED | Kiểm tra 3V3/SPI/RST |
| ERR_102 | CARD_READ_TIMEOUT | Kiểm tra thẻ, wiring, khoảng cách |
| ERR_201 | WIFI_CONNECTION_LOST | Reconnect non-blocking, xét offline policy |
| ERR_202 | SERVER_TIMEOUT | Retry/backoff, xét cache |
| ERR_203 | DEVICE_UNAUTHORIZED | Kiểm tra deviceCode/token |
| ERR_301 | CARD_NOT_FOUND | Deny + log |
| ERR_302 | CARD_BLOCKED | Deny + alert |
| ERR_303 | CARD_REVOKED | Deny + alert |
| ERR_304 | CARD_EXPIRED | Deny + log |
| ERR_305 | ACCESS_DENIED | Kiểm tra CardDoorPermission |
| ERR_401 | BRUTE_FORCE_ALERT | High alert |
| ERR_501 | OFFLINE_CACHE_WRITE_FAILED | Kiểm tra NVS/cache payload |
| ERR_502 | OFFLINE_QUEUE_STORAGE_FULL | Maintenance/sync alert |
| ERR_503 | OFFLINE_SYNC_FAILED | Giữ queue, retry backoff |

---

## 7. Enterprise Gap Analysis

| Enterprise standard | MVP Bản Final Chuẩn | Hướng nâng cấp production |
|---|---|---|
| Network | Wi-Fi STA 2.4 GHz | Ethernet controller như W5500 kết hợp PoE PD module/PoE splitter hoặc board PoE để nhận data + power trên cùng cáp LAN |
| Card security | UID/MIFARE Classic | DESFire EV2/EV3, AES mutual authentication |
| Reader protocol | RC522 SPI local | OSDP RS-485/Secure Channel cho reader công nghiệp; Wiegand chỉ legacy |
| Fire interface | Firmware ESP32 điều khiển mô hình lab | Cửa thoát hiểm phải theo quy chuẩn PCCC địa phương, thiết bị được chứng nhận và đơn vị/kỹ sư có thẩm quyền; MVP không dùng cho cửa thoát hiểm |
| Anti-passback | Một reader độc lập | Cặp reader in/out, global state, occupancy/anti-passback rules |
| Audit trail | SQLite AccessLog | Append-only policy, DB role hạn chế UPDATE/DELETE, HMAC/signed hash chain, backup và immutable/WORM storage khi phù hợp |

---

## 8. Ghi chú bảo vệ đồ án

- Không nói “100% chống clone”, “100% không memory leak” hoặc “latency chắc chắn dưới X ms” nếu chưa đo.
- Không nói CRC32 là mã hóa.
- Không nói relay optocoupler luôn cách ly hoàn toàn.
- Không nói HTTP request-response tự hỗ trợ server push command.
- Nêu rõ MVP minh chứng năng lực thiết kế/triển khai; Enterprise Gap Analysis chứng minh hiểu giới hạn và đường nâng cấp.
