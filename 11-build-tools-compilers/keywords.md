# 🔑 Senior Keywords — Build Tools, JS Compilers & Bundler Architecture

> Từ khóa chuyên sâu về Compilers, AST, Webpack, Vite và Rust Tools.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Compiler Pipeline | `AST (Abstract Syntax Tree)` | Cấu trúc cây cú pháp trừu tượng được sinh ra từ Lexer/Parser, làm đầu vào cho Babel, SWC, ESBuild biến đổi code | `Abstract Syntax Tree AST Parser Babel SWC` |
| JIT Architecture | `V8 Ignition Interpreter & TurboFan JIT` | Ignition thực thi Bytecode nhanh, TurboFan thu thập thông tin kiểu dữ liệu (Type Feedback) để biên dịch Machine Code tối ưu | `V8 Ignition TurboFan JIT compiler Inline Cache` |
| Plugin Architecture | `Webpack Tapable Plugin Hooks` | Thư viện Tapable cung cấp cơ chế Event Hooks (Sync/Async, Bail, Waterfall) cho toàn bộ hệ thống Loader và Plugin của Webpack | `Webpack Tapable hooks loader plugin architecture` |
| Optimization | `Tree-Shaking & SideEffects Flag` | Kỹ thuật loại bỏ Dead Code dựa trên phân tích tĩnh ES Modules (`import/export`) và khai báo `'sideEffects': false` trong `package.json` | `Tree Shaking ES Modules sideEffects dead code elimination` |
| Dev Server | `Vite Native ESM Dev Server` | Vite không bundle mã nguồn ở môi trường dev mà phục vụ trực tiếp các file qua Browser Native ES Modules, nén bằng esbuild | `Vite Native ESM Dev Server esbuild HMR` |
| Native Tooling | `SWC & esbuild Rust/Go Engines` | Thế hệ Build Tools viết bằng ngôn ngữ hệ thống (Rust/Go) chạy nhanh gấp 10-100 lần so với các công cụ thuần JS | `SWC esbuild Rust Go compiler performance` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **SplitChunks Plugin Chunk Graph**: Thuật toán gom nhóm modules trùng lặp tạo ra các vendor chunks tối ưu cho browser caching dài hạn.
- **Hot Module Replacement (HMR) Internals**: Cơ chế duy trì kết nối WebSocket giữa Dev Server và Browser để thay thế module code mà không reload toàn trang.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Scope Hoisting (ModuleConcatenationPlugin)**: Gộp tất cả các module vào cùng 1 closure scope để giảm chi phí gọi function wrapper và giảm dung lượng bundle.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Migrating Monolith from Webpack to Vite + SWC**: Case study thực tế về các bẫy khi chuyển đổi dự án lớn sang Vite (như CommonJS modules, polyfills, environment variables).

