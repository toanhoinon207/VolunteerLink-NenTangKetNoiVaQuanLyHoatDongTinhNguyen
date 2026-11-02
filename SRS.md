
# TÀI LIỆU YÊU CẦU PHẦN MỀM (SRS)

# HỆ THỐNG VOLUNTEERLINK - NỀN TẢNG KẾT NỐI & QUẢN LÝ TÌNH NGUYỆN

## 1. GIỚI THIỆU (INTRODUCTION)

### 1.1 Mục đích (Purpose)

Tài liệu SRS này xác định đầy đủ, chi tiết các yêu cầu nghiệp vụ, yêu cầu chức năng và phi chức năng cho hệ thống VolunteerLink.

Tài liệu này phục vụ làm căn cứ kỹ thuật chính thức cho:

- **Developer:** Triển khai cơ sở dữ liệu MySQL, xây dựng RESTful API bằng FastAPI, phát triển giao diện bằng HTML/CSS (Tailwind)/JS.
- **Tester (QA/QC):** Lập kịch bản kiểm thử (Test Cases) và kiểm tra nghiệm thu.
- **Project Manager / Team Lead:** Quản lý tiến độ triển khai và phân chia công việc.

### 1.2 Phạm vi dự án (Project Scope)

VolunteerLink là nền tảng Web Application kết nối Tình nguyện viên (TNV) với các Tổ chức / Chiến dịch tình nguyện.

Phạm vi xử lý: Quản lý tài khoản, khám phá chiến dịch, đăng ký tham gia, duyệt ứng viên, điểm danh ghi nhận giờ công và xuất báo cáo thống kê.

**Công nghệ cam kết:**

- **Frontend:** Single-Page Application (SPA) / Multi-Page chuẩn HTML5, TailwindCSS, Vanilla JavaScript (Fetch API).
- **Backend:** RESTful API phát triển bằng Python (FastAPI), SQLAlchemy ORM, Pydantic, PyJWT.
- **Database:** MySQL 8.0+.

### 1.3 Thuật ngữ & Từ viết tắt (Definitions & Abbreviations)

| Từ viết tắt | Thuật ngữ đầy đủ | Ý nghĩa |
|---|---|---|
| SRS | Software Requirements Specification | Tài liệu Yêu cầu Phần mềm. |
| TNV | Tình nguyện viên | Người dùng đăng ký tham gia chiến dịch (Role User). |
| BTC | Ban tổ chức / Quản trị viên | Người tạo và điều hành chiến dịch (Role Admin). |
| JWT | JSON Web Token | Chuẩn mã hóa phục vụ xác thực người dùng. |
| ORM | Object-Relational Mapping | Công cụ ánh xạ bảng CSDL thành đối tượng code (SQLAlchemy). |
| DDL | Data Definition Language | Ngôn ngữ định nghĩa cấu trúc bảng trong SQL. |

## 2. MÔ TẢ TỔNG QUAN (OVERALL DESCRIPTION)

### 2.1 Kiến trúc tổng quan (Product Architecture)

Hệ thống phát triển theo kiến trúc Client-Server phân tách hoàn toàn (Decoupled Architecture):

- **Client Side:** Nhận request từ người dùng, render giao diện động qua DOM JS, gửi HTTP Request (JSON) tới Backend.
- **Server Side:** FastAPI chịu trách nhiệm Routing, Middleware (CORS, JWT Auth), Validation (Pydantic), và truy vấn CSDL qua ORM.
- **Database:** Lưu trữ dữ liệu quan hệ mã hóa chuẩn UTF-8 (utf8mb4).

```text
[ HTML / Tailwind / JS ] <--- HTTP/JSON (REST API) ---> [ FastAPI (Python) ] <--- ORM ---> [ MySQL 8.0 ]
```

### 2.2 Các vai trò người dùng (User Roles)

Hệ thống phân chia 3 vai trò chính:

**Guest (Khách chưa đăng nhập):**

Người dùng vãng lai truy cập hệ thống.

Quyền hạn: Xem trang chủ, danh sách chiến dịch, tìm kiếm/lọc chiến dịch, đăng ký/đăng nhập.

**User (Tình nguyện viên):**

Người dùng đã xác thực tài khoản.

Quyền hạn: Toàn bộ quyền Guest + Cập nhật Profile cá nhân, nộp đơn đăng ký chiến dịch, hủy đơn đăng ký, xem lịch sử tham gia và tổng số giờ công tích lũy.

**Admin (Ban tổ chức / Quản trị hệ thống):**

Tài khoản được phân quyền quản trị.

Quyền hạn: Toàn bộ quyền User + Tạo/Sửa/Xóa chiến dịch, duyệt/từ chối đơn đăng ký, điểm danh & ghi nhận giờ công, quản lý danh sách người dùng toàn hệ thống, xem báo cáo Dashboard.

