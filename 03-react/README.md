# ⚛️ React Deep Dive

> Tài liệu học React toàn diện — từ Hooks đến State Management và Performance.

---

## 📂 Cấu Trúc Thư Mục

| Thư mục | Nội dung |
|---------|---------|
| [`hooks-patterns/`](./hooks-patterns/README.md) | Tất cả React Hooks — phân loại, use cases, code examples, câu hỏi phỏng vấn |
| [`component-patterns/`](./component-patterns/README.md) | Các mẫu thiết kế component (Compound Component, Render Props, Controlled/Uncontrolled...) |
| [`state-management/`](./state-management/) | Context, Zustand, Redux, React Query |
| [`performance/`](./performance/) | Optimization, profiling, lazy loading |
| [`interviews/`](./interviews/) | Câu hỏi phỏng vấn React tổng hợp |

---

## 🎯 Bắt Đầu Với Hooks

👉 **[React Hooks — Toàn Tập](./hooks-patterns/README.md)**

Bao gồm **5 nhóm hook** với đầy đủ tài liệu:

1. 🗂️ [State Hooks](./hooks-patterns/01-state-hooks.md) — `useState`, `useReducer`
2. 🔗 [Context & Ref Hooks](./hooks-patterns/02-context-ref-hooks.md) — `useContext`, `useRef`, `useImperativeHandle`
3. ⚡ [Effect Hooks](./hooks-patterns/03-effect-hooks.md) — `useEffect`, `useLayoutEffect`, `useInsertionEffect`
4. 🚀 [Performance Hooks](./hooks-patterns/04-performance-hooks.md) — `useMemo`, `useCallback`, `useTransition`, `useDeferredValue`
5. 🆕 [React 19+ Hooks](./hooks-patterns/05-new-hooks.md) — `use`, `useOptimistic`, `useActionState`, `useFormStatus`

---

## 🧩 Các Mẫu Thiết Kế Component (Component Patterns)

👉 **[React Component Patterns — Toàn Tập](./component-patterns/README.md)**

Bao gồm **5 mẫu thiết kế** và tài liệu phỏng vấn:

1. 📦 [Compound Component](./component-patterns/01-compound-component.md) — Thiết kế component hỗn hợp qua Context
2. 📦 [Render Props](./component-patterns/02-render-props.md) — Tái sử dụng logic kèm render function
3. 📦 [Controlled & Uncontrolled](./component-patterns/03-controlled-uncontrolled.md) — Quản lý nguồn state/ref trong form
4. 📦 [Higher-Order Component](./component-patterns/04-hoc.md) — Composition component nâng cao
5. 📦 [State Reducer & Control Props](./component-patterns/05-state-reducer-control-props.md) — Can thiệp sâu và kiểm soát state từ xa
6. 🧠 [Phỏng vấn & Tình huống](./component-patterns/interviews.md) — Các câu hỏi thực chiến cho Middle/Senior
