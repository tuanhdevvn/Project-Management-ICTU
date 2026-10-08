# Hướng dẫn sử dụng

| Mục | Nội dung |
| --- | --- |
| Hệ thống | PMS-ICTU |
| Phiên bản | 1.0 |
| Đường dẫn | `http://localhost:8080` |
| Liên kết | [Triển khai Docker](06-trien-khai-docker.md) |

Tài liệu này mô tả cách dùng hệ thống sau khi cụm Docker đã chạy. Cách dựng môi trường nằm ở tài liệu triển khai.

## 1. Đăng nhập lần đầu

1. Mở `http://localhost:8080`.
2. Với người vận hành, đăng nhập bằng email và mật khẩu quản trị đã khai báo khi triển khai.
3. Với người dùng thường, chọn Đăng ký, nhập họ tên, email và mật khẩu có ít nhất 8 ký tự, gồm chữ và số.
4. Sau khi đăng ký, quay lại Đăng nhập.

Phiên làm việc kéo dài 8 giờ. Hết hạn, hệ thống yêu cầu đăng nhập lại.

## 2. Bảng điều khiển

Sau đăng nhập, trang chủ hiện:

- Số dự án đang tham gia.
- Số công việc của bạn theo bốn trạng thái: Cần làm, Đang làm, Chờ duyệt, Hoàn thành.
- Việc đến hạn trong 7 ngày tới.

Khi chưa có dự án, trang ở trạng thái trống.

## 3. Tạo dự án

1. Vào mục Dự án.
2. Chọn Tạo dự án.
3. Nhập tên (từ 3 ký tự), mô tả, ngày bắt đầu và ngày kết thúc.
4. Lưu.

Người tạo là chủ dự án. Trạng thái ban đầu là Lên kế hoạch. Có thể chuyển sang Đang thực hiện, Tạm dừng hoặc Hoàn thành. Dự án đã hoàn thành không nhận thêm công việc.

## 4. Mời thành viên

1. Mở dự án.
2. Chọn Thành viên, rồi Thêm.
3. Nhập email của tài khoản đã đăng ký.
4. Chọn vai trò Thành viên hoặc Quản lý.
5. Lưu.

Người được mời thấy dự án trong danh sách và nhận thông báo. Chủ dự án có thể chuyển quyền sở hữu cho một quản lý trước khi rời vai trò chủ.

## 5. Giao việc

1. Mở bảng công việc của dự án.
2. Tạo việc: tiêu đề, mô tả, ưu tiên, hạn và người thực hiện.
3. Thẻ mới nằm ở cột Cần làm.

Bốn cột theo thứ tự: Cần làm, Đang làm, Chờ duyệt, Hoàn thành. Mỗi lần chỉ chuyển sang cột kề bên, tiến hoặc lùi một bước.

Thành viên được gán có thể cập nhật mô tả và trạng thái việc của mình, đồng thời bình luận. Việc đổi người thực hiện, sửa hạn và xóa việc thuộc về quản lý dự án hoặc quản trị viên.

Ưu tiên gồm Thấp, Trung bình, Cao và Khẩn cấp.

## 6. Theo dõi tiến độ

Trong chi tiết dự án, phần trăm tiến độ là số việc Hoàn thành chia cho tổng số việc. Quản lý dự án xem thêm nhật ký: tạo dự án, thêm người, tạo việc và mỗi lần đổi trạng thái.

## 7. Thông báo

Biểu tượng chuông mở danh sách thông báo của riêng bạn. Chọn một dòng để đánh dấu đã đọc và mở dự án hoặc công việc liên quan, nếu đích đó còn tồn tại.

Các sự kiện có thông báo:

- Được thêm vào dự án.
- Được gán công việc.
- Người khác đổi trạng thái công việc của bạn.

## 8. Quản trị tài khoản

Mục này chỉ hiện với quản trị viên.

- Tìm người dùng theo tên hoặc email.
- Khóa tài khoản để chặn đăng nhập, sau đó có thể mở lại.
- Đổi vai trò hệ thống giữa Người dùng và Quản trị viên.

Hệ thống không cho quản trị viên tự khóa chính mình và không cho khóa quản trị viên cuối cùng.

## 9. Đăng xuất và đổi mật khẩu

Vào menu tài khoản ở góc màn hình để đổi mật khẩu hoặc đăng xuất. Sau khi đổi mật khẩu, lần đăng nhập sau dùng mật khẩu mới.
