# 🔑 Senior Keywords — React 19, RSC & Architecture Interviews

> Từ khóa dành riêng cho phỏng vấn React Senior & React 19.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| React 19 | `React Server Components (RSC)` | Thành phần component chỉ chạy trên Server, không gửi JavaScript bundle về Client, hỗ trợ truy cập DB trực tiếp | `React 19 Server Components RSC bundle size` |
| React 19 | `Server Actions` | Phương thức bất đồng bộ chạy trên Server được gọi trực tiếp từ Form hoặc Client component | `React 19 Server Actions form mutation` |
| React Compiler | `React Compiler (Forget)` | Compiler tự động chèn `useMemo`/`useCallback` ở cấp độ AST, loại bỏ nhu cầu tối ưu thủ công | `React Compiler Forget auto memoization` |
| Data Fetching | `React 19 `use()` API` | API mới cho phép unwrap Promise hoặc Context linh hoạt bên trong vòng lặp hoặc điều kiện | `React 19 use API promise unwrapping` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **RSC Wire Format**: Định dạng luồng dữ liệu JSON dạng tree gửi từ Server về Client để Hydrate UI.
- **Action State & Form Status**: Các hook mới (`useActionState`, `useFormStatus`, `useOptimistic`) quản lý trạng thái submit form và optimistic update.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Zero-Bundle-Size Components**: Sử dụng RSC cho các thư viện nặng (Markdown parser, Date formatter) mà không tốn KB bundle nào ở Client.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Client Component Boundary ('use client')**: Giải thích ranh giới giữa Server và Client Component và bẫy import Server Component vào Client Component.

