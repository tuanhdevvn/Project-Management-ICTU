# Kế hoạch kiểm thử

| Mục | Nội dung |
| --- | --- |
| Hệ thống | PMS-ICTU |
| Phiên bản | 1.0 |
| Liên kết | [SRS](01-srs-dac-ta-yeu-cau.md), [Ca sử dụng](02-dac-ta-use-case.md), [Docker](06-trien-khai-docker.md) |

## 1. Mục tiêu

Xác nhận các yêu cầu chức năng trong SRS và khả năng chạy cụm Docker Desktop. Kiểm thử chấp nhận thực hiện trên bản dựng bằng `docker compose`, qua trình duyệt tại `http://localhost:8080`.

## 2. Phạm vi

Trong phạm vi:

- Ca sử dụng UC-01 đến UC-15.
- Phân quyền tại API, không chỉ trên giao diện.
- Bền vững dữ liệu sau khi tắt và bật container.
- Cổng publish, pgAdmin, dashboard Grafana và ba câu LogQL.

Ngoài phạm vi:

- Kiểm thử tải ngoài mức NFR-01.
- Kiểm thử xâm nhập chuyên sâu.
- Tương thích trình duyệt ngoài Chrome và Edge bản hiện hành.

## 3. Môi trường

| Hạng mục | Giá trị |
| --- | --- |
| Nền tảng | Docker Desktop, engine đang chạy |
| Đường vào | `http://localhost:8080` |
| Dữ liệu | Volume mới, tạo bằng `down -v` rồi `up -d --build` trước đợt kiểm thử |
| Tài khoản sẵn | Admin hạt giống từ `.env` |
| Trình duyệt | Chrome hoặc Edge |

Trước mỗi đợt, ghi phiên bản tài liệu, ngày chạy và hệ điều hành máy host vào biên bản.

## 4. Dữ liệu kiểm thử

| Mã | Vai trò | Cách tạo |
| --- | --- | --- |
| AD-01 | Quản trị viên | Tài khoản hạt giống |
| US-01 | Người dùng, sau đó là owner dự án | AD-01 cấp tài khoản |
| US-02 | Thành viên được mời | AD-01 cấp tài khoản |
| US-03 | Người không thuộc dự án | AD-01 cấp tài khoản, không được mời |

Mật khẩu kiểm thử đạt quy tắc: ít nhất 8 ký tự, có chữ và số. Không dùng mật khẩu này ngoài máy local.

## 5. Chiến lược

| Mức | Cách làm | Thời điểm |
| --- | --- | --- |
| Đơn vị | Kiểm tra service nghiệp vụ: chuyển trạng thái, chặn owner tự khóa, tính tiến độ | Khi đã có mã API |
| Tích hợp | Gọi API kèm PostgreSQL trong Compose | Sau khi API nối được CSDL |
| Chấp nhận | Thao tác trên trình duyệt theo bảng ca bên dưới | Trước khi bàn giao |
| Triển khai | Các tiêu chí DEP-01 đến DEP-13 | Cùng đợt chấp nhận |

Một ca đạt khi kết quả quan sát khớp cột "Kết quả mong đợi" và cơ sở dữ liệu không phát sinh bản ghi ngoài mô tả.

## 6. Ca kiểm thử chấp nhận

