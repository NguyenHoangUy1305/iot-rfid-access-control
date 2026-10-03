# 📘 LỘ TRÌNH THỰC HIỆN CHI TIẾT — DỰ ÁN 1
## Đề tài: Hệ thống kiểm soát cửa RFID & IoT tích hợp Desktop App (ESP32 + Fastify + Electron)

> **Tác giả:** Kỹ sư IoT & Hệ thống nhúng (NguyenHoangUy1305)  
> **Repository:** [NguyenHoangUy1305/iot-rfid-access-control](https://github.com/NguyenHoangUy1305/iot-rfid-access-control)  
> **Thời gian:** 11 tuần (Khởi động: 01/11/2026 – Nghiệm thu v1.0.0: 15/01/2027)  
> **Mục tiêu tối thượng:** Hoàn thành sản phẩm khả dụng tối thiểu (**MVP**) chạy ổn định trước; chỉ triển khai tính năng nâng cao khi MVP đã nghiệm thu.  
> **Nguyên tắc kỹ sư:** Một tuần = Một sản phẩm con chạy được + Commit Git + Ảnh/Video/Ghi chép kỹ thuật.

---

## MỤC LỤC
1. [PHẠM VI MVP VÀ HƯỚNG NÂNG CAO](#1-phạm-vi-mvp-và-hướng-nâng-cao)
2. [LỘ TRÌNH CHI TIẾT 11 TUẦN THỰC CHIẾN (01/11/2026 – 15/01/2027)](#2-lộ-trình-chi-tiết-11-tuần-thực-chiến)
   - Tuần 1 (01/11–07/11): Môi trường phát triển và I/O cơ bản
   - Tuần 2 (08/11–14/11): Giao tiếp SPI RC522 và đọc chuẩn hóa UID
   - Tuần 3 (15/11–21/11): Kiểm soát cửa cục bộ (Local Access Control)
   - Tuần 4 (22/11–28/11): Kết nối Wi-Fi và Kiến trúc Firmware FSM
   - Tuần 5 (29/11–05/12): Nền tảng Backend Fastify + Prisma + SQLite
   - Tuần 6 (06/12–12/12): Xây dựng Business Access API & Token
   - Tuần 7 (13/12–19/12): Tích hợp ESP32 gọi API xác thực thời gian thực
   - Tuần 8 (20/12–26/12): Xây dựng ứng dụng Desktop Electron MVP
   - Tuần 9 (27/12–02/01): CRUD Cư dân, Phân quyền RBAC & Cảnh báo bất thường
   - Tuần 10 (03/01–09/01): Cơ chế Offline Whitelist Cache & Log Sync
   - Tuần 11 (10/01–15/01): Đo đạc chỉ số, Tài liệu, Video & Đóng gói .exe Release
3. [CHECKLIST NGHIỆM THU TRƯỚC KHI RELEASE V1.0.0](#3-checklist-nghiệm-thu-trước-khi-release-v100)
4. [BẢNG TỔNG HỢP BẪY PHẦN CỨNG & CÁCH PHÒNG TRÁNH (HARDWARE GOTCHAS)](#4-bảng-tổng-hợp-bẫy-phần-cứng--cách-phòng-tránh)
5. [BẢNG MÃ LỖI HỆ THỐNG (SYSTEM ERROR CODES)](#5-bảng-mã-lỗi-hệ-thống)
6. [BỘ CÂU HỎI PHỎNG VẤN KỸ THUẬT KINH ĐIỂN DÀNH CHO DỰ ÁN 1](#6-bộ-câu-hỏi-phỏng-vấn-kỹ-thuật-kinh-điển)

---

## 1. PHẠM VI MVP VÀ HƯỚNG NÂNG CAO

### 🎯 MVP Bắt buộc (Hoàn thành trước 15/01/2027)
```text
ESP32 + RC522 đọc UID
→ Whitelist / API Authorization
→ Relay (GPIO 26) + LED Xanh (GPIO 27) + LED Đỏ (GPIO 33) + Buzzer (GPIO 25)
→ Fastify + TypeScript + Prisma + SQLite
→ X-Device-Token xác thực thiết bị
→ Card Lifecycle: ACTIVE / BLOCKED / REVOKED / EXPIRED
→ Access Logs + Rule-based Anomaly Alerts
→ Electron + React Desktop App (Xem log, CRUD thẻ, cấp/khóa thẻ)
→ Offline Whitelist Cache (NVS Flash) + Log Queue (LittleFS) / Sync
→ Test Report (Đo latency 30 lần) + README + Video Demo + Đóng gói file .exe
```

### 🚀 Tính năng Nâng cao (Advanced Features — Chỉ làm khi còn dư thời gian)
```text
- WebSocket live event stream (MVP dùng REST Polling 3-5s cho ổn định)
- Xuất báo cáo định dạng Excel / CSV
- Gửi thông báo khẩn cấp qua Telegram Bot
- Counter mã hóa trong Data Block của thẻ MIFARE Classic
- Mở cửa từ xa có xác nhận mật khẩu Admin 2 bước
- Cảm biến từ Reed Switch phát hiện cửa bị giữ mở quá lâu (Door-Held-Open)
- Cảm biến chống cạy phá (Tamper Switch)
- Mở rộng nhiều cửa / nhiều thiết bị ESP32 (Chuyển sang PostgreSQL / Docker)
```

---

## 2. LỘ TRÌNH CHI TIẾT 11 TUẦN THỰC CHIẾN

### Tuần 1 — 01/11–07/11: Môi trường phát triển và I/O cơ bản
* **Kiến thức cần học:** Git cơ bản (`init`, `add`, `commit`, `push`, `branch`), PlatformIO IDE, cấu trúc PlatformIO `platformio.ini`, Serial Monitor, các mức logic GPIO, quy tắc điện áp 3.3V/5V/GND, nguyên lý tránh chân Strapping Pins.
* **Nhiệm vụ thực hành:**
  - Cài đặt công cụ: VS Code, Git, PlatformIO IDE extension, Node.js v20 LTS, Postman.
  - Khởi tạo repo `iot-rfid-access-control` và tạo khung thư mục: `firmware/`, `server/`, `desktop/`, `docs/`.
  - Nạp chương trình Blink test LED on-board trên ESP32 DevKit V1.
  - Cắm mạch breadboard thử nghiệm:
    - LED Xanh: `GPIO 27` (qua điện trở hạn dòng $220\Omega$).
    - LED Đỏ: `GPIO 33` (qua điện trở hạn dòng $220\Omega$).
    - Active Buzzer: `GPIO 25` (phát tiếng bíp bíp).
    - Relay Module: `GPIO 26` (kiểm tra tiếng đóng/ngắt tiếp điểm cơ khí, **chưa cắm khóa Solenoid 12V**).
  - Tạo file `docs/learning-log.md` ghi lại nhật ký thực hành từng ngày.
* **Nghiệm thu:** ESP32 nạp code mượt mà, Serial log in rõ ràng, LED/Buzzer/Relay đóng mở đúng nhịp. Commit Git: `week-01-hardware-io`.

---

### Tuần 2 — 08/11–14/11: Giao tiếp SPI RC522 và đọc chuẩn hóa UID
* **Kiến thức cần học:** Giao thức truyền thông đồng bộ nối tiếp SPI (Master/Slave, CPOL, CPHA), chức năng từng chân `SCK`, `MOSI`, `MISO`, `SS`, cấu trúc mã định danh duy nhất UID của thẻ ISO/IEC 14443A, thư viện `MFRC522`.
* **Nhiệm vụ thực hành:**
  - Đấu nối 7 chân RC522 với ESP32:
    - `SS / SDA`: **GPIO 21**
    - `SCK`: **GPIO 18**
    - `MOSI`: **GPIO 23**
    - `MISO`: **GPIO 19**
    - `RST`: **GPIO 22**
    - `VCC`: **Chân 3V3 của ESP32** (⚠️ *Tuyệt đối cấm cắm 5V*).
    - `GND`: Chân GND chung.
  - Cài đặt thư viện `miguelbalboa/MFRC522`, chạy ví dụ mẫu `DumpInfo.ino` để kiểm tra firmware module đọc thẻ.
  - Viết hàm chuẩn hóa chuỗi UID: Chuyển mảng bytes thành chuỗi hex viết thường đồng nhất (ví dụ: `A4:3C:9B:10` $	o$ `a43c9b10`).
  - Thử nghiệm quét tối thiểu 3 thẻ mẫu khác nhau, lặp lại 30 lần liên tiếp để đo độ nhạy.
  - Lập trình cơ chế chống quét lặp lại (Debounce thẻ): Bỏ qua nếu cùng 1 thẻ quét lại trong vòng $2	ext{ giây}$.
  - Chụp ảnh mạch phần cứng và quay video clip ngắn đọc mã thẻ in ra Serial Monitor.
* **Nghiệm thu:** Đọc thẻ nhạy trong khoảng cách $1 \sim 3	ext{ cm}$, không bị lỗi `Communication Error` liên tục. Commit Git: `week-02-rfid-read`.

---

### Tuần 3 — 15/11–21/11: Kiểm soát cửa cục bộ (Local Access Control)
* **Kiến thức cần học:** Máy trạng thái đơn giản, cấu trúc `enum`, kỹ thuật chống dội phím tiếp điểm cơ khí (Software Debounce với `millis()`), nguyên lý tải cảm kháng Solenoid và Diode Flyback 1N4007.
* **Nhiệm vụ thực hành:**
  - Khai báo một mảng Whitelist thẻ cục bộ tạm thời trong firmware (chứa 2 thẻ hợp lệ).
  - Lập trình logic kiểm tra:
    - Quẹt thẻ hợp lệ $	o$ Bật Relay trong $3	ext{ giây}$ + Bật LED xanh + Buzzer kêu 2 tiếng bíp ngắn.
    - Quẹt thẻ lạ $	o$ Bật LED đỏ $2	ext{ giây}$ + Buzzer kêu 1 tiếng bíp dài + Không mở Relay.
  - Đấu nối nút bấm mở cửa khẩn cấp (Exit Button): Cắm vào **GPIO 32** cấu hình `pinMode(32, INPUT_PULLUP)`. Viết giải thuật lọc dội phím bằng `millis()`, ngưỡng ổn định $50	ext{ ms}$.
  - Chạy thử nghiệm 50 lần quẹt thẻ hợp lệ/không hợp lệ và bấm nút Exit.
  - **Lắp mạch khóa Solenoid 12V:** Chỉ thực hiện sau khi logic Relay và còi đèn đã hoạt động hoàn hảo:
    - Nối nguồn 12V 2A rời vào khóa qua tiếp điểm Relay (COM - NO).
    - Hàn Diode Flyback 1N4007 song song ngược cực tính với 2 cọc cuộn dây khóa (Cathode có vạch trắng nối vào cực $+12	ext{V}$).
* **Nghiệm thu:** Thẻ hợp lệ mở khóa cơ học $3	ext{ giây}$ rồi tự khóa lại an toàn; thẻ lạ bị từ chối; bấm nút Exit mở cửa tức thì; không bị reset ESP32. Commit Git: `week-03-local-access`.

---

### Tuần 4 — 22/11–28/11: Kết nối Wi-Fi và Kiến trúc Firmware FSM
* **Kiến thức cần học:** Wi-Fi Station Mode (STA), cơ chế Reconnect non-blocking sử dụng Software Timer, mô hình Finite State Machine (FSM), nguyên tắc tách module mã nguồn (Clean Architecture).
* **Nhiệm vụ thực hành:**
  - Lập trình kết nối Wi-Fi qua thư viện `WiFi.h`; in địa chỉ IP nội bộ, cường độ sóng RSSI và trạng thái ra Serial Monitor.
  - Xây dựng cơ chế tự động kết nối lại (Auto-Reconnect) định kỳ mỗi 5 giây bằng non-blocking timer, **tuyệt đối không dùng vòng lặp `while` chặn luồng chính**.
  - Tách mã nguồn firmware thành các module độc lập:
    - `RfidReader`: Quản lý giao tiếp SPI RC522.
    - `DoorController`: Quản lý Relay, Buzzer, LED và nút Exit.
    - `NetworkManager`: Quản lý kết nối Wi-Fi.
    - `AppConfig`: Chứa cấu hình chân pinout và thông số hệ thống.
  - Cài đặt máy trạng thái `DoorState`: `LOCKED`, `VERIFYING`, `UNLOCKED`, `DENIED`, `OFFLINE_CHECK`.
  - Thử nghiệm tắt router Wi-Fi: Bấm nút Exit cửa vẫn phải mở bình thường, vi điều khiển không bị treo. Bật lại router: ESP32 tự động bắt lại mạng.
* **Nghiệm thu:** Firmware chạy liên tục 24h không bị rò rỉ bộ nhớ; tự động phục hồi kết nối mạng khi router bật lại. Commit Git: `week-04-wifi-firmware-architecture`.

---

### Tuần 5 — 29/11–05/12: Nền tảng Backend Fastify + Prisma + SQLite
* **Kiến thức cần học:** TypeScript cơ bản, Fastify Framework, kiến trúc RESTful API, Schema-driven validation, mô hình ORM Prisma, cơ sở dữ liệu quan hệ SQLite, công cụ kiểm thử Postman.
* **Nhiệm vụ thực hành:**
  - Khởi tạo dự án `server/`: Cài đặt `fastify`, `typescript`, `@prisma/client`, `prisma`, `dotenv`, `zod`.
  - Viết endpoint kiểm tra sức khỏe hệ thống: `GET /api/health` trả về `{ status: "ok", timestamp: ... }`.
  - Thiết kế file `prisma/schema.prisma` gồm các bảng cốt lõi:
    - `User`: Quản trị viên hệ thống.
    - `Resident`: Cư dân / nhân viên tòa nhà.
    - `Card`: Quản lý thẻ RFID (UID, trạng thái `ACTIVE`, `BLOCKED`, `REVOKED`, `EXPIRED`).
    - `Door`: Thông tin cửa ra vào.
    - `Device`: Thiết bị ESP32 (Mã thiết bị `deviceCode`, mã băm token `tokenHash`).
    - `AccessLog`: Nhật ký quẹt thẻ (UID, thời gian, kết quả, mã nguyên nhân).
    - `SecurityAlert`: Cảnh báo an ninh.
  - Chạy lệnh migration: `npx prisma migrate dev`.
  - Viết script `prisma/seed.ts` nạp sẵn dữ liệu mẫu: 1 tài khoản Admin, 3 cư dân mẫu, 5 thẻ RFID kiểm thử.
  - Chuẩn hóa định dạng lỗi trả về của API: `{ success: false, errorCode: "...", message: "..." }`.
* **Nghiệm thu:** Migration và Seed chạy thành công tạo file cơ sở dữ liệu `dev.db`; gọi Postman kiểm tra `GET /api/health` trả về mã 200 OK. Commit Git: `week-05-backend-foundation`.

---

### Tuần 6 — 06/12–12/12: Xây dựng Business Access API & Token
* **Kiến thức cần học:** Cơ chế xác thực thiết bị qua HTTP Header (`X-Device-Token`), Data Validation với thư viện Zod, mã phản hồi HTTP tiêu chuẩn, nguyên lý phân quyền truy cập cửa.
* **Nhiệm vụ thực hành:**
  - Xây dựng endpoint cốt lõi xác thực quẹt thẻ: `POST /api/device/access/verify`.
  - Kiểm tra và xác thực token thiết bị `X-Device-Token` khớp với bản ghi trong cơ sở dữ liệu.
  - Kiểm tra tính hợp lệ của mã thẻ:
    - Thẻ có trạng thái `ACTIVE` và còn hạn sử dụng $	o$ Trả về `allowed: true`, `reasonCode: "ACCESS_GRANTED"`, thời gian mở cửa `unlockDurationMs: 3000`.
    - Thẻ bị khóa (`BLOCKED`) $	o$ Trả về `allowed: false`, `reasonCode: "CARD_BLOCKED"`.
    - Thẻ bị thu hồi (`REVOKED`) $	o$ Trả về `allowed: false`, `reasonCode: "CARD_REVOKED"`.
    - Thẻ không tồn tại trong hệ thống $	o$ Trả về `allowed: false`, `reasonCode: "CARD_NOT_FOUND"`.
  - Tự động ghi bản ghi vào bảng `AccessLog` cho **mọi yêu cầu quẹt thẻ**.
  - Tự động tạo bản ghi cảnh báo trong bảng `SecurityAlert` khi phát hiện thẻ bị khóa hoặc thu hồi cố tình quẹt cửa.
  - Xây dựng bộ test case tự động trên Postman từ TC01 đến TC06.
* **Nghiệm thu:** API xử lý chính xác 100% các trạng thái thẻ và phản hồi kết quả trong thời gian $< 30	ext{ ms}$. Commit Git: `week-06-access-api`.

---

### Tuần 7 — 13/12–19/12: Tích hợp ESP32 gọi API xác thực thời gian thực
* **Kiến thức cần học:** Thư viện `HTTPClient` trên ESP32, phân tích cú pháp JSON bằng `ArduinoJson` v6, cấu hình Timeout kết nối mạng, xử lý mã phản hồi HTTP.
* **Nhiệm vụ thực hành:**
  - Viết hàm gửi HTTP Request từ ESP32: Đóng gói JSON `{ deviceCode, uid, timestamp }`, đính kèm Header `X-Device-Token` gửi tới endpoint `POST /api/device/access/verify`.
  - Phân tích gói tin phản hồi JSON từ Server:
    - Nếu `allowed == true`: Kích chân **GPIO 26** mở Relay trong $3	ext{ giây}$, bật LED xanh **GPIO 27**, còi bíp đôi.
    - Nếu `allowed == false`: Bật LED đỏ **GPIO 33**, còi bíp dài, không mở Relay.
  - Xử lý khi Server bị lỗi hoặc Timeout ($> 1.5	ext{ giây}$): Nhấp nháy LED đỏ báo lỗi kết nối.
  - In log chi tiết quá trình gửi nhận gói tin ra Serial Monitor để kiểm tra độ trễ.
  - Thực hiện bài đo kiểm tra 30 lần quẹt thẻ trực tiếp: Thay đổi trạng thái thẻ từ `ACTIVE` sang `BLOCKED` trên cơ sở dữ liệu và quẹt thẻ ngay để kiểm chứng phản hồi tức thì của phần cứng.
* **Nghiệm thu:** Quẹt thẻ thực tế mở cửa trong vòng $< 200	ext{ ms}$; thay đổi quyền trên máy chủ tác động ngay lập tức tới hành vi của cửa. Commit Git: `week-07-esp32-api-integration`.

---

### Tuần 8 — 20/12–26/12: Xây dựng ứng dụng Desktop Electron MVP
* **Kiến thức cần học:** Kiến trúc Electron (Main Process, Renderer Process, Preload Script), mô hình bảo mật IPC (`contextIsolation: true`, `nodeIntegration: false`), React Component & State Management, gọi REST API từ máy khách Desktop.
* **Nhiệm vụ thực hành:**
  - Khởi tạo dự án `desktop/` sử dụng Electron Forge / Vite + React 18 + TypeScript + TailwindCSS.
  - Cấu hình bảo mật chuẩn công nghiệp trong `main.ts` và `preload.ts`: Cấm hoàn toàn việc expose Node API trực tiếp ra giao diện web.
  - Thiết kế màn hình đăng nhập quản trị viên đơn giản.
  - Xây dựng màn hình Dashboard tổng quan:
    - Thẻ thống kê tổng số cư dân, số thẻ đang hoạt động, số lượt quẹt thẻ trong ngày.
    - Bảng danh sách nhật ký truy cập gần nhất (sử dụng cơ chế REST Polling định kỳ $3 \sim 5	ext{ giây}$ gọi `GET /api/logs` để cập nhật dữ liệu mới).
  - Xây dựng màn hình danh sách cư dân (`Residents`) và danh sách thẻ (`Cards`).
* **Nghiệm thu:** Ứng dụng Desktop khởi động nhanh, giao diện đẹp hiện đại, đọc và hiển thị chính xác dữ liệu thực tế từ máy chủ Backend Fastify. Commit Git: `week-08-electron-mvp`.

---

### Tuần 9 — 27/12–02/01: CRUD Cư dân, Phân quyền RBAC & Cảnh báo bất thường
* **Kiến thức cần học:** Mã hóa mật khẩu với `bcrypt`, phân quyền dựa trên vai trò (RBAC: Admin vs Guard), thuật toán cửa sổ thời gian trượt (Sliding-Window Rule) để phát hiện hành vi quét thẻ bất thường.
* **Nhiệm vụ thực hành:**
  - Hoàn thiện đầy đủ các chức năng quản trị trên Desktop:
    - Thêm cư dân mới, sửa thông tin, xóa cư dân.
    - Cấp phát thẻ mới (**Card Enrollment**): Cho phép nhập tay mã UID hoặc nhận mã UID từ lượt quẹt gần nhất.
    - Khóa thẻ khẩn cấp (`BLOCKED`) và thu hồi thẻ (`REVOKED`) chỉ bằng 1 cú click chuột.
    - Bộ lọc lịch sử truy cập theo ngày, theo cư dân, theo trạng thái thành công/thất bại.
  - Cài đặt quy tắc phát hiện bất thường (Rule-Based Anomaly Detection) trên Backend:
    - **RULE_01:** Phát hiện $\ge 3$ lần quẹt mã thẻ không tồn tại trong vòng $60	ext{ giây}$ $	o$ Tạo cảnh báo mức Warning.
    - **RULE_02:** Phát hiện $\ge 5$ lần quẹt thẻ bị từ chối liên tiếp trong vòng $60	ext{ giây}$ $	o$ Tạo cảnh báo mức High Severity Alert.
  - Xây dựng trang Cảnh báo an ninh (`Alerts`) trên Desktop App: Hiển thị danh sách cảnh báo màu đỏ nhấp nháy, có nút "Xác nhận đã xử lý" (`resolve`).
* **Nghiệm thu:** Vòng đời của thẻ (Cấp phát $	o$ Hoạt động $	o$ Khóa $	o$ Thu hồi) vận hành hoàn hảo; hệ thống phát hiện chính xác hành vi quét thẻ lạ liên tục và tạo cảnh báo an ninh tức thì. Commit Git: `week-09-rbac-alerts`.

---

### Tuần 10 — 03/01–09/01: Cơ chế Offline Whitelist Cache & Log Sync
* **Kiến thức cần học:** Thư viện `Preferences.h` (lưu trữ trên bộ nhớ NVS Flash), hệ thống tệp tin `LittleFS`, cơ chế hàng đợi bất đồng bộ (Queue), tính chất Idempotent khi đồng bộ dữ liệu.
* **Nhiệm vụ thực hành:**
  - Đồng bộ danh sách thẻ hợp lệ (Whitelist Cache) từ Server về lưu vào bộ nhớ **NVS Flash** của ESP32 (lưu tối đa 50 thẻ hoạt động gần nhất).
  - Lập trình chế độ ngoại tuyến (**Offline Mode**):
    - Khi gửi HTTP request xác thực mà bị mất kết nối Wi-Fi hoặc Server quá thời gian timeout $1.5	ext{ giây}$:
    - ESP32 tự động chuyển sang kiểm tra mã UID trong NVS Flash Cache.
    - Nếu thẻ có trong Cache $	o$ Kích mở Relay mở cửa bình thường; tạo một bản ghi sự kiện gồm `{ eventId, deviceCode, uid, timestamp, result: "OFFLINE_GRANTED" }` và ghi tuần tự vào file hàng đợi trên **LittleFS**.
    - Nếu thẻ không có trong Cache $	o$ Từ chối mở cửa.
  - Xây dựng endpoint `POST /api/device/logs/sync` trên Backend: Tiếp nhận danh sách các bản ghi sự kiện ngoại tuyến và xử lý Idempotent dựa trên `eventId` để **không bao giờ bị ghi đè hay tạo bản ghi trùng lặp**.
  - Kiểm thử thực tế: Rút dây mạng router Wi-Fi, quẹt thẻ hợp lệ mở cửa, cắm lại dây mạng $	o$ ESP32 tự động kết nối lại và đẩy bản ghi ngoại tuyến lên Server đồng bộ đầy đủ.
* **Nghiệm thu:** Rút mạng cửa vẫn mở được cho cư dân có trong cache; cắm mạng lại dữ liệu lịch sử tự động xuất hiện đầy đủ trên màn hình Desktop mà không bị mất hay trùng bản ghi nào. Commit Git: `week-10-offline-sync`.

---

### Tuần 11 — 10/01–15/01: Đo đạc chỉ số, Tài liệu, Video & Đóng gói .exe Release
* **Nhiệm vụ thực hành:**
  - Chạy toàn bộ bộ kiểm thử nghiệm thu từ TC01 đến TC10.
  - Đo đạc thực nghiệm các chỉ số hiệu năng (đo tối thiểu 30 lần):
    - Độ trễ mở cửa khi Online (từ lúc quẹt thẻ đến khi Relay đóng): Giá trị trung bình, min, max.
    - Độ trễ mở cửa khi Offline qua Flash Cache.
    - Thời gian tự động kết nối lại Wi-Fi khi có mạng.
    - Tỷ lệ đồng bộ nhật ký ngoại tuyến thành công ($100\%$).
  - Chụp ảnh chất lượng cao mô hình mạch phần cứng thực tế, vẽ lại sơ đồ kiến trúc hệ thống và sơ đồ khối rõ ràng.
  - Hoàn thiện file `README.md` chính thức của repository: Giới thiệu đề tài, bảng pinout, hướng dẫn cài đặt từ A đến Z, ảnh chụp giao diện Desktop.
  - Kiểm tra an toàn bảo mật mã nguồn: Đảm bảo file `.env` chứa mật khẩu Wi-Fi, token bí mật đã được thêm vào `.gitignore`, tạo file mẫu `.env.example`.
  - Sử dụng Electron Builder đóng gói toàn bộ ứng dụng Desktop thành **file cài đặt `.exe` độc lập**.
  - Quay video demo chất lượng cao thời lượng $3 \sim 5	ext{ phút}$ trình diễn toàn bộ các tính năng: Quẹt thẻ Online, khóa thẻ từ xa, rút mạng thử nghiệm Offline, tạo cảnh báo Anomaly và mở phần mềm Desktop.
  - Tạo Git Tag chính thức: `v1.0.0` trên GitHub. Nghiệm thu hoàn tất xuất sắc toàn bộ Dự án 1 trước kỳ nghỉ Tết Nguyên Đán!
* **Nghiệm thu:** Bộ cài đặt `.exe` chạy độc lập mượt mà; video demo hoàn chỉnh; mã nguồn sạch đẹp sẵn sàng đưa vào CV ứng tuyển.

---

## 3. CHECKLIST NGHIỆM THU TRƯỚC KHI RELEASE V1.0.0

```text
[x] Phần cứng: Module RC522 chạy bằng nguồn 3.3V ổn định; dây cắm SPI ngắn dưới 15cm.
[x] Phần cứng: Relay sử dụng GPIO 26, không bị tự kích giật tiếp điểm khi ESP32 khởi động lại.
[x] Phần cứng: Nút Exit dùng GPIO 32, Buzzer dùng GPIO 25, LED dùng GPIO 27/33 (Không dính Strapping Pins).
[x] Bảo mật: Không commit mật khẩu Wi-Fi, Device Token bí mật, file Database thật lên GitHub công khai.
[x] Nghiệp vụ thẻ: Thẻ ACTIVE / BLOCKED / REVOKED / UNKNOWN phản hồi chính xác 100%.
[x] Audit Log: Mọi sự kiện quẹt thẻ thành công hay thất bại đều được ghi nhận vào nhật ký truy cập.
[x] Báo động: Quét thẻ sai liên tục >= 5 lần kích hoạt cảnh báo an ninh High Severity Alert.
[x] Chế độ Offline: Chỉ cấp quyền mở cửa cho các thẻ hợp lệ đã được lưu trong NVS Whitelist Cache.
[x] Đồng bộ dữ liệu: Khi mạng phục hồi, nhật ký sự kiện ngoại tuyến từ LittleFS đồng bộ lên Server không bị trùng lặp.
[x] Phần mềm Desktop: Cấu hình an toàn contextIsolation=true và nodeIntegration=false.
[x] Thực nghiệm: Thực hiện tối thiểu 30 lần đo đạc độ trễ phản hồi online và offline, tính giá trị trung bình.
[x] Hồ sơ: Đầy đủ file README.md, ảnh chụp mạch, sơ đồ khối, video demo thực tế và file cài đặt .exe.
```

---

## 4. BẢNG TỔNG HỢP BẪY PHẦN CỨNG & CÁCH PHÒNG TRÁNH

| Bẫy phần cứng thực tế | Hiện tượng gặp phải | Nguyên nhân kỹ thuật | Giải pháp kỹ thuật chuẩn xác |
| :--- | :--- | :--- | :--- |
| **Bẫy chân Strapping Pin (GPIO 2, 4, 12, 15)** | ESP32 bị treo cứng bootloop, không nạp được code qua USB | GPIO 0, 2, 4, 12, 15 là chân cấu hình chế độ nạp bootloader; nội trở của Relay/Buzzer kéo sai điện áp lúc khởi động. | **Chuyển toàn bộ linh kiện sang các chân I/O an toàn:** Relay $	o$ **GPIO 26**, Buzzer $	o$ **GPIO 25**, LED $	o$ **GPIO 27/33**, Nút Exit $	o$ **GPIO 32**, RC522 SS $	o$ **GPIO 21**. |
| **Bẫy cắm nhầm nguồn 5V vào RC522** | Cháy chip MFRC522, module nóng ran, ngửi thấy mùi khét | Chip NXP MFRC522 sản xuất theo công nghệ CMOS chỉ chịu điện áp tối đa $3.6\text{V}$. | **Chỉ cắm chân VCC vào cọc 3V3 của ESP32**. Kiểm tra kỹ bằng đồng hồ vạn năng VOM trước khi bật nguồn. |
| **Bẫy xung ngược cảm ứng Back-EMF** | ESP32 bị reset liên tục (Brownout), màn hình Desktop mất kết nối | Khi Relay ngắt dòng khóa Solenoid 12V, cuộn cảm sinh điện áp ngược $V = -L rac{di}{dt}$ vọt lên $> 200\text{V}$ truyền ngược qua mass. | **Bắt buộc mắc Diode Flyback 1N4007** song song ngược cực tính với cuộn dây khóa. Sử dụng module Relay có cách ly quang Opto PC817. |
| **Bẫy dội phím nút Exit Button** | Nhấn nút mở cửa 1 lần nhưng hệ thống ghi nhận mở/đóng liên tục 5-10 lần | Tiếp điểm kim loại của nút bấm cơ bị nảy lò xo trong khoảng $5 \sim 20\text{ ms}$ trước khi ổn định. | **Lọc dội phần cứng:** Mắc tụ gốm $100\text{ nF}$ song song với nút. **Phần mềm:** Sử dụng timer `millis()` lọc ngưỡng $50\text{ ms}$. |
| **Bẫy dây cắm SPI quá dài** | Module RC522 chập chờn, lúc đọc được thẻ lúc báo lỗi `Communication Error` | Bus SPI chạy xung nhịp $10\text{ MHz}$ rất nhạy cảm với điện dung ký sinh và nhiễu sóng khi dùng dây cắm Dupont dài $> 20\text{ cm}$. | Giữ chiều dài dây nối SPI ngắn dưới $15\text{ cm}$. Đi dây mass GND kẹp cạnh dây clock SCK để triệt tiêu nhiễu xuyên âm (Crosstalk). |

---

## 5. BẢNG MÃ LỖI HỆ THỐNG (SYSTEM ERROR CODES)

```text
+-----------+-----------------------------------+---------------------------------------------------------------+
| Mã Lỗi    | Tên Lỗi Kỹ Thuật                  | Nguyên nhân & Hướng khắc phục                                 |
+-----------+-----------------------------------+---------------------------------------------------------------+
| ERR_101   | RC522_INIT_FAILED                 | Không tìm thấy module RC522 qua SPI (Kiểm tra chân SS/SCK/VCC)|
| ERR_102   | CARD_READ_TIMEOUT                 | Thẻ lướt qua quá nhanh hoặc thẻ bị hỏng chip RFID bên trong    |
| ERR_103   | AUTH_KEY_FAILED                   | Sai mã khóa Key A/Key B khi cố gắng đọc dữ liệu Sector Trailer |
| ERR_201   | WIFI_CONNECTION_LOST              | Mất kết nối tới Access Point Wi-Fi (Tự động kích hoạt Offline)|
| ERR_202   | SERVER_TIMEOUT                    | Server backend không phản hồi trong 1.5s (Kích hoạt Fallback)  |
| ERR_301   | CARD_NOT_FOUND_IN_DB              | Mã UID chưa được cấp phát trong hệ thống (Báo từ chối truy cập)|
| ERR_302   | CARD_REVOKED_OR_BLOCKED           | Thẻ đã bị quản trị viên khóa khẩn cấp do báo mất thẻ          |
| ERR_303   | CARD_EXPIRED                      | Thẻ cư dân đã hết hạn hợp đồng thuê nhà                        |
| ERR_401   | BRUTE_FORCE_DETECTED              | Phát hiện quẹt thẻ sai liên tục >= 5 lần/phút (Hú còi báo động)|
| ERR_501   | NVS_FLASH_FULL                    | Bộ nhớ NVS Flash đầy dung lượng danh sách Whitelist Cache     |
+-----------+-----------------------------------+---------------------------------------------------------------+
```

---

## 6. BỘ CÂU HỎI PHỎNG VẤN KỸ THUẬT KINH ĐIỂN

1. **Câu hỏi:** *Tại sao bạn lại chọn các chân GPIO 26, 25, 27, 33, 32 và tránh các chân GPIO 0, 2, 4, 12, 15 khi điều khiển Relay, Buzzer và Nút nhấn?*  
   **Trả lời:** Trên vi điều khiển ESP32, các chân GPIO 0, 2, 4, 12, 15 là các chân cấu hình khởi động (**Strapping Pins**). Trong quá trình bật nguồn (Power-on Reset), vi xử lý sẽ kiểm tra mức điện áp logic trên các chân này để quyết định chế độ boot (chạy code từ Flash hay nạp firmware qua UART, điện áp Flash 3.3V hay 1.8V). Nếu gắn Relay vào GPIO 4 hay Buzzer vào GPIO 2, nội trở của các linh kiện này có thể vô tình kéo chân đó xuống LOW hoặc lên HIGH, khiến ESP32 bị treo cứng (Bootloop) hoặc làm Relay tự giật đóng ngắt ngoài ý muốn. Do đó, việc chuyển toàn bộ ngoại vi công suất sang các chân I/O tiêu chuẩn như GPIO 26, 25, 27, 33, 32 là giải pháp chuẩn xác của kỹ sư phần cứng chuyên nghiệp để đảm bảo vi điều khiển luôn khởi động an toàn 100%.

2. **Câu hỏi:** *Thẻ MIFARE Classic 1K có thể bị sao chép (Clone) bằng các thẻ Magic Card giá rẻ. Bạn giải quyết vấn đề bảo mật này như thế nào trong đồ án?*  
   **Trả lời:** Trong đồ án, em không tuyên bố "chống clone tuyệt đối" bằng mã UID vì tiêu chuẩn MIFARE Classic 1K đã có những giới hạn bảo mật được công bố. Thay vào đó, em thiết kế hệ thống theo mô hình phòng thủ đa tầng (**Defense-in-Depth**):
   - Mọi quyền hạn mở cửa đều do máy chủ Backend quản lý tập trung và xác thực thời gian thực qua trạng thái thẻ (`ACTIVE`, `BLOCKED`, `REVOKED`).
   - Mọi lượt quẹt thẻ đều được ghi lại trong Access Audit Log.
   - Hệ thống tích hợp thuật toán phát hiện bất thường (Rule-Based Anomaly Detection): Nếu phát hiện hành vi quét thẻ lạ hoặc thẻ bị từ chối liên tiếp $\ge 5$ lần trong 60 giây, hệ thống tự động khóa cổng đọc và kích hoạt cảnh báo an ninh mức cao.
   - Ở bản nâng cao, hệ thống hỗ trợ ghi một Rolling Counter động vào Data Block của thẻ để phát hiện các phiên bản thẻ sao chép cũ bị rollback.

3. **Câu hỏi:** *Tại sao khi điều khiển khóa Solenoid 12V bằng Relay lại bắt buộc phải có Diode Flyback mắc song song với cuộn dây?*  
   **Trả lời:** Cuộn dây khóa Solenoid là tải thuần cảm có độ tự cảm $L$. Khi Relay ngắt điện, dòng điện giảm đột ngột về 0 trong thời gian cực ngắn ($dt 	o 0$). Theo định luật tự cảm Faraday-Lenz: $V_{	ext{kick}} = -L rac{di}{dt}$, cuộn cảm sẽ phóng ra một sức điện động cảm ứng ngược cực lớn (Back-EMF) từ $200	ext{V}$ đến $400	ext{V}$. Xung áp nhọn này sinh ra tia lửa điện đánh cháy tiếp điểm cơ khí của Relay, phát ra bức xạ điện từ (EMI) truyền ngược qua đường mass làm ESP32 bị khởi động lại (Brownout Reset). Diode Flyback 1N4007 mắc ngược cực tính sẽ tạo thành một mạch vòng kín cho dòng điện cảm ứng tự tuần hoàn và tiêu tán năng lượng từ trường $rac{1}{2}LI^2$ thành nhiệt một cách an toàn.

4. **Câu hỏi:** *Hệ thống của bạn xử lý thế nào khi đường truyền Wi-Fi bị đứt hoặc máy chủ Backend bị treo?*  
   **Trả lời:** Hệ thống được thiết kế với cơ chế chịu lỗi ngoại tuyến (Offline Caching) độc lập:
   - Khi còn mạng, danh sách các thẻ hợp lệ gần nhất được đồng bộ và lưu vào bộ nhớ **NVS Flash** của ESP32.
   - Khi mất mạng (phát hiện qua cơ chế timeout $1.5	ext{ giây}$), ESP32 tự động chuyển sang chế độ Offline: Tra cứu cục bộ trong NVS Flash, nếu thẻ hợp lệ thì vẫn mở cửa bình thường cho cư dân.
   - Sự kiện mở cửa ngoại tuyến được lưu vào hàng đợi trên bộ nhớ **LittleFS** (để tránh làm mòn Flash NVS).
   - Khi mạng Wi-Fi phục hồi, tiến trình nền tự động gửi toàn bộ hàng đợi sự kiện lên endpoint `/api/device/logs/sync` với cơ chế Idempotent dựa trên `eventId` để cập nhật Database mà không bao giờ bị trùng lặp dữ liệu.
