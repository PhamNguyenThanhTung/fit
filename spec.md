# PRODUCT SPECIFICATION: INITIALIZE FITNESS APP MICROSERVICES

**Author:** Phạm Nguyễn Thanh Tùng
**Role of AI:** Expert DevOps and Backend Boilerplate Generator.
**Constraint:** Do NOT write any logic for Deep Learning, Computer Vision, or NLP. Only set up the folder architecture, Git, and the Docker environment for the base backend.

---

## 0. AGENT SAFETY RULES (đọc và tuân thủ trước khi làm bất kỳ Task nào)

1. **Fail-Safe (ngắt mạch khẩn cấp):** Nếu một lệnh/đoạn code bị lỗi và 3 lần sửa liên tiếp đều thất bại, BẮT BUỘC DỪNG LẠI ngay. Tuyệt đối không tự ý thử thêm cách giải quyết đoán mò. In ra thông báo lỗi cuối cùng, phân tích nguyên nhân nghi ngờ, và chờ chỉ thị tiếp theo từ tôi.
2. **Step-by-step Checkpoint:** Thực thi lần lượt từng Task theo đúng thứ tự bên dưới. Sau khi hoàn thành mỗi Task, DỪNG LẠI, in báo cáo ngắn gọn (đã làm gì, file nào được tạo/sửa, kết quả kỳ vọng), và KHÔNG được tự chuyển sang Task tiếp theo cho đến khi tôi gõ "Tiếp tục" hoặc "Next".
3. **Human-in-the-loop:** Không tự động chạy các lệnh can thiệp hệ thống (cài package, xóa file, chạy `docker-compose up`, khởi động server...) mà không hỏi xác nhận trước, trừ khi tôi đã bật chế độ auto-approve rõ ràng.
4. **No silent guessing:** Nếu thiếu thông tin để hoàn thành 1 Task (ví dụ không chắc version image nào ổn định), hãy hỏi tôi thay vì tự chọn đại và chạy tiếp.

---

## 1. Mục tiêu (Objective)

Khởi tạo cấu trúc Monorepo cho dự án Fitness App. Thiết lập môi trường Docker chạy sẵn mã nguồn mở `wger` để làm REST API Backend cơ bản.

**Nguồn tham chiếu chính thức:** https://github.com/wger-project/wger — trước khi viết bất kỳ file cấu hình nào (docker-compose.yml, .env), hãy tham khảo README và thư mục cấu hình Docker của repo này (ví dụ `docker/` hoặc `extras/docker/` nếu có) để lấy đúng biến môi trường, tên image, và service khuyến nghị. Không tự đoán cấu hình nếu chưa xác nhận được từ nguồn chính thức.

## 2. Cấu trúc thư mục (Folder Architecture)

```
├── backend/            # Chứa source code wger (sẽ pull qua Docker)
├── cv-service/         # (Trống) Sẽ tự code Pytorch/YOLO sau
├── chatbot-service/    # (Trống) Sẽ tự code Qwen/Transformer sau
├── .gitignore          # Cấu hình chuẩn cho Python, Node, và Docker
├── docker-compose.yml  # File chạy wger backend
├── .env                # Biến môi trường local dev
└── README.md           # Giới thiệu dự án và hướng dẫn chạy môi trường
```

## 3. Nhiệm vụ của AI (Tasks)

### Task 1: Git & Architecture
- Khởi tạo `git init`.
- Tạo các thư mục `backend`, `cv-service`, `chatbot-service`.
- Tạo file `.gitignore` bỏ qua: `venv`, `.env`, `__pycache__`, `node_modules`, model weights (`*.pt`, `*.onnx`), và các file docker volume local nếu có.
- Viết file `README.md` chuyên nghiệp, giải thích kiến trúc 3 thành phần, kèm hướng dẫn chạy nhanh (quick start).

### Task 2: Docker & wger Setup
- Trước tiên, kiểm tra README/docs Docker trong repo chính thức (https://github.com/wger-project/wger) để xác nhận cấu trúc docker-compose khuyến nghị hiện tại — không tự bịa cấu hình.
- Tạo `docker-compose.yml` để kéo wger về. **Dùng tag version cụ thể, ổn định gần nhất** (kiểm tra trên Docker Hub trước, không dùng `latest`) để tránh breaking change bất ngờ.
- Cấu hình các service: `web`, `db` (PostgreSQL), `cache` (Redis), `celery`.
- **Named volume cho `db`** để data không mất khi chạy `docker-compose down`.
- **`depends_on` với `condition: service_healthy`** (hoặc cơ chế wait-for-db tương đương) cho service `web`, tránh race condition lúc `db` chưa sẵn sàng nhận connection.
- Expose port `8000` ra localhost.
- Tạo file `.env` chứa biến môi trường cơ bản (SECRET_KEY giả, cấu hình DB) để chạy được ngay. Ghi chú rõ trong README: **file `.env` này chỉ dùng cho local dev, phải đổi SECRET_KEY thật trước khi deploy** (liên quan Giai đoạn 5 trong roadmap).

### Task 3: Verification (Kịch bản kiểm thử)
- In ra lệnh khởi động: `docker-compose up -d`.
- Cung cấp lệnh `curl` gọi thử: `GET http://localhost:8000/api/v2/exercise/`.
- **Lưu ý quan trọng:** exercise database mặc định của wger có thể KHÔNG có sẵn dữ liệu (rỗng) cho đến khi seed/import. Nếu response rỗng, hướng dẫn tôi cách import exercise database mặc định của wger — không tự bịa ví dụ JSON kỳ vọng cho các bài tập cụ thể (VD: Muscle-up, Handstand) nếu chưa xác nhận được data đó tồn tại sẵn trong image.

## 4. Nguyên tắc thực thi
- Thực hiện tuần tự từng Task, tuân thủ đầy đủ AGENT SAFETY RULES ở mục 0.
- Sau mỗi Task: báo cáo ngắn gọn, dừng chờ xác nhận.
- Không giải thích lan man, chỉ sinh code và tạo file, trừ khi được hỏi.