# QLSV - Student Management System

Hệ thống quản lý sinh viên được xây dựng bằng Laravel, cho phép quản lý thông tin sinh viên, điểm số, điểm danh và các hoạt động học thuật.

## 📋 Yêu cầu hệ thống

- **Hệ điều hành**: Windows 10/11, macOS, hoặc Linux
- **Docker Desktop**: Phiên bản mới nhất (bao gồm Docker Compose)
- **Git**: Để clone repository
- **Trình duyệt**: Chrome, Firefox, Edge, hoặc Safari

## 🐳 Cài đặt Docker

Nếu chưa có Docker Desktop:

1. Tải về từ [docker.com](https://www.docker.com/products/docker-desktop)
2. Cài đặt và khởi động Docker Desktop
3. Đảm bảo Docker Desktop đang chạy (icon whale ở taskbar)

## 📥 Clone repository

```bash
git clone <repository-url>
cd QLSV
```

## 🔧 Chuẩn bị môi trường

Dự án đã bao gồm các file cấu hình Docker cần thiết:
- `docker-compose.yml` - Cấu hình các dịch vụ
- `Dockerfile` - Build image cho Laravel app
- `.env` - Cấu hình môi trường
- `.dockerignore` - Loại trừ file khỏi build

Nếu thiếu bất kỳ file nào, tham khảo [plan.md](plan.md) để tạo lại.
(Prompt để agent thực thi `plan.md`)

## ⚡ Khởi động nhanh (5 phút)

### Yêu cầu
- Docker Desktop đã được cài đặt và đang chạy
- Terminal (PowerShell / Command Prompt)

### Bước 1: Khởi động Docker containers
```bash
cd QLSV
docker compose up -d --build
```
Đợi 2-3 phút để build image (lần đầu có thể lâu hơn).

### Bước 2: Kiểm tra containers đang chạy
```bash
docker compose ps
```
Tất cả containers phải ở trạng thái **Up**. Nếu có lỗi, xem phần Troubleshooting.

### Bước 3: Khởi tạo database (chỉ lần đầu)
```bash
docker compose exec app php artisan migrate
docker compose exec app php artisan db:seed
```
Lệnh này sẽ tạo bảng database và nạp dữ liệu mẫu.

### Bước 4: Truy cập ứng dụng
Mở trình duyệt và truy cập:
- **Laravel App**: http://localhost:8001
- **Frontend Dev Server**: http://localhost:5174
- **MailHog (Email Test UI)**: http://localhost:8025

✅ **Hoàn tất!** Ứng dụng đã sẵn sàng sử dụng.

## 📋 Thông tin đăng nhập mặc định

### Admin Account
- **Email**: admin@example.com
- **Password**: password

### Database Connection (từ bên ngoài)
- Host: localhost
- Port: 3307
- Database: studentmanagement
- Username: root
- Password: root

## 🛠️ Lệnh phổ biến

### Xem logs
```bash
docker compose logs --tail=20 app          # Logs từ Laravel app
docker compose logs --tail=20 vite         # Logs từ Vite dev server
docker compose logs mysql                   # Logs từ MySQL
```

### Chạy Artisan commands
```bash
docker compose exec app php artisan migrate:fresh --seed    # Reset & seed database
docker compose exec app php artisan tinker                   # Laravel REPL
docker compose exec app php artisan key:generate             # Tạo app key mới
```

### Dừng & khởi động lại
```bash
docker compose down         # Dừng tất cả containers
docker compose up -d        # Khởi động lại
docker compose restart app  # Khởi động lại riêng app container
```

## ⚠️ Troubleshooting nhanh

### Containers không khởi động
```bash
docker system prune -f
docker compose up -d --build --no-cache
```

### Lỗi port đã sử dụng
Kiểm tra process sử dụng port:
```bash
netstat -ano | findstr :8001
```

### Database connection error
- Chắc chắn MySQL container đang chạy: `docker compose ps`
- Xem logs MySQL: `docker compose logs mysql`

### Vite không hot reload
- Kiểm tra vite container: `docker compose ps`
- Xem logs: `docker compose logs vite`

## 📚 Tài liệu bổ sung

- **[plan.md](plan.md)**: Hướng dẫn toàn diện với Docker
- **[thinking.md](thinking.md)**: Quá trình xây dựng setup
- **[result.md](result.md)**: Kết quả Docker hóa

## 🚀 Phát triển

1. **Backend**: Chỉnh sửa code trong `app/`, `routes/`, `config/` - tự động reflect vào container
2. **Frontend**: Chỉnh sửa CSS/JS trong `resources/` - Vite tự động hot reload
3. **Database**: Import/export dữ liệu thông qua MySQL trên port 3307
4. **Email**: Kiểm tra test email trên http://localhost:8025

## 📞 Hỗ trợ

Nếu gặp vấn đề, kiểm tra:
1. Docker Desktop đang chạy
2. Ports 8001, 5174, 8025, 3307, 6380 không bị chiếm
3. Logs của containers: `docker compose logs`

Chúc mừng! Dự án QLSV đã sẵn sàng. 🎉