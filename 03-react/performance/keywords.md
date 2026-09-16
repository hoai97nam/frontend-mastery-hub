# 🔑 Senior Keywords — React Rendering Optimization & Profiling

> Từ khóa tối ưu hóa hiệu năng render, code splitting và caching.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Rendering Mechanics | `Reconciliation & Diffing Algorithm` | Thuật toán so sánh O(n) của React dựa trên Component Type và Key Props để quyết định Reuse hay Unmount/Remount | `React Reconciliation Diffing Algorithm key prop` |
| Optimization | `Shallow Equality Comparison` | Cơ chế so sánh nông của `React.memo` và `PureComponent` để bỏ qua re-render nếu props không đổi tham chiếu | `React memo shallow equality comparison` |
| Optimization | `Context Selector Pattern` | Kỹ thuật ngăn chặn re-render hàng loạt do Context API bằng cách bọc selector hoặc thư viện Zustand/use-context-selector | `React Context selector pattern re-render optimization` |
| Code Splitting | `Suspense Boundary & Code Splitting` | Tách bundle ứng dụng theo route/component bằng `React.lazy` và hiển thị UI chờ với `Suspense` | `React lazy Suspense code splitting route based` |
| Data Caching | `Stale-While-Revalidate (SWR)` | Chiến lược caching hiển thị dữ liệu cũ trong cache ngay lập tức đồng thời gửi request ngầm cập nhật dữ liệu mới | `React Query SWR Stale While Revalidate cache invalidation` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Component Unmounting Trigger**: Khi `key` hoặc `type` của component thay đổi, React hủy toàn bộ DOM cũ và tạo lại state từ đầu.
- **Batching Updates (Automatic Batching)**: React 18 gộp nhiều lần gọi `setState` trong cùng microtask/event handler thành 1 lần re-render duy nhất.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Moving State Down**: Đẩy state xuống component nhánh thấp nhất để hạn chế phạm vi ảnh hưởng của re-render.
- **Passing Component as Prop**: Truyền component dưới dạng `children` hoặc prop để tận dụng tính chất lưu giữ tham chiếu không bị re-render.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **React.memo Overuse Trap**: Chi phí so sánh props của `React.memo` đôi khi đắt hơn việc cho phép component re-render nhẹ nhàng.

