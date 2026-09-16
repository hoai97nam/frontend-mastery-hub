# 🔑 Senior Keywords — Graph Data Structures & DAG Algorithms

> Từ khóa về Đồ thị và Đồ thị có hướng không chu trình.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Graph Representation | `Adjacency Matrix vs Adjacency List` | Biểu diễn đồ thị bằng ma trận vuông O(V^2) space tra cứu O(1) vs danh sách kề O(V+E) space tối ưu bộ nhớ | `Graph Adjacency Matrix vs Adjacency List space complexity` |
| Graph Traversal | `BFS Shortest Path & DFS Backtracking` | Duyệt theo chiều rộng (BFS) tìm đường đi ngắn nhất trên đồ thị không trọng số vs Duyệt theo chiều sâu (DFS) phục vụ tìm kiếm quay lui | `BFS Shortest Path DFS Backtracking Graph` |
| DAG Architecture | `Directed Acyclic Graph (DAG)` | Đồ thị có hướng không chu trình là mô hình cốt lõi của các hệ thống Build Tool (Webpack dependency graph, Git commits) | `Directed Acyclic Graph DAG build system dependency` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Cycle Detection in Graph**: Phát hiện chu trình bằng DFS (dùng 3 màu Unvisited/Visiting/Visited) hoặc Kahn's Algorithm.
- **Topological Sorting Order**: Thứ tự tuyến tính các đỉnh sao cho với mọi cạnh u -> v, u luôn đứng trước v.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Sparse Graph Optimization**: Luôn ưu tiên Adjacency List cho đồ thị thưa (Sparse Graph) để tiết kiệm tài nguyên.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Module Dependency Circular Reference**: Cách Webpack phát hiện và xử lý lỗi circular dependency trong đồ thị module.

