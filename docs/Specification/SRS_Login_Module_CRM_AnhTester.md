# SOFTWARE REQUIREMENTS SPECIFICATION (SRS)
## Module: Đăng nhập (Admin Login) – Perfex CRM | Anh Tester Demo

| Thuộc tính | Giá trị |
|---|---|
| Hệ thống | Perfex CRM v3.1.6 (bản demo Anh Tester) |
| URL module | `https://crm.anhtester.com/admin/authentication` |
| Phiên bản tài liệu | 1.0 |
| Ngày lập | 18/11/2026 |
| Trạng thái | Draft – cần BA/PO review |
| Nguồn thông tin | Khảo sát trực tiếp giao diện + HTML của trang (reverse-engineered), kết hợp hành vi mặc định của Perfex CRM |

> **Ghi chú về độ tin cậy:** Các yêu cầu được đánh dấu **[Quan sát]** là đã xác minh trực tiếp trên trang. Các yêu cầu đánh dấu **[Giả định]** dựa trên hành vi mặc định của Perfex CRM / CodeIgniter và **cần xác nhận** khi test thực tế (xem mục 10).

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu mô tả các yêu cầu chức năng và phi chức năng của module **Đăng nhập dành cho nhân viên/quản trị (Staff/Admin)** trên hệ thống Perfex CRM, làm cơ sở cho việc thiết kế test case, test manual và test automation.

### 1.2 Phạm vi
**Trong phạm vi:**
- Màn hình Đăng nhập Admin (`/admin/authentication`)
- Chức năng "Remember me"
- Liên kết và màn hình "Forgot Password" (`/admin/authentication/forgot_password`)
- Điều hướng khi truy cập trang Admin mà chưa đăng nhập

**Ngoài phạm vi:**
- Đăng nhập cổng khách hàng (Client portal – `/authentication/login`)
- Màn hình đặt lại mật khẩu từ link email (`reset_password`)
- Xác thực 2 lớp (2FA), reCAPTCHA – chỉ đề cập ở mức cấu hình tùy chọn
- Quản lý tài khoản nhân viên (tạo, khóa, phân quyền)

### 1.3 Thuật ngữ

| Thuật ngữ | Giải thích |
|---|---|
| Staff / Admin | Người dùng nội bộ có tài khoản trong module Staff của CRM |
| CSRF token | Mã chống giả mạo request, gửi kèm form ẩn `csrf_token_name` |
| Session | Phiên làm việc sau khi đăng nhập thành công |
| Remember me | Tùy chọn duy trì đăng nhập sau khi đóng trình duyệt |
| BR | Business Rule – quy tắc nghiệp vụ |
| FR / NFR | Functional / Non-Functional Requirement |

### 1.4 Đối tượng sử dụng tài liệu
BA, Developer, Tester (Manual/Automation), PO.

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh
Perfex CRM là hệ thống quản lý quan hệ khách hàng. Mọi chức năng trong khu vực `/admin/*` yêu cầu người dùng đăng nhập bằng tài khoản Staff. Module Đăng nhập là cổng vào duy nhất của khu vực quản trị.

### 2.2 Tác nhân (Actors)

| Actor | Mô tả |
|---|---|
| Khách (Guest) | Người dùng chưa đăng nhập, truy cập trang login |
| Staff | Nhân viên có tài khoản active |
| Admin | Staff có quyền quản trị (ví dụ: `admin@example.com`) |
| Hệ thống Email | Gửi email hướng dẫn đặt lại mật khẩu |

### 2.3 Môi trường vận hành
- Ứng dụng web, truy cập qua HTTPS
- Trình duyệt: Chrome, Firefox, Edge, Safari (phiên bản hiện hành)
- Responsive: viewport `width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no` **[Quan sát]**
- Ngôn ngữ giao diện: Tiếng Anh (`lang="en"`) **[Quan sát]**

---

## 3. Đặc tả giao diện (UI Specification)

### 3.1 Màn hình Login – `/admin/authentication`

**Tiêu đề tab trình duyệt:** `Perfex CRM | Anh Tester Demo - Login` **[Quan sát]**

**Bố cục** (căn giữa màn hình, nền xám nhạt, form dạng card trắng) **[Quan sát]**:

```
            [ Logo ANHTESTER – Automation Testing ]   (link về https://crm.anhtester.com/)
                           Login
        ┌──────────────────────────────────────────┐
        │ Email Address                            │
        │ [______________________________________] │
        │ Password                                 │
        │ [______________________________________] │
        │ [ ] Remember me                          │
        │ [               Login                  ] │
        │ Forgot Password?                         │
        └──────────────────────────────────────────┘
```

