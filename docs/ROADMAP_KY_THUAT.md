# 📘 SỔ TAY KỸ THUẬT & LỘ TRÌNH THỰC HIỆN DỰ ÁN 1
## Đề tài: Hệ thống kiểm soát cửa RFID & IoT tích hợp Desktop App (ESP32 + Fastify + Electron)

> **Người thực hiện:** Kỹ sư IoT / Embedded  
> **Thời gian:** 10 - 14 tuần (Khởi động: 06/10/2026)  
> **Ghi chú tác giả:** Tài liệu này ghi lại toàn bộ cơ sở lý thuyết, kiến trúc gói tin, bẫy phần cứng thực tế và lộ trình chi tiết từng tuần theo phong cách ghi chép kỹ thuật thực chiến.

---

## PHẦN 1: CƠ SỞ LÝ THUYẾT & NGUYÊN LÝ HOẠT ĐỘNG

### 1.1. Công nghệ RFID 13.56MHz & Chuẩn ISO/IEC 14443A
* **Nguyên lý cảm ứng điện từ:** Đầu đọc RC522 liên tục phát ra trường điện từ tần số 13.56 MHz qua cuộn anten PCB. Khi thẻ RFID (MIFARE Classic 1K) đi vào vùng từ trường (khoảng cách 1–4 cm), cuộn dây bên trong thẻ nhận năng lượng cảm ứng, tự nạp điện cho chip và phát ngược lại mã định danh (Load Modulation).
* **Cấu trúc bộ nhớ thẻ MIFARE Classic 1K:**
  * Bộ nhớ gồm **1024 bytes**, chia làm **16 Sector** (Sector 0 đến Sector 15).
  * Mỗi Sector có **4 Block** (mỗi Block chứa 16 bytes dữ liệu).
  * **Sector 0 - Block 0 (Nhà sản xuất):** Chứa mã **UID** (4 bytes hoặc 7 bytes) và dữ liệu xuất xưởng. Ở thẻ chuẩn chính hãng, block này chỉ đọc (Read-only) và được ghi cố định từ nhà máy.
  * **Block 3 của mỗi Sector (Sector Trailer):** Chứa 2 khóa bảo mật (Key A - 6 bytes, Key B - 6 bytes) và 4 bytes Access Bits quy định quyền đọc/ghi.
* **Bản chất về lỗ hổng "Clone thẻ" và cách tiếp cận an toàn:**
  * Hiện nay trên thị trường có các loại thẻ Trung Quốc ("Magic Card" UID Gen 1 / Gen 2) cho phép dùng lệnh đặc biệt để ghi đè cả Sector 0 Block 0. Do đó, bất kỳ ai có đầu đọc cầm tay giá rẻ đều có thể sao chép UID sang thẻ trắng khác.
  * **Giải pháp kỹ thuật của dự án:** Không coi UID là khóa bảo mật duy nhất! Hệ thống bảo vệ bằng cách:
    1. Quản lý trạng thái thẻ tập trung trên Server (`ACTIVE`, `BLOCKED`, `REVOKED`).
    2. Kiểm tra quyền thẻ theo từng cửa và khung giờ.
    3. Đọc/ghi **Counter sử dụng** vào Block dữ liệu được bảo vệ bằng Key A/Key B bí mật. Nếu thẻ clone dùng bản sao cũ có counter nhỏ hơn server, hệ thống lập tức khóa và báo động.
    4. Ghi nhận dấu vết (Audit log) và phát hiện hành vi quẹt dò mã liên tục (Brute-force Anomaly).

### 1.2. Giao thức giao tiếp vi điều khiển: SPI vs I2C cho RC522
* Module RC522 hỗ trợ cả SPI, I2C và UART. Nhưng **bắt buộc chọn SPI** vì:
  * **Tốc độ:** Bus SPI chạy xung nhịp lên tới 10 MHz (so với 400 kHz của I2C), giúp đọc thẻ tức thì, giảm độ trễ khi người dùng quẹt lướt qua nhanh.
  * **Khả năng chống nhiễu:** Dây SPI truyền tín hiệu clock và data riêng biệt (`MOSI`, `MISO`, `SCK`, `SS`), ổn định hơn nhiều khi nối dây cắm dài 10–20cm trên mô hình cửa.

### 1.3. Cơ chế giao tiếp mạng: REST API qua Wi-Fi
* ESP32 đóng vai trò là HTTP Client, định dạng dữ liệu gửi nhận là **JSON**.
* **Cấu trúc gói tin Request khi quẹt thẻ:**
  ```http
  POST /api/access/verify HTTP/1.1
  Host: 192.168.1.100:3000
  Content-Type: application/json
  x-device-token: d7a8f9c2b1e44a39872e...
  
  {
    "deviceCode": "DOOR_ENTRY_01",
    "uid": "8A3B21F0",
    "counter": 12,
    "timestamp": 1711234567
  }
  ```
