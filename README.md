# Campus Hub – Đóng gói và chạy bằng Docker

- **Họ tên:** Nguyễn Thế Kiên
- **Mã sinh viên:** <MÃ SV>
- **Source gốc:** dinhthanhnam/campus-hub-base

## Cấu trúc bổ sung
- `backend/Dockerfile`: multi-stage, `eclipse-temurin:21-jdk` build jar, `eclipse-temurin:21-jre` chạy jar bằng user không phải root
- `backend/.dockerignore`: loại `build/`, `.gradle/`, IDE, log, `.env`
- `compose.yml`: 2 service `mysql` (MySQL 8.0) và `backend`, chung network `campus-net`
- `.env.example`: mẫu biến môi trường (file `.env` thật không push)

## Lệnh chạy
1. `cp .env.example .env` rồi đổi mật khẩu
2. Test: `cd backend && ./gradlew test` (cần JDK 21), hoặc chạy trong container:
   `docker run --rm -v "$PWD/backend":/app -w /app eclipse-temurin:21-jdk ./gradlew test`
3. Build image: `docker build -t campus-hub-backend:1.0 ./backend`
4. Chạy hệ thống: `docker compose up -d --build`
5. Trạng thái: `docker compose ps`
6. Dừng: `docker compose down` (thêm `-v` nếu muốn xóa cả dữ liệu MySQL)

## Kiểm tra
- Backend: http://localhost:8080
- Health: `curl http://localhost:8080/actuator/health` trả về HTTP 200, `"status":"UP"`, `db: UP (MySQL)`
- API: `curl http://localhost:8080/api/categories`

## Kết nối database
Backend nhận cấu hình qua biến môi trường `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` (đã khai báo sẵn trong `application.yml`).
Chuỗi kết nối dùng tên service `mysql` thay cho localhost: `jdbc:mysql://mysql:3306/campus_hub`.
Backend chỉ khởi động khi healthcheck của MySQL báo healthy (`depends_on: condition: service_healthy`). Flyway tự tạo bảng khi khởi động.

## Minh chứng
1. Gradle test: ![gradle test](docs/images/1-gradle-test.png)
2. Docker image: ![docker images](docs/images/2-docker-images.png)
3. Trạng thái service: ![compose ps](docs/images/3-compose-ps.png)
4. Health endpoint: ![health](docs/images/4-health.png)