## 3. THIẾT KẾ CƠ SỞ DỮ LIỆU CHI TIẾT (DATABASE SCHEMA - MYSQL)

**Sơ đồ liên kết thực thể (Entity Relationships):**

```text
users (1) --- (N) campaigns (Admin tạo chiến dịch)
users (1) --- (N) registrations (TNV đăng ký chiến dịch)
categories (1) --- (N) campaigns (Danh mục chứa nhiều chiến dịch)
campaigns (1) --- (N) registrations (Một chiến dịch có nhiều lượt đăng ký)
registrations (1) --- (N) attendance (Một lượt đăng ký có thể điểm danh nhiều buổi)
```

### 3.1 Cấu trúc các Bảng (Table Specifications)

#### 1. Bảng users (Người dùng)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| id | VARCHAR(36) | PRIMARY KEY | Khóa chính (UUID v4) |
| full_name | VARCHAR(100) | NOT NULL | Họ và tên |
| email | VARCHAR(150) | NOT NULL, UNIQUE | Email đăng nhập |
| password_hash | VARCHAR(255) | NOT NULL | Mật khẩu đã mã hóa (Bcrypt) |
| phone | VARCHAR(20) | NULL | Số điện thoại |
| avatar_url | TEXT | NULL | Link ảnh đại diện |
| role | ENUM | NOT NULL, DEFAULT 'USER' | Values: 'GUEST', 'USER', 'ADMIN' |
| bio | TEXT | NULL | Giới thiệu ngắn |
| skills | JSON | NULL | Danh sách kỹ năng ["Y tế", "Dạy học"] |
| status | ENUM | NOT NULL, DEFAULT 'ACTIVE' | Values: 'ACTIVE', 'BLOCKED' |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Thời gian tạo |
| updated_at | TIMESTAMP | ON UPDATE CURRENT_TIMESTAMP | Thời gian cập nhật |

#### 2. Bảng categories (Danh mục chiến dịch)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| id | INT | PRIMARY KEY, AUTO_INCREMENT | Khóa chính |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Tên danh mục (Môi trường, Giáo dục,...) |
| slug | VARCHAR(100) | NOT NULL, UNIQUE | Đường dẫn thân thiện (moi-truong) |

#### 3. Bảng campaigns (Chiến dịch tình nguyện)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| id | VARCHAR(36) | PRIMARY KEY | Khóa chính (UUID v4) |
| title | VARCHAR(255) | NOT NULL | Tiêu đề chiến dịch |
| slug | VARCHAR(255) | NOT NULL, UNIQUE | Slug đường dẫn |
| description | TEXT | NOT NULL | Mô tả chi tiết chiến dịch |
| category_id | INT | FOREIGN KEY (categories.id) | Danh mục |
| location | VARCHAR(255) | NOT NULL | Địa điểm diễn ra |
| start_date | DATETIME | NOT NULL | Ngày bắt đầu |
| end_date | DATETIME | NOT NULL | Ngày kết thúc |
| registration_deadline | DATETIME | NOT NULL | Hạn chót đăng ký |
| max_participants | INT | NOT NULL, DEFAULT 0 | Số lượng cần tuyển |
| current_participants | INT | NOT NULL, DEFAULT 0 | Số lượng đã duyệt |
| status | ENUM | DEFAULT 'PUBLISHED' | Values: 'DRAFT', 'PUBLISHED', 'ONGOING', 'COMPLETED', 'CANCELLED' |
| created_by | VARCHAR(36) | FOREIGN KEY (users.id) | Admin tạo chiến dịch |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Ngày tạo |
| updated_at | TIMESTAMP | ON UPDATE CURRENT_TIMESTAMP | Ngày cập nhật |

#### 4. Bảng registrations (Đơn đăng ký tham gia)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| id | VARCHAR(36) | PRIMARY KEY | Khóa chính (UUID v4) |
| campaign_id | VARCHAR(36) | FOREIGN KEY (campaigns.id) | Mã chiến dịch |
| user_id | VARCHAR(36) | FOREIGN KEY (users.id) | Mã TNV đăng ký |
| status | ENUM | DEFAULT 'PENDING' | Values: 'PENDING', 'APPROVED', 'REJECTED', 'CANCELLED' |
| note | TEXT | NULL | Lời nhắn từ TNV khi đăng ký |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Thời gian đăng ký |
| Constraint | UNIQUE | (campaign_id, user_id) | Mỗi TNV chỉ đăng ký 1 lần/chiến dịch |

