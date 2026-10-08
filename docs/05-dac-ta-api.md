# Đặc tả API

| Mục | Nội dung |
| --- | --- |
| Hệ thống | PMS-ICTU |
| Phiên bản API | 1.0 |
| Gốc đường dẫn | `/api` |
| Định dạng | JSON, UTF-8 |
| Liên kết | [Thiết kế hệ thống](03-thiet-ke-he-thong.md), [CSDL](04-thiet-ke-csdl.md) |

## 1. Quy ước

- Xác thực: tiêu đề `Authorization: Bearer <jwt>`, trừ đăng ký và đăng nhập.
- Thời gian trong JSON dùng ISO-8601.
- Danh sách dùng phân trang query `page` (bắt đầu từ 1) và `pageSize` (mặc định 20, tối đa 100).
- Phản hồi danh sách:

```json
{
  "data": [],
  "meta": { "page": 1, "pageSize": 20, "total": 0 }
}
```

- Phản hồi một bản ghi bọc trong `{ "data": { } }`.
- Lỗi dùng cấu trúc ở mục xử lý lỗi của tài liệu thiết kế hệ thống.

Ngày nghiệp vụ gửi dạng `YYYY-MM-DD`.

## 2. Auth

### POST `/api/auth/register`

Không cần token.

Yêu cầu:

```json
{
  "fullName": "Nguyễn Văn A",
  "email": "a@ictu.edu.vn",
  "password": "Matkhau12",
  "confirmPassword": "Matkhau12"
}
```

Phản hồi `201`: `{ "data": { "id", "fullName", "email" } }`.

### POST `/api/auth/login`

```json
{ "email": "a@ictu.edu.vn", "password": "Matkhau12" }
```

Phản hồi `200`:

```json
{
  "data": {
    "token": "<jwt>",
    "expiresIn": 28800,
    "user": {
      "id": "uuid",
      "fullName": "Nguyễn Văn A",
      "email": "a@ictu.edu.vn",
      "systemRole": "USER"
    }
  }
}
```

### GET `/api/auth/me`

Trả hồ sơ người dùng hiện tại.

### POST `/api/auth/change-password`

```json
{
  "currentPassword": "Matkhau12",
  "newPassword": "Matkhau34",
  "confirmPassword": "Matkhau34"
}
```

Phản hồi `204` khi thành công.

## 3. Người dùng

Chỉ `ADMIN`.

| Phương thức | Đường dẫn | Mục đích |
| --- | --- | --- |
| GET | `/api/users?q=&page=&pageSize=` | Tìm theo tên hoặc email |
| PATCH | `/api/users/{id}/status` | Body `{ "status": "LOCKED" }` hoặc `ACTIVE` |
| PATCH | `/api/users/{id}/role` | Body `{ "systemRole": "ADMIN" }` hoặc `USER` |

## 4. Dự án

| Phương thức | Đường dẫn | Quyền | Mục đích |
| --- | --- | --- | --- |
| GET | `/api/projects` | Đã đăng nhập | Dự án mình tham gia; admin thấy tất cả |
| POST | `/api/projects` | Mọi tài khoản đang hoạt động | Tạo dự án, người tạo thành `OWNER` |
| GET | `/api/projects/{id}` | Thành viên hoặc admin | Chi tiết kèm số việc theo trạng thái |
| PATCH | `/api/projects/{id}` | Owner, manager, admin | Sửa tên, mô tả, ngày, trạng thái |
| DELETE | `/api/projects/{id}` | Owner hoặc admin | Xóa dự án |

Body tạo và sửa:

```json
{
  "name": "Xây dựng PMS",
  "description": "Đồ án môn học",
  "startDate": "2026-10-08",
  "endDate": "2026-12-20",
  "status": "ACTIVE"
}
```

Khi tạo, trường `status` bỏ qua và luôn lưu `PLANNING`. Mọi tài khoản đang hoạt động đều gọi được API này và trở thành chủ dự án vừa tạo. Quyền sửa, đóng và xóa vẫn theo vai trò trong từng dự án.

Query danh sách bổ sung: `status`.

Chi tiết trả thêm:

```json
{
  "myRole": "OWNER",
  "taskSummary": {
    "TODO": 2,
    "IN_PROGRESS": 1,
    "REVIEW": 0,
    "DONE": 3,
    "progress": 50
  }
}
```

`progress` là phần trăm làm tròn của số việc `DONE` trên tổng việc. Tổng bằng 0 thì `progress` bằng 0.