**Bảng mô tả trường dữ liệu** **[Quan sát]**:

| # | Thành phần | Loại | id / name | Thuộc tính HTML | Bắt buộc | Mô tả |
|---|---|---|---|---|---|---|
| 1 | Logo | Image + link | – | `alt="Perfex CRM \| Anh Tester Demo"`, href `/` | – | Click điều hướng về trang chủ |
| 2 | Tiêu đề "Login" | Text | – | – | – | Tiêu đề form |
| 3 | Email Address | Textbox | `email` / `email` | `type="email"`, `autofocus` | Có (server-side) | Nhận con trỏ khi tải trang |
| 4 | Password | Textbox | `password` / `password` | `type="password"` | Có (server-side) | Ký tự được che (masked) |
| 5 | Remember me | Checkbox | `remember` / `remember` | `value="estimate"`, mặc định không tick | Không | Duy trì đăng nhập |
| 6 | Login | Button | – | `type="submit"`, `btn-primary btn-block` | – | Submit form |
| 7 | Forgot Password? | Link | – | href `/admin/authentication/forgot_password` | – | Mở màn hình quên mật khẩu |
| 8 | CSRF token | Hidden | `csrf_token_name` | giá trị 32 ký tự hex, thay đổi theo phiên | – | Chống CSRF |

**Form:** `method="post"`, `action="/admin/authentication"`, `accept-charset="utf-8"`. **[Quan sát]**

> Lưu ý: Các input **không** có thuộc tính `required` hay `maxlength` → validation "bắt buộc nhập" được thực hiện phía server. Riêng `type="email"` sẽ kích hoạt validation định dạng email của trình duyệt (HTML5) khi trường có giá trị. **[Quan sát]**

### 3.2 Màn hình Forgot Password – `/admin/authentication/forgot_password`

**[Quan sát]**

| # | Thành phần | Loại | id / name | Mô tả |
|---|---|---|---|---|
| 1 | Logo | Image + link | – | Giống màn hình Login |
| 2 | Tiêu đề "Forgot Password" | Text | – | – |
| 3 | Email Address | Textbox | `email` / `email` | `type="email"`, không `required` |
| 4 | Confirm | Button | – | Submit (`POST /admin/authentication/forgot_password`) |
| 5 | CSRF token | Hidden | `csrf_token_name` | – |

> Ghi nhận: màn hình **không có** link "Back to Login" — người dùng phải dùng nút Back của trình duyệt. (Đề xuất cải tiến – xem mục 10.)

---

## 4. Yêu cầu chức năng (Functional Requirements)

### FR-01: Hiển thị màn hình đăng nhập
| | |
|---|---|
| Mô tả | Khi Guest truy cập `/admin/authentication`, hệ thống hiển thị form đăng nhập như mục 3.1. |
| Điều kiện | Người dùng chưa đăng nhập. |
| Kết quả | Form hiển thị đầy đủ các thành phần; con trỏ tự động focus vào ô Email; ô Password trống; Remember me không được tick. **[Quan sát]** |
| Ngoại lệ | Nếu người dùng đã đăng nhập, truy cập URL này sẽ chuyển hướng về Dashboard. **[Giả định]** |

### FR-02: Đăng nhập thành công
| | |
|---|---|
| Mô tả | Staff nhập Email và Password hợp lệ, nhấn **Login** (hoặc phím Enter). |
| Tiền điều kiện | Tài khoản tồn tại và đang ở trạng thái Active. |
| Xử lý | Hệ thống kiểm tra CSRF token → validate dữ liệu → đối chiếu email + mật khẩu (đã băm) → tạo session. |
| Kết quả | Chuyển hướng tới `/admin` (Dashboard). Nếu trước đó người dùng truy cập một trang admin cụ thể khi chưa đăng nhập, có thể được chuyển về đúng trang đó. **[Giả định]** |
| Hậu điều kiện | Ghi nhận thời điểm đăng nhập cuối (last login) và IP của staff. **[Giả định]** |

### FR-03: Validate trường bắt buộc
| | |
|---|---|
| Mô tả | Email và Password là trường bắt buộc. |
| Kết quả mong đợi | Bỏ trống trường nào thì hiển thị thông báo lỗi của trường đó, không đăng nhập. **[Giả định – thông báo mặc định CodeIgniter]**: <br>• `The Email Address field is required.` <br>• `The Password field is required.` |
| Ghi chú | Các trường không có `required` nên form vẫn được gửi lên server khi để trống. **[Quan sát]** |