#### 5. Bảng attendance (Điểm danh & Giờ công)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|---|---|---|---|
| id | VARCHAR(36) | PRIMARY KEY | Khóa chính (UUID v4) |
| registration_id | VARCHAR(36) | FOREIGN KEY (registrations.id) | Mã đơn đăng ký |
| check_in_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | Thời điểm điểm danh |
| hours_contributed | DECIMAL(4,2) | NOT NULL, DEFAULT 0.00 | Số giờ đóng góp ghi nhận |
| status | ENUM | DEFAULT 'PRESENT' | Values: 'PRESENT', 'ABSENT' |
| verified_by | VARCHAR(36) | FOREIGN KEY (users.id) | Admin thực hiện điểm danh |

## 4. QUY TRÌNH NGHIỆP VỤ & YÊU CẦU CHỨC NĂNG (FUNCTIONAL REQUIREMENTS)

### 4.1 Phân hệ 1: Xác thực & Quản lý Tài khoản (Auth & User)

#### UC-01: Đăng ký tài khoản (Register)

**Actor:** Guest

**Luồng xử lý (Flow):**

1. Guest nhập full_name, email, password.
2. Frontend Validate định dạng dữ liệu (Email chuẩn, Password >= 6 ký tự).
3. Backend kiểm tra email đã tồn tại trong bảng users chưa.
4. Nếu chưa: Hash password bằng mã hóa Bcrypt, tạo bản ghi mới với role='USER', trả về status 201 Created.

#### UC-02: Đăng nhập (Login)

**Actor:** Guest / User / Admin

**Luồng xử lý (Flow):**

1. Người dùng nhập email và password.
2. Backend kiểm tra thông tin, so sánh chuỗi password hash.
3. Nếu hợp lệ: Kiểm tra trạng thái status == 'ACTIVE'.
4. Trả về access_token (JWT chứa sub: user_id, role, exp), lưu tại Client (LocalStorage/Cookie).

#### UC-03: Quản lý Hồ sơ cá nhân (Profile)

**Actor:** User / Admin

**Luồng xử lý (Flow):**

1. Xem thông tin cá nhân: Họ tên, Email, SĐT, Kỹ năng, Bio.
2. Xem Tổng số giờ đóng góp: Hệ thống tự động tính tổng hours_contributed từ bảng attendance liên kết qua các registrations có status='APPROVED'.
3. Cập nhật thông tin: Cho phép sửa full_name, phone, bio, skills.

### 4.2 Phân hệ 2: Khám phá & Đăng ký Chiến dịch (Campaigns & Registrations)

#### UC-04: Danh sách & Tìm kiếm chiến dịch

**Actor:** Guest, User, Admin

**Luồng xử lý (Flow):**

1. Hiển thị danh sách chiến dịch có status='PUBLISHED' dạng Grid/List card.
2. Hỗ trợ lọc theo category_id, tìm kiếm theo từ khóa title / location.
3. Hỗ trợ phân trang (Pagination) limit/offset.

#### UC-05: Đăng ký tham gia chiến dịch

**Actor:** User

**Điều kiện tiên quyết:** Đã đăng nhập (role='USER'), chiến dịch còn trong hạn đăng ký (registration_deadline >= NOW()), và chưa vượt quá số lượng tuyển.

**Luồng xử lý (Flow):**

1. User nhấn "Đăng ký tham gia" tại trang chi tiết chiến dịch, nhập lời nhắn (note).
2. Backend kiểm tra ràng buộc UNIQUE(campaign_id, user_id).
3. Tạo bản ghi mới trong registrations với status='PENDING'.

#### UC-06: Hủy đăng ký chiến dịch

**Actor:** User

**Luồng xử lý (Flow):**

1. User vào danh sách "Chiến dịch đã đăng ký".
2. Chọn chiến dịch có status='PENDING', nhấn "Hủy đăng ký".
3. Cập nhật trạng thái trong CSDL thành CANCELLED.

### 4.3 Phân hệ 3: Quản trị & Điểm danh (Admin Operations)

#### UC-07: Tạo mới & Quản lý chiến dịch

**Actor:** Admin

**Luồng xử lý (Flow):**

1. Admin nhập thông tin: Tiêu đề, Mô tả, Danh mục, Địa điểm, Thời gian (Bắt đầu, Kết thúc, Hạn đăng ký), Số lượng tuyển tối đa.
2. Backend sinh slug tự động, gán created_by = current_user.id.
3. Admin có quyền đổi trạng thái chiến dịch (PUBLISHED -> ONGOING -> COMPLETED / CANCELLED).

#### UC-08: Phê duyệt đơn đăng ký TNV

**Actor:** Admin

**Luồng xử lý (Flow):**

