# 📦 REPO CONTEXT — frontend-mastery-hub

> **Mục đích file này:** Tổng hợp toàn bộ kiến thức về repo để paste vào cuộc hội thoại mới, giúp AI assistant nắm bối cảnh ngay lập tức mà không cần khám phá lại từ đầu.

---

## 1. TỔNG QUAN

### Repo này dành cho ai?
- **Frontend developer** cấp độ **Middle → Senior**
- Người đang chuẩn bị **phỏng vấn kỹ thuật** (câu hỏi từ Mid đến Senior/Architect)
- Người muốn **hệ thống hóa kiến thức** thay vì học rải rác

### Mục đích
Hệ thống hóa kiến thức Frontend toàn diện — lý thuyết, code thực chiến, và câu hỏi phỏng vấn — theo từng chủ đề, viết bằng **tiếng Việt**.

### Tech Stack học trong repo
| Tầng | Công nghệ |
|------|-----------|
| **Languages** | JavaScript (ESNext), TypeScript, C# (study patterns) |
| **Frameworks** | React, Angular, Vue, .NET (patterns) |
| **Styling** | Tailwind CSS, SCSS |
| **State** | Redux, Pinia, RxJS, Signals |
| **Architecture** | Micro Frontends (Module Federation, Web Components) |
| **Patterns** | SOLID, GoF Design Patterns |
| **Data Structures** | Array → Graph + Modern DS |

---

## 2. CẤU TRÚC THƯ MỤC (rút gọn)

