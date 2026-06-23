# 🚀 React Performance — Tối Ưu Hiệu Năng Toàn Diện

> Tài liệu chuyên sâu về Performance trong React — từ lý thuyết đến thực chiến và phỏng vấn level **Middle → Senior**.

---

## 📚 Mục Lục Tài Liệu

| # | File | Chủ đề | Level |
|---|------|--------|-------|
| 1 | [Rendering Optimization](./01-rendering-optimization.md) | Re-render, React.memo, reconciliation, Profiler | ⭐⭐ Mid |
| 2 | [Code Splitting & Lazy Loading](./02-code-splitting.md) | lazy(), Suspense, bundle analysis, route-based splitting | ⭐⭐ Mid |
| 3 | [State & Data Flow Performance](./03-state-data-perf.md) | State shape, Context perf, selector pattern, Zustand | ⭐⭐⭐ Mid+ |
| 4 | [Network & Data Fetching](./04-network-data.md) | Caching, prefetch, stale-while-revalidate, React Query | ⭐⭐⭐ Senior |
| 5 | [Interview Q&A](./05-interview-qna.md) | 25+ câu hỏi tình huống thực chiến | ⭐⭐⭐ Mid–Senior |

---

## 🗺️ Mental Model — "Tắc Đường" Ở Đâu?

```
User Action
    │
    ▼
[JavaScript Execution]  ──→ Nặng? → Web Workers, memoize, virtualize
    │
    ▼
[React Reconciliation]  ──→ Re-render? → memo, key, state shape
    │
    ▼
[Commit to DOM]         ──→ Layout thrash? → useLayoutEffect, FLIP
    │
    ▼
[Browser Paint/Composite] → CSS perf, GPU layers, will-change
    │
    ▼
[Network]               ──→ Slow? → Code splitting, prefetch, cache
```

---

## 🎯 Quick Decision Guide

**Khi component re-render quá nhiều:**
→ [`01-rendering-optimization.md`](./01-rendering-optimization.md)

**Khi bundle quá nặng, load chậm:**
→ [`02-code-splitting.md`](./02-code-splitting.md)

**Khi Context / State update gây lag:**
→ [`03-state-data-perf.md`](./03-state-data-perf.md)

**Khi API calls không hiệu quả, waterfall:**
→ [`04-network-data.md`](./04-network-data.md)

**Chuẩn bị phỏng vấn:**
→ [`05-interview-qna.md`](./05-interview-qna.md)

---

## ⚡ Top 5 "Quick Wins" Ngay Hôm Nay

1. **Bật React DevTools Profiler** → Xác định component nào render nhiều nhất.
2. **Thêm `key` đúng cách** → Tránh React re-create DOM không cần thiết.
3. **Wrap heavy child bằng `React.memo`** + ổn định props reference.
4. **Code-split route-level** bằng `React.lazy` + `Suspense`.
5. **Dùng `useTransition`** cho search/filter lists → UI không bị freeze.

---

## 🔗 Tài Liệu Liên Quan

- [Performance Hooks (useMemo/useCallback/useTransition)](../hooks-patterns/04-performance-hooks.md)
- [React Docs — Performance](https://react.dev/learn/render-and-commit)
- [web.dev — Core Web Vitals](https://web.dev/vitals/)
