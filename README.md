# Hệ thống Quản lý Dự án (PMS-ICTU)

Ứng dụng web quản lý dự án, thành viên và công việc, triển khai cục bộ bằng Docker Desktop.

Phiên bản tài liệu: 1.0 — 08/10/2026.

## Tài liệu

Bộ đặc tả nằm trong thư mục [`docs/`](docs/README.md):

1. [Đặc tả yêu cầu phần mềm](docs/01-srs-dac-ta-yeu-cau.md)
2. [Đặc tả ca sử dụng](docs/02-dac-ta-use-case.md)
3. [Thiết kế hệ thống](docs/03-thiet-ke-he-thong.md)
4. [Thiết kế cơ sở dữ liệu](docs/04-thiet-ke-csdl.md)
5. [Đặc tả API](docs/05-dac-ta-api.md)
6. [Triển khai Docker Desktop](docs/06-trien-khai-docker.md)
7. [Kế hoạch kiểm thử](docs/07-ke-hoach-kiem-thu.md)
8. [Hướng dẫn sử dụng](docs/08-huong-dan-su-dung.md)

## Phạm vi phiên bản 1.0

- Đăng ký, đăng nhập, phân quyền quản trị viên và người dùng.
- Dự án, thành viên, công việc theo bốn trạng thái, bình luận, nhật ký.
- Bảng điều khiển và thông báo trong hệ thống.
- Chạy bằng Docker Compose: Nginx, giao diện React, API Node.js, PostgreSQL 16.

Mã nguồn ứng dụng chưa nằm trong kho này. Tài liệu là bản đặc tả để triển khai tiếp.