* **Cấu trúc gói tin Response từ Server:**
  ```json
  {
    "allowed": true,
    "reasonCode": "ACCESS_GRANTED",
    "residentName": "Nguyen Van A",
    "unlockDurationMs": 3000,
    "nextCounter": 13
  }
  ```

---

## PHẦN 2: BẪY PHẦN CỨNG & KINH NGHIỆM THỰC CHIẾN (HARDWARE GOTCHAS)

1. **Lỗi sụt áp Brownout Detector Reset của ESP32:**
   * *Hiện tượng:* Khi nạp code chạy riêng thì bình thường, nhưng hễ ESP32 bắt đầu phát Wi-Fi hoặc gọi API là vi điều khiển tự khởi động lại (Serial báo lỗi `Brownout detector was triggered`).
   * *Nguyên nhân:* Module RF Wi-Fi của ESP32 tiêu thụ dòng tức thời lên tới 250mA - 300mA khi truyền tin (TX burst). Cổng USB máy tính hoặc dây cáp chất lượng kém làm sụt áp chân 3.3V xuống dưới 2.8V.
   * *Cách khắc phục:* 
     * Dùng cáp USB xịn có lõi đồng dày.
     * Cắm thêm 1 tụ hóa (Electrolytic capacitor) **10µF đến 100µF** (điện áp chịu đựng 10V–16V) nối song song giữa chân `3V3` và `GND` ngay gần bo mạch để bù áp tức thời.
2. **Chân GPIO Strapping cấm kỵ:**
   * Không dùng các chân `GPIO 0`, `GPIO 2`, `GPIO 12`, `GPIO 15` để nối Relay hoặc Buzzer nếu có trở kéo ngoài không mong muốn, vì các chân này quy định chế độ nạp Bootloader khi ESP32 vừa khởi động. Nếu bị kéo sai mức logic, ESP32 sẽ không chạy chương trình.
   * *Khuyến nghị:* Dùng `GPIO 26` cho Relay, `GPIO 12` cho Buzzer (sau khi boot xong), `GPIO 14` cho LED đỏ, `GPIO 27` cho LED xanh.
3. **Nguồn cấp cho Relay 5V:**
   * Module Relay 5V dùng cuộn hút 5V. Chân `VCC` của Relay phải nối vào chân `VIN` (hoặc `5V`) của ESP32 (lấy trực tiếp từ nguồn USB), **không được nối vào chân 3V3** vì chân 3.3V của ESP32 không đủ dòng nuôi cuộn hút relay.

---

## PHẦN 3: LỘ TRÌNH THỰC HIỆN CHI TIẾT 10 TUẦN

### Tuần 1: Cài đặt công cụ, Git & Khảo sát phần cứng (06/10 - 12/10)
* [ ] Cài đặt VS Code, Git, PlatformIO IDE, Node.js v20 (LTS), Postman.
* [ ] Tạo kho lưu trữ GitHub `iot-rfid-access-control`, tạo các thư mục: `firmware`, `server`, `desktop`, `docs`.
* [ ] Kiểm tra bo mạch ESP32: Cắm cáp nạp, cài driver CP210x, nạp code Blink LED trên chân GPIO 2.
* [ ] Ghi lại video ngắn xác nhận mạch nạp hoạt động tốt.

### Tuần 2: Nối dây & Lập trình Module RC522 (13/10 - 19/10)
* [ ] Nối dây RC522 qua chuẩn SPI: SDA->GPIO 5, SCK->GPIO 18, MOSI->GPIO 23, MISO->GPIO 19, RST->GPIO 22.
* [ ] Nạp chương trình mẫu đọc thẻ: Lấy được mã UID (ví dụ `A4:3C:9B:10`) in ra Serial Monitor tốc độ 115200.
* [ ] Lập trình cơ cấu chấp hành: Nối Relay vào GPIO 26, Buzzer vào GPIO 12, LED Xanh/Đỏ vào GPIO 27/14.
* [ ] Viết hàm `testHardware()`: Quẹt thẻ bất kỳ -> Relay đóng 3 giây, LED xanh bật; rút thẻ ra -> kêu 1 tiếng bíp.

### Tuần 3: Thiết kế Database & Đặc tả REST API (20/10 - 26/10)
* [ ] Vẽ sơ đồ thực thể mối quan hệ (ERD): `User`, `Resident`, `Card`, `Door`, `AccessLog`, `SecurityAlert`.
* [ ] Viết tài liệu đặc tả API chuẩn: Input format, Header xác thực, Output status code.
* [ ] Chuẩn bị kịch bản kiểm thử: Phân loại 10 mã trạng thái (`ACCESS_GRANTED`, `CARD_BLOCKED`,...).

