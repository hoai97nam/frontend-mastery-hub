# 🔑 Senior Keywords — CSS Architecture, Layout Engines & Rendering Pipeline

> Từ khóa về Rendering Engine của trình duyệt, layout contexts và CSS hiện đại.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Browser Engine | `Critical Rendering Path` | Các bước xử lý HTML/CSS: DOM + CSSOM -> Render Tree -> Layout (Reflow) -> Paint -> Composite | `Browser Critical Rendering Path Layout Paint Composite` |
| Rendering Optimization | `Composite Layers & GPU Acceleration` | Đưa element vào layer riêng bằng `will-change: transform` hoặc `translateZ(0)` để GPU đảm nhận Composite mà không kích hoạt Layout/Paint | `CSS GPU Acceleration Composite Layer transform` |
| Layout Mechanics | `Block Formatting Context (BFC)` | Môi trường render độc lập giúp chứa float elements và chống margin collapse giữa các sibling box | `CSS Block Formatting Context BFC margin collapse` |
| Modern Layout | `CSS Container Queries` | Truy vấn style dựa trên kích thước của container thay vì viewport (`@container (min-width: 400px)`) | `CSS Container Queries vs Media Queries` |
| CSS Architecture | `Cascade Layers (@layer)` | Quản lý độ ưu tiên của CSS theo các tầng định sẵn (base, components, utilities) không phụ thuộc vào CSS Specificity | `CSS Cascade Layers layer specificity` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Stacking Context & `z-index`**: Điều kiện tạo Stacking Context mới (`opacity < 1`, `transform`, `isolation: isolate`) quyết định thứ tự vẽ đè các phần tử.
- **Subgrid Layout**: Cho phép grid item con thừa hưởng các đường grid line của grid container cha.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Reflow Triggering Properties**: Tránh đọc các thuộc tính `offsetHeight`, `scrollTop` liên tục gây ra Synchronous Layout Thrashing.
- **`content-visibility: auto`**: Bỏ qua quá trình render & layout đối với các phần tử ngoài màn hình (Off-screen content) để tăng tốc độ FCP/LCP.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Layout Thrashing in Animation**: Phân tích và khắc phục hiện tượng giật lag khi sửa đổi DOM style rồi đọc lại thuộc tính kích thước trong cùng 1 frame.

