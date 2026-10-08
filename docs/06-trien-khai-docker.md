# Triển khai bằng Docker Desktop

| Mục | Nội dung |
| --- | --- |
| Hệ thống | PMS-ICTU |
| Phiên bản | 1.0 |
| Công cụ | Docker Desktop, Docker Compose v2 |
| Liên kết | [Thiết kế hệ thống](03-thiet-ke-he-thong.md) |

## 1. Mục tiêu triển khai

Người nghiệm thu chỉ cần Docker Desktop. Không cài Node.js, PostgreSQL hay Nginx lên máy host. Một lệnh dựng bốn dịch vụ, trình duyệt mở `http://localhost:8080`.

## 2. Yêu cầu máy

| Hạng mục | Mức tối thiểu |
| --- | --- |
| Docker Desktop | Bản còn được hỗ trợ, bật engine Linux containers |
| CPU / RAM cấp cho Docker | 2 CPU, 4 GB RAM |
| Đĩa trống | 5 GB cho image và volume |
| Cổng host còn trống | 8080 |

Kiểm tra nhanh:

```bash
docker version
docker compose version
```

Cả hai lệnh cần in phiên bản client và server. Nếu server không chạy, mở Docker Desktop và đợi trạng thái Engine running.

## 3. Thành phần

| Dịch vụ | Image / cách dựng | Vai trò | Cổng |
| --- | --- | --- | --- |
| `gateway` | Nginx 1.27 | Nhận lưu lượng từ host, phát tệp giao diện và chuyển `/api` | Host `8080` → container `80` |
| `web` | Build từ `apps/web` | Ứng dụng React đã build, chỉ trong mạng nội bộ | Không publish |
| `api` | Build từ `apps/api`, Node.js 20 | REST API | Không publish, lắng nghe `3000` trong mạng |
| `postgres` | `postgres:16` | Cơ sở dữ liệu | Không publish, lắng nghe `5432` trong mạng |

`web` có thể được gộp vào image `gateway` ở bước hiện thực (Nginx phục vụ tệp tĩnh và ủy quyền API). Đặc tả chấp nhận cả hai cách, miễn là từ phía host chỉ có cổng 8080 và đường dẫn `/` cùng `/api` giữ nguyên.

```mermaid
flowchart TB
    subgraph host [Máy host]
      Browser[Trình duyệt localhost:8080]
    end
    subgraph compose [Docker Compose - mạng pms_net]
      Gateway[gateway]
      Web[web]
      Api[api]
      Db[postgres]
      Vol[(volume pms_pg_data)]
    end
    Browser --> Gateway
    Gateway --> Web
    Gateway --> Api
    Api --> Db
    Db --> Vol
```

## 4. Mạng, volume, khởi động

| Đối tượng | Tên | Quy tắc |
| --- | --- | --- |
| Mạng | `pms_net` | Bridge, các dịch vụ gọi nhau bằng tên |
| Volume | `pms_pg_data` | Gắn vào `/var/lib/postgresql/data` |
| Khởi tạo lược đồ | `database/init` | Mount vào `/docker-entrypoint-initdb.d` ở chế độ chỉ đọc |

Script trong `docker-entrypoint-initdb.d` chỉ chạy khi volume còn trống. Sửa file SQL sau đó không tự áp lên dữ liệu cũ. Khi cần dựng lại từ đầu, xóa volume bằng lệnh có chủ đích ở mục 8.

Thứ tự phụ thuộc:

1. `postgres` đạt healthcheck `pg_isready`.
2. `api` khởi động sau đó và chờ cơ sở dữ liệu.
3. `gateway` khởi động sau `api` và `web`.

Healthcheck của `api`: `GET /api/health` trả `200` và thân `{ "data": { "status": "ok" } }`. Endpoint này không yêu cầu JWT.

## 5. Biến môi trường

Tệp mẫu `.env.example` nằm ở gốc kho. Tệp `.env` thật không đưa vào git.

| Biến | Ví dụ | Dùng ở | Ý nghĩa |
| --- | --- | --- | --- |
| `POSTGRES_DB` | `pms` | postgres, api | Tên cơ sở dữ liệu |
| `POSTGRES_USER` | `pms` | postgres, api | Người dùng CSDL |
| `POSTGRES_PASSWORD` | đặt riêng | postgres, api | Mật khẩu CSDL |
| `DATABASE_URL` | `postgres://pms:matkhau@postgres:5432/pms` | api | Chuỗi kết nối, host là tên dịch vụ |
| `JWT_SECRET` | chuỗi ngẫu nhiên dài | api | Khóa ký token |
| `JWT_EXPIRES_IN` | `8h` | api | Hạn token |
| `SEED_ADMIN_NAME` | `Quản trị hệ thống` | api | Tên admin hạt giống |
| `SEED_ADMIN_EMAIL` | `admin@pms.local` | api | Email admin |
| `SEED_ADMIN_PASSWORD` | đặt riêng | api | Mật khẩu admin lần đầu |
| `WEB_PORT` | `8080` | gateway | Cổng publish ra host |

