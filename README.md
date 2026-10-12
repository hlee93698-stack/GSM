# GSM / QSM — Nền Tảng Di Chuyển Thông Minh Thuần Điện (EV Smart Mobility)

## 📖 Mô Tả Bài Toán & Bối Cảnh Thực Tiễn

### 1. Đặt Vấn Đề (Problem Statement)
Trong bối cảnh đô thị hóa nhanh và mục tiêu Net Zero hướng tới giao thông xanh bền vững, nhu cầu di chuyển nội đô bằng các phương tiện thuần điện (Electric Vehicles - EV) ngày càng gia tăng. Tuy nhiên, các giải pháp công nghệ vận tải hiện nay thường gặp phải những thách thức:
- **Thiếu tính toán tác động môi trường:** Người dùng không đo lường được lượng khí thải $\text{CO}_2$ giảm thiểu khi ưu tiên sử dụng phương tiện xanh thay thế xe động cơ đốt trong.
- **Quản lý trạng thái cuốc xe phức tạp:** Quy trình điều phối từ khâu tiếp nhận thông tin, định vị, tìm tài xế, đón khách đến hoàn tất cuốc xe dễ rơi vào các trạng thái bất hợp lệ (*invalid states* hoặc *race conditions*) nếu không được kiểm soát theo mô hình chuẩn hóa.
- **Lộ trình thực tế đa điểm dừng:** Nhu cầu đón trả bạn bè hoặc ghé nhiều điểm dọc đường đòi hỏi tính toán khoảng cách và giá cước động linh hoạt.
- **Trải nghiệm bản đồ thiếu trực quan:** Các ứng dụng thường phụ thuộc vào các dịch vụ bản đồ thương mại trả phí cao hoặc giải pháp giả lập tĩnh, thiếu tính năng xoay đầu xe theo hướng đường (*smooth bearing*) và định vị địa chỉ thực tế hai chiều.

---

### 2. Mục Tiêu Dự Án (Project Objectives)
Xây dựng một nền tảng dịch vụ di chuyển thông minh thuần điện hoàn chỉnh (**Quick Smart Mobility - QSM / GSM**) với các mục tiêu trọng tâm:
1. **Quản lý quy trình đặt xe chặt chẽ (Finite State Machine):** Ứng dụng mô hình máy trạng thái hữu hạn để điều phối toàn bộ vòng đời cuốc xe từ lúc chọn tuyến đường cho đến khi hoàn thành hoặc hủy bỏ.
2. **Bản đồ số & Điều hướng thời gian thực (Real-time GIS & Routing):** 
   - Tích hợp bản đồ mã nguồn mở OpenStreetMap (OSM) và Leaflet hoàn toàn miễn phí, không phụ thuộc API key trả phí.
   - Định tuyến đường bộ chính xác (OSRM engine) hỗ trợ đa điểm dừng (*Multi-stop Routing*).
   - Định vị địa lý hai chiều (*Forward/Reverse Geocoding*) tự động nhận diện tên đường và địa chỉ tiếng Việt.
   - Nội suy góc xoay xe mượt mà (*Smooth Bearing Interpolation*) mô phỏng chuyển động thực tế của phương tiện dọc theo cung đường.
3. **Đo lường & Báo cáo phát thải $\text{CO}_2$:** Tự động tính toán lượng $\text{CO}_2$ tiết kiệm được sau mỗi chuyến đi dựa trên độ dài quãng đường và loại phương tiện (Xe máy điện VinFast Feliz S, Taxi VinFast VF 5 Plus, SUV hạng sang VinFast VF 9).
4. **Phân quyền người dùng & Động cơ khuyến mãi thông minh (RBAC & Tier-based Voucher):**
   - Phân cấp hội viên (Tiêu chuẩn - Standard, Hạng Vàng - Gold, Kim Cương VIP - Platinum).
   - Kiểm soát quyền áp dụng mã giảm giá nghiêm ngặt theo cấp bậc tài khoản nhằm bảo đảm tính công bằng và chống lạm dụng mã khuyến mãi.
