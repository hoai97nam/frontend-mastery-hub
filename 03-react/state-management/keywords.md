# 🔑 Senior Keywords — React State Management Paradigms

> Từ khóa về các trường phái quản lý state trong hệ sinh thái React.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| State Paradigm | `Atomic State (Jotai / Recoil)` | Mô hình quản lý state theo các nguyên tử độc lập (Atoms) tự động theo dõi phụ thuộc qua đồ thị | `React Atomic State Jotai Recoil dependency graph` |
| State Paradigm | `Selector-driven Store (Zustand)` | Thư viện quản lý state đơn giản dựa trên Flux pattern, dùng selector để chọn đúng slice state tránh re-render | `Zustand selector re-render optimization Flux` |
| Immutability | `Structural Sharing & Immer` | Kỹ thuật tạo đối tượng mới chỉ thay đổi nhánh dữ liệu bị sửa đổi (Copy-on-Write) giúp so sánh tham chiếu O(1) | `Immer Copy on Write structural sharing immutability` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Proxy-based Change Tracking**: Sử dụng ES6 Proxy để tự động lắng nghe thuộc tính nào được đọc trong component (như Valtio/MobX).
- **Normalized State Structure**: Phẳng hóa cấu trúc state phức tạp thành dạng hashmap (`byId`, `allIds`) giống database.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Transient State Updates**: Cập nhật DOM trực tiếp qua ref mà không kích hoạt chu kỳ re-render của React cho các hiệu ứng tần suất cao (Slider, Mouse move).

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Redux vs Zustand Decision Matrix**: So sánh Redux Toolkit (dự án enterprise khổng lồ, middleware phức tạp) vs Zustand (gọn nhẹ, không boilerplate).