```
frontend-mastery-hub/
│
├── README.md                        # Tổng quan + roadmap 6 phase
│
├── 01-javascript/                   # JavaScript Core
│   ├── README.md
│   ├── concepts/                    # Lý thuyết JS sâu
│   │   ├── closure.md               # Closure & Memory Leak (467 dòng)
│   │   ├── event-loop.md            # Event Loop cơ bản
│   │   ├── event-loop-questions.md  # Q&A Event Loop
│   │   └── lab.md                   # Lab thực hành
│   ├── core-concepts/               # (trống - chưa có nội dung)
│   ├── es6-plus/                    # (trống - chưa có nội dung)
│   ├── web-apis/                    # (trống - chưa có nội dung)
│   └── interviews/                  # Câu hỏi phỏng vấn JS
│       ├── senior-questions.md
│       ├── advanced-async.md
│       ├── design-patterns.md
│       └── performance-v8.md
│
├── 02-styling/                      # CSS / SCSS / Tailwind
│   ├── README.md
│   ├── css-advanced/                # (trống)
│   ├── scss-structure/              # (trống)
│   └── tailwind-lab/                # (trống)
│
├── 03-react/                        # React Deep Dive ✅ (có nội dung đầy đủ)
│   ├── README.md
│   ├── hooks-patterns/              # React Hooks toàn tập
│   │   ├── README.md
│   │   ├── 01-state-hooks.md        # useState, useReducer
│   │   ├── 02-context-ref-hooks.md  # useContext, useRef, useImperativeHandle
│   │   ├── 03-effect-hooks.md       # useEffect, useLayoutEffect, useInsertionEffect
│   │   ├── 04-performance-hooks.md  # useMemo, useCallback, useTransition, useDeferredValue
│   │   └── 05-new-hooks.md          # React 19+: use, useOptimistic, useActionState, useFormStatus
│   ├── component-patterns/          # React Component Design Patterns ✅
│   │   ├── README.md
│   │   ├── 01-compound-component.md # Compound Component
│   │   ├── 02-render-props.md       # Render Props
│   │   ├── 03-controlled-uncontrolled.md # Controlled & Uncontrolled
│   │   ├── 04-hoc.md                # Higher-Order Component (HOC)
│   │   ├── 05-state-reducer-control-props.md # State Reducer & Control Props
│   │   └── interviews.md            # Q&A + 2 tình huống thiết kế thực tế
│   ├── performance/                 # Tối ưu hiệu năng React ✅
│   │   ├── README.md
│   │   ├── 01-rendering-optimization.md  # Re-render, memo, reconciliation
│   │   ├── 02-code-splitting.md          # lazy(), Suspense, bundle
│   │   ├── 03-state-data-perf.md         # Context perf, Zustand, selector
│   │   ├── 04-network-data.md            # Caching, React Query
│   │   └── 05-interview-qna.md           # 25+ câu hỏi phỏng vấn
│   ├── state-management/            # (trống - Context, Zustand, Redux)
│   └── interviews/                  # (trống)
│
├── 04-angular/                      # Angular
│   ├── README.md
│   ├── architecture/                # (trống)
│   ├── dependency-injection/        # (trống)
│   ├── rxjs-signals/                # (trống)
│   └── interviews/                  # (trống)
│
├── 05-vue/                          # Vue 3
│   ├── README.md
│   ├── composition-api/             # (trống)
│   ├── pinia-state/                 # (trống)
│   ├── ecosystem/                   # (trống)
│   └── interviews/                  # (trống)
│
├── 06-coding-challenges/            # Bài tập thực hành
│   ├── algorithms/                  # (trống)
│   └── ui-components/              # (trống)
│
├── 07-micro-frontends/              # Micro Frontends ✅ (có nội dung đầy đủ)
│   ├── README.md                    # Tổng quan MFE + Decision Matrix
│   ├── component-sharing.md         # Module Federation, Web Components
│   ├── communication.md             # Giao tiếp giữa các MFE
│   ├── core-mechanisms.md           # Routing, CSS isolation, CI/CD
│   └── interviews.md                # Q&A Senior-level
│
├── 08-design-patterns/              # Design Patterns (C# Focus) ✅ (có nội dung đầy đủ)
│   ├── README.md                    # SOLID + Decision Matrix
│   ├── creational-patterns.md       # Singleton, Factory, Builder, Prototype
│   ├── structural-patterns.md       # Adapter, Decorator, Facade, Proxy...
│   ├── behavioral-patterns.md       # Strategy, Observer, Command, State...
│   ├── hidden-patterns.md           # DI, Repository, Specification, Null Object
│   └── interviews.md                # Q&A + 5 tình huống thiết kế thực tế
│
├── 09-data-structures/              # Data Structures ✅ (đang xây dựng)
│   ├── README.md                    # Roadmap 4 tuần + template mỗi file
│   ├── 01-linear/                   # Linear DS ✅ Hoàn thành
│   │   ├── README.md
│   │   ├── array.md
│   │   ├── linked-list.md
│   │   ├── stack.md
│   │   ├── queue.md
│   │   └── deque.md
│   ├── 02-hash-based/               # Hash DS ✅ Hoàn thành
│   │   ├── README.md
│   │   ├── hash-map.md              # ✅ Hoàn thành
│   │   ├── hash-set.md              # ✅ Hoàn thành
│   │   └── bloom-filter.md          # ✅ Hoàn thành
│   ├── 03-tree/                     # Cây phân cấp ✅ Hoàn thành
│   │   ├── README.md
│   │   ├── binary-tree.md           # ✅ Hoàn thành
│   │   ├── binary-search-tree.md    # ✅ Hoàn thành
│   │   ├── heap.md                  # ✅ Hoàn thành
│   │   ├── trie.md                  # ✅ Hoàn thành
│   │   └── segment-tree.md          # ✅ Hoàn thành
│   ├── 04-graph/                    # ❌ Chưa tạo
│   ├── 05-modern/                   # ❌ Chưa tạo
│   └── interviews/                  # ❌ Chưa tạo
│
└── docs/
    └── superpowers/                 # (trống)
```

---

## 3. MAPPING — NỘI DUNG CHI TIẾT TỪNG MODULE

### 📁 01-javascript

| File | Nội dung |
|------|---------|
| `concepts/closure.md` | Closure & Memory Leak: lý thuyết, debug, tình huống giả định, câu hỏi bẫy, React/Vue specific, cách đo bằng DevTools |
| `concepts/event-loop.md` | Call Stack, Web APIs, Callback Queue, Microtask Queue, cách hoạt động Event Loop |
| `concepts/event-loop-questions.md` | Q&A Event Loop cho phỏng vấn |
| `concepts/lab.md` | Bài tập thực hành JS core |
| `interviews/senior-questions.md` | Câu hỏi phỏng vấn Senior JS tổng hợp |
| `interviews/advanced-async.md` | Promise, async/await, concurrency |
| `interviews/design-patterns.md` | Design patterns trong JS |
| `interviews/performance-v8.md` | V8 engine, JIT, memory management |

### 📁 03-react (Module đầy đủ nhất)

