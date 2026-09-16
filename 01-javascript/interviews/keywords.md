# 🔑 Senior Keywords — V8 Engine Optimization & Advanced Async

> Tổng hợp từ khóa nâng cao dành riêng cho vòng phỏng vấn JavaScript Senior/Architect.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| V8 Internal | `Hidden Classes (Shapes)` | Cơ chế V8 tạo ra cấu trúc lớp ẩn đại diện cho layout của object để tối ưu hóa truy xuất thuộc tính | `V8 Hidden Classes Map Transition` |
| V8 Internal | `Inline Caches (IC)` | Kỹ thuật lưu lại vị trí bộ nhớ của thuộc tính tại call-site (Monomorphic -> Polymorphic -> Megamorphic) | `V8 Inline Caches Monomorphic Megamorphic` |
| V8 Internal | `TurboFan Deoptimization` | Quá trình V8 hạ cấp code từ machine code tối ưu về bytecode khi phát hiện type thay đổi đột ngột (Bailout) | `V8 TurboFan Deoptimization Bailout` |
| Async Pattern | `Promise Concurrency Control` | Điều phối thực thi song song có giới hạn (Limit Concurrency) sử dụng Queue để tránh cạn kiệt tài nguyên mạng | `Promise concurrency limit pool throttle` |
| Profiling Tool | `Heap Snapshot Delta Analysis` | Kỹ thuật so sánh 2 bản chụp bộ nhớ Heap Snapshot để tìm ra chính xác các object bị lọt lướt GC | `Chrome DevTools Memory Heap Snapshot Delta` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Ignition Interpreter & TurboFan JIT**: Kiến trúc 2 tầng của V8: Ignition dịch code nhanh sang Bytecode, TurboFan biên dịch Bytecode nóng sang Machine Code tối ưu.
- **Fast Properties vs Slow Properties**: V8 lưu thuộc tính dạng In-object/Fast (array-backed) hoặc Dictionary/Slow (hashmap-backed) tùy theo cách thao tác object.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Object Monomorphism**: Giữ cho đối tượng có hình dạng (shape) không đổi để V8 áp dụng Inline Cache hiệu quả nhất.
- **Unhandled Rejection Tracking**: Quản lý và log tất cả các Promise Rejection không được catch để tránh crash ứng dụng ở môi trường Node.js/Browser.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Why `delete` Operator Harms Performance**: Giải thích lý do dùng `delete` khiến V8 chuyển object sang Dictionary Mode (Slow Properties) và phá hỏng Hidden Class.
- **Promise.all vs Promise.allSettled Fail-Fast**: Phân tích sự khác biệt về cơ chế dừng khi gặp lỗi đầu tiên của `Promise.all` so với thu thập toàn bộ trạng thái của `allSettled`.

