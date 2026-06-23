# ⚛️ React Hooks — Toàn Tập

> Tài liệu học và ôn tập React Hooks: phân loại, use cases, code examples, và câu hỏi phỏng vấn.

---

## 📚 Mục Lục Nhóm Hook

React Hooks được chia thành **5 nhóm chính** theo mục đích sử dụng:

| Nhóm | Hooks | Mô tả |
|------|-------|-------|
| 🗂️ [State Hooks](./01-state-hooks.md) | `useState`, `useReducer` | Quản lý trạng thái cục bộ |
| 🔗 [Context & Ref Hooks](./02-context-ref-hooks.md) | `useContext`, `useRef`, `useImperativeHandle` | Chia sẻ dữ liệu & thao tác DOM |
| ⚡ [Effect Hooks](./03-effect-hooks.md) | `useEffect`, `useLayoutEffect`, `useInsertionEffect` | Side effects & DOM sync |
| 🚀 [Performance Hooks](./04-performance-hooks.md) | `useMemo`, `useCallback`, `useTransition`, `useDeferredValue` | Tối ưu render |
| 🆕 [React 19+ Hooks](./05-new-hooks.md) | `use`, `useOptimistic`, `useActionState`, `useFormStatus` | Async & Server Actions |

---

## 🗂️ Danh Sách File Chi Tiết

### Nhóm 1 — State Hooks
📄 [01-state-hooks.md](./01-state-hooks.md)
- `useState` — State cơ bản, functional update, lazy initialization
- `useReducer` — State phức tạp, action-based logic

### Nhóm 2 — Context & Ref Hooks
📄 [02-context-ref-hooks.md](./02-context-ref-hooks.md)
- `useContext` — Consume context, tránh prop drilling
- `useRef` — DOM access, mutable values, tránh re-render
- `useImperativeHandle` — Expose imperative API qua forwardRef

### Nhóm 3 — Effect Hooks
📄 [03-effect-hooks.md](./03-effect-hooks.md)
- `useEffect` — Data fetching, subscriptions, cleanup
- `useLayoutEffect` — DOM measurement trước khi browser paint
- `useInsertionEffect` — CSS-in-JS injection (nâng cao)

### Nhóm 4 — Performance Hooks
📄 [04-performance-hooks.md](./04-performance-hooks.md)
- `useMemo` — Memoize computed values
- `useCallback` — Memoize functions
- `useTransition` — Non-urgent state updates
- `useDeferredValue` — Defer expensive renders

### Nhóm 5 — React 19+ Hooks
📄 [05-new-hooks.md](./05-new-hooks.md)
- `use` — Đọc Promise / Context trong render
- `useOptimistic` — Optimistic UI updates
- `useActionState` — Form action state
- `useFormStatus` — Pending state của form

---

## 🗺️ Lộ Trình Học Đề Xuất

```
useState → useReducer → useContext → useRef → useEffect
    → useMemo / useCallback → useTransition → React 19 Hooks
```

---

## 🎯 Quick Reference — Rules of Hooks

> Hai quy tắc bất di bất dịch:

1. **Chỉ gọi Hook ở top level** — Không gọi trong loop, condition, hay nested function.
2. **Chỉ gọi Hook trong React function** — Function Component hoặc Custom Hook.

---

## 🔗 Tài Liệu Liên Quan

- [React Docs — Built-in Hooks](https://react.dev/reference/react)
- [Custom Hooks Patterns](../state-management/)
- [Performance Deep Dive](../performance/)
