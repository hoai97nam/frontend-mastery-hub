# 🔑 Senior Keywords — Pinia Store Architecture & State Management

> Từ khóa về Pinia store trong Vue 3.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Store Architecture | `Setup Store vs Option Store` | Phân biệt viết Store theo phong cách Composition API (`setup()`) linh hoạt vs Option API truyền thống | `Pinia Setup Store vs Option Store` |
| Developer Experience | `HMR State Preservation` | Khả năng cập nhật code trong Store mà vẫn giữ nguyên giá trị State hiện tại trong quá trình phát triển (Hot Module Replacement) | `Pinia HMR state preservation` |
| Store Augmentation | `Pinia Plugins & $onAction` | Lắng nghe và can thiệp vào trước/sau khi Action thực thi thông qua `$onAction` để làm logger hay Undo/Redo | `Pinia plugins onAction middleware` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Store Subscriptions**: `$subscribe` tự động theo dõi thay đổi state và flush ghi xuống LocalStorage.
- **SSR State Hydration**: Tự động serialize state từ Server và hydrate tương thích ở Client mà không gây lệch data.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Resetting State Strategy**: Cơ chế reset state nguyên tử cho Setup Store bằng cách đóng gói hàm trả về state ban đầu.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Pinia vs Vuex Differences**: Tại sao Pinia thay thế Vuex: Loại bỏ Mutation, hỗ trợ TypeScript hoàn hảo, không còn lồng module phức tạp.

