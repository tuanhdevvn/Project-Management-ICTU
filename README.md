# Hệ thống Quản lý Dự án (PMS-ICTU)

Đề 14 môn Triển khai và Quản trị Hệ thống Phần mềm. Ứng dụng web quản lý dự án, task và thành viên, chạy bằng Docker Compose trên Docker Desktop.

Phiên bản tài liệu: 1.1 — 08/10/2026.

## Tài liệu

Bắt đầu tại [đối chiếu đề của thầy](docs/00-yeu-cau-de-tai.md). Mục lục đầy đủ nằm trong [`docs/`](docs/README.md).

## Phạm vi

- Website: đăng nhập, dự án, thành viên, công việc. Quản trị viên cấp tài khoản.
- Hạ tầng được chấm: Nginx reverse proxy, PostgreSQL, pgAdmin, Prometheus, Grafana, Loki, Promtail và hardening.
- Ba commit kỹ thuật: Nginx, giám sát, log tập trung.

Mã nguồn ứng dụng chưa nằm trong kho này.
