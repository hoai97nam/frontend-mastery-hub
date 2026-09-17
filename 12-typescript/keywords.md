# 🔑 Senior Keywords — TypeScript Core & Framework Architecture

> Tổng hợp các từ khóa, cơ chế và thuật ngữ cốt lõi về TypeScript ở cấp độ **Senior / Architect** phục vụ cho việc tra cứu, thiết kế hệ thống và gây ấn tượng trong phỏng vấn kỹ thuật.

---

## 📑 Mục Lục (Table of Contents)
- [📋 Bảng Tra Cứu Từ Khóa Senior](#-bảng-tra-cứu-từ-khóa-senior)
- [🎯 Chi Tiết Theo Nhóm Chuyên Sâu](#-chi-tiết-theo-nhóm-chuyên-sâu)
  - [1. Cơ chế bên dưới (Under the Hood)](#1-cơ-chế-bên-dưới-under-the-hood)
  - [2. Tối ưu hiệu năng & Bộ nhớ (Compiler Performance)](#2-tối-ưu-hiệu-năng--bộ-nhớ-compiler-performance)
  - [3. Kiến trúc & Design Patterns (Architecture)](#3-kiến-trúc--design-patterns-architecture)
  - [4. Bẫy phỏng vấn & Case Studies (Interview Triggers)](#4-bẫy-phỏng-vấn--case-studies-interview-triggers)

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| **Type System** | **Structural Typing (Duck Typing)** | TypeScript so sánh kiểu dựa trên cấu trúc hình dạng (shape/properties) thay vì tên lớp hay khai báo rõ ràng như Java/C# (Nominal Typing). | Structural vs Nominal subtyping, Shape matching |
| **Type System** | **Type Erasure** | Toàn bộ type annotations và interfaces hoàn toàn bị loại bỏ khi biên dịch sang JavaScript, không tồn tại ở runtime và không làm tốn bộ nhớ thực thi. | TypeScript AST emission, erased types runtime cost |
| **Advanced Types** | **Conditional Types & `infer`** | Kỹ thuật meta-programming trên kiểu dữ liệu: cho phép rẽ nhánh kiểu dựa trên điều kiện `T extends U ? X : Y` và trích xuất kiểu tự động qua từ khóa `infer`. | Type-level pattern matching, infer keyword TypeScript |
| **Advanced Types** | **Mapped Types & Template Literal Types** | Biến đổi tập hợp keys của type thành một type mới, kết hợp chuỗi literal ở type-level để tự động sinh ra các type động cho Event Bus, I18n, Route Params. | Mapped type remapping, Template literal types TS |
| **Type Soundness** | **Variance (Covariance & Contravariance)** | Quy tắc tương thích kiểu cho các kiểu phức hợp (Generics/Functions): Covariance giữ nguyên chiều kế thừa (bản kiểm tra output), Contravariance đảo ngược chiều kế thừa (bản kiểm tra input parameters). | TypeScript strict Function Types, Function parameter bivariance |
| **Type Narrowing** | **Discriminated Unions (Tagged Unions)** | Mẫu thiết kế state bằng cách dùng một property chung cố định (ví dụ `kind` hoặc `type`) để TypeScript tự động narrowing chính xác 100% trong `switch/if`. | Algebraic Data Types (ADT), Tagged Union Pattern |
| **Type Safety** | **`satisfies` Operator** | Kiểm tra giá trị thỏa mãn một kiểu dữ liệu mà **không làm mất** đi kiểu cụ thể thực sự (literal inference) của giá trị đó (khác với Type Annotation `: Type`). | TS 4.9 satisfies operator vs type assertion |
| **Type Safety** | **`const` Type Parameters** | Giữ nguyên kiểu hằng số (readonly literal type inference) trực tiếp tại Generic declaration mà không cần người dùng viết `as const` mỗi khi gọi hàm. | TS 5.0 const type parameters, literal inference |
| **Declarations** | **Declaration Merging & Ambient Modules** | Cơ chế tự động gộp các `interface` trùng tên hoặc mở rộng định nghĩa thư viện ngoài thông qua file `.d.ts` (`declare module`). | TS declaration merging, ambient module declarations |
| **Performance** | **Project References & Incremental Build** | Giải pháp tăng tốc độ compile cho monorepo lớn bằng cách chia nhỏ project thành các sub-projects có `tsconfig.json` độc lập và dùng `tsc --build`. | TS project references, incremental compilation `tsbuildinfo` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)
- **Type Checker Engine & Symbol Table**: Hiểu cách `tsc` dựng AST, tạo Binders và Resolve Symbols. Biết được lý do tại sao các Mapped Types lồng nhau quá sâu có thể làm tràn bộ nhớ compiler (`Type instantiation is excessively deep and possibly infinite`).
- **Nominal vs Structural Subtyping**: Giải thích được tại sao 2 `interface` có tên khác nhau hoàn toàn nhưng có cùng cấu trúc thuộc tính thì vẫn gán được cho nhau trong TypeScript. Biết cách dùng **Branded Types / Opaque Types** (`type Brand<K, T> = K & { __brand: T }`) để giả lập Nominal Typing cho Currency, UserID, UserEmail.
- **Function Parameter Bivariance vs Contravariance**: Hiểu lý do cờ `strictFunctionTypes` ép các tham số hàm từ Bivariant sang Contravariant để ngăn ngừa lỗi runtime method parameter substitution.

### 2. Tối ưu hiệu năng & Bộ nhớ (Compiler Performance)
- **Avoiding Deeply Nested Utility Types**: Hạn chế viết các type đệ quy vô hạn mà không có điều kiện dừng (`Recursion depth limit`).
- **Interface Extends vs Type Intersections**: Ưu tiên `interface B extends A` hơn `type B = A & C` vì TypeScript compiler cache các kết quả so sánh `interface` theo name hash, còn `&` buộc compiler phải đánh giá lại mọi properties mỗi lần type check.
- **SkipLibCheck & IsolatedModules**: Cấu hình `skipLibCheck: true` trong `tsconfig.json` để bỏ qua type-checking trong `node_modules`, dùng `isolatedModules: true` để đảm bảo code hoàn toàn tương thích với các bundler dựa trên SWC / esbuild.

### 3. Kiến trúc & Design Patterns (Architecture)
- **Polymorphic Component Typings**: Kỹ thuật viết Component React dùng `as` prop (`<Button as="a" href="...">`) với type safety tuyệt đối cho mọi HTML Attributes tương ứng.
- **Single Source of Truth với Schema Validation**: Dùng Zod/Valibot để định nghĩa Schema Runtime, từ đó suy luận type tĩnh bằng `z.infer<typeof Schema>`, đảm bảo sync 100% giữa API Runtime data và TypeScript Types.
- **Generic Repository & Factory Patterns**: Áp dụng Generics và Type Constraints để xây dựng Data Access Layer hoặc API Client dùng chung cho toàn bộ dự án Enterprise.

### 4. Bẫy phỏng vấn & Case Studies (Interview Triggers)
- **Bẫy `any` Leakage**: Giải thích cơ chế `any` truyền nhiễm sang toàn bộ codebase và cách chặn đứng bằng `unknown` hoặc cờ `noImplicitAny`.
- **Bẫy Index Signature**: Tại sao `Record<string, User>` trả về `User` thay vì `User | undefined` khi truy cập key không tồn tại, và cách sửa bằng cờ `noUncheckedIndexedAccess`.
- **Exhaustiveness Checking với `never`**: Cách viết `default: const _exhaustiveCheck: never = check; throw new Error(...)` trong `switch-case` để compiler bắt buộc developer phải xử lý đủ mọi enum values khi bổ sung case mới.
