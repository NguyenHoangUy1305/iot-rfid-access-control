# 📘 SỔ TAY KỸ THUẬT & LỘ TRÌNH THỰC HIỆN CHI TIẾT DỰ ÁN 1
## Đề tài: Hệ thống kiểm soát cửa RFID & IoT tích hợp Desktop App (ESP32 + Fastify + Electron)

> **Tác giả:** Kỹ sư IoT & Hệ thống nhúng (NguyenHoangUy1305)  
> **Repository:** [NguyenHoangUy1305/iot-rfid-access-control](https://github.com/NguyenHoangUy1305/iot-rfid-access-control)  
> **Thời gian:** 10 - 12 tuần (Khởi động chính thức: 01/11/2026 - Hoàn thành trước Tết: 15/01/2027)  
> **Mục tiêu:** Cung cấp lộ trình hành động chi tiết từng tuần, danh mục công việc từng ngày, các bẫy phần cứng thực tế, bảng mã lỗi và câu hỏi phỏng vấn kỹ thuật liên quan đến hệ thống kiểm soát truy cập cửa thông minh.

---

## MỤC LỤC
1. [PHẦN 1: TỔNG QUAN LỘ TRÌNH & MỤC TIÊU CỐT LÕI](#phần-1-tổng-quan-lộ-trình--mục-tiêu-cốt-lõi)
2. [PHẦN 2: LỘ TRÌNH CHI TIẾT 10 TUẦN TRIỂN KHAI (01/11/2026 - 15/01/2027)](#phần-2-lộ-trình-chi-tiết-10-tuần-triển-khai)
   - Tuần 1: Cài đặt công cụ, Git & Khảo sát phần cứng IO (01/11 - 07/11/2026)
   - Tuần 2: Nối dây & Lập trình Module RC522 SPI (08/11 - 14/11/2026)
   - Tuần 3: Mạch công suất Relay, Solenoid Lock & Diode Flyback (15/11 - 21/11/2026)
   - Tuần 4: Driver Wi-Fi Reconnect & Finite State Machine (22/11 - 28/11/2026)
   - Tuần 5: Thiết kế Database Prisma + Fastify REST Backend (29/11 - 05/12/2026)
   - Tuần 6: Tích hợp WebSocket thời gian thực & Xác thực thẻ (06/12 - 12/12/2026)
   - Tuần 7: Xây dựng ứng dụng Desktop Quản trị (Electron + React) (13/12 - 19/12/2026)
   - Tuần 8: Hoàn thiện CRUD Cư dân, Phân quyền & Báo cáo (20/12 - 26/12/2026)
   - Tuần 9: Xây dựng thuật toán Anomaly Detection (Phát hiện Brute-Force) (27/12 - 02/01/2027)
   - Tuần 10: Cơ chế Offline Cache Flash NVS & Đóng gói sản phẩm (03/01 - 15/01/2027)
3. [PHẦN 3: BẢNG TỔNG HỢP BẪY PHẦN CỨNG & CÁCH KHẮC PHỤC (HARDWARE GOTCHAS)](#phần-3-bảng-tổng-hợp-bẫy-phần-cứng--cách-khắc-phục)
4. [PHẦN 4: BẢNG MÃ LỖI HỆ THỐNG (SYSTEM ERROR CODES)](#phần-4-bảng-mã-lỗi-hệ-thống)
5. [PHẦN 5: BỘ CÂU HỎI PHỎNG VẤN KỸ THUẬT DÀNH CHO DỰ ÁN 1](#phần-5-bộ-câu-hỏi-phỏng-vấn-kỹ-thuật)

---

# PHẦN 1: TỔNG QUAN LỘ TRÌNH & MỤC TIÊU CỐT LÕI

Dự án được thiết kế theo mô hình xoắn ốc (Spiral Model), chia làm 4 trụ cột kỹ thuật vững chắc:
1. **Firmware C++ (ESP32):** Điều khiển ngoại vi tốc độ cao qua SPI, xử lý ngắt nút bấm (Hardware/Software Debounce), quản lý kết nối Wi-Fi tự động và lưu trữ Offline Cache trong NVS Flash.
2. **Backend Server (Node.js + Fastify + TypeScript):** Kiến trúc hướng dịch vụ (Service-Oriented), xử lý xác thực quyền hạn thẻ trong $< 50\text{ ms}$, quản lý giao dịch bằng Prisma ORM với SQLite/PostgreSQL, truyền phát sự kiện thời gian thực qua WebSocket.
3. **Desktop App (Electron + React + TailwindCSS):** Giao diện quản trị cư dân, cấp phát thẻ RFID mới, giám sát trực quan trạng thái mở/khóa cửa theo thời gian thực và cấu hình hệ thống từ xa.
4. **An ninh & Độ tin cậy (Security & High Reliability):** Phòng thủ chống sao chép thẻ (Clone Card Protection), chống tấn công dò mã (Brute-force Anomaly Detection) và mạch bảo vệ chống xung ngược cảm ứng Back-EMF.

---

# PHẦN 2: LỘ TRÌNH CHI TIẾT 10 TUẦN TRIỂN KHAI

### Tuần 1: Cài đặt công cụ, Git & Khảo sát phần cứng IO (01/11 - 07/11/2026)
* **Mục tiêu kỹ thuật:** Thiết lập hoàn chỉnh môi trường phát triển trên máy tính, kiểm tra phần cứng ESP32 và các linh kiện xuất nhập (IO) cơ bản.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (01/11):* Kiểm tra danh mục linh kiện Shopee đã nhận; tạo repo GitHub `NguyenHoangUy1305/iot-rfid-access-control`.
  * *Thứ Hai (02/11):* Cài đặt VS Code, Git, PlatformIO IDE, Node.js v20 LTS, Postman.
  * *Thứ Ba (03/11):* Cài đặt driver CP210x / CH340 cho ESP32. Tạo project PlatformIO đầu tiên (`framework = arduino`, `board = esp32doit-devkit-v1`).
  * *Thứ Tư (04/11):* Viết chương trình nạp thử nghiệm nháy LED chân GPIO 2; làm quen với cấu trúc `setup()` và `loop()`.
  * *Thứ Năm (05/11):* Đấu nối breadboard: LED xanh (GPIO 16), LED đỏ (GPIO 17) qua điện trở hạn dòng $220\Omega$; Active Buzzer (GPIO 2).
  * *Thứ Sáu (06/11):* Viết các hàm điều khiển âm thanh: `beepSuccess()` (2 tiếng bíp ngắn $80\text{ ms}$), `beepError()` (1 tiếng bíp dài $600\text{ ms}$).
  * *Thứ Bảy (07/11):* Đấu nối Module Relay 5V (GPIO 4); kiểm tra tiếng đóng/ngắt tiếp điểm cơ khí "tạch tạch". Commit code Tuần 1 lên GitHub.
* **Tiêu chí nghiệm thu:** Nạp code thành công vào ESP32, bấm phím điều khiển LED sáng, còi kêu đúng âm sắc và Relay đóng ngắt chuẩn xác.

---

### Tuần 2: Nối dây & Lập trình Module RC522 SPI (08/11 - 14/11/2026)
* **Mục tiêu kỹ thuật:** Làm chủ bus giao tiếp SPI, giao tiếp thành công với module đọc thẻ MFRC522 ở tần số 13.56 MHz, đọc UID và giải mã cấu trúc thẻ MIFARE Classic 1K.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (08/11):* Đấu nối 7 chân RC522 vào ESP32 (SDA $\to$ GPIO 5, SCK $\to$ GPIO 18, MOSI $\to$ GPIO 23, MISO $\to$ GPIO 19, RST $\to$ GPIO 22, 3V3, GND). ⚠️ *Tuyệt đối kiểm tra kỹ chân 3.3V*.
  * *Thứ Hai (09/11):* Cài đặt thư viện `miguelbalboa/MFRC522` trên PlatformIO; chạy ví dụ `DumpInfo.ino` để kiểm tra firmware của module RC522.
  * *Thứ Ba (10/11):* Viết hàm trích xuất chuỗi mã UID dạng Hexadecimal (`8A3B21F0`) từ mảng bytes `mfrc522.uid.uidByte`.
  * *Thứ Tư (11/11):* Lập trình xác thực khóa bí mật `Key A` (mặc định `0xFF 0xFF 0xFF 0xFF 0xFF 0xFF`) để đọc dữ liệu từ Sector 1 Block 4.
  * *Thứ Năm (12/11):* Thử nghiệm ghi chuỗi mã định danh người dùng vào Block dữ liệu; đọc lại để kiểm tra tính toàn vẹn.
  * *Thứ Sáu (13/11):* Đấu nối nút nhấn kim loại Exit Button vào GPIO 15 (kéo trở Pull-up nội); lập trình hàm xử lý chống dội (Software Debounce) sử dụng `millis()`.
  * *Thứ Bảy (14/11):* Tích hợp: Quẹt thẻ trắng $\to$ còi bíp ngắn $\to$ in mã UID ra Serial Monitor $\to$ đóng ngắt Relay mô phỏng mở cửa. Commit code Tuần 2.
* **Tiêu chí nghiệm thu:** Đọc được chính xác 100% mã UID của 3 thẻ mẫu khác nhau; nút nhấn Exit bấm êm ái, không bị hiện tượng kích hoạt đúp do dội phím.

---

### Tuần 3: Mạch công suất Relay, Solenoid Lock & Diode Flyback (15/11 - 21/11/2026)
* **Mục tiêu kỹ thuật:** Xây dựng phần cứng công suất thực tế; xử lý triệt để bài toán sụt áp và xung ngược cảm ứng Back-EMF khi đóng ngắt cuộn dây khóa 12V.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (15/11):* Hàn song song 1 Diode 1N4007 vào 2 cực cuộn dây khóa Solenoid 12V (Vạch trắng Cathode nối vào $+12\text{V}$).
  * *Thứ Hai (16/11):* Nối tiếp điểm Relay (COM $\to$ GND nguồn 12V, NO $\to$ Dây âm khóa từ). Nguồn dương $+12\text{V}$ cấp thẳng vào khóa.
  * *Thứ Ba (17/11):* Đấu nối nguồn Adapter 12V 2A qua mạch hạ áp Buck LM2596, vặn biến trở tinh chỉnh điện áp ra đúng $5.0\text{V}$ để nuôi ESP32 và Relay.
  * *Thứ Tư (18/11):* Chạy bài test stress: Kích mở khóa liên tục 50 lần (mỗi lần mở 3 giây, nghỉ 2 giây).
  * *Thứ Năm (19/11):* Dùng đồng hồ đo dao động (Oscilloscope hoặc VOM) kiểm tra xem điện áp chân 3.3V của ESP32 có bị sụt áp dưới $3.0\text{V}$ không.
  * *Thứ Sáu (20/11):* Kiểm tra nhiệt độ cuộn dây khóa Solenoid (lưu ý: Solenoid 12V chỉ nên duy trì đóng dòng tối đa $5 \sim 8\text{ giây}$ để tránh nóng cháy cuộn dây).
  * *Thứ Bảy (21/11):* Đóng hộp bảo vệ mô hình phần cứng; cố định dây dẫn bằng dây rút và ống bọc gọn gàng. Commit báo cáo phần cứng Tuần 3.
* **Tiêu chí nghiệm thu:** Hệ thống đóng ngắt khóa 12V trơn tru 50 lần liên tiếp mà ESP32 không bị hiện tượng Brownout Reset (sập nguồn reset lại vi điều khiển).

---

### Tuần 4: Driver Wi-Fi Reconnect & Finite State Machine (22/11 - 28/11/2026)
* **Mục tiêu kỹ thuật:** Xây dựng tầng mạng không dây ổn định cho ESP32; chuyển đổi kiến trúc mã nguồn sang Máy trạng thái hữu hạn (FSM).
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (22/11):* Lập trình module kết nối Wi-Fi sử dụng thư viện `WiFi.h`. Cấu hình IP tĩnh hoặc DHCP với cơ chế lưu thông tin mạng trong NVS.
  * *Thứ Hai (23/11):* Viết hàm kiểm tra và tự động kết nối lại mạng (Auto-Reconnect): Sử dụng non-blocking timer, cứ mỗi 5 giây kiểm tra `WiFi.status()`.
  * *Thứ Ba (24/11):* Tách cấu trúc code firmware thành các module hướng đối tượng: `RFIDManager`, `DoorController`, `NetworkManager`, `BuzzerLED`.
  * *Thứ Tư (25/11):* Xây dựng máy trạng thái `DoorState`: `STATE_LOCKED`, `STATE_UNLOCKED`, `STATE_WAIT_CLOSE`, `STATE_ALARM`.
  * *Thứ Năm (26/11):* Lập trình đồng bộ thời gian thực qua giao thức SNTP (`pool.ntp.org`) để đồng hồ nội của ESP32 khớp chuẩn theo múi giờ GMT+7.
  * *Thứ Sáu (27/11):* Thử nghiệm tắt router Wi-Fi: Kiểm tra xem firmware có bị treo cứng hay vẫn phản hồi nút nhấn Exit bình thường.
  * *Thứ Bảy (28/11):* Bật lại router Wi-Fi: ESP32 phải tự động kết nối lại mạng trong vòng dưới 10 giây mà không cần bấm nút reset vật lý. Commit code Tuần 4.
* **Tiêu chí nghiệm thu:** Firmware chạy liên tục 24h không bị rò rỉ bộ nhớ (Heap Memory ổn định $> 180\text{ KB}$); tự động bắt lại sóng Wi-Fi khi mất mạng.

---

### Tuần 5: Thiết kế Database Prisma + Fastify REST Backend (29/11 - 05/12/2026)
* **Mục tiêu kỹ thuật:** Khởi tạo dự án Backend với Node.js, Fastify Framework, TypeScript và thiết kế cơ sở dữ liệu quan hệ hoàn chỉnh bằng Prisma ORM.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (29/11):* Khởi tạo dự án `server/`, cài đặt `fastify`, `typescript`, `@prisma/client`, `prisma`, `dotenv`, `zod`.
  * *Thứ Hai (30/11):* Thiết kế file `schema.prisma`: Các bảng `Resident`, `Card`, `AccessLog`, `Device`, `SystemConfig`.
  * *Thứ Ba (01/12):* Tạo file migration và chạy `prisma migrate dev`. Viết file `seed.ts` nạp sẵn 10 bản ghi cư dân mẫu và 5 thẻ RFID kiểm thử.
  * *Thứ Tư (02/12):* Xây dựng endpoint cốt lõi: `POST /api/access/verify`
    - Nhận vào: `{ deviceCode, uid, counter, timestamp }`
    - Xử lý: Tra cứu trạng thái thẻ (`ACTIVE`, `LOCKED`, `EXPIRED`), kiểm tra phân quyền cửa.
    - Phản hồi: `{ status: "GRANTED" | "DENIED", reason, residentName, duration }` trong $< 30\text{ ms}$.
  * *Thứ Năm (03/12):* Viết middleware xác thực thiết bị `x-device-token` để bảo vệ API không bị gọi trái phép.
  * *Thứ Sáu (04/12):* Viết các API CRUD: `GET /api/cards`, `POST /api/cards`, `PUT /api/cards/:id/block`, `GET /api/logs`.
  * *Thứ Bảy (05/12):* Dùng Postman tạo test suite tự động kiểm thử toàn bộ các kịch bản: Thẻ hợp lệ, thẻ lạ, thẻ bị khóa, thẻ hết hạn. Commit code Tuần 5.
* **Tiêu chí nghiệm thu:** API phản hồi với độ trễ trung bình $< 25\text{ ms}$; kiểm thử tự động trên Postman đạt 100% Pass.

---

### Tuần 6: Tích hợp WebSocket thời gian thực & Xác thực thẻ (06/12 - 12/12/2026)
* **Mục tiêu kỹ thuật:** Kết nối hoàn chỉnh giữa phần cứng ESP32 và máy chủ Backend qua mạng Wi-Fi nội bộ; truyền phát sự kiện quẹt thẻ tức thời lên máy tính qua WebSocket.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (06/12):* Cài đặt `@fastify/websocket` trên server; tạo kênh WebSocket `/ws/events` để đẩy luồng sự kiện (Event Streaming).
  * *Thứ Hai (07/12):* Cài đặt thư viện `ArduinoJson` v6 trên ESP32; viết hàm đóng gói gói tin JSON từ dữ liệu quẹt thẻ.
  * *Thứ Ba (08/12):* Sử dụng `HTTPClient` của ESP32 gửi request `POST /api/access/verify` tới IP máy chủ backend.
  * *Thứ Tư (09/12):* ESP32 giải mã phản hồi JSON từ Server:
    - Nếu `GRANTED`: Kích chân GPIO 4 mở Relay trong 5 giây, bật LED xanh, còi bíp đôi.
    - Nếu `DENIED`: Bật nhấp nháy LED đỏ 3 lần, còi bíp dài, không mở khóa.
  * *Thứ Năm (10/12):* Server bắt sự kiện quẹt thẻ thành công/thất bại và phát tin nhắn WebSocket `ACCESS_EVENT` đến tất cả các client đang kết nối.
  * *Thứ Sáu (11/12):* Tối ưu hóa Keep-Alive TCP giữa ESP32 và Server để giảm độ trễ bắt tay mạng (Handshake Latency).
  * *Thứ Bảy (12/12):* Đo đạc tổng độ trễ từ lúc chạm thẻ vào đầu đọc RC522 đến khi khóa cơ rút chốt mở: Mục tiêu $< 250\text{ ms}$. Commit code Tuần 6.
* **Tiêu chí nghiệm thu:** Quẹt thẻ thực tế mở cửa ngay lập tức trong chớp mắt; log hiển thị chuẩn xác trên terminal của Server.

---

### Tuần 7: Xây dựng ứng dụng Desktop Quản trị (Electron + React) (13/12 - 19/12/2026)
* **Mục tiêu kỹ thuật:** Xây dựng ứng dụng phần mềm Desktop chuyên nghiệp với Electron, React 18, TypeScript và TailwindCSS.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (13/12):* Khởi tạo dự án `desktop/` sử dụng Electron Forge / Vite + React + TypeScript.
  * *Thứ Hai (14/12):* Cấu hình an toàn cho Electron: Bật `contextIsolation: true`, tắt `nodeIntegration: false`, thiết lập `preload.ts` làm cầu nối IPC an toàn.
  * *Thứ Ba (15/12):* Thiết kế layout Dashboard hiện đại chuẩn UI/UX: Thanh bên Sidebar điều hướng, thanh trạng thái kết nối Server (Online/Offline), bảng hiển thị nhanh thống kê.
  * *Thứ Tư (16/12):* Kết nối WebSocket từ React Desktop vào Backend Fastify: Hiển thị bảng nhật ký quẹt thẻ (Live Access Feed) cập nhật theo thời gian thực không cần F5.
  * *Thứ Năm (17/12):* Hiển thị avatar cư dân, phòng ban, thời gian chính xác từng giây và trạng thái thẻ (Thành công: thẻ xanh, Từ chối: thẻ đỏ).
  * *Thứ Sáu (18/12):* Xây dựng nút bấm "MỞ CỬA TỪ XA KHẨN CẤP" trên Desktop: Nhấn nút gửi lệnh xuống Server $	o$ Server gửi lệnh mở khóa ngay lập tức.
  * *Thứ Bảy (19/12):* Kiểm thử giao diện trên Windows 11; tinh chỉnh animation mượt mà. Commit code Tuần 7.
* **Tiêu chí nghiệm thu:** Phần mềm Desktop khởi động dưới 2 giây; nhận thông báo quẹt thẻ từ ESP32 hiển thị trên màn hình máy tính chỉ sau $0.05\text{ giây}$.

---

### Tuần 8: Hoàn thiện CRUD Cư dân, Phân quyền & Báo cáo (20/12 - 26/12/2026)
* **Mục tiêu kỹ thuật:** Hoàn thiện đầy đủ các tính năng quản trị nghiệp vụ thực tế cho ban quản lý tòa nhà/văn phòng.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (20/12):* Màn hình **Quản lý cư dân**: Thêm mới, sửa thông tin, xóa, phân loại theo căn hộ / phòng ban.
  * *Thứ Hai (21/12):* Chức năng **Cấp thẻ mới (Card Enrollment)**: Đưa đầu đọc vào chế độ học thẻ (Learning Mode), quẹt thẻ mới để tự động điền mã UID vào form.
  * *Thứ Ba (22/12):* Chức năng **Khóa thẻ khẩn cấp (Card Revocation)**: Khi cư dân báo mất thẻ, quản trị viên bấm nút "Khóa thẻ", hệ thống lập tức từ chối thẻ này trong lần quẹt tới.
  * *Thứ Tư (23/12):* Màn hình **Tra cứu lịch sử truy cập (Audit Logs)**: Lọc lịch sử theo khoảng ngày, theo tên cư dân, theo trạng thái thành công/thất bại.
  * *Thứ Năm (24/12):* Tính năng **Xuất báo cáo (Export Excel / CSV)**: Cho phép xuất danh sách điểm danh và lịch sử quẹt thẻ ra file Excel chỉ bằng 1 cú click.
  * *Thứ Sáu (25/12):* Phân quyền tài khoản quản trị (Role-Based Access Control): Phân biệt tài khoản Admin (toàn quyền) và Guard (bảo vệ chỉ xem log).
  * *Thứ Bảy (26/12):* Kiểm tra tính toàn vẹn dữ liệu cơ sở dữ liệu khi thực hiện nhiều thao tác thêm/xóa/sửa liên tục. Commit code Tuần 8.
* **Tiêu chí nghiệm thu:** Thao tác cấp phát và khóa thẻ phản ánh ngay lập tức xuống phần cứng; xuất file Excel đầy đủ thông tin chuẩn font Tiếng Việt UTF-8.

---

### Tuần 9: Xây dựng thuật toán Anomaly Detection (Phát hiện Brute-Force) (27/12 - 02/01/2027)
* **Mục tiêu kỹ thuật:** Xây dựng hệ thống phòng vệ an ninh chủ động, phát hiện các hành vi quét thẻ bất thường, tấn công dò mã hoặc cạy cửa trái phép.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (27/12):* Phân tích các mô hình tấn công kiểm soát cửa: Quẹt dò mã thẻ lạ liên tục, quẹt thẻ hợp lệ ngoài giờ quy định, cửa bị giữ mở quá lâu (Door Held Open).
  * *Thứ Hai (28/12):* Xây dựng module `AnomalyDetector` trên Backend: Sử dụng thuật toán cửa sổ thời gian trượt (Sliding Time Window).
  * *Thứ Ba (29/12):* Cài đặt quy tắc bảo vệ:
    - Nếu 1 đầu đọc nhận liên tiếp **> 5 lần quẹt thẻ bị từ chối trong vòng 60 giây**:
    - Server tự động gắn cờ báo động đỏ `SECURITY_ALERT_BRUTE_FORCE`.
  * *Thứ Tư (30/12):* Khi phát hiện bất thường: Server gửi tín hiệu hạ lệnh cho ESP32 hú còi báo động liên tục $30\text{ giây}$ và khóa tạm thời đầu đọc $3\text{ phút}$.
  * *Thứ Năm (31/12):* Trên ứng dụng Desktop: Bật hộp thông báo cảnh báo an ninh màu đỏ nhấp nháy toàn màn hình kèm âm thanh còi hú báo động; bắt buộc quản trị viên phải bấm nút "Xác nhận đã xử lý" mới tắt báo động.
  * *Thứ Sáu (01/01/2027):* Tích hợp gửi thông báo khẩn cấp qua Telegram Bot tới điện thoại của ban quản lý.
  * *Thứ Bảy (02/01/2027):* Thử nghiệm tấn công mô phỏng: Cầm thẻ lạ quẹt liên tục 6 lần để kích hoạt hệ thống phòng thủ. Commit code Tuần 9.
* **Tiêu chí nghiệm thu:** Hệ thống phát hiện chính xác 100% các hành vi quét thẻ bất thường; còi báo động tại cửa hú to và màn hình Desktop cảnh báo tức thì.

---

### Tuần 10: Cơ chế Offline Cache Flash NVS & Đóng gói sản phẩm (03/01 - 15/01/2027)
* **Mục tiêu kỹ thuật:** Hoàn thiện cơ chế chịu lỗi ngoại tuyến (Offline Caching) trên Flash NVS của ESP32; đóng gói hoàn chỉnh phần mềm Desktop và nghiệm thu đồ án trước Tết Nguyên Đán.
* **Kế hoạch hành động từng ngày:**
  * *Chủ Nhật (03/01):* Tìm hiểu thư viện `Preferences.h` (lưu trữ Key-Value trên bộ nhớ NVS Flash không bay hơi của ESP32).
  * *Thứ Hai (04/01):* Viết hàm đồng bộ danh sách thẻ hợp lệ (Local Whitelist Hash Table) từ Server về lưu vào NVS Flash mỗi khi khởi động hoặc có thẻ mới.
  * *Thứ Ba (05/01):* Lập trình logic Fallback: Khi hàm gửi HTTP request bị timeout (quá $1.5\text{ giây}$ không thấy Server phản hồi) $	o$ Tự động chuyển sang chế độ `OFFLINE_MODE`.
  * *Thứ Tư (06/01):* Trong chế độ Offline: Tra cứu nhanh mã UID trong bộ nhớ Flash nội vi; nếu có $	o$ mở cửa và lưu sự kiện vào hàng đợi ngoại tuyến `offline_queue`.
  * *Thứ Năm (07/01):* Khi mạng phục hồi: ESP32 tự động xả hàng đợi, gửi bù toàn bộ các bản ghi quẹt thẻ ngoại tuyến lên Server để cập nhật Database.
  * *Thứ Sáu (08/01):* Kiểm thử ngắt mạng thực tế: Rút dây mạng router Wi-Fi 30 phút, quẹt thẻ thử nghiệm mở cửa, cắm lại dây mạng $	o$ kiểm tra dữ liệu đồng bộ đầy đủ trên Server.
  * *Thứ Bảy (09/01):* Tối ưu hóa mã nguồn, dọn dẹp các lệnh `Serial.print()` thừa, kiểm tra rò rỉ bộ nhớ (Memory Leak).
  * *10/01 - 12/01:* Đóng gói ứng dụng Desktop thành bộ cài đặt `.exe` hoàn chỉnh sử dụng Electron Builder.
  * *13/01 - 15/01:* Quay video demo toàn diện (chạy các kịch bản: Online, Offline, Thẻ lạ, Báo động Brute-force, Desktop Management), hoàn tất tài liệu báo cáo kỹ thuật. Nghiệm thu trọn vẹn dự án trước Tết Nguyên Đán!
* **Tiêu chí nghiệm thu:** Đóng gói thành công file cài đặt Desktop `.exe`; hệ thống vận hành ổn định ngoại tuyến và trực tuyến; video demo chuyên nghiệp sẵn sàng đưa vào CV ứng tuyển.

---

# PHẦN 3: BẢNG TỔNG HỢP BẪY PHẦN CỨNG & CÁCH KHẮC PHỤC

| Bẫy phần cứng thực tế | Hiện tượng gặp phải | Nguyên nhân gốc rễ | Giải pháp kỹ thuật chuẩn xác |
| :--- | :--- | :--- | :--- |
| **Bẫy cắm nhầm nguồn 5V vào RC522** | Cháy chip MFRC522, module nóng ran, ngửi thấy mùi khét | Chip NXP MFRC522 sản xuất theo công nghệ CMOS chỉ chịu điện áp tối đa $3.6\text{V}$. | **Chỉ cắm chân VCC vào cọc 3V3 của ESP32**. Kiểm tra kỹ bằng đồng hồ vạn năng VOM trước khi bật nguồn. |
| **Bẫy xung ngược cảm ứng Back-EMF** | ESP32 bị reset liên tục (Brownout), màn hình Desktop mất kết nối | Khi Relay ngắt dòng khóa Solenoid 12V, cuộn cảm sinh điện áp ngược $V = -L rac{di}{dt}$ vọt lên $> 200\text{V}$ truyền ngược qua mass. | **Bắt buộc mắc Diode Flyback 1N4007** song song ngược cực tính với cuộn dây khóa. Sử dụng module Relay có cách ly quang Opto PC817. |
| **Bẫy dội phím nút Exit Button** | Nhấn nút mở cửa 1 lần nhưng hệ thống ghi nhận mở/đóng liên tục 5-10 lần | Tiếp điểm kim loại của nút bấm cơ bị nảy lò xo trong khoảng $5 \sim 20\text{ ms}$ trước khi ổn định. | **Lọc dội phần cứng:** Mắc tụ gốm $100\text{ nF}$ song song với nút. **Phần mềm:** Sử dụng timer `millis()` lọc ngưỡng $50\text{ ms}$. |
| **Bẫy dây cắm SPI quá dài** | Module RC522 chập chờn, lúc đọc được thẻ lúc báo lỗi `Communication Error` | Bus SPI chạy xung nhịp $10\text{ MHz}$ rất nhạy cảm với điện dung ký sinh và nhiễu sóng khi dùng dây cắm Dupont dài $> 20\text{ cm}$. | Giữ chiều dài dây nối SPI ngắn dưới $15\text{ cm}$. Đi dây mass GND kẹp cạnh dây clock SCK để triệt tiêu nhiễu xuyên âm (Crosstalk). |
| **Bẫy sụt áp nguồn cấp LM2596** | Khi khóa Solenoid đóng chốt, đèn LED trên ESP32 nháy tắt rồi khởi động lại | Nguồn Adapter 12V công suất quá yếu (loại 0.5A) bị sụt áp đột ngột khi cuộn hút khóa kéo dòng đỉnh $1.5\text{ A}$. | **Sử dụng Adapter 12V DC tối thiểu 2A**. Mắc thêm tụ điện hóa dung lượng lớn $1000\mu\text{F} / 25\text{V}$ tại đầu vào nguồn 12V. |

---

# PHẦN 4: BẢNG MÃ LỖI HỆ THỐNG (SYSTEM ERROR CODES)

```text
+-----------+-----------------------------------+---------------------------------------------------------------+
| Mã Lỗi    | Tên Lỗi Kỹ Thuật                  | Nguyên nhân & Hướng khắc phục                                 |
+-----------+-----------------------------------+---------------------------------------------------------------+
| ERR_101   | RC522_INIT_FAILED                 | Không tìm thấy module RC522 qua SPI (Kiểm tra chân CS/SCK/VCC)|
| ERR_102   | CARD_READ_TIMEOUT                 | Thẻ lướt qua quá nhanh hoặc thẻ bị hỏng chip RFID bên trong    |
| ERR_103   | AUTH_KEY_FAILED                   | Sai mã khóa Key A/Key B khi cố gắng đọc dữ liệu Sector Trailer |
| ERR_201   | WIFI_CONNECTION_LOST              | Mất kết nối tới Access Point Wi-Fi (Tự động kích hoạt Offline)|
| ERR_202   | SERVER_TIMEOUT                    | Server backend không phản hồi trong 1.5s (Kích hoạt Fallback)  |
| ERR_301   | CARD_NOT_FOUND_IN_DB              | Mã UID chưa được cấp phát trong hệ thống (Báo từ chối truy cập)|
| ERR_302   | CARD_REVOKED_OR_BLOCKED           | Thẻ đã bị quản trị viên khóa khẩn cấp do báo mất thẻ          |
| ERR_303   | CARD_EXPIRED                      | Thẻ cư dân đã hết hạn hợp đồng thuê nhà                        |
| ERR_401   | BRUTE_FORCE_DETECTED              | Phát hiện quẹt thẻ sai liên tục > 5 lần/phút (Hú còi báo động)|
| ERR_501   | NVS_FLASH_FULL                    | Bộ nhớ Flash nội đầy hàng đợi offline (Cần kết nối để đồng bộ)|
+-----------+-----------------------------------+---------------------------------------------------------------+
```

---

# PHẦN 5: BỘ CÂU HỎI PHỎNG VẤN KỸ THUẬT DÀNH CHO DỰ ÁN 1

Dưới đây là 5 câu hỏi phỏng vấn kinh điển mà các nhà tuyển dụng IoT / Embedded Engineer thường hỏi khi thấy dự án RFID Access Control trên CV của bạn, kèm theo câu trả lời chuẩn chỉ:

1. **Câu hỏi:** *Tại sao bạn lại chọn giao tiếp SPI thay vì I2C cho module đọc thẻ RC522?*  
   **Trả lời:** Module RC522 hỗ trợ cả SPI, I2C và UART. Em chọn SPI vì xung nhịp bus SPI đạt tới $10\text{ MHz}$ (vượt trội so với $400\text{ kHz}$ của I2C Fast-mode), giúp thời gian đọc mã thẻ diễn ra tức thì dưới $10\text{ ms}$, người dùng quẹt lướt nhanh qua cửa không bị trễ. Ngoài ra, giao tiếp SPI truyền tín hiệu đồng hồ và dữ liệu trên các đường độc lập (`MOSI`, `MISO`, `SCK`) nên khả năng chống nhiễu trên mô hình dây nối thực tế tốt hơn nhiều.

2. **Câu hỏi:** *Nếu kẻ gian dùng đầu đọc sao chép (Clone) UID của thẻ cư dân sang thẻ trắng, hệ thống của bạn xử lý thế nào?*  
   **Trả lời:** Hệ thống của em không coi UID là phương thức bảo mật duy nhất. Dự án áp dụng mô hình bảo vệ đa tầng: Bên cạnh UID, hệ thống sử dụng khóa bí mật `Key A` riêng để đọc một mã số đếm động (Rolling Counter) được mã hóa trong Sector Data Block. Mỗi lần quẹt thẻ thành công, ESP32 sẽ ghi tăng Counter thêm 1 và đồng bộ lên Server. Nếu kẻ gian dùng thẻ clone có Counter cũ nhỏ hơn dữ liệu trên Server, hệ thống sẽ lập tức khóa cổng đọc và kích hoạt còi báo động đỏ. Đồng thời, thuật toán Anomaly Detection sẽ tự động khóa thẻ nếu phát hiện dò mã bất thường.

3. **Câu hỏi:** *Tại sao khi điều khiển khóa Solenoid 12V bằng Relay lại bắt buộc phải có Diode Flyback mắc song song với cuộn dây?*  
   **Trả lời:** Cuộn dây khóa Solenoid là tải thuần cảm có độ tự cảm $L$. Khi Relay ngắt điện, dòng điện giảm đột ngột về 0 trong thời gian cực ngắn ($dt \to 0$). Theo định luật tự cảm $V = -L rac{di}{dt}$, cuộn cảm sẽ phóng ra một sức điện động cảm ứng ngược cực lớn (Back-EMF) từ $200\text{ V}$ đến $400\text{ V}$. Xung áp nhọn này sẽ đánh thủng tiếp điểm cơ khí của Relay, sinh tia lửa điện gây nhiễu điện từ (EMI) truyền ngược qua đường mass làm vi điều khiển ESP32 bị reset (Brownout). Diode Flyback 1N4007 mắc ngược cực tính sẽ tạo thành vòng kín triệt tiêu năng lượng từ trường $\frac{1}{2}LI^2$ thành nhiệt trên điện trở nội của cuộn dây một cách an toàn.

4. **Câu hỏi:** *Làm thế nào để hệ thống vẫn mở được cửa khi mất kết nối mạng Wi-Fi hoặc máy chủ Backend bị sập?*  
   **Trả lời:** Em xây dựng cơ chế chịu lỗi ngoại tuyến (Offline Caching) sử dụng bộ nhớ Flash NVS của ESP32. Khi hệ thống còn online, danh sách mã băm (Hash Table) của các thẻ hợp lệ sẽ được đồng bộ và lưu vào Flash. Khi mất kết nối Wi-Fi (phát hiện qua cơ chế timeout $1.5\text{ giây}$), ESP32 tự động chuyển sang `OFFLINE_MODE`, tra cứu cục bộ trong NVS để quyết định mở cửa và lưu nhật ký quẹt thẻ tạm thời vào hàng đợi Flash. Khi mạng phục hồi, tiến trình nền tự động đẩy các bản ghi offline lên Server để đồng bộ lại cơ sở dữ liệu.

5. **Câu hỏi:** *Trong ứng dụng Desktop Electron, bạn bảo mật giao tiếp giữa Renderer Process (Giao diện React) và Main Process như thế nào?*  
   **Trả lời:** Em tuân thủ nghiêm ngặt chuẩn bảo mật của Electron: Cấu hình `contextIsolation: true` và `nodeIntegration: false` để ngăn chặn hoàn toàn mã JavaScript độc hại từ giao diện can thiệp vào tài nguyên hệ điều hành của máy tính. Mọi giao tiếp giữa React và hệ điều hành đều bắt buộc phải đi qua file `preload.ts` sử dụng `contextBridge.exposeInMainWorld()`, chỉ cung cấp các hàm IPC được kiểm duyệt chặt chẽ một cách an toàn.