### FR-04: Validate định dạng Email
| | |
|---|---|
| Mô tả | Email phải đúng định dạng `local@domain`. |
| Phía client | Trình duyệt chặn submit và hiển thị tooltip HTML5 (ví dụ thiếu ký tự `@`). **[Quan sát – do `type="email"`]** |
| Phía server | Nếu bypass client, hiển thị `The Email Address field must contain a valid email address.` **[Giả định]** |

### FR-05: Đăng nhập thất bại – sai thông tin
| | |
|---|---|
| Mô tả | Email không tồn tại **hoặc** mật khẩu sai. |
| Kết quả | Hiển thị thông báo chung `Invalid email or password`, giữ người dùng ở trang login, ô Password bị xóa. **[Giả định]** |
| BR liên quan | BR-03 (không tiết lộ trường nào sai). |

### FR-06: Đăng nhập thất bại – tài khoản bị vô hiệu hóa
| | |
|---|---|
| Mô tả | Email + mật khẩu đúng nhưng tài khoản Staff ở trạng thái Inactive. |
| Kết quả | Từ chối đăng nhập, hiển thị thông báo `Inactive Account` (hoặc tương đương). **[Giả định]** |

### FR-07: Remember me
| | |
|---|---|
| Mô tả | Khi tick "Remember me" và đăng nhập thành công, hệ thống lưu cookie auto-login. |
| Kết quả | Đóng và mở lại trình duyệt trong thời hạn cookie → vẫn ở trạng thái đăng nhập. Không tick → phiên kết thúc theo vòng đời session. **[Giả định]** |
| Ghi chú | Thời hạn cookie cần xác nhận (xem mục 10). |

### FR-08: Bảo vệ trang Admin khi chưa đăng nhập
| | |
|---|---|
| Mô tả | Mọi URL `/admin/*` (trừ các trang authentication) yêu cầu đăng nhập. |
| Kết quả | Truy cập `/admin/dashboard` khi chưa đăng nhập → chuyển hướng về `/admin/authentication`. **[Quan sát]** |

### FR-09: Điều hướng Forgot Password
| | |
|---|---|
| Mô tả | Click link "Forgot Password?" trên màn hình Login. |
| Kết quả | Mở `/admin/authentication/forgot_password` cùng tab. **[Quan sát]** |

### FR-10: Yêu cầu đặt lại mật khẩu
| | |
|---|---|
| Mô tả | Người dùng nhập Email và nhấn **Confirm**. |
| Kết quả – email hợp lệ, tồn tại | Gửi email chứa link đặt lại mật khẩu; hiển thị thông báo kiểu `Check your email for further instructions resetting your password`. **[Giả định]** |
| Kết quả – để trống / sai định dạng | Hiển thị lỗi validate tương tự FR-03/FR-04. **[Giả định]** |
| Kết quả – email không tồn tại | Hiển thị thông báo lỗi (cần xác nhận nội dung; xem rủi ro user enumeration ở mục 10). **[Giả định]** |
| Kết quả – tài khoản Inactive | Không gửi email, hiển thị thông báo tài khoản không hoạt động. **[Giả định]** |

### FR-11: Đăng xuất (liên quan)
| | |
|---|---|
| Mô tả | Sau khi đăng xuất, session và cookie Remember me bị hủy; nhấn Back không truy cập lại được trang admin. **[Giả định]** |

---

## 5. Quy tắc nghiệp vụ (Business Rules)

| Mã | Quy tắc | Nguồn |
|---|---|---|
| BR-01 | Email và Password là bắt buộc. | Giả định |
| BR-02 | Email phải đúng định dạng email. | Quan sát (`type=email`) |
| BR-03 | Thông báo lỗi đăng nhập không được tiết lộ email hay mật khẩu sai (thông báo chung). | Giả định |
| BR-04 | Chỉ tài khoản Staff ở trạng thái Active mới đăng nhập được. | Giả định |
| BR-05 | Mật khẩu được che khi nhập và không bị giữ lại trên form sau khi đăng nhập thất bại. | Quan sát (`type=password`) / Giả định |
| BR-06 | Mọi request POST phải kèm CSRF token hợp lệ; token sai/hết hạn → request bị từ chối. | Quan sát (hidden field) / Giả định (hành vi) |
| BR-07 | Email so khớp không phân biệt hoa thường; khoảng trắng đầu/cuối được loại bỏ. | Giả định – cần xác nhận |
| BR-08 | Mật khẩu phân biệt hoa thường. | Giả định |
| BR-09 | Toàn bộ khu vực `/admin/*` yêu cầu phiên đăng nhập hợp lệ. | Quan sát |