`DATABASE_URL` dùng hostname `postgres`, không dùng `localhost`. Trong một container, `localhost` là chính container đó.

## 6. Tệp Compose tham chiếu

Đường dẫn dự kiến: `deploy/docker-compose.yml`.

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - pms_pg_data:/var/lib/postgresql/data
      - ../database/init:/docker-entrypoint-initdb.d:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 20
    networks: [pms_net]
    restart: unless-stopped

  api:
    build:
      context: ../apps/api
    environment:
      DATABASE_URL: ${DATABASE_URL}
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: ${JWT_EXPIRES_IN}
      SEED_ADMIN_NAME: ${SEED_ADMIN_NAME}
      SEED_ADMIN_EMAIL: ${SEED_ADMIN_EMAIL}
      SEED_ADMIN_PASSWORD: ${SEED_ADMIN_PASSWORD}
    depends_on:
      postgres:
        condition: service_healthy
    networks: [pms_net]
    restart: unless-stopped

  web:
    build:
      context: ../apps/web
    networks: [pms_net]
    restart: unless-stopped

  gateway:
    image: nginx:1.27
    ports:
      - "${WEB_PORT:-8080}:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on: [web, api]
    networks: [pms_net]
    restart: unless-stopped

networks:
  pms_net:

volumes:
  pms_pg_data:
```

`deploy/nginx.conf` cần:

- `/api/` ủy quyền tới `http://api:3000`.
- `/` phục vụ giao diện. Nếu `web` là container riêng, dùng `proxy_pass` tới dịch vụ đó; nếu tệp tĩnh nằm trong image gateway thì dùng `root`.

Tiêu đề `Host` và `X-Forwarded-For` được chuyển tiếp. Kích thước thân yêu cầu cho phép tối thiểu 1 MB.

## 7. Quy trình chạy

Thực hiện tại thư mục gốc kho mã, sau khi đã có mã nguồn ứng dụng và tệp `.env`.

```bash
cp .env.example .env
docker compose -f deploy/docker-compose.yml up -d --build
docker compose -f deploy/docker-compose.yml ps
```

Kỳ vọng cột trạng thái của cả bốn dịch vụ là running, `postgres` và `api` là healthy.

Mở `http://localhost:8080`. Đăng nhập bằng tài khoản trong `SEED_ADMIN_EMAIL` và `SEED_ADMIN_PASSWORD`.

Xem log khi cần:

```bash
docker compose -f deploy/docker-compose.yml logs -f api
```

Dừng cụm, giữ dữ liệu:

```bash
docker compose -f deploy/docker-compose.yml down
```

## 8. Dữ liệu và đặt lại

| Việc | Lệnh | Kết quả |
| --- | --- | --- |
| Tắt cụm | `docker compose -f deploy/docker-compose.yml down` | Container mất, volume còn |
| Bật lại | `docker compose -f deploy/docker-compose.yml up -d` | Dữ liệu cũ còn |
| Xóa cả dữ liệu | `docker compose -f deploy/docker-compose.yml down -v` | Volume `pms_pg_data` bị xóa |

Chỉ dùng `down -v` khi cố ý dựng lại cơ sở dữ liệu trống.

## 9. Tiêu chí nghiệm thu triển khai

| Mã | Tiêu chí |
| --- | --- |
| DEP-01 | `docker compose up -d --build` thoát mã 0 trên Docker Desktop |
| DEP-02 | Chỉ cổng 8080 được publish; `5432` không mở trên host |
| DEP-03 | `http://localhost:8080` trả trang đăng nhập |
| DEP-04 | `http://localhost:8080/api/health` trả trạng thái ok |
| DEP-05 | Đăng nhập được bằng tài khoản hạt giống |
| DEP-06 | Tạo một dự án, tắt cụm bằng `down`, bật lại, dự án vẫn còn |
| DEP-07 | Tệp `.env` không nằm trong git |

## 10. Xử lý sự cố thường gặp

| Hiện tượng | Hướng xử lý |
| --- | --- |
| `port is already allocated` trên 8080 | Đổi `WEB_PORT` trong `.env` hoặc tắt tiến trình đang giữ cổng |
| `api` thoát ngay | Xem log API; thường do `DATABASE_URL` sai hostname hoặc `JWT_SECRET` trống |
| Trang web mở được nhưng gọi API lỗi | Kiểm tra `nginx.conf` đã trỏ `/api/` tới `http://api:3000` |
| Sửa SQL mà lược đồ không đổi | Volume đã khởi tạo. Dùng `down -v` rồi `up` lại nếu được phép xóa dữ liệu |
| Docker báo engine chưa chạy | Mở Docker Desktop, đợi engine, chạy lại `docker version` |
