# Yêu cầu đề tài của giảng viên

| Mục | Nội dung |
| --- | --- |
| Môn | Triển khai và Quản trị Hệ thống Phần mềm |
| Lớp | K23 |
| Đề | Đề 14 — Hệ thống Quản lý Dự án (Project Management) |
| Nguồn | `docs/De_Tai_Thuc_Hanh - TKQT HTPM K23.pdf` |
| Ngày đối chiếu | 08/10/2026 |

Tài liệu này gắn bộ đặc tả với đề thầy giao. Phần nghiệp vụ (dự án, công việc, thành viên) nằm ở các tài liệu 01–05 và 08. Phần được chấm điểm nằm ở triển khai Docker Compose.

## 1. Đề 14

| Hạng mục | Yêu cầu của thầy | Cách làm trong đồ án |
| --- | --- | --- |
| Phần mềm | Ứng dụng web quản lý dự án, task, thành viên | Giữ các tài liệu nghiệp vụ đã có. Mức tối thiểu để đúng đề là dự án, công việc và thành viên |
| Cơ sở dữ liệu | PostgreSQL + pgAdmin | PostgreSQL 16 và pgAdmin 4 trong Compose |
| Kỹ thuật | Giống yêu cầu chung | Nginx, Prometheus, Grafana, Loki, Promtail, hardening |
| Stack gợi ý | Node.js (Nest hoặc Express) hoặc Django | Node.js 20 và Express, React cho giao diện |

## 2. Sáu hạng mục bắt buộc

| Bước | Việc phải có | Commit gợi ý | Điểm |
| --- | --- | --- | --- |
| 1 | Mã nguồn và file cấu hình trên GitHub, README hướng dẫn chạy. Tài khoản GitHub đặt theo mã số sinh viên | Commit tài liệu và mã ứng dụng phải có nội dung rõ | 1.5, chung với toàn bộ lịch sử |
| 2 | Ứng dụng web chạy ổn định, nối được database, pgAdmin hoạt động | Cùng mốc ứng dụng, trước hoặc cùng commit Nginx | 1.5 |
| 3 | Nginx reverse proxy, truy cập website qua Nginx, có HTTPS tự ký hoặc security headers | Commit 1 | 1.5 |
| 4 | Prometheus thu metrics; Grafana có dashboard cho container, web server và database | Commit 2 | 1.5 |
| 5 | Loki + Promtail, truy vấn LogQL ít nhất 2–3 câu | Commit 3 | 1.5 |
| 6 | Ít nhất 3–4 biện pháp hardening | Nằm trong các commit trên, được mô tả trong README và báo cáo | 1.5 |

Mục trình bày và demo toàn hệ thống bằng một lệnh Docker Compose: 1.0 điểm. Báo cáo tổng hợp tối thiểu 10 trang, bìa theo mẫu báo cáo thực tập, có hình minh họa kết quả sáu bước.

Thầy yêu cầu đủ 3 commit có nội dung. Ba commit được chấm là commit Nginx, commit Prometheus + Grafana, và commit Loki + Promtail. Không gộp cả ba vào một commit. Không cần sửa commit tài liệu đã đẩy trước đó.

## 3. Việc demo phải mở được

| Thành phần | Đường dẫn mặc định |
| --- | --- |
| Website qua Nginx | `http://localhost:8080` |
| HTTPS tự ký | `https://localhost:8443` |
| pgAdmin | `http://127.0.0.1:5050` |
| Grafana | `http://127.0.0.1:3001` |
| Câu LogQL | Chạy trong Grafana Explore, datasource Loki |

PostgreSQL không mở cổng ra máy host. pgAdmin nối tới host `postgres` trong mạng Docker.

## 4. Ba câu LogQL tối thiểu

Các câu này được lưu tại `deploy/observability/logql.md` khi hiện thực, và đưa vào báo cáo.

```logql
{service="api"}
```

```logql
{service="api"} |= "error"
```

```logql
sum by (service) (count_over_time({service=~"api|gateway"}[5m]))
```

## 5. Hardening bắt buộc trong đồ án

| Mã | Biện pháp |
| --- | --- |
| HD-01 | Container ứng dụng, Nginx và exporter chạy bằng user không phải root |
| HD-02 | Tách mạng `edge`, `app`, `data`, `obs`. Postgres chỉ nằm trên mạng `data` |
| HD-03 | Mật khẩu database, pgAdmin, Grafana và JWT nằm trong `.env`, không commit |
| HD-04 | Nginx gửi security headers |
| HD-05 | API dùng user database chỉ có quyền trên database `pms`, không dùng superuser |
| HD-06 | Cổng pgAdmin và Grafana chỉ gắn `127.0.0.1` |

## 6. Dàn ý báo cáo

Bìa: tên trường, môn Triển khai và Quản trị Hệ thống Phần mềm, đề 14, họ tên, mã sinh viên, lớp, giảng viên.

| Phần | Nội dung cần có ảnh |
| --- | --- |
| Mở đầu | Lý do chọn đề 14, phạm vi |
| Cấu trúc hệ thống | Sơ đồ container và mạng |
| Cách hoạt động | Luồng người dùng vào Nginx, API, PostgreSQL |
| Bước GitHub | Ảnh repository và 3 commit |
| Ứng dụng và database | Ảnh website, ảnh pgAdmin thấy bảng |
| Nginx | Ảnh truy cập qua cổng 8080 và response header |
| Giám sát | Ảnh dashboard Grafana: container, Nginx, PostgreSQL |
| Log | Ảnh 3 câu LogQL có kết quả |
| Hardening | Bảng biện pháp và ảnh minh chứng, ví dụ `docker inspect` user không phải root |
| Kết luận | Những gì đã chạy và phần chưa làm |

## 7. Kho GitHub hiện tại

Remote đang là `https://github.com/tuanhdevvn/Project-Management-ICTU.git`. Đề yêu cầu tài khoản GitHub đặt theo mã số sinh viên. Cần đối chiếu với thầy trước khi đổi tên tài khoản hoặc tạo repository mới.
