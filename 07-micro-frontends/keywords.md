# 🔑 Senior Keywords — Micro Frontends Architecture & Federation

> Từ khóa chuyên sâu về kiến trúc Micro Frontends.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Integration Model | `Module Federation (Host & Remote)` | Giải pháp chia sẻ code runtime giữa các ứng dụng độc lập bằng Webpack/Vite mà không cần build-time npm package | `Webpack Module Federation Host Remote Shared Scope` |
| Scope Sharing | `Shared Scope & Singleton Dependency` | Cơ chế đàm phán phiên bản thư viện chung (React, Vue) giữa các MFE để chỉ load 1 bản duy nhất trên browser | `Module Federation Shared Scope Singleton version resolution` |
| CSS Isolation | `Shadow DOM Encapsulation` | Cô lập hoàn toàn CSS của Micro Frontend bằng Shadow Root chặn mọi style toàn cục lọt vào hoặc thoát ra | `Shadow DOM CSS encapsulation Micro Frontends` |
| Cross-MFE Router | `Global Route Synchronization` | Đồng bộ trạng thái URL giữa Shell App và các Micro Apps con mà không gây ra lặp vô tận (Infinite Loop navigation) | `Micro Frontends Routing Synchronization Single-SPA` |
| Cross-MFE Communication | `Custom Event Bus & Shared Store` | Giao tiếp giữa các MFE bằng `window.dispatchEvent(new CustomEvent())` hoặc RxJS Shared Event Bus | `Micro Frontends Communication CustomEvent EventBus` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Web Components Custom Elements**: Đóng gói MFE thành một thẻ HTML tùy chỉnh (`<my-mfe-app>`) chạy được ở bất kỳ framework nào.
- **Remote Entry Manifest**: File manifest `remoteEntry.js` chứa danh sách các module được export và phiên bản dependencies.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Asset Preloading for Remotes**: Preload file `remoteEntry.js` của các MFE quan trọng để khi chuyển tab ứng dụng hiển thị tức thì.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Micro Frontends Decision Matrix**: Khi nào NÊN dùng MFE (nhiều team độc lập, tech stack hỗn hợp) và khi nào KHÔNG NÊN (team nhỏ, ứng dụng đơn giản gây ra chi phí quản lý phức tạp).

