# 🔑 Senior Keywords — React Component Design Patterns

> Từ khóa về các mô hình thiết kế component chuyên nghiệp cấp Senior.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Design Pattern | `Compound Components` | Mô hình chia nhỏ component phức tạp thành các sub-components cùng chia sẻ state ngầm qua Context API | `React Compound Components Pattern Context API` |
| Design Pattern | `State Reducer Pattern` | Cho phép user bên ngoài can thiệp và kiểm soát luồng biến đổi state của component thông qua reducer tùy chỉnh | `React State Reducer Pattern inversion of control` |
| Design Pattern | `Control Props Pattern` | Cung cấp cơ chế Hybrid State: vừa hỗ trợ Uncontrolled (tự quản lý) vừa cho phép Controlled (bị điều khiển từ bên ngoài) | `React Control Props Pattern uncontrolled controlled component` |
| Design Pattern | `Render Props & Children as Function` | Kỹ thuật đảo ngược quyền kiểm soát render HTML cho component cha bằng cách truyền callback render | `React Render Props Pattern inversion of control` |
| Ref Forwarding | `forwardRef & useImperativeHandle` | Ủy quyền ref cho DOM con hoặc tùy biến imperative API xuất ra cho component cha | `React forwardRef useImperativeHandle imperative API` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Inversion of Control (IoC)**: Nguyên lý giao quyền quyết định render hoặc xử lý state cho nơi tiêu thụ component.
- **HOC Static Method Hoisting**: Sao chép các thuộc tính static khi bọc component bằng Higher-Order Component (`hoist-non-react-statics`).

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Context Splitting**: Tách Context dữ liệu và Context chứa dispatch function để tránh re-render phụ thuộc không cần thiết.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Controlled vs Uncontrolled Form Performance**: So sánh hiệu năng khi render form 100 fields: Controlled re-render mỗi keystroke vs Uncontrolled dùng Ref đọc khi Submit.

