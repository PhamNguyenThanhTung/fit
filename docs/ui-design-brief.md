# DESIGN BRIEF: FITNESS APP UI KIT — DARK PERFORMANCE THEME

**Vai trò của AI:** Product/UI designer + frontend developer.
**Output mong muốn:** Bộ màn hình UI hoàn chỉnh (React + Tailwind), giao diện mobile-first,
tái hiện đủ các luồng chính của một fitness app, phong cách tối màu, hướng đến nam giới
nhưng không loại trừ đối tượng khác.

---

## 1. Định hướng thiết kế (KHÔNG được tự ý đổi)

**Tinh thần chung:** "Performance gear" — cảm giác giống đồng hồ thể thao / thiết bị tập
luyện cao cấp, chính xác, có kỷ luật. Tránh mọi cliché "nam tính" kiểu đỏ-đen gắt, tránh
neon chói trên nền đen tuyền.

### Bảng màu (dùng đúng các mã hex sau, không tự chọn màu khác)

| Vai trò | Hex | Ghi chú |
|---|---|---|
| Nền chính | `#14171C` | Than chì lạnh, không phải đen thuần |
| Nền card/surface | `#1D2128` | |
| Nền surface nổi (modal, sheet) | `#252A33` | |
| Accent chính | `#E8A33D` | Vàng đồng — dùng cho CTA, số liệu nổi bật, tiến độ |
| Accent phụ | `#4A7FA5` | Xanh thép — dùng cho biểu đồ, icon dữ liệu |
| Success | `#6B9B6B` | Xanh rêu nhạt |
| Warning/Alert | `#C9704A` | Cam đất, dùng cảnh báo form sai (liên quan module AI Check Form sau này) |
| Chữ chính | `#E8E6E1` | Trắng ngà |
| Chữ phụ | `#8A8F98` | Xám trung tính |
| Border/divider | `#2C313B` | |

### Typography
- **Display/số liệu lớn** (cân nặng, %, reps, giờ): font condensed/bold, cảm giác kỹ thuật —
  ví dụ `Barlow Condensed` hoặc `Oswald`, weight 600–700.
- **Body/UI text**: font geometric sans dễ đọc — `Inter` hoặc `Manrope`, weight 400–500.
- **Số liệu dashboard là điểm nhấn thị giác chính** — luôn lớn hơn hẳn text xung quanh,
  dùng accent màu vàng đồng cho con số quan trọng nhất trên mỗi màn hình.

### Layout & phong cách hình khối
- Card bo góc vừa phải (`rounded-xl`, ~12–16px), không bo tròn quá mức (tránh cảm giác "app trẻ em").
- Biểu đồ dùng dạng đường/thanh mảnh, không dùng gradient rực rỡ — line chart mảnh màu
  xanh thép, fill area rất nhạt (~10% opacity).
- Icon: outline mảnh (1.5px stroke), không dùng icon filled màu mè.
- Motion: chuyển màn hình mượt (200-300ms ease), progress ring/bar có animation khi load,
  KHÔNG lạm dụng hiệu ứng — giữ cảm giác "chính xác" chứ không "vui nhộn".

### Signature element (yếu tố nhận diện riêng, dùng xuyên suốt)
Một **vòng tiến độ dạng radial (radial performance ring)** lấy cảm hứng từ mặt đồng hồ thể
thao — dùng lặp lại ở màn hình dashboard (calo, %mục tiêu), màn hình workout (thời gian còn
lại), màn hình profile (streak). Đây là điểm nhấn hình ảnh chính của cả app, các màn hình
khác giữ tối giản để vòng tròn này nổi bật.

---

## 2. Danh sách màn hình cần dựng (tham khảo cấu trúc ảnh mockup gốc, đổi theme tối)

1. **Onboarding/Splash** — logo, tagline ngắn, nút "Get Started".
2. **Sign up / Sign in** — form tối giản, input viền mảnh, nút CTA màu vàng đồng.
3. **Setup profile** — chọn mục tiêu (giảm cân/tăng cơ/duy trì), thông tin cơ bản.
4. **Dashboard chính** — radial ring calo, số liệu tuần (cân nặng, reps, thời gian tập),
   biểu đồ xu hướng nhỏ.
5. **Workout log / Lịch sử tập** — danh sách buổi tập, mỗi item có icon bài tập + số liệu.
6. **Progress / Biểu đồ tiến độ** — line chart cân nặng theo thời gian, bar chart theo tuần.
7. **Calendar** — lịch tháng đánh dấu ngày đã tập (chấm nhỏ màu vàng đồng).
8. **Map/Route tracking** (nếu có chạy bộ) — nền tối, đường route màu vàng đồng nổi trên nền map tối.
9. **Chi tiết bài tập đang thực hiện** — chừa sẵn khu vực hiển thị **skeleton overlay** (dùng
   cho module AI Check Form sau này — để trống/placeholder, không cần logic thật ở bước này).
10. **Chat với AI Coach** — bong bóng chat tối giản, chừa sẵn UI cho chatbot-service sau này.

> Lưu ý: Task này CHỈ dựng UI/UX tĩnh (component + mock data), KHÔNG kết nối logic AI thật.
> Các khu vực dành cho `cv-service` (skeleton overlay) và `chatbot-service` (chat logic) chỉ
> cần placeholder UI, sẽ nối API thật ở giai đoạn sau của dự án.

---

## 3. Yêu cầu kỹ thuật

- React functional components, Tailwind CSS (dùng CSS variable cho bảng màu ở mục 1 để dễ
  đổi theme sau này).
- Mobile-first, responsive tối thiểu xuống 375px width.
- Component tái sử dụng: `<RadialProgress />`, `<StatCard />`, `<WorkoutListItem />`,
  `<BottomNav />` — tách riêng, không viết lặp code giữa các màn hình.
- Giữ accessibility cơ bản: contrast đủ đọc được trên nền tối, focus state rõ ràng cho input/button.
- Không cần kết nối API thật — dùng mock data cứng trong component.

## 4. Quy trình thực hiện
- Trước khi code, tóm tắt lại token system (màu, font, layout) theo đúng bảng ở mục 1 để
  xác nhận đã hiểu đúng brief.
- Dựng từng màn hình một, sau mỗi màn hình dừng lại cho tôi xem trước khi làm màn tiếp theo.
- Không tự ý đổi bảng màu, font, hoặc thêm hiệu ứng ngoài những gì đã mô tả ở mục 1.
