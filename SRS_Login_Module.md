# SRS – Module Đăng nhập (Login)

| Thông tin | Giá trị |
|---|---|
| Hệ thống | Perfex CRM – Anh Tester Demo |
| URL | https://crm.anhtester.com/admin/authentication |
| Module | Authentication – Login (kèm Forgot Password) |
| Phiên bản tài liệu | 1.0 (Draft) |
| Ngày lập | 2026-10-28 |
| Nguồn thông tin | Khảo sát giao diện & DOM của trang Login và Forgot Password (chỉ quan sát, chưa thực hiện đăng nhập) |

> **Quy ước:** Mục đánh dấu **[Quan sát]** là thông tin lấy trực tiếp từ giao diện/DOM. Mục đánh dấu **[Giả định – cần xác nhận]** là hành vi suy luận từ chuẩn của ứng dụng web/Perfex CRM, chưa kiểm chứng trên hệ thống – tester cần xác minh khi thực thi.

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu mô tả yêu cầu chức năng và phi chức năng của module Đăng nhập trong trang quản trị (Admin) của Perfex CRM, làm cơ sở thiết kế test case và kiểm thử thủ công.

### 1.2 Phạm vi
- **Trong phạm vi:** Màn hình Login, checkbox Remember me, liên kết Forgot Password và màn hình Forgot Password (chỉ phần điều hướng và form nhập email).
- **Ngoài phạm vi:** Dashboard sau đăng nhập, phân quyền chi tiết, đăng ký tài khoản, Customer portal login, 2FA (không thấy trên giao diện).

### 1.3 Đối tượng sử dụng
| Vai trò | Mô tả |
|---|---|
| Staff / Admin | Nhân viên nội bộ có tài khoản trong hệ thống, đăng nhập để vào khu vực `/admin` |

### 1.4 Thuật ngữ
| Thuật ngữ | Ý nghĩa |
|---|---|
| CSRF token | Token chống tấn công Cross-Site Request Forgery, gửi kèm form |
| Remember me | Duy trì phiên đăng nhập lâu hơn phiên trình duyệt |

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh
Login là cổng vào bắt buộc của khu vực quản trị. Người dùng chưa xác thực khi truy cập các trang `/admin/*` được chuyển về trang này **[Giả định – cần xác nhận]**.

### 2.2 Giao diện màn hình Login **[Quan sát]**
Trang có tiêu đề tab: `Perfex CRM | Anh Tester Demo - Login`.

| STT | Thành phần | Loại | Thuộc tính quan sát được |
|---|---|---|---|
| 1 | Logo Anh Tester | Image (link về `/`) | Alt: "Perfex CRM \| Anh Tester Demo" |
| 2 | Tiêu đề "Login" | Heading | – |
| 3 | Email Address | Input `type=email`, `name=email`, `id=email` | Autofocus khi tải trang; không có `required`, `maxlength`, `placeholder`, `autocomplete` |
| 4 | Password | Input `type=password`, `name=password`, `id=password` | Không có `required`, `maxlength`, `placeholder`, `autocomplete` |
| 5 | Remember me | Checkbox `name=remember`, `id=remember` | Mặc định không chọn |
| 6 | Login | Button `type=submit` (xanh, full-width) | – |
| 7 | Forgot Password? | Link → `/admin/authentication/forgot_password` | – |
| 8 | CSRF token | Input hidden `name=csrf_token_name` | Có trong form |

Form submit: `POST https://crm.anhtester.com/admin/authentication`, không có `novalidate` (trình duyệt tự kiểm tra định dạng email của `type=email`).

### 2.3 Giao diện màn hình Forgot Password **[Quan sát]**
URL: `/admin/authentication/forgot_password`. Gồm: heading "Forgot Password", input Email Address (`type=email`), nút **Confirm**, CSRF token ẩn.

### 2.4 Ràng buộc & giả định
- Hệ thống chạy trên nền web, truy cập qua trình duyệt, giao thức HTTPS.
- Tài khoản test do đơn vị vận hành demo cung cấp; dữ liệu demo có thể bị reset bởi người dùng khác.

---

## 3. Yêu cầu chức năng