| Mã | Việc cần làm | Kết quả mong đợi | Liên kết |
| --- | --- | --- | --- |
| TC-01 | AD-01 cấp tài khoản US-01 với email chưa dùng | Tài khoản xuất hiện trong danh sách, vai trò người dùng | FR-USER-04 |
| TC-02 | AD-01 cấp lại đúng email US-01 | Báo email đã được sử dụng | FR-USER-04 |
| TC-03 | AD-01 cấp tài khoản với mật khẩu tạm `1234567` | Báo chưa đạt quy tắc mật khẩu, chưa tạo tài khoản | FR-USER-04 |
| TC-03b | US-03 mở trang đăng nhập và tìm form đăng ký; gọi `POST /api/auth/register` | Không có form đăng ký; API trả 404 | FR-AUTH-01 |
| TC-04 | US-01 đăng nhập bằng mật khẩu tạm | Vào màn hình buộc đổi mật khẩu, chưa vào được bảng điều khiển | FR-AUTH-02, FR-AUTH-06 |
| TC-05 | US-01 đăng nhập sai mật khẩu | Báo một câu chung, không nói email có tồn tại hay không | FR-AUTH-02 |
| TC-06 | Đổi mật khẩu US-01 rồi đăng nhập bằng mật khẩu mới | Vào bảng điều khiển, không còn màn hình buộc đổi mật khẩu | FR-AUTH-04, FR-AUTH-06 |
| TC-07 | Xóa token (đăng xuất) rồi mở thẳng đường dẫn dự án | Quay về đăng nhập | FR-AUTH-03, FR-AUTH-05 |
| TC-08 | AD-01 tìm US-01 trong danh sách người dùng | Thấy đúng email | FR-USER-01 |
| TC-09 | AD-01 khóa US-01 | US-01 không đăng nhập được | FR-USER-02 |
| TC-10 | AD-01 tự khóa chính mình | Bị từ chối, tài khoản AD-01 vẫn hoạt động | FR-USER-02 |
| TC-11 | AD-01 mở khóa US-01 | US-01 đăng nhập lại được | FR-USER-02 |
| TC-12 | US-01 tạo dự án ngày kết thúc trước ngày bắt đầu | Bị từ chối, chưa có dự án | FR-PRJ-01 |
| TC-13 | US-01 tạo dự án hợp lệ tên "Đồ án PMS" | US-01 là owner, trạng thái Lên kế hoạch | FR-PRJ-01 |
| TC-14 | US-03 mở danh sách dự án | Không thấy "Đồ án PMS" | FR-PRJ-02 |
| TC-15 | US-01 mời US-02 với vai trò thành viên | US-02 thấy dự án và có thông báo được mời | FR-MEM-01, FR-NOTI-01 |
| TC-16 | Mời lại đúng email US-02 | Báo đã là thành viên | FR-MEM-01 |
| TC-17 | US-01 tạo việc ưu tiên cao, gán US-02, hạn trong 3 ngày | Thẻ nằm cột Cần làm; US-02 có thông báo được gán | FR-TASK-01, FR-NOTI-01 |
| TC-18 | US-02 chuyển việc đó sang Đang làm rồi sang Chờ duyệt | Mỗi lần một bước, cột đổi đúng | FR-TASK-03 |
| TC-19 | US-02 chuyển từ Cần làm thẳng sang Hoàn thành | Bị từ chối nếu trạng thái hiện tại không kề | FR-TASK-03 |
| TC-20 | US-02 sửa người thực hiện của việc | Bị từ chối | FR-TASK-02 |
| TC-21 | US-02 bình luận "Đã xong phần sơ đồ" | Bình luận hiện kèm tên US-02 | FR-CMT-01 |
| TC-22 | US-01 mở nhật ký dự án | Có dòng tạo dự án, thêm thành viên, tạo việc, đổi trạng thái | FR-CMT-02 |
| TC-23 | US-02 mở bảng điều khiển | Thấy việc sắp đến hạn | FR-DASH-01 |
| TC-24 | Đưa việc sang Hoàn thành, xem tiến độ dự án chỉ có một việc | Tiến độ 100% | FR-DASH-02 |
| TC-25 | US-01 chuyển dự án sang Hoàn thành rồi tạo việc mới | Bị từ chối | FR-PRJ-05 |
| TC-26 | US-02 gọi xóa dự án | Bị từ chối | FR-PRJ-06 |
| TC-27 | US-01 xóa dự án sau khi xác nhận | US-02 không còn thấy dự án | FR-PRJ-06 |
| TC-28 | Tạo lại dự án, tắt cụm bằng `down`, bật lại, đăng nhập | Dự án vẫn còn | NFR-05, DEP-06 |
| TC-29 | Kiểm tra cổng host | Có 8080; không có 5432 | DEP-02, NFR-04 |
| TC-30 | Mở `/api/health` | Trả trạng thái ok | DEP-04 |

## 7. Ca phân quyền qua API

Thực hiện bằng công cụ gọi HTTP, dùng token của US-03.

| Mã | Lời gọi | Kết quả |
| --- | --- | --- |
| TC-31 | `GET /api/projects/{id}` của dự án không tham gia | 404 |
| TC-32 | `POST /api/projects/{id}/tasks` | 403 hoặc 404 |
| TC-33 | `GET /api/users` | 403 |
| TC-34 | `GET /api/projects` không kèm token | 401 |

## 8. Biên bản

Mỗi ca ghi: mã, người chạy, ngày, đạt hoặc không đạt, ảnh hưởng nếu không đạt, ghi chú.

Đợt kiểm thử đạt khi:

- TC-01 đến TC-34 không còn ca bắt buộc nào thất bại.
- DEP-01 đến DEP-13 trong tài liệu Docker đều đạt.
- Lỗi còn lại được ghi là ngoài phạm vi hoặc đã có cách xử lý chấp nhận được với giảng viên.

## 9. Rủi ro kiểm thử

| Rủi ro | Cách giảm |
| --- | --- |
| Volume cũ làm dữ liệu hạt giống không tạo lại | Chạy `down -v` trước đợt, chỉ trên máy local |
| Cổng 8080 bị ứng dụng khác chiếm | Đổi `WEB_PORT` và ghi rõ URL thực tế vào biên bản |
| Token hết hạn giữa chừng | Đăng nhập lại; hạn mặc định 8 giờ đủ cho một đợt |
