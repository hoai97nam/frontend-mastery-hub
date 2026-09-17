# 🎯 Senior TypeScript Interview Questions & Type-Level Programming Challenges

> **Mục đích:** Bộ câu hỏi phỏng vấn phân loại cấp độ (Mid → Senior → Architect) kèm lời giải chi tiết, phân tích bẫy Type System, thử thách lập trình ở cấp độ kiểu (Type-Level Programming) và kỹ năng tối ưu hiệu năng biên dịch cho các dự án quy mô lớn.

---

## 📑 Mục Lục (Table of Contents)
- [📌 Tổng Quan Bộ Câu Hỏi](#-tổng-quan-bộ-câu-hỏi)
- [❓ 1. Core Type System Internals](#-1-core-type-system-internals)
- [🧩 2. Type-Level Programming Challenges (Thử thách Code Type)](#-2-type-level-programming-challenges-thử-thách-code-type)
- [🏗️ 3. Framework Architecture & Type Safety](#️-3-framework-architecture--type-safety)
- [⚡ 4. Compiler Performance & Monorepo Optimization](#-4-compiler-performance--monorepo-optimization)

---

## 📌 Tổng Quan Bộ Câu Hỏi

| Nhóm chủ đề | Số lượng câu hỏi | Trọng tâm đánh giá Senior |
|-------------|-------------------|--------------------------|
| **1. Core Type System Internals** | 6 Câu hỏi | Structural vs Nominal, Type Erasure, Variance, Subtyping |
| **2. Type-Level Programming Puzzles** | 7 Câu hỏi | Utility Types tự viết (`infer`, Conditional Types, Mapped Types) |
| **3. Framework Architecture & Type Safety** | 6 Câu hỏi | Polymorphic Types, Generics, Zod Schema Sync, State Stores |
| **4. Compiler Performance & Monorepo** | 6 Câu hỏi | Project References, `tsc --build`, Incremental Builds, Strict Flags |

---

## ❓ 1. Core Type System Internals

### Q1: Phân biệt Structural Typing (TypeScript) và Nominal Typing (Java/C#). Làm thế nào để giả lập Nominal Typing trong TypeScript?

**Trả lời:**
- **Structural Typing (Duck Typing)**: TypeScript kiểm tra tính tương thích dựa trên **cấu trúc thuộc tính (shape)** của type. Nếu Object A chứa tất cả các thuộc tính mà Type B yêu cầu, A có thể gán cho B cho dù tên khai báo khác nhau.
- **Nominal Typing**: Java/C# kiểm tra tính tương thích dựa trên **tên lớp (Class/Interface Name) và quan hệ kế thừa khai báo tường minh**.

```typescript
// Structural Example:
interface Point2D { x: number; y: number; }
interface Vector2D { x: number; y: number; }
let p: Point2D = { x: 10, y: 20 };
let v: Vector2D = p; // ✅ OK hoàn toàn trong TypeScript vì cấu trúc trùng khớp!

// Giả lập Nominal Typing với Branded / Opaque Types:
declare const brandSymbol: unique symbol;
type Brand<T, BrandName extends string> = T & { readonly [brandSymbol]: BrandName };

type USD = Brand<number, "USD">;
type EUR = Brand<number, "EUR">;

let walletUSD = 100 as USD;
let walletEUR = 100 as EUR;

// walletUSD = walletEUR; // ❌ LỖI BIÊN DỊCH! TypeScript coi đây là 2 kiểu hoàn toàn khác nhau.
```

---

### Q2: Sự khác biệt bản chất giữa `any`, `unknown`, `never` và `void` là gì?

**Trả lời:**
- **`any` (Unsound Type)**: Tắt Type Checker. Cho phép gán bất kỳ giá trị nào VÀ cho phép gọi bất kỳ property/method nào mà không kiểm tra.
- **`unknown` (Top Type)**: Cho phép gán bất kỳ giá trị nào VÀO `unknown`, nhưng **KHÔNG cho phép tương tác/gọi phương thức** trừ khi thực hiện Type Narrowing (`typeof`, `instanceof`, Type Guard).
- **`never` (Bottom Type)**: Biểu diễn tập hợp rỗng. Không có bất kỳ giá trị nào có thể gán cho `never` (ngoại trừ chính `never`). Thường dùng cho các hàm không bao giờ return (throw Error, infinite loop) hoặc nhánh code rỗng không thể chạm tới.
- **`void`**: Biểu diễn sự vắng mặt của giá trị trả về trong hàm (hàm chạy xong nhưng không return giá trị nào, hoặc trả về `undefined`).

---

### Q3: Covariance và Contravariance ảnh hưởng thế nào đến Type Safety của hàm trong TypeScript?

**Trả lời:**
- **Covariance (Thuận chiều)**: `Dog extends Animal` $\Rightarrow$ `List<Dog> extends List<Animal>`. Trả về kiểu con trong return value là hoàn toàn an toàn.
- **Contravariance (Nghịch chiều)**: `Dog extends Animal` $\Rightarrow$ `(a: Animal) => void extends (d: Dog) => void`. Hàm nhận tham số cha có thể thay thế cho hàm nhận tham số con.

```typescript
type Fn<in T, out R> = (arg: T) => R; 
// Từ TS 4.7+, ta có thể ghi rõ chú thích variance: `in` cho Contravariant, `out` cho Covariant.
```
Bật cờ `strictFunctionTypes: true` buộc TypeScript kiểm tra nghiêm ngặt tính Contravariant của các tham số truyền vào hàm, ngăn chặn việc gọi các phương thức không tồn tại ở runtime.

---

## 🧩 2. Type-Level Programming Challenges (Thử thách Code Type)

### Q4: Hãy tự viết Utility Type `DeepReadonly<T>` biến tất cả thuộc tính (kể cả thuộc tính lồng nhau) thành `readonly`.

**Lời giải Senior:**

```typescript
export type DeepReadonly<T> = T extends (...args: any[]) => any
  ? T // Giữ nguyên hàm
  : T extends ReadonlyArray<infer U>
  ? ReadonlyArray<DeepReadonly<U>> // Xử lý Mảng
  : T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> } // Đệ quy qua từng key của Object
  : T; // Primitive types (string, number, boolean)

// --- TEST CASE ---
type ComplexObj = {
  user: {
    profile: {
      name: string;
      tags: string[];
    };
  };
};

type ReadonlyComplex = DeepReadonly<ComplexObj>;
// ReadonlyComplex.user.profile.name = "Bob"; // ❌ Lỗi: Cannot assign to 'name' because it is a read-only property.
```

---

### Q5: Viết Utility Type `SnakeToCamelCase<S extends string>` biến chuỗi snake_case thành camelCase ở cấp độ kiểu.

**Lời giải Senior:**

```typescript
export type SnakeToCamelCase<S extends string> =
  S extends `${infer First}_${infer Rest}`
    ? `${Uncapitalize<First>}${Capitalize<SnakeToCamelCase<Rest>>}`
    : Uncapitalize<S>;

// --- TEST CASE ---
type T1 = SnakeToCamelCase<"user_first_name">; // "userFirstName"
type T2 = SnakeToCamelCase<"is_senior_developer">; // "isSeniorDeveloper"
```

---

### Q6: Xây dựng Utility Type `TupleToUnion<T>` và `TupleToIntersection<T>`.

**Lời giải Senior:**

```typescript
// 1. Tuple to Union (Dùng Indexed Access [number] hoặc infer)
type TupleToUnion<T extends readonly any[]> = T[number];

type UnionRes = TupleToUnion<["ADMIN", "USER", "GUEST"]>; // "ADMIN" | "USER" | "GUEST"

// 2. Tuple to Intersection (Kỹ thuật ép Contravariance với infer)
type TupleToIntersection<T extends readonly any[]> = 
  (T extends any[] ? (arg: T[number]) => void : never) extends (arg: infer U) => void
    ? U
    : never;

type InterRes = TupleToIntersection<[{ a: string }, { b: number }]>; // { a: string } & { b: number }
```

---

## 🏗️ 3. Framework Architecture & Type Safety

### Q7: Tại sao toán tử `satisfies` (TS 4.9+) lại ưu việt hơn Type Assertion (`as`) và Type Annotation (`:`) trong việc định nghĩa Config Objects?

**Trả lời:**
- **Type Assertion (`as Type`)**: Bỏ qua kiểm tra lỗi của compiler nếu có ép kiểu không hoàn toàn khớp (Unsound). Dễ gây lỗi runtime.
- **Type Annotation (`: Type`)**: Bắt buộc tuân thủ Type, nhưng làm **nhoè (widen)** kiểu cụ thể. Không gọi được phương thức riêng của thuộc tính con.
- **`satisfies`**: Đảm bảo vừa validate đúng cấu trúc interface vừa **giữ nguyên Type gốc cụ thể (Literal Type Inference)**.

```typescript
type RouteConfig = Record<string, { path: string; exact?: boolean }>;

// Với satisfies:
const routes = {
  home: { path: '/' },
  user: { path: '/user/:id', exact: true },
} satisfies RouteConfig;

routes.home.path; // ✅ OK
routes.user.exact; // ✅ TS tự biết exact là boolean mà không bị rộng thành undefined!
```

---

### Q8: Làm thế nào để giải quyết vấn đề Exhaustiveness Checking trong Switch-case statement để đảm bảo khi thêm Enum value mới, compiler sẽ báo lỗi lập tức?

**Lời giải Senior:**

```typescript
type Action = 
  | { type: 'LOGIN'; payload: { username: string } }
  | { type: 'LOGOUT' }
  | { type: 'REFRESH_TOKEN' }; // Mới thêm vào!

function reducer(state: any, action: Action) {
  switch (action.type) {
    case 'LOGIN': return state;
    case 'LOGOUT': return state;
    // THIẾU 'REFRESH_TOKEN'!
    
    default: {
      // Ép kiểu gán vào variable type `never`
      const _exhaustiveCheck: never = action;
      throw new Error(`Unhandled action type: ${_exhaustiveCheck}`);
    }
  }
}
// ❌ LỖI BIÊN DỊCH NGAY LẬP TỨC: Type '{ type: "REFRESH_TOKEN"; }' is not assignable to type 'never'.
```

---

## ⚡ 4. Compiler Performance & Monorepo Optimization

### Q9: Làm sao để tối ưu thời gian Type Check (`tsc`) trong một Monorepo có hàng trăm nghìn dòng code?

**Trả lời của Architect:**
1. **Dùng TypeScript Project References (`tsconfig.json` refs)**: Chia Monorepo thành các sub-projects có `tsconfig.json` riêng với `"composite": true`. Khi biên dịch dùng `tsc --build` (`tsc -b`), TypeScript chỉ type-check lại các package có thay đổi (Incremental Build nhờ file `.tsbuildinfo`).
2. **Kích hoạt `skipLibCheck: true`**: Bỏ qua việc check lại các file `.d.ts` nằm trong `node_modules`.
3. **Phân tách Type Checking và Code Transpilation**:
   - Dùng **Vite / SWC / esbuild** cho quá trình dev server & production build bundle (chỉ xóa type annotation bằng Type Erasure, siêu nhanh).
   - Chạy `tsc --noEmit` trên CI/CD pipeline riêng để verify Type Safety.
4. **Tránh Type Intersections quá rộng & Recursive Types sâu**: Ưu tiên `interface extends` thay cho `type A = B & C & D` để tận dụng cơ chế caching hash table của compiler.

---

### Q10: Cờ `noUncheckedIndexedAccess` trong `tsconfig.json` có tác dụng gì và tại sao mọi dự án Senior nên bật cờ này?

**Trả lời:**
- Mặc định, khi bạn khai báo `const map: Record<string, User> = {}`, truy cập `map["abc"]` sẽ trả về kiểu `User`. Tuy nhiên ở runtime, nếu key `"abc"` không tồn tại, giá trị thực tế lại là `undefined` $\Rightarrow$ Dễ gây ra lỗi runtime crash `Cannot read properties of undefined`.
- Khi bật `noUncheckedIndexedAccess: true`, TypeScript sẽ tự động suy luận kiểu truy cập index là `User | undefined`, bắt buộc nhà phát triển phải check null/undefined an toàn trước khi thao tác!