### FR-LOGIN-01 – Hiển thị màn hình đăng nhập
| Mục | Nội dung |
|---|---|
| Mô tả | Khi truy cập `/admin/authentication`, hệ thống hiển thị form đăng nhập |
| Tiền điều kiện | Người dùng chưa đăng nhập |
| Yêu cầu | (1) Hiển thị đủ các thành phần ở mục 2.2. (2) Con trỏ tự động focus vào ô Email. (3) Ô Password che ký tự nhập. (4) Remember me mặc định bỏ chọn. |
| Mức ưu tiên | Cao |

### FR-LOGIN-02 – Đăng nhập với thông tin hợp lệ
| Mục | Nội dung |
|---|---|
| Mô tả | Người dùng nhập Email và Password đúng của tài khoản đang hoạt động rồi nhấn **Login** |
| Tiền điều kiện | Tài khoản tồn tại, đang active |
| Luồng chính | 1. Nhập Email. 2. Nhập Password. 3. (Tuỳ chọn) tick Remember me. 4. Nhấn Login (hoặc Enter). 5. Hệ thống xác thực. 6. Chuyển vào khu vực quản trị (Dashboard) **[Giả định]**. |
| Hậu điều kiện | Phiên đăng nhập được tạo; người dùng truy cập được các chức năng theo quyền |
| Mức ưu tiên | Cao |

### FR-LOGIN-03 – Đăng nhập với thông tin không hợp lệ
| Mục | Nội dung |
|---|---|
| Mô tả | Hệ thống từ chối khi thông tin sai |
| Các trường hợp | Sai password; email không tồn tại; sai cả hai |
| Yêu cầu | Ở lại trang Login, hiển thị thông báo lỗi chung (không tiết lộ email hay password sai) **[Giả định – cần xác nhận nội dung thông báo]**; không tạo phiên |
| Mức ưu tiên | Cao |

### FR-LOGIN-04 – Kiểm tra dữ liệu đầu vào
| Mục | Nội dung |
|---|---|
| Yêu cầu | (1) Để trống Email và/hoặc Password: không đăng nhập được, có thông báo lỗi **[Giả định – ô input không có `required` nên validate nhiều khả năng ở phía server]**. (2) Email sai định dạng (vd `abc`, `abc@`): trình duyệt chặn submit bởi `type=email`. (3) Khoảng trắng đầu/cuối email: cần xác nhận có được trim không. (4) Không giới hạn `maxlength` ở client: cần xác nhận hệ thống xử lý chuỗi rất dài không lỗi. |
| Mức ưu tiên | Cao |

### FR-LOGIN-05 – Remember me
| Mục | Nội dung |
|---|---|
| Mô tả | Khi tick, phiên đăng nhập được duy trì sau khi đóng và mở lại trình duyệt; khi không tick, phiên kết thúc khi đóng trình duyệt **[Giả định – cần xác nhận thời hạn cookie]** |
| Mức ưu tiên | Trung bình |

### FR-LOGIN-06 – Quên mật khẩu
| Mục | Nội dung |
|---|---|
| Mô tả | Nhấn **Forgot Password?** chuyển tới `/admin/authentication/forgot_password`; người dùng nhập Email rồi nhấn **Confirm** |
| Yêu cầu | (1) Điều hướng đúng. (2) Email sai định dạng bị chặn. (3) Email hợp lệ: hệ thống gửi hướng dẫn đặt lại mật khẩu và hiển thị thông báo **[Giả định – cần xác nhận; trên môi trường demo có thể không gửi mail]**. (4) Email không tồn tại: hiển thị thông báo tương ứng. |
| Mức ưu tiên | Trung bình |

### FR-LOGIN-07 – Điều hướng & kiểm soát truy cập
| Mục | Nội dung |
|---|---|
| Yêu cầu | (1) Chưa đăng nhập mà truy cập trực tiếp trang `/admin/*` → chuyển về Login **[Giả định]**. (2) Đã đăng nhập mà truy cập lại `/admin/authentication` → chuyển về Dashboard **[Giả định]**. (3) Click logo → về trang chủ `/`. |
| Mức ưu tiên | Trung bình |

### FR-LOGIN-08 – Đăng xuất & hiệu lực phiên
| Mục | Nội dung |
|---|---|
| Yêu cầu | Sau khi đăng xuất, nhấn nút Back của trình duyệt không được hiển thị lại nội dung quản trị; phiên hết hạn buộc đăng nhập lại **[Giả định – cần xác nhận]** |
| Mức ưu tiên | Trung bình |

