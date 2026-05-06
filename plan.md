# Kế hoạch và Hướng dẫn chạy dự án QLSV bằng Docker

## Kế hoạch chạy dự án bằng Docker

Dự án có thể chạy bằng Docker Compose, không cần XAMPP. Cần thêm `docker-compose.yml` và `.env` tương ứng để container `app` và `mysql` kết nối đúng.

### 1. Công cụ cần có
- Docker Desktop (Windows) (Đã có)
- Docker Compose (Đã có trong Docker Desktop)
- Composer và npm có thể chạy trong container hoặc trên máy host

### 2. Cấu hình `docker-compose.yml`
Tạo file `docker-compose.yml` trong gốc dự án với nội dung chính:
- service `app`
  - image PHP 8.2 hoặc custom Dockerfile
  - mount mã nguồn `./:/var/www/html`
  - cài ext PHP cần thiết
  - expose port `8001` (để tránh xung đột với dự án reshop trên cổng 8000)
  - phụ thuộc `mysql`
- service `mysql`
  - image `mysql:8.1`
  - ports `3307:3306`
  - environment:
    - `MYSQL_ROOT_PASSWORD=root`
    - `MYSQL_DATABASE=studentmanagement`
    - `MYSQL_USER=root`
    - `MYSQL_PASSWORD=root`
- service `node` nếu cần tách build assets

### 3. Cấu hình `.env` cho Docker
Tạo `.env` từ `.env.example` và sửa:
- `DB_HOST=mysql`
- `DB_PORT=3306`
- `DB_DATABASE=studentmanagement`
- `DB_USERNAME=root`
- `DB_PASSWORD=root`
- `APP_URL=http://localhost`
- `REDIS_PORT=6380`
- `MAIL_PORT=1025`
- Đổi `SESSION_DRIVER=file`
- Đổi `CACHE_STORE=file`

### 4. Các bước khởi tạo trong Docker
1. Chạy:
   - `docker compose up -d`
2. Cài dependencies:
   - `docker compose exec app composer install`
   - `docker compose exec app npm install`
3. Tạo khóa ứng dụng:
   - `docker compose exec app php artisan key:generate`
4. Migrate database:
   - `docker compose exec app php artisan migrate`
5. Nếu cần session database:
   - `docker compose exec app php artisan session:table`
   - `docker compose exec app php artisan migrate`

### 5. Chạy ứng dụng
- `docker compose exec app php artisan serve --host=0.0.0.0 --port=8001`
- Mở `http://localhost:8001`

### 6. Ghi chú
- Docker thay thế MySQL cục bộ, nên không cần MySQL trên Windows nếu container chạy tốt.
- Nếu dùng Laravel Sail, có thể dùng `./vendor/bin/sail up -d` và cấu hình `.env` tương tự.
- Nếu muốn tách frontend, dùng container Node hoặc build assets trên host.
- Cổng MySQL expose ra host là 3307, Redis 6380, Mail SMTP 1025, Laravel 8001, Vite dev 5174 (cần chỉnh `vite.config.js` để set `server.port = 5174`).

---

## Hướng dẫn thực thi

## Tổng quan
Dự án này sử dụng Docker Compose để chạy môi trường phát triển, bao gồm Laravel app, MySQL, Redis, MailHog và Vite dev server. Hướng dẫn này giúp bạn khởi động dự án một cách nhanh chóng mà không cần cài đặt PHP, MySQL, Node.js trên máy host.

## Yêu cầu hệ thống
- **Docker Desktop** (phiên bản mới nhất cho Windows)
- **Docker Compose** (bao gồm trong Docker Desktop)
- **Git** (để clone dự án nếu cần)

## Các file cấu hình cần thiết
Dự án đã bao gồm các file sau:
- `docker-compose.yml`: Cấu hình các dịch vụ (app, mysql, redis, mailhog, vite)
- `Dockerfile`: Build image tùy chỉnh cho Laravel app
- `.env`: Cấu hình môi trường (database, mail, etc.)
- `.dockerignore`: Loại trừ file không cần thiết khỏi build context