---

## 6. Use Case

### UC-01: Đăng nhập hệ thống
| Mục | Nội dung |
|---|---|
| Actor | Staff / Admin |
| Tiền điều kiện | Có tài khoản Active; đang ở trang Login |
| Luồng chính | 1. Người dùng nhập Email. <br>2. Người dùng nhập Password. <br>3. (Tùy chọn) Tick Remember me. <br>4. Nhấn Login hoặc Enter. <br>5. Hệ thống xác thực thành công. <br>6. Hệ thống chuyển tới Dashboard. |
| Luồng thay thế | **A1** – Bỏ trống trường bắt buộc → hiển thị lỗi (FR-03), quay lại bước 1. <br>**A2** – Sai định dạng email → trình duyệt chặn submit (FR-04). <br>**A3** – Sai email/mật khẩu → hiển thị `Invalid email or password` (FR-05). <br>**A4** – Tài khoản Inactive → từ chối (FR-06). <br>**A5** – CSRF token hết hạn (để trang mở lâu) → request bị từ chối, yêu cầu tải lại trang (BR-06). |
| Hậu điều kiện | Session được tạo; last login được cập nhật. |

### UC-02: Quên mật khẩu
| Mục | Nội dung |
|---|---|
| Actor | Staff |
| Luồng chính | 1. Tại trang Login, click "Forgot Password?". <br>2. Nhập Email đã đăng ký. <br>3. Nhấn Confirm. <br>4. Hệ thống gửi email hướng dẫn và hiển thị thông báo thành công. |
| Luồng thay thế | **A1** – Email trống/sai định dạng → lỗi validate. <br>**A2** – Email không tồn tại → thông báo lỗi. <br>**A3** – Tài khoản Inactive → thông báo tài khoản không hoạt động. |

---

## 7. Yêu cầu phi chức năng (Non-Functional Requirements)

| Mã | Loại | Yêu cầu |
|---|---|---|
| NFR-01 | Bảo mật | Toàn bộ trang đăng nhập phục vụ qua HTTPS. **[Quan sát]** |
| NFR-02 | Bảo mật | Form có CSRF token. **[Quan sát]** |
| NFR-03 | Bảo mật | Mật khẩu lưu dạng băm (hash), không truyền/lưu dạng plain text ngoài request HTTPS. **[Giả định]** |
| NFR-04 | Bảo mật | Cookie session nên có cờ `HttpOnly`, `Secure`. Cookie không đọc được qua `document.cookie` khi chưa đăng nhập. **[Quan sát một phần]** |
| NFR-05 | Bảo mật | Chống brute-force: giới hạn số lần đăng nhập sai / reCAPTCHA. **Hiện chưa thấy reCAPTCHA trên trang** – cần xác nhận cấu hình. **[Quan sát]** |
| NFR-06 | Bảo mật | Không có lỗi SQL Injection / XSS khi nhập ký tự đặc biệt vào Email/Password. |
| NFR-07 | Hiệu năng | Trang Login tải < 3 giây; phản hồi đăng nhập < 2 giây trong điều kiện bình thường. |
| NFR-08 | Khả dụng | Hỗ trợ submit bằng phím Enter; thứ tự Tab: Email → Password → Remember me → Login → Forgot Password?. |
| NFR-09 | Khả dụng | Ô Email tự động focus khi tải trang. **[Quan sát]** |
| NFR-10 | Accessibility | Mỗi input có `<label for>` liên kết đúng id. **[Quan sát]** |
| NFR-11 | Responsive | Hiển thị tốt trên mobile (≥ 320px), tablet, desktop. |
| NFR-12 | Tương thích | Hoạt động trên Chrome, Firefox, Edge, Safari phiên bản mới nhất. |

---

## 8. Tiêu chí chấp nhận (Acceptance Criteria)

