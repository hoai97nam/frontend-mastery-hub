# 🔑 Senior Keywords — Nuxt 3 Framework & Vue Ecosystem

> Từ khóa về Nuxt 3, Router và Server Engine.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Server Engine | `Nitro Engine` | Engine máy chủ đa nền tảng của Nuxt 3 hỗ trợ nén bundle, Zero-config triển khai lên Serverless/Edge Workers | `Nuxt 3 Nitro engine serverless edge deployment` |
| Universal Rendering | `Hybrid Rendering & SWR` | Cấu hình chiến lược render theo từng route (SSR, SSG, SPA, SWR) trong cùng 1 ứng dụng Nuxt | `Nuxt 3 Hybrid Rendering Route Rules SWR` |
| Built-in Component | `Teleport Portal` | Render phần tử HTML đến một vị trí DOM bất kỳ ngoài cây component cha (thích hợp làm Modal, Notification) | `Vue 3 Teleport DOM portal` |
| Component Caching | `KeepAlive Component` | Lưu giữ instance và trạng thái DOM của component trong bộ nhớ khi chuyển tab để tránh render lại | `Vue 3 KeepAlive component state caching` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Auto-imports AST Transform**: Nuxt tự động quét và import composables, components ở cấp độ biên dịch AST.
- **Vue Router Navigation Guards**: Luồng kiểm soát điều hướng: `beforeEach` -> `beforeEnter` -> `beforeRouteEnter` -> `afterEach`.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **`useAsyncData` Data Deduplication**: Tránh gửi request trùng lặp ở cả Server và Client trong quá trình Hydration.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Nuxt Hydration Mismatch**: Khắc phục lỗi render HTML không trùng khớp giữa Server và Client khi truy cập thông tin browser.