### Tuần 4: Xây dựng Backend Fastify + Prisma + SQLite (27/10 - 02/11)
* [ ] Khởi tạo dự án Node.js TypeScript: Cấu hình `tsconfig.json`, `package.json`.
* [ ] Viết file `prisma/schema.prisma` và chạy lệnh `npx prisma migrate dev --name init`.
* [ ] Viết seed script tạo dữ liệu mẫu: 1 Admin, 3 Cư dân, 5 Thẻ với các trạng thái khác nhau.
* [ ] Dựng endpoint `GET /api/health` và cấu hình CORS cho phép Desktop App gọi vào.

### Tuần 5: Hoàn thiện CRUD nghiệp vụ Backend (03/11 - 09/11)
* [ ] Viết API Quản lý Cư dân: Thêm, sửa, xóa, tìm kiếm theo tên hoặc căn hộ.
* [ ] Viết API Quản lý Thẻ: Gán thẻ cho cư dân, cập nhật trạng thái (`ACTIVE`, `BLOCKED`, `REVOKED`).
* [ ] Viết API Quản lý Cửa: Tạo Device Token riêng biệt cho từng cổng ESP32.
* [ ] Dùng Postman chạy toàn bộ Collection kiểm thử tự động.

### Tuần 6: Tích hợp ESP32 gọi API xác thực thời gian thực (10/11 - 16/11)
* [ ] Lập trình ESP32 kết nối Wi-Fi tự động; tự động reconnect nếu mất tín hiệu.
* [ ] Khi RC522 phát hiện thẻ: Lấy UID, đóng gói JSON và gửi HTTP POST lên Server.
* [ ] Nhận JSON phản hồi từ Server: Nếu `allowed = true` thì mở relay; nếu `false` thì bật còi báo động.
* [ ] Đo thời gian phản hồi: Từ lúc chạm thẻ vào đầu đọc đến lúc relay kêu "tách" (mục tiêu: < 300ms trong mạng LAN).

### Tuần 7: Ghi nhận Log & Thuật toán phát hiện bất thường (17/11 - 23/11)
* [ ] Server tự động lưu mỗi lượt quẹt vào bảng `AccessLog`.
* [ ] Cài đặt thuật toán phát hiện bất thường: Nếu trong vòng 60 giây có 3 lần quẹt thẻ lạ hoặc thẻ bị khóa liên tiếp tại cùng 1 cửa -> Tạo một bản ghi cảnh báo nguy hiểm trong `SecurityAlert`.
* [ ] Thêm mã hóa bcrypt cho mật khẩu Admin và sinh token JWT khi đăng nhập.

### Tuần 8: Xây dựng giao diện Desktop App với Electron + React (24/11 - 30/11)
* [ ] Khởi tạo template Electron + React + Vite + TypeScript.
* [ ] Cấu hình kiến trúc: Main process quản lý cửa sổ Windows, Renderer process hiển thị giao diện React.
* [ ] Dựng Layout chuẩn: Sidebar định hướng, Header hiển thị thông tin tài khoản Admin đang trực.
* [ ] Xây dựng màn hình **Dashboard**: Các thẻ thống kê số thẻ active, số lượt ra vào trong ngày.

### Tuần 9: Hoàn thiện các màn hình quản trị trên Desktop (01/12 - 07/12)
* [ ] Màn hình **Quản lý Thẻ**: Bảng dữ liệu có tìm kiếm, nút Switch Khóa/Mở thẻ tức thời.
* [ ] Màn hình **Lịch sử ra vào**: Phân trang, lọc theo kết quả (Thành công/Thất bại), xuất file báo cáo.
* [ ] Màn hình **Cảnh báo an ninh**: Hộp thông báo màu đỏ nhấp nháy khi phát hiện sự kiện bất thường.

### Tuần 10: Cơ chế Offline Cache & Đóng gói sản phẩm (08/12 - 14/12)
* [ ] Lập trình bộ nhớ Flash ESP32 (LittleFS / NVS): Lưu danh sách 50 thẻ hợp lệ cục bộ.
* [ ] Khi mất Wi-Fi: ESP32 tự chuyển sang chế độ Offline, kiểm tra thẻ qua cache Flash để mở cửa, lưu log vào bộ nhớ tạm.
* [ ] Khi có Wi-Fi lại: Tự động gửi gói tin sync đồng bộ toàn bộ log tạm về server.
* [ ] Đóng gói phần mềm Desktop ra file `.exe` cài đặt cho Windows bằng Electron Builder.
* [ ] Chạy trọn vẹn 10 kịch bản kiểm thử (TC01 đến TC10), quay video demo 3 phút và hoàn tất báo cáo.