## Bước 1: Chuẩn bị
1. Đảm bảo Docker Desktop đang chạy trên máy Windows.
2. Mở PowerShell hoặc Command Prompt.
3. Điều hướng đến thư mục gốc dự án:
   ```powershell
   cd QLSV
   ```

## Bước 2: Khởi động containers
Chạy lệnh sau để build và khởi động tất cả dịch vụ:
```bash
docker compose up -d --build
```

Lệnh này sẽ:
- Build image cho `app` service từ Dockerfile
- Khởi động các containers: `app`, `mysql`, `redis`, `mailhog`, `vite`
- Mount mã nguồn vào container để có thể chỉnh sửa code trực tiếp

**Thời gian đầu tiên:** Có thể mất 5-10 phút để build image và download dependencies.

## Bước 3: Cài đặt dependencies (chỉ lần đầu)
Nếu đây là lần đầu chạy hoặc chưa có thư mục `vendor`/`node_modules`:
```bash
docker compose exec app composer install
docker compose exec app npm install
```

## Bước 4: Cấu hình ứng dụng Laravel
```bash
docker compose exec app php artisan key:generate
docker compose exec app php artisan migrate
```

Nếu cần tạo bảng session (tùy chọn):
```bash
docker compose exec app php artisan session:table
docker compose exec app php artisan migrate
```

## Bước 5: Truy cập ứng dụng
Sau khi tất cả containers đang chạy, bạn có thể truy cập:
- **Laravel App**: http://localhost:8001
- **Vite Dev Server** (hot reload cho frontend): http://localhost:5174
- **MailHog UI** (xem email test): http://localhost:8025

## Lệnh quản lý containers
### Xem trạng thái containers
```bash
docker compose ps
```

### Xem logs của một service
```bash
docker compose logs --tail=20 app
docker compose logs --tail=20 vite
docker compose logs --tail=20 mysql
```

### Dừng tất cả containers
```bash
docker compose down
```

### Khởi động lại sau khi chỉnh sửa code
```bash
docker compose restart app
```

### Rebuild image (khi thay đổi Dockerfile)
```bash
docker compose up -d --build
```

## Cấu hình kết nối database (từ bên ngoài)
Nếu cần kết nối MySQL từ công cụ bên ngoài (như MySQL Workbench):
- **Host**: localhost
- **Port**: 3307
- **Database**: studentmanagement
- **Username**: root
- **Password**: root

## Troubleshooting
### Lỗi "port already in use"
- Đảm bảo không có ứng dụng khác đang sử dụng port 8001, 5174, 8025, 3307, 6380.
- Kiểm tra: `netstat -ano | findstr :8001`

### Lỗi build image
- Xóa cache: `docker system prune -f`
- Rebuild: `docker compose up -d --build --no-cache`

### Lỗi database connection
- Đảm bảo container mysql đã khởi động: `docker compose ps`
- Kiểm tra logs: `docker compose logs mysql`

### Vite không hot reload
- Đảm bảo container vite đang chạy: `docker compose ps`
- Kiểm tra logs: `docker compose logs vite`

## Ghi chú kỹ thuật
- **PHP Version**: 8.2
- **MySQL Version**: 8.1
- **Node.js Version**: 18 (trong container)
- **Ports**:
  - Laravel: 8001
  - Vite: 5174
  - MySQL: 3307 (host) / 3306 (container)
  - Redis: 6380 (host) / 6379 (container)
  - MailHog SMTP: 1025 / UI: 8025

## Phát triển
- Chỉnh sửa code trong thư mục `app/`, `resources/`, etc. sẽ tự động reflect vào container.
- Frontend assets sẽ auto-reload nhờ Vite dev server.
- Database data được persist trong named volume `mysql_data`.