**hooks-patterns/**

| File | Hooks được cover |
|------|-----------------|
| `01-state-hooks.md` | `useState` (functional update, lazy init), `useReducer` |
| `02-context-ref-hooks.md` | `useContext`, `useRef`, `useImperativeHandle` |
| `03-effect-hooks.md` | `useEffect`, `useLayoutEffect`, `useInsertionEffect` |
| `04-performance-hooks.md` | `useMemo`, `useCallback`, `useTransition`, `useDeferredValue` |
| `05-new-hooks.md` | `use`, `useOptimistic`, `useActionState`, `useFormStatus` (React 19+) |

**component-patterns/**

| File | Nội dung chính | Level |
|------|----------------|-------|
| `01-compound-component.md` | Compound Component (Context API vs cloneElement) | ⭐⭐ Mid |
| `02-render-props.md` | Render Props, Children as a Function, vs Hooks | ⭐⭐ Mid |
| `03-controlled-uncontrolled.md` | Controlled & Uncontrolled Components, State sync pitfalls | ⭐ Mid |
| `04-hoc.md` | Higher-Order Component, forwardRef, copy static methods | ⭐⭐ Mid |
| `05-state-reducer-control-props.md` | State Reducer Pattern & Control Props Pattern (Hybrid state) | ⭐⭐⭐ Senior |
| `interviews.md` | Q&A + 2 tình huống thiết kế hệ thống thực tế | ⭐⭐⭐ Senior |

**performance/**

| File | Chủ đề | Level |
|------|--------|-------|
| `01-rendering-optimization.md` | Re-render, React.memo, reconciliation, Profiler | ⭐⭐ Mid |
| `02-code-splitting.md` | lazy(), Suspense, bundle analysis | ⭐⭐ Mid |
| `03-state-data-perf.md` | State shape, Context perf, Zustand, selector | ⭐⭐⭐ Mid+ |
| `04-network-data.md` | Caching, prefetch, stale-while-revalidate, React Query | ⭐⭐⭐ Senior |
| `05-interview-qna.md` | 25+ câu phỏng vấn tình huống | ⭐⭐⭐ Senior |

### 📁 07-micro-frontends

| File | Nội dung |
|------|---------|
| `README.md` | Định nghĩa MFE, so sánh Monolith/Microservices/MFE, 4 mô hình tích hợp (Build-time, Iframe, SSI, Module Federation), Decision Matrix |
| `component-sharing.md` | Webpack/Vite Module Federation, Web Components, build-time vs run-time sharing |
| `communication.md` | Custom Events, Shared State, URL params, PostMessage, EventBus patterns |
| `core-mechanisms.md` | Routing đồng bộ, CSS isolation (Shadow DOM, CSS Modules, BEM scope), shared dependencies, Error Boundary, CI/CD |
| `interviews.md` | Q&A nâng cao + 5 tình huống thiết kế thực tế cho Senior/Architect |

### 📁 08-design-patterns

| File | Patterns được cover |
|------|-------------------|
| `README.md` | SOLID (5 nguyên lý), mối quan hệ SOLID ↔ Patterns, Decision Matrix 10 patterns phổ biến |
| `creational-patterns.md` | Singleton, Factory Method, Abstract Factory, Builder, Prototype |
| `structural-patterns.md` | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy |
| `behavioral-patterns.md` | Strategy, Observer, Command, Iterator, State, Template Method, Mediator, Chain of Responsibility, Visitor, Memento, Interpreter |
| `hidden-patterns.md` | Dependency Injection, Repository & Unit of Work, Specification, Fluent API, Null Object, Double Dispatch |
| `interviews.md` | Q&A nâng cao + 5 tình huống thiết kế thực tế |

### 📁 09-data-structures

**01-linear/** — đã hoàn thành

| File | Data Structure | Big-O highlight |
|------|---------------|----------------|
| `array.md` | Array | Access O(1), Insert O(n) |
| `linked-list.md` | Linked List | Insert/Delete O(1)*, Access O(n) |
| `stack.md` | Stack | Push/Pop O(1) — LIFO |
| `queue.md` | Queue | Enqueue/Dequeue O(1) — FIFO |
| `deque.md` | Deque | O(1) cả 2 đầu |

**02-hash-based/** — đã hoàn thành

| File | Data Structure | Mô tả |
|------|---------------|-------|
| `hash-map.md` | HashMap | Ánh xạ key → value, O(1) tra cứu |
| `hash-set.md` | HashSet | Tập hợp không trùng lặp, O(1) membership |
| `bloom-filter.md` | Bloom Filter | Cấu trúc xác suất tiết kiệm bộ nhớ, kiểm tra membership |

**03-tree/** — đã hoàn thành

| File | Data Structure | Mô tả |
|------|---------------|-------|
| `binary-tree.md` | Binary Tree | Cây nhị phân cơ bản (mỗi nút tối đa 2 con) |
| `binary-search-tree.md` | BST | Cây tìm kiếm nhị phân, trung bình O(log n) |
| `heap.md` | Heap | Priority queue, truy cập min/max trong O(1) |
| `trie.md` | Trie | Cây tiền tố, prefix search/autocomplete |
| `segment-tree.md` | Segment Tree | Range queries (sum/min/max) và point updates trong O(log n) |

**Roadmap chưa tạo (có trong 09-data-structures/README.md):**
- `04-graph/`: Graph, DAG
- `05-modern/`: LRU Cache, Skip List, Persistent DS, Rope, Union-Find, HyperLogLog
- `interviews/qna.md`: 40+ câu hỏi phỏng vấn

---

## 4. QUY ƯỚC & CONVENTIONS

### Đặt tên thư mục
- **Dùng số thứ tự 2 chữ số** cho module cấp 1: `01-javascript`, `02-styling`, `03-react`...
- **Dùng số thứ tự** cho sub-module trong data-structures: `01-linear`, `02-hash-based`...
- **Kebab-case** cho tất cả tên thư mục: `hooks-patterns`, `state-management`, `micro-frontends`
- **Tên mô tả chủ đề** không dùng viết tắt khó hiểu

### Đặt tên file
- **Kebab-case**: `closure.md`, `event-loop.md`, `hash-map.md`
- **Có số thứ tự** khi trong một series có thứ tự học tập: `01-state-hooks.md`, `02-context-ref-hooks.md`
- Luôn có `README.md` ở mỗi thư mục làm **index/entry point**
- File phỏng vấn đặt tên `interviews.md` hoặc trong thư mục `interviews/`

### Cấu trúc nội dung — Module lớn (README.md)

```markdown
# [Emoji] Tên Module — Tiêu đề mô tả (tiếng Việt)

> Quote ngắn mô tả triết lý/mục tiêu

---

## 📂 Cấu Trúc Thư Mục
| File | Nội dung |  ← Bảng mapping file → nội dung

## 1. Khái niệm chính

## 2. So sánh / Decision Matrix
| Tiêu chí | Option A | Option B |  ← Bảng so sánh

## 3. Nội dung sâu (mermaid diagrams, code examples)
```

### Cấu trúc nội dung — Mỗi Data Structure file

```markdown
# [Emoji] Tên Data Structure

## 🧠 Định nghĩa      ← Giải thích như nói với người ngoài ngành
## ⏱️ Big-O Table     ← Bảng complexity đầy đủ
## 💻 Code JS         ← Implementation code chạy được, có comment
## 🔧 Trong Framework ← React/Vue/Angular/Node dùng ở đâu
## 🕵️ Kỹ thuật ẩn    ← Điều ít người nhận ra
## ❓ Bẫy phỏng vấn   ← Câu hỏi mẹo
```

### Cấu trúc nội dung — React Hooks files

```markdown
# [Emoji] Tên Hook Group

## Giới thiệu hook
## Cú pháp
## Use cases (kèm code example)
## Anti-patterns / Lỗi phổ biến
## So sánh với hooks liên quan
## Câu hỏi phỏng vấn
```

### Cấu trúc nội dung — interviews.md

```markdown
# Q&A Phỏng vấn [Chủ đề]

## Level phân cấp (Junior / Middle / Senior)

**Q:** Câu hỏi?
**A:** Trả lời đầy đủ

## Tình huống thiết kế thực tế (dành cho Senior)
```

### Quy ước Markdown
- **Emoji** ở đầu heading chính: 📁 🔵 🟢 🟡 🔴 🟣 🎯 🧠 ⚡ 🚀 ⭐
- **Bold** cho từ khóa quan trọng lần đầu xuất hiện
- **Code block** với language tag: ` ```javascript `, ` ```mermaid `, ` ```typescript `
- **Bảng Markdown** cho comparison, decision matrix, Big-O table
- **Mermaid diagrams** cho architecture diagrams và relationship maps
- Văn bản **tiếng Việt** là chủ đạo, thuật ngữ kỹ thuật giữ nguyên tiếng Anh
- Dùng `>` blockquote cho định nghĩa ngắn gọn ở đầu file

---

## 5. TRẠNG THÁI HOÀN THÀNH TỪNG MODULE

| Module | Thư mục | Trạng thái | Ghi chú |
|--------|---------|-----------|---------|
| JavaScript Core | `01-javascript/` | 🟡 Một phần | `concepts/` 4 file, `interviews/` 4 file; phần còn lại trống |
| Styling | `02-styling/` | 🔴 Chưa có nội dung | Cấu trúc thư mục tạo sẵn |
| React | `03-react/` | 🟢 Đầy đủ | hooks-patterns (5 file) + component-patterns (6 file) + performance (5 file) |
| Angular | `04-angular/` | 🔴 Chưa có nội dung | Chỉ có README placeholder |
| Vue | `05-vue/` | 🔴 Chưa có nội dung | Chỉ có README placeholder |
| Coding Challenges | `06-coding-challenges/` | 🔴 Chưa có nội dung | Thư mục tạo sẵn, trống |
| Micro Frontends | `07-micro-frontends/` | 🟢 Đầy đủ | 5 file, nội dung phong phú |
| Design Patterns | `08-design-patterns/` | 🟢 Đầy đủ | 6 file, C# focus, SOLID + GoF đầy đủ |
| Data Structures | `09-data-structures/` | 🟢 Đầy đủ | linear, hash, tree, graph, và modern đã hoàn thành 100% |

---

## 6. LIÊN KẾT CHÉO GIỮA CÁC MODULE

```
01-javascript  ←→  09-data-structures    # JS Array/Map/Set là built-in DS
08-design-patterns  →  09-data-structures # Observer dùng Queue, Decorator dùng Stack
09-data-structures  →  06-coding-challenges # Bài tập thực hành DS
03-react  ←→  09-data-structures         # React Fiber dùng DAG, useState/undo dùng Stack
07-micro-frontends  →  03-react           # Module Federation với React
```

---

## 7. LỊCH SỬ PHÁT TRIỂN

- **2026-06-28**: Tạo module `09-data-structures/` — phân loại theo nhóm chức năng (Linear, Hash, Tree, Graph, Modern), template file chuẩn, roadmap 4 tuần. Đã hoàn thành: `01-linear/` (5 DS: Array, Linked List, Stack, Queue, Deque), `02-hash-based/hash-map.md`.
- **2026-06-28**: Hoàn thành `02-hash-based/` (HashSet, Bloom Filter) và `03-tree/` (Binary Tree, BST, Heap, Trie, Segment Tree).
- **2026-06-28**: Hoàn thành các cấu trúc còn lại: `04-graph/` (Graph, DAG), `05-modern/` (LRU Cache, Skip List, Persistent DS, Rope, Disjoint-Set, HyperLogLog, Count-Min Sketch) và file phỏng vấn `interviews/qna.md`.
- **2026-06-28**: Bổ sung chủ đề Component Design Patterns vào module React (`03-react/component-patterns/`): Compound Component, Render Props, Controlled/Uncontrolled, HOC, State Reducer & Control Props, cùng tài liệu phỏng vấn chi tiết.

---

## 8. GHI CHÚ QUAN TRỌNG CHO CUỘC HỘI THOẠI MỚI

Khi tiếp tục làm việc với repo này, agent cần biết:

1. **Ngôn ngữ chính**: Tiếng Việt cho giải thích, tiếng Anh cho thuật ngữ kỹ thuật
2. **Level đối tượng**: Middle → Senior frontend developer
3. **File template DS**: Xem mục 4 — "Cấu trúc nội dung — Mỗi Data Structure file"
4. **Việc còn thiếu trong 09-data-structures**: Đã hoàn thành toàn bộ 100%.
5. **Conventions bắt buộc**: Kebab-case tên file, emoji ở heading, Mermaid cho diagrams, bảng Big-O, code JS chạy được có comment, section "🔧 Trong Framework" cho mỗi DS
6. **Path repo**: `c:\Users\t14\Documents\frontend-mastery-hub\`
7. **GitHub**: `hoai97nam/frontend-mastery-hub`
