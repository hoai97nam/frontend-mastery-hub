# 🔑 Senior Keywords — Angular Architecture & Ivy Engine

> Từ khóa kiến trúc Angular, Compiler Ivy và Change Detection.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Compiler Engine | `Ivy Incremental DOM` | Ivy biên dịch template thành các câu lệnh JS nguyên tử thao tác Incremental DOM giúp tối ưu bộ nhớ và Tree-shaking | `Angular Ivy Incremental DOM vs Virtual DOM` |
| Change Detection | `ChangeDetectionStrategy.OnPush` | Chiến lược bỏ qua kiểm tra Change Detection cho component nếu tham chiếu `@Input` không thay đổi | `Angular ChangeDetectionStrategy OnPush markForCheck` |
| Zoneless | `Zoneless Angular & Signals` | Kiến trúc mới loại bỏ Zone.js monkey-patching, sử dụng Signals để kích hoạt cập nhật DOM chính xác theo nguyên tử | `Angular Zoneless signals change detection` |
| Architecture | `Standalone Components` | Mô hình ứng dụng Angular không cần `NgModule`, quản lý dependencies trực tiếp trong component decorator | `Angular Standalone Components NgModule migration` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Locality Principle in Ivy**: Mỗi component được biên dịch độc lập chỉ với thông tin của chính nó, tăng tốc đáng kể build time.
- **Zone.js Monkey Patching**: Zone.js ghi đè tất cả các sự kiện bất đồng bộ (`setTimeout`, `Promise`, `addEventListener`) để kích hoạt Change Detection toàn cục.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Detaching Change Detector**: Sử dụng `ChangeDetectorRef.detach()` để tạm ngưng Change Detection cho các bảng dữ liệu hàng ngàn dòng.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **ExpressionChangedAfterItHasBeenCheckedError**: Giải thích nguyên nhân xảy ra lỗi do thay đổi state trong lifecycle hook `ngAfterViewInit` và giải pháp dùng `microtask` hoặc `Signals`.

