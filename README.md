# Fitness App — Monorepo

Hệ thống Fitness App gồm 3 thành phần độc lập, phát triển theo kiến trúc microservices.
Repo này là monorepo chứa toàn bộ source code và môi trường dev bằng Docker.

**Author:** Phạm Nguyễn Thanh Tùng

---

## 1. Kiến trúc

```
fit/
├── backend/            # REST API backend — chạy mã nguồn mở wger qua Docker
├── cv-service/         # Computer Vision service (PyTorch / YOLO) — chưa implement
├── chatbot-service/    # Chatbot service (Qwen / Transformer) — chưa implement
├── docker-compose.yml  # Orchestration cho backend stack
├── .env.example        # Template biến môi trường (commit)
├── .env                # Biến môi trường local dev (KHÔNG commit)
├── .wger-prod.env.ref  # Bản prod.env gốc của wger, để tra cứu option
└── README.md
```

### 1.1. Ba thành phần

| Service | Vai trò | Tech stack | Trạng thái |
|---|---|---|---|
| `backend` | REST API lõi: user, workout, exercise DB, nutrition, log tập luyện. Là nguồn dữ liệu trung tâm cho 2 service còn lại. | wger (Django + DRF), PostgreSQL, Redis, Celery | Chạy qua Docker |
| `cv-service` | Nhận video/frame từ client, đếm rep và đánh giá form động tác. | Python, PyTorch, YOLO | Trống — sẽ code sau |
| `chatbot-service` | Trợ lý hội thoại: tư vấn giáo án, giải đáp dinh dưỡng dựa trên dữ liệu từ `backend`. | Python, Transformer (Qwen) | Trống — sẽ code sau |

Luồng dự kiến: client gọi `backend` cho dữ liệu nghiệp vụ; `cv-service` và `chatbot-service`
là các service phụ trợ, tiêu thụ REST API của `backend` qua network nội bộ của Docker.

Ở giai đoạn này chỉ dựng môi trường cho `backend`. Hai service AI còn lại chỉ có thư mục
rỗng, chưa có logic Deep Learning / Computer Vision / NLP.

---

## 2. Quick start

Yêu cầu: Docker Desktop (hoặc Docker Engine + Compose v2), port `8000` còn trống.

```bash
# 1. Clone repo
git clone <repo-url> fit && cd fit

# 2. Tạo file .env cho local dev (xem mục 3)
cp .env.example .env

# 3. Khởi động backend stack
docker compose up -d

# 4. Kiểm tra API
curl http://localhost:8000/api/v2/exercise/
```

Backend chạy tại http://localhost:8000, REST API base path `/api/v2/`.

> Lần khởi động đầu tiên mất vài phút: container phải pull image, chạy migration và
> collectstatic. Theo dõi bằng `docker compose logs -f`.

Dừng stack:

```bash
docker compose down          # giữ lại data trong named volume
docker compose down -v       # XÓA luôn volume, mất toàn bộ data DB
```

---

## 3. Biến môi trường (`.env`)

File `.env` ở thư mục gốc chứa cấu hình cho local dev: `SECRET_KEY`, thông tin kết nối
PostgreSQL, Redis, Celery và các tuỳ chọn của wger. Tạo bằng `cp .env.example .env`.

Ba file liên quan:

| File | Vai trò | Commit? |
|---|---|---|
| `.env.example` | Template có chú thích đầy đủ, giá trị `SECRET_KEY` là placeholder | Có |
| `.env` | File thật docker compose đọc | **Không** |
| `.wger-prod.env.ref` | Bản `config/prod.env` gốc của wger, để tra các option chưa dùng | Có |

Các biến quan trọng:

| Biến | Ý nghĩa |
|---|---|
| `SECRET_KEY` | Khoá ký session/CSRF của Django. Để trống → sinh mới mỗi lần restart, session bị vô hiệu hoá |
| `SITE_URL` | URL gốc, ảnh hưởng link trong email và API response |
| `POSTGRES_USER` / `_PASSWORD` / `_DB` | Container `db` đọc trực tiếp để khởi tạo DB lần đầu |
| `DJANGO_CACHE_LOCATION` | Redis DB 1 cho cache Django |
| `CELERY_BROKER` / `_BACKEND` | Redis DB 2 cho queue Celery |
| `WGER_INSTANCE` | Instance nguồn để đồng bộ exercise (mục 4.3) |
| `NUMBER_OF_PROXIES` | Đặt `0` vì stack này không có nginx (prod.env gốc để `1`) |