---

## 4. Yêu cầu phi chức năng

| Mã | Loại | Yêu cầu |
|---|---|---|
| NFR-SEC-01 | Bảo mật | Form có CSRF token; gửi sai/thiếu token bị từ chối **[Quan sát có token; hành vi từ chối cần xác nhận]** |
| NFR-SEC-02 | Bảo mật | Dữ liệu đăng nhập truyền qua HTTPS; password không xuất hiện trong URL |
| NFR-SEC-03 | Bảo mật | Chống SQL Injection / XSS ở các trường email, password |
| NFR-SEC-04 | Bảo mật | Thông báo lỗi không tiết lộ tài khoản có tồn tại hay không |
| NFR-SEC-05 | Bảo mật | Cơ chế chống brute-force (khoá tạm/giới hạn số lần) **[Giả định – cần xác nhận]** |
| NFR-SEC-06 | Bảo mật | Password không bị trình duyệt hiển thị dạng rõ; ô password là `type=password` **[Quan sát]** |
| NFR-USA-01 | Khả dụng | Nhấn Enter ở ô Email/Password thực hiện submit; thứ tự Tab: Email → Password → Remember me → Login → Forgot Password |
| NFR-USA-02 | Khả dụng | Label gắn với input (click label focus đúng ô) **[Quan sát có label]** |
| NFR-UI-01 | Giao diện | Hiển thị đúng trên Chrome/Firefox/Edge bản mới; responsive trên mobile/tablet |
| NFR-PERF-01 | Hiệu năng | Trang Login tải < 3 giây; xử lý xác thực < 3 giây trong điều kiện mạng bình thường **[Giả định]** |

---

## 5. Quy tắc nghiệp vụ

| Mã | Quy tắc |
|---|---|
| BR-01 | Định danh đăng nhập là **Email**, không dùng username |
| BR-02 | Chỉ tài khoản staff đang active mới đăng nhập được **[Giả định]** |
| BR-03 | Email so khớp không phân biệt hoa/thường; password phân biệt hoa/thường **[Giả định – cần xác nhận]** |

---

## 6. Dữ liệu kiểm thử gợi ý

| Nhóm | Email | Password | Kết quả mong đợi |
|---|---|---|---|
| Hợp lệ | Tài khoản admin demo do người dùng cung cấp | Mật khẩu tương ứng | Đăng nhập thành công |
| Sai password | Email hợp lệ | `wrong_pass` | Lỗi, ở lại Login |
| Email không tồn tại | `notexist@example.com` | `123456` | Lỗi, ở lại Login |
| Trống | (rỗng) | (rỗng) | Lỗi / bị chặn |
| Email sai định dạng | `abc`, `abc@`, `@x.com` | bất kỳ | Trình duyệt chặn |
| Khoảng trắng | ` email ` | hợp lệ | Xác nhận có trim không |
| Ký tự đặc biệt/injection | `' OR 1=1 --`, `<script>alert(1)</script>` | bất kỳ | Bị từ chối, không lỗi hệ thống |
| Chuỗi dài | 256+ ký tự | 256+ ký tự | Xử lý an toàn |

> Thông tin đăng nhập thật không được lưu trong tài liệu này.

---

## 7. Ma trận truy vết (rút gọn)

| Yêu cầu | Nhóm test case gợi ý |
|---|---|
| FR-LOGIN-01 | UI/hiển thị mặc định |
| FR-LOGIN-02 | Đăng nhập thành công, Enter key |
| FR-LOGIN-03 | Đăng nhập thất bại |
| FR-LOGIN-04 | Validate input, boundary |
| FR-LOGIN-05 | Remember me |
| FR-LOGIN-06 | Forgot password |
| FR-LOGIN-07/08 | Điều hướng, session |
| NFR-SEC-* | Bảo mật |

---

## 8. Điểm cần xác nhận sau khi thực thi test

1. Nội dung thông báo lỗi chính xác khi sai thông tin / để trống.
2. Trang đích sau đăng nhập thành công.
3. Thời hạn phiên khi bật/tắt Remember me.
4. Cơ chế khoá tài khoản/CAPTCHA khi nhập sai nhiều lần.
5. Hành vi của Forgot Password trên môi trường demo.
6. Xử lý khoảng trắng và phân biệt hoa/thường của email.
