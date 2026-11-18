# source-ai-test-manual

Repo thực hành **Manual Testing có hỗ trợ AI** (khóa Claude Testing Manual). Repo lưu các tài liệu kiểm thử như SRS và test case. Hệ thống được dùng để thực hành là bản demo **Perfex CRM của Anh Tester**.

## Hệ thống dùng để kiểm thử (AUT)

| Thông tin | Giá trị |
|---|---|
| Ứng dụng | Perfex CRM v3.1.6 – Anh Tester Demo |
| Trang đăng nhập Admin | https://crm.anhtester.com/admin/authentication |
| Tài khoản test | Liên hệ người phụ trách khóa học (không lưu mật khẩu trong repo) |

## Cấu trúc thư mục

```
source-ai-test-manual/
├── README.md
├── .gitignore
└── docs/
    ├── SRS_Login_Module_CRM_AnhTester.md   # Đặc tả yêu cầu (SRS) module Đăng nhập
    └── TestCase/
        ├── TC_Login.txt                     # Test case module Đăng nhập
        └── TC_Dashboard.txt                 # Test case module Dashboard (đang trống)
```

## Danh sách tài liệu

| Tài liệu | Mô tả | Trạng thái |
|---|---|---|
| [SRS_Login_Module_CRM_AnhTester.md](docs/SRS_Login_Module_CRM_AnhTester.md) | SRS module Login, gồm đặc tả UI, yêu cầu chức năng (FR), yêu cầu phi chức năng (NFR), quy tắc nghiệp vụ (BR), use case, tiêu chí chấp nhận, ma trận truy vết và dữ liệu test | Draft |
| [TC_Login.txt](docs/TestCase/TC_Login.txt) | Các bước test đăng nhập | Đang soạn |
| [TC_Dashboard.txt](docs/TestCase/TC_Dashboard.txt) | Test case Dashboard | Chưa có nội dung |

## Quy trình làm việc

1. **Khảo sát hệ thống:** truy cập AUT và ghi nhận giao diện, trường dữ liệu và luồng xử lý.
2. **Viết SRS:** lưu tại `docs/`, mỗi module một file `SRS_<Module>_....md`.
   - Gắn nhãn **[Quan sát]** cho yêu cầu đã kiểm chứng trên hệ thống.
   - Gắn nhãn **[Giả định]** cho yêu cầu cần BA/PO xác nhận.
3. **Thiết kế test case:** dựa trên SRS, lưu tại `docs/TestCase/`, đặt tên `TC_<Module>.*`.
4. **Thực thi và báo cáo:** chạy test case trên AUT và ghi nhận kết quả hoặc bug.

## Quy ước

- Tài liệu viết bằng **tiếng Việt**. Thuật ngữ kỹ thuật, tên trường và thông báo hệ thống giữ nguyên tiếng Anh.
- Mỗi test case nên tham chiếu mã yêu cầu trong SRS (ví dụ `FR-03`, `AC-04`) để đảm bảo truy vết.
- **Không commit mật khẩu** hay thông tin đăng nhập thật vào repo.
- Commit message ngắn gọn, mô tả rõ thay đổi (ví dụ: `add TC login negative cases`).

## Kế hoạch tiếp theo

- [ ] Xác nhận các yêu cầu đang gắn nhãn [Giả định] trong SRS Login bằng cách test thực tế
- [ ] Hoàn thiện `TC_Login` theo tiêu chí chấp nhận AC-01 → AC-10
- [ ] Viết SRS và test case cho module Dashboard