| Mã | Given | When | Then |
|---|---|---|---|
| AC-01 | Tài khoản `admin@example.com` Active | Nhập đúng email + mật khẩu, nhấn Login | Vào Dashboard `/admin` |
| AC-02 | Ở trang Login | Để trống cả 2 trường, nhấn Login | Hiển thị lỗi bắt buộc cho Email và Password |
| AC-03 | Ở trang Login | Nhập email `admin@example` thiếu domain hợp lệ / `adminexample.com` | Không đăng nhập; hiển thị lỗi định dạng |
| AC-04 | Ở trang Login | Nhập email đúng, mật khẩu sai | Hiển thị `Invalid email or password`; ô Password trống |
| AC-05 | Ở trang Login | Nhập email không tồn tại | Hiển thị cùng thông báo như AC-04 |
| AC-06 | Đã tick Remember me và đăng nhập thành công | Đóng, mở lại trình duyệt, vào `/admin` | Vẫn ở trạng thái đăng nhập |
| AC-07 | Chưa đăng nhập | Truy cập trực tiếp `/admin/dashboard` | Chuyển hướng về `/admin/authentication` |
| AC-08 | Ở trang Login | Click "Forgot Password?" | Mở trang Forgot Password |
| AC-09 | Ở trang Forgot Password | Nhập email hợp lệ đã đăng ký, nhấn Confirm | Thông báo đã gửi email; email reset được nhận |
| AC-10 | Ở trang Login | Nhập Password | Ký tự hiển thị dạng che (●) |

---

## 9. Ma trận truy vết (Traceability)

| Requirement | Business Rule | Use Case | Acceptance Criteria |
|---|---|---|---|
| FR-01 | – | UC-01 | AC-10 |
| FR-02 | BR-04 | UC-01 | AC-01 |
| FR-03 | BR-01 | UC-01 (A1) | AC-02 |
| FR-04 | BR-02 | UC-01 (A2) | AC-03 |
| FR-05 | BR-03, BR-08 | UC-01 (A3) | AC-04, AC-05 |
| FR-06 | BR-04 | UC-01 (A4) | – |
| FR-07 | – | UC-01 | AC-06 |
| FR-08 | BR-09 | – | AC-07 |
| FR-09 | – | UC-02 | AC-08 |
| FR-10 | BR-01, BR-02 | UC-02 | AC-09 |
| FR-11 | – | – | – |

---

## 10. Giả định, câu hỏi mở và đề xuất

### 10.1 Câu hỏi cần BA/PO xác nhận
1. Nội dung chính xác các thông báo lỗi (FR-03, FR-04, FR-05, FR-06, FR-10) – hiện dựa trên mặc định Perfex CRM.
2. Có giới hạn số lần đăng nhập sai / khóa tài khoản tạm thời không? Có bật reCAPTCHA không?
3. Thời hạn cookie "Remember me" là bao lâu?
4. Thời gian timeout session khi không thao tác?
5. Email có được trim khoảng trắng và so khớp không phân biệt hoa thường không?
6. Độ dài tối đa của Email/Password? (Form không đặt `maxlength`.)
7. Sau đăng nhập, có redirect về đúng trang đã yêu cầu trước đó không?
8. Trang Forgot Password với email không tồn tại hiển thị thông báo gì? (Rủi ro lộ thông tin tài khoản – user enumeration.)
9. Link đặt lại mật khẩu có hiệu lực trong bao lâu?

### 10.2 Đề xuất cải tiến
- Thêm link **"Back to Login"** trên trang Forgot Password.
- Thêm thuộc tính `required` và `autocomplete="username"` / `autocomplete="current-password"` cho input để cải thiện UX và tương thích trình quản lý mật khẩu.
- Cho phép zoom trên mobile (bỏ `user-scalable=no, maximum-scale=1`) để đáp ứng accessibility (WCAG 1.4.4).
- Thay `value="estimate"` của checkbox Remember me bằng giá trị có ý nghĩa (vd. `1`) – chỉ là vấn đề kỹ thuật, không ảnh hưởng người dùng.
- Cân nhắc thêm tính năng hiện/ẩn mật khẩu (icon con mắt).

---

## 11. Dữ liệu test tham khảo

| Mục đích | Email | Password |
|---|---|---|
| Tài khoản Admin hợp lệ | `admin@example.com` | *(được cung cấp riêng – không lưu trong tài liệu)* |
| Email không tồn tại | `notexist@example.com` | bất kỳ |
| Email sai định dạng | `admin@`, `adminexample.com`, `@example.com` | bất kỳ |
| Ký tự đặc biệt / injection | `' OR '1'='1`, `<script>alert(1)</script>` | `' OR '1'='1` |
| Khoảng trắng | `  admin@example.com  ` | mật khẩu đúng |
| Hoa thường | `ADMIN@EXAMPLE.COM` | mật khẩu đúng / đổi hoa thường |