> **Cảnh báo bảo mật:** `.env` trong repo này chỉ dùng cho **local development**.
> `SECRET_KEY` là giá trị dev dùng chung và mật khẩu DB là mật khẩu mặc định (`wger`).
> Trước khi deploy (Giai đoạn 5 — Deployment trong roadmap) **bắt buộc** phải:
> - sinh `SECRET_KEY` mới, ngẫu nhiên, không tái sử dụng giá trị trong repo
>   (`python -c "import secrets; print(secrets.token_urlsafe(50))"`);
> - đổi mật khẩu PostgreSQL;
> - giữ `DJANGO_DEBUG=False`;
> - đặt `SITE_URL` / `CSRF_TRUSTED_ORIGINS` theo domain thật;
> - quản lý secret qua secret manager hoặc biến môi trường của CI/CD, không commit vào git.
>
> `.env` đã được đưa vào `.gitignore`. Không bao giờ commit file này.

---

## 4. Kiểm thử (Verification)

### 4.1. Khởi động stack

```bash
docker compose up -d
docker compose ps          # tất cả service phải ở state running / healthy
docker compose logs -f web # theo dõi migration + collectstatic lần đầu
```

Lần đầu chạy mất vài phút (pull image, migrate, collectstatic). Service `web` có
`start_period: 300s` trong healthcheck nên đừng lo nếu nó báo `starting` một lúc.

### 4.2. Gọi thử REST API

```bash
curl http://localhost:8000/api/v2/exercise/
```

Response mong đợi là JSON phân trang của DRF, dạng:

```json
{"count": 0, "next": null, "previous": null, "results": []}
```

> `count` là **0** sau khi cài mới. Đây là đúng, không phải lỗi: image wger không
> đóng gói sẵn exercise database. Xem mục 4.3 để nạp dữ liệu.

Kiểm tra thêm (không cần auth):

```bash
curl http://localhost:8000/api/v2/exerciseinfo/       # exercise kèm ảnh, cơ, thiết bị
curl http://localhost:8000/api/v2/muscle/             # danh sách nhóm cơ (có sẵn từ migration)
curl http://localhost:8000/api/v2/daytype/            # smoke test endpoint tĩnh
```

Các endpoint gắn với user (`/api/v2/workout/`, `/api/v2/nutritionplan/`...) trả
`401`/`403` nếu chưa auth — đó là hành vi đúng, không phải lỗi cấu hình.

### 4.3. Nạp exercise database

wger đồng bộ exercise từ instance công khai cấu hình ở `WGER_INSTANCE`
(mặc định `https://wger.de`). Cần internet.

```bash
# 1. Đồng bộ exercise (bắt buộc, nhanh)
docker compose exec web python3 manage.py sync-exercises

# 2. Tải ảnh minh hoạ (tuỳ chọn, nặng hơn)
docker compose exec web python3 manage.py download-exercise-images

# 3. Tải video minh hoạ (tuỳ chọn, nặng nhất)
docker compose exec web python3 manage.py download-exercise-videos

# 4. Làm mới cache API để response phản ánh data mới
docker compose exec web python3 manage.py warmup-exercise-api-cache --force
```

Xong bước 1, gọi lại `/api/v2/exercise/` sẽ thấy `count` > 0.

Trong `.env` đã bật `SYNC_EXERCISES_CELERY=True`, nên celery cũng tự đồng bộ định kỳ
(1 lần/tuần, ở thời điểm random chọn khi server start). Chạy tay như trên để có data ngay.

### 4.4. Tạo superuser để vào trang admin

```bash
docker compose exec web python3 manage.py createsuperuser
```

Sau đó đăng nhập tại http://localhost:8000/en/user/login (admin: `/django-admin/`).

### 4.5. Dọn dẹp

```bash
docker compose down     # giữ data
docker compose down -v  # xoá volume, mất toàn bộ data DB
```

---

## 5. Roadmap

| Giai đoạn | Nội dung | Trạng thái |
|---|---|---|
| 1 | Khởi tạo monorepo, Git, kiến trúc thư mục | Xong |
| 2 | Dựng backend wger bằng Docker (web, db, cache, celery) | Xong — chờ chạy `docker compose up -d` để nghiệm thu |
| 3 | `cv-service`: đếm rep + đánh giá form động tác | Chưa bắt đầu |
| 4 | `chatbot-service`: trợ lý hội thoại | Chưa bắt đầu |
| 5 | Deployment: hardening secret, HTTPS, CI/CD | Chưa bắt đầu |