Phiên bản 1.0 cho mọi tài khoản `USER` đã đăng nhập được tạo dự án. Đây là cách để một thành viên trở thành quản lý của dự án do mình tạo. SRS giới hạn sửa và xóa theo vai trò trong từng dự án.

## 5. Thành viên dự án

| Phương thức | Đường dẫn | Quyền |
| --- | --- | --- |
| GET | `/api/projects/{id}/members` | Thành viên hoặc admin |
| POST | `/api/projects/{id}/members` | Owner, manager, admin |
| PATCH | `/api/projects/{id}/members/{userId}` | Owner hoặc admin |
| DELETE | `/api/projects/{id}/members/{userId}` | Owner, manager, admin |

Thêm thành viên:

```json
{ "email": "b@ictu.edu.vn", "projectRole": "MEMBER" }
```

Đổi vai trò: `{ "projectRole": "MANAGER" }`. Không dùng endpoint này để gán `OWNER`.

Chuyển sở hữu:

`POST /api/projects/{id}/transfer-ownership`

```json
{ "newOwnerId": "uuid" }
```

Người nhận phải đang là `MANAGER` của dự án. Chỉ owner hiện tại hoặc admin được gọi.

## 6. Công việc

| Phương thức | Đường dẫn | Quyền |
| --- | --- | --- |
| GET | `/api/projects/{id}/tasks` | Thành viên hoặc admin |
| POST | `/api/projects/{id}/tasks` | Owner, manager, admin |
| GET | `/api/tasks/{id}` | Thành viên của dự án chứa việc, hoặc admin |
| PATCH | `/api/tasks/{id}` | Theo FR-TASK-02 và FR-TASK-03 |
| DELETE | `/api/tasks/{id}` | Owner, manager, admin |

Query của danh sách: `status`, `assigneeId`, `priority`, `sort=dueDate`.

Body tạo:

```json
{
  "title": "Thiết kế lược đồ CSDL",
  "description": "Hoàn thiện từ điển dữ liệu",
  "priority": "HIGH",
  "dueDate": "2026-10-20",
  "assigneeId": "uuid"
}
```

Body sửa cho phép các trường trên cộng `status`. Thành viên thường chỉ gửi `description` và `status` cho việc của mình.

## 7. Bình luận và nhật ký

| Phương thức | Đường dẫn | Quyền |
| --- | --- | --- |
| GET | `/api/tasks/{id}/comments` | Thành viên dự án hoặc admin |
| POST | `/api/tasks/{id}/comments` | Thành viên dự án hoặc admin |
| GET | `/api/projects/{id}/activities` | Owner, manager, admin |

Body bình luận: `{ "body": "Đã cập nhật mục 3." }`.

Nhật ký trả `action`, `actorName`, `metadata`, `createdAt`, sắp mới nhất trước.

## 8. Bảng điều khiển và thông báo

### GET `/api/dashboard`

```json
{
  "data": {
    "projectCount": 2,
    "myTasks": { "TODO": 1, "IN_PROGRESS": 2, "REVIEW": 0, "DONE": 4 },
    "dueSoon": [
      {
        "id": "uuid",
        "title": "Thiết kế lược đồ CSDL",
        "projectId": "uuid",
        "projectName": "Xây dựng PMS",
        "dueDate": "2026-10-20",
        "priority": "HIGH"
      }
    ]
  }
}
```

`dueSoon` gồm việc chưa `DONE`, hạn trong 7 ngày tính từ ngày hiện tại của máy chủ, gán cho người gọi.

### GET `/api/notifications?unreadOnly=true`

### POST `/api/notifications/{id}/read`

Đánh dấu một thông báo đã đọc. Phản hồi `204`.

### POST `/api/notifications/read-all`

Đánh dấu mọi thông báo của người gọi là đã đọc. Phản hồi `204`.

## 9. Mã trạng thái dùng chung

| Mã | Khi dùng |
| --- | --- |
| 200 | Đọc hoặc cập nhật thành công có thân bài |
| 201 | Tạo mới |
| 204 | Thành công không có thân bài |
| 401 | Chưa đăng nhập hoặc token không hợp lệ |
| 403 | Sai quyền |
| 404 | Không thấy tài nguyên trong phạm vi được xem |
| 409 | Trùng email, trùng thành viên, khóa admin cuối cùng |
| 422 | Lỗi kiểm tra dữ liệu hoặc chuyển trạng thái không hợp lệ |
| 500 | Lỗi máy chủ |

Thành viên không thuộc dự án khi gọi chi tiết dự án nhận 404, để không tiết lộ sự tồn tại của dự án.