5. **Cổng quản trị vận hành tập trung (Admin Dashboard):** Thống kê doanh thu, lưu lượng chuyến xe, quản lý tài khoản người dùng, tài xế và xuất báo cáo dữ liệu định dạng CSV/Excel.

---

### 3. Kiến Trúc Hệ Thống (System Architecture)

Dự án được xây dựng theo kiến trúc **Client - Server (RESTful API)** kết hợp kiến trúc phân lớp sạch sẽ (**Layered Architecture**):

```
                   ┌─────────────────────────────────────────┐
                   │               CLIENT (SPA)              │
                   │    React 19 + TypeScript + Vite         │
                   │  - State Machine (Finite State Machine) │
                   │  - Leaflet GIS & OSRM Engine            │
                   │  - 3D EV Showcase (Three.js)            │
                   └────────────────────┬────────────────────┘
                                        │
                               HTTP / RESTful API
                               (JSON & Bearer Auth)
                                        │
                   ┌────────────────────▼────────────────────┐
                   │             SERVER (Backend)            │
                   │      Node.js + Express + TypeScript     │
                   ├─────────────────────────────────────────┤
                   │  Controllers (Request/Response Handler) │
                   │  Middlewares (JWT Auth & Role Guard)    │
                   │  Services (Fare & Business Rules)       │
                   │  Database Adapter (File-based JSON DB)  │
                   └─────────────────────────────────────────┘
```

---

### 4. Các Phân Hệ Chức Năng Chính

| Phân Hệ | Mô Tả Nghiệp Vụ | Công Nghệ Triển Khai |
| :--- | :--- | :--- |
| **Giao diện Người dùng (Customer Portal)** | Đặt xe nhanh, chọn phương tiện (Bike, Car, Luxury), tùy chỉnh lộ trình đa điểm, chọn lịch hẹn trước, đánh giá chuyến đi và đổi chủ đề Sáng / Tối (Cyber EV Cyan). | React 19, Lucide Icons, Three.js, CSS Variables. |
| **Bản đồ & Định vị (Map Engine)** | Hiển thị bản đồ Leaflet + OSM, tự động chuyển đổi giao diện Dark/Light mode qua CSS filter, ghim điểm đón/điểm đến, xoay đầu xe mượt mà theo góc rẽ. | Leaflet, OSRM Routing API, Nominatim Geocoding. |
| **Quản lý Chuyến đi (Trip Lifecycle)** | Kiểm soát luồng cuốc xe chặt chẽ theo State Machine: `IDLE` $\to$ `COLLECTING_INFO` $\to$ `SEARCHING_DRIVER` $\to$ `DRIVER_ASSIGNED` $\to$ `DRIVER_ARRIVING` $\to$ `TRIP_STARTED` $\to$ `TRIP_COMPLETED`. | React useReducer, Finite State Machine (FSM). |
| **Hệ thống Khuyến mãi (Voucher Engine)** | Quản lý mã giảm giá theo số tiền cố định hoặc phần trăm, áp dụng điều kiện giá trị đơn tối thiểu và phân quyền theo hạng thẻ (Standard/Gold/Platinum). | Express Router, TypeScript Validators, RBAC. |
| **Cổng Quản trị (Admin Portal)** | Bảng điều khiển dành riêng cho quản trị viên: theo dõi KPI doanh thu, số chuyến xe, quản lý khóa/mở tài khoản, duyệt voucher và xuất báo cáo CSV. | Admin Dashboard, REST API. |
| **Lưu Trữ Dữ Liệu (Data Persistence)** | Hệ thống lưu trữ dữ liệu dạng JSON File-based DB (`qsm-store.json`), có sẵn schema chuẩn DDL (`schema.sql`) để chuyển giao sang PostgreSQL/MySQL. | Node.js File System (`fs`), bcryptjs, JWT. |
