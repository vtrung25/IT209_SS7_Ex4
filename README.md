# Bài 4 — Nginx Reverse Proxy cho Spring Boot

## Cấu hình định tuyến

- `/` phục vụ trang tĩnh trong `/var/www/html/`.
- `/api/` chuyển tiếp tới Spring Boot ở `127.0.0.1:8082` và giữ nguyên URI. Vì vậy `/api/health` tới backend với đúng path `/api/health`.
- Các header `Host`, `X-Real-IP`, `X-Forwarded-For` và `X-Forwarded-Proto` được chuyển tiếp để backend nhận diện request gốc.

## Triển khai trên Ubuntu

Từ thư mục chứa `index.html` và `spring-proxy.conf`:

```bash
sudo install -d -o root -g root -m 0755 /var/www/html
sudo install -o root -g root -m 0644 index.html /var/www/html/index.html
sudo install -o root -g root -m 0644 spring-proxy.conf /etc/nginx/sites-available/spring-proxy.conf
```

Kiểm tra cấu hình đang bật trước khi dùng `default_server`:

```bash
ls -l /etc/nginx/sites-enabled/
```

Nếu cấu hình `default` hiện có đang chiếm `default_server` trên cổng 80, hãy kiểm tra nội dung và chỉ vô hiệu hóa symlink đó nếu không còn cần dùng. Sau đó kích hoạt cấu hình mới và kiểm tra trước khi reload:

```bash
sudo ln -sfn /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/spring-proxy.conf
sudo nginx -t
sudo systemctl reload nginx
```

## Kiểm tra

```bash
curl -i http://localhost/
curl -i http://localhost/api/health
```

Trang chủ cần trả `200 OK` và chứa thông tin học viên. `/api/health` cần trả response từ backend; mã 502 thường có nghĩa ứng dụng chưa lắng nghe ở `127.0.0.1:8082` hoặc backend chưa sẵn sàng. Cấu hình này chuyển tiếp nguyên path, nên backend cần có route `/api/health`.
