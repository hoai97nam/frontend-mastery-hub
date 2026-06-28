# 🔴 Graph Data Structures — Đồ thị

> **Đồ thị** = mạng lưới bao gồm các **đỉnh (vertices/nodes)** và các **cạnh (edges)** kết nối chúng. Khác với cây, đồ thị không có khái niệm cha-con rõ ràng và có thể chứa các vòng lặp (cycles).

---

## Các cấu trúc trong nhóm

| DS | Biểu diễn phổ biến | Tìm kiếm / Duyệt | Khi nào dùng |
|----|--------------------|------------------|--------------|
| [Graph](./graph.md) | Adjacency List, Adjacency Matrix | BFS, DFS | Mạng xã hội, Bản đồ đường đi, Mạng lưới quan hệ |
| [DAG (Directed Acyclic Graph)](./dag.md) ⭐ | Adjacency List (không vòng có hướng) | Topological Sort | Quản lý dependency (Webpack, Yarn), Luồng xử lý công việc (Airflow, Git commits), React Fiber Tree |

---

## Cách biểu diễn Đồ thị

```
Đồ thị mẫu:
 (A) ─── (B)
  │       │
  │       │
 (C) ─── (D)
```

### 1. Adjacency Matrix (Ma trận kề)
Dùng mảng 2 chiều kích thước `V x V` để biểu diễn các kết nối.
```javascript
const matrix = [
  [0, 1, 1, 0], // A kết nối với B, C
  [1, 0, 0, 1], // B kết nối với A, D
  [1, 0, 0, 1], // C kết nối với A, D
  [0, 1, 1, 0]  // D kết nối với B, C
];
```
- **Ưu điểm**: Kiểm tra cạnh `(u, v)` có tồn tại hay không trong `O(1)`.
- **Nhược điểm**: Tốn bộ nhớ `O(V²)` bất kể số cạnh nhiều hay ít (Sparse graph).

### 2. Adjacency List (Danh sách kề) — Phổ biến nhất
Mỗi đỉnh liên kết với một danh sách chứa các đỉnh kề của nó.
```javascript
const graph = {
  A: ['B', 'C'],
  B: ['A', 'D'],
  C: ['A', 'D'],
  D: ['B', 'C']
};
```
- **Ưu điểm**: Tiết kiệm bộ nhớ `O(V + E)`. Rất tối ưu khi duyệt các đỉnh lân cận.
- **Nhược điểm**: Kiểm tra cạnh `(u, v)` mất `O(degree(u))` thay vì `O(1)`.

---

## 2 thuật toán duyệt đồ thị cơ bản

### 1. DFS (Depth-First Search) — Duyệt theo chiều sâu
Duyệt sâu nhất có thể dọc theo mỗi nhánh trước khi quay lui. Dùng **Stack** (hoặc Đệ quy).
- **Ứng dụng**: Tìm chu trình (cycle detection), Topological Sort, giải mê cung.

### 2. BFS (Breadth-First Search) — Duyệt theo chiều rộng
Duyệt tất cả các đỉnh lân cận ở mức hiện tại trước khi chuyển sang mức tiếp theo. Dùng **Queue**.
- **Ứng dụng**: Tìm đường đi ngắn nhất (shortest path) trên đồ thị không trọng số, kiểm tra tính liên thông.