1. Admin xem danh sách các ứng viên đăng ký chiến dịch (Trạng thái PENDING).
2. Chọn Duyệt (APPROVED) hoặc Từ chối (REJECTED).
3. Ràng buộc tự động: Khi chuyển trạng thái sang APPROVED, CSDL thực hiện tăng current_participants = current_participants + 1 trong bảng campaigns.

#### UC-09: Điểm danh & Ghi nhận giờ công

**Actor:** Admin

**Luồng xử lý (Flow):**

1. Tại buổi hoạt động, Admin mở danh sách TNV đã được duyệt (status='APPROVED').
2. Thực hiện điểm danh: Nhập số giờ đóng góp (hours_contributed), đánh dấu PRESENT.
3. Tạo bản ghi trong bảng attendance ghi nhận ID Admin điểm danh (verified_by).

## 5. THIẾT KẾ RESTful API SPECS (FASTAPI)

**Quy chuẩn Response JSON chung:**

```json
{
  "success": true,
  "message": "Thông báo kết quả",
  "data": { ... }
}
```

### 5.1 Danh sách Endpoints

| Nhóm | Method | Endpoint | Auth | Quyền | Mô tả |
|---|---|---|---|---|---|
| Auth | POST | /api/v1/auth/register | No | Public | Đăng ký tài khoản mới |
| | POST | /api/v1/auth/login | No | Public | Đăng nhập lấy JWT Access Token |
| User | GET | /api/v1/users/me | Yes | User, Admin | Lấy thông tin tài khoản hiện tại |
| | PUT | /api/v1/users/me | Yes | User, Admin | Cập nhật thông tin cá nhân |
| | GET | /api/v1/users/me/history | Yes | User | Lịch sử tham gia & tổng số giờ công |
| Campaign | GET | /api/v1/campaigns | No | Public | Lấy danh sách chiến dịch (Lọc/Search) |
| | GET | /api/v1/campaigns/{id} | No | Public | Xem chi tiết 1 chiến dịch |
| | POST | /api/v1/campaigns | Yes | Admin | Tạo chiến dịch mới |
| | PUT | /api/v1/campaigns/{id} | Yes | Admin | Cập nhật thông tin chiến dịch |
| | DELETE | /api/v1/campaigns/{id} | Yes | Admin | Xóa/Hủy chiến dịch |
| Registration | POST | /api/v1/campaigns/{id}/register | Yes | User | Đăng ký tham gia chiến dịch |
| | POST | /api/v1/registrations/{id}/cancel | Yes | User | Hủy đăng ký (Khi còn PENDING) |
| | GET | /api/v1/campaigns/{id}/registrations | Yes | Admin | Lấy danh sách ứng viên đăng ký |
| | PATCH | /api/v1/registrations/{id}/status | Yes | Admin | Phê duyệt/Từ chối ứng viên |
| Attendance | POST | /api/v1/attendance/check-in | Yes | Admin | Điểm danh & tích lũy giờ công TNV |

## 6. YÊU CẦU PHI CHỨC NĂNG (NON-FUNCTIONAL REQUIREMENTS)

### 6.1 BẢO MẬT (SECURITY)

**Mã hóa mật khẩu:** Bắt buộc sử dụng thuật toán hash Bcrypt / Argon2. Tuyệt đối không lưu Plaintext Password.

**Xác thực API:** Các Endpoint yêu cầu quyền hạn bắt buộc truyền JWT trong HTTP Header: Authorization: Bearer \<token>.

**Phòng chống tấn công:**

- **SQL Injection:** Xử lý triệt để thông qua truy vấn Parameterized Query của SQLAlchemy ORM.
- **XSS (Cross-Site Scripting):** Sanitize dữ liệu đầu vào và Encode HTML Output tại Client.
- **CORS:** Cấu hình FastAPI CORSMiddleware chỉ cho phép Domain được duyệt truy cập.

### 6.2 HIỆU NĂNG & MỞ RỘNG (PERFORMANCE & SCALABILITY)

**Thời gian phản hồi (Response Time):** API xử lý trung bình < 200ms với quy mô 1.000 request/phút.

**Cơ sở dữ liệu:** Đánh Index thích hợp cho các cột tìm kiếm thường xuyên (email, slug, status, category_id).

**Dung lượng trang:** Giao diện HTML/CSS/JS thuần nhẹ, tối ưu nạp CDN, trang tải hoàn tất dưới 1.5s.

### 6.3 GIAO DIỆN & TRẢI NGHIỆM (UI/UX)

**Responsive Design:** Giao diện hiển thị tốt trên cả Desktop, Tablet và Mobile nhờ TailwindCSS Grid/Flexbox.

**Đồng bộ trạng thái:** Cung cấp Role Switcher / Navigation linh hoạt cho từng đối tượng người dùng.
