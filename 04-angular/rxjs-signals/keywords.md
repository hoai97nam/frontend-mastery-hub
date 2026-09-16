# 🔑 Senior Keywords — RxJS Reactive Streams & Angular Signals

> Từ khóa về lập trình phản ứng RxJS và Angular Signals.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Reactive Streams | `Cold vs Hot Observables` | Phân biệt Observable thụ động tạo producer mới mỗi subscribe (Cold) vs Observable chủ động chia sẻ stream (Hot - Subject) | `RxJS Cold vs Hot Observables Subject` |
| Higher-Order Mapping | `switchMap vs concatMap vs mergeMap` | Các toán tử flatten stream: `switchMap` (hủy request cũ), `concatMap` (thực thi tuần tự), `mergeMap` (thực thi song song), `exhaustMap` (bỏ qua request mới) | `RxJS higher order mapping operators switchMap mergeMap concatMap exhaustMap` |
| Reactivity Primitive | `Angular Signals Graph` | Cấu trúc đồ thị phản ứng tự động theo dõi phụ thuộc (Producer -> Consumer) không cần unsubcribe thủ công | `Angular Signals graph fine grained reactivity` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **RxJS Subscription Memory Leak**: Unsubscribe không đủ làm rò rỉ bộ nhớ khi Stream giữ tham chiếu đến component instance.
- **Signal Effect Teardown**: Cơ chế cleanup tự động gọi trước khi Effect chạy lại hoặc khi context bị destroy.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **`takeUntilDestroyed` Operator**: Toán tử mới tự động hủy Subscription theo vòng đời của component/service trong Angular 16+.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **AsyncPipe vs Manual Subscribe**: Lý do luôn ưu tiên dùng `async` pipe trong template để Angular tự động subscribe/unsubscribe.

