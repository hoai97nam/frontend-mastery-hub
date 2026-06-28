# 🗂️ Data Structures — Frontend Mastery Hub

> **Triết lý**: Hiểu đúng data structure = viết code nhanh hơn, debug ít hơn, phỏng vấn tự tin hơn.

Không chỉ học "cho biết" — module này giúp bạn **nhận ra** data structure đang ẩn trong React hooks, Webpack bundle, hay Redis cache của mình.

---

## 🗺️ Roadmap học tập

```
Tuần 1: Linear (Array → Linked List → Stack → Queue → Deque)
Tuần 2: Hash-based (HashMap → HashSet → Bloom Filter)
Tuần 3: Tree (Binary Tree → BST → Heap → Trie)
Tuần 4: Graph (Graph → DAG) + Modern (LRU Cache → Skip List → ...)
```

---

## 📁 Cấu trúc thư mục

```
09-data-structures/
├── 01-linear/          🔵 Tuyến tính — nền tảng
├── 02-hash-based/      🟢 Hash — tra cứu O(1)
├── 03-tree/            🟡 Cây — phân cấp & tìm kiếm
├── 04-graph/           🔴 Đồ thị — quan hệ phức tạp
├── 05-modern/          🟣 Hiện đại — ít biết nhưng hay gặp
└── interviews/         🎯 QnA phỏng vấn tổng hợp
```

---

## 🔵 Nhóm 1: Linear — Tuyến tính

> Dữ liệu xếp **theo hàng** — mỗi phần tử có vị trí rõ ràng

| File | Data Structure | Một dòng mô tả |
|------|---------------|----------------|
| [array.md](./01-linear/array.md) | Array | Dãy ô nhớ liên tiếp — truy cập O(1) theo index |
| [linked-list.md](./01-linear/linked-list.md) | Linked List | Chuỗi nút, mỗi nút trỏ đến nút tiếp theo |
| [stack.md](./01-linear/stack.md) | Stack | Chồng đĩa — LIFO, cái vào sau ra trước |
| [queue.md](./01-linear/queue.md) | Queue | Hàng đợi — FIFO, cái vào trước ra trước |
| [deque.md](./01-linear/deque.md) | Deque | Queue 2 đầu — vào/ra ở cả hai phía |

---

## 🟢 Nhóm 2: Hash-based — Tra cứu O(1)

> Ánh xạ **key → value** với tốc độ cực nhanh

| File | Data Structure | Một dòng mô tả |
|------|---------------|----------------|
| [hash-map.md](./02-hash-based/hash-map.md) | HashMap | Từ điển — tìm theo tên thay vì số thứ tự |
| [hash-set.md](./02-hash-based/hash-set.md) | HashSet | Tập hợp không trùng lặp |
| [bloom-filter.md](./02-hash-based/bloom-filter.md) | Bloom Filter ⭐ | "Có thể có" — tiết kiệm bộ nhớ kiểm tra tồn tại |

---

## 🟡 Nhóm 3: Tree — Cấu trúc phân cấp

> Dữ liệu có **quan hệ cha-con** — tìm kiếm hiệu quả

| File | Data Structure | Một dòng mô tả |
|------|---------------|----------------|
| [binary-tree.md](./03-tree/binary-tree.md) | Binary Tree | Mỗi nút tối đa 2 con |
| [binary-search-tree.md](./03-tree/binary-search-tree.md) | BST | Cây tìm kiếm có thứ tự — O(log n) |
| [heap.md](./03-tree/heap.md) | Heap | Luôn trả về min/max nhanh nhất |
| [trie.md](./03-tree/trie.md) | Trie ⭐ | Cây tiền tố — autocomplete, spell check |
| [segment-tree.md](./03-tree/segment-tree.md) | Segment Tree | Truy vấn khoảng — sum/min/max range |
| [red-black-tree.md](./03-tree/red-black-tree.md) | Red-Black Tree | Tự cân bằng — ẩn trong JS Map, TreeMap |

---

## 🔴 Nhóm 4: Graph — Quan hệ phức tạp

> Dữ liệu có **nhiều quan hệ** không phải cha-con đơn giản

| File | Data Structure | Một dòng mô tả |
|------|---------------|----------------|
| [graph.md](./04-graph/graph.md) | Graph | Mạng lưới node và edge |
| [dag.md](./04-graph/dag.md) | DAG ⭐ | Đồ thị không vòng có hướng — Webpack, React Fiber |

---

## 🟣 Nhóm 5: Modern — Hiện đại & Ít phổ biến

> Xuất hiện trong **hệ thống thực tế** nhưng hiếm khi được dạy

| File | Data Structure | Một dòng mô tả |
|------|---------------|----------------|
| [lru-cache.md](./05-modern/lru-cache.md) | LRU Cache ⭐ | Giữ dữ liệu thường dùng, bỏ dữ liệu cũ |
| [skip-list.md](./05-modern/skip-list.md) | Skip List | Linked list nhiều tầng — Redis dùng |
| [persistent-data-structure.md](./05-modern/persistent-data-structure.md) | Persistent DS ⭐ | Immutable nhưng vẫn hiệu quả — Redux |
| [rope.md](./05-modern/rope.md) | Rope | Chuỗi khổng lồ — VS Code, Monaco Editor |
| [disjoint-set.md](./05-modern/disjoint-set.md) | Union-Find | Nhóm kết nối — Kruskal, network topology |
| [probabilistic.md](./05-modern/probabilistic.md) | HyperLogLog / Count-Min Sketch ⭐ | Đếm xấp xỉ với bộ nhớ cực nhỏ |

---

## 🎯 Phỏng vấn

| File | Nội dung |
|------|---------|
| [interviews/qna.md](./interviews/qna.md) | 40+ câu hỏi phỏng vấn thực tế, chia theo level |

---

## 📖 Template mỗi file

Mỗi data structure file đều có:
- 🧠 **Định nghĩa** — giải thích như nói với người ngoài ngành
- ⏱️ **Big-O Table** — complexity rõ ràng
- 💻 **Code JavaScript** — chạy được, có comment
- 🔧 **Trong Framework** — React/Vue/Angular/Node dùng ở đâu
- 🕵️ **Kỹ thuật ẩn** — điều ít người nhận ra
- ❓ **Bẫy phỏng vấn** — câu hỏi mẹo

---

## 🔗 Kết nối với các module khác

- **[08-design-patterns](../08-design-patterns/)** — Observer Pattern dùng Queue, Decorator dùng Stack
- **[01-javascript](../01-javascript/)** — JS Array/Map/Set là built-in DS
- **[06-coding-challenges](../06-coding-challenges/)** — Bài tập thực hành DS
