# 🔑 Senior Keywords — Algorithmic Patterns & Problem Solving

> Từ khóa các dạng bài tập thuật toán kinh điển dành cho Frontend Senior.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Algorithmic Pattern | `Sliding Window Pattern` | Kỹ thuật duy trì một cửa sổ (cố định hoặc linh hoạt) trên mảng/chuỗi để giải bài toán subarray với độ phức tạp O(n) | `Sliding Window Algorithm Pattern subarray` |
| Algorithmic Pattern | `Monotonic Stack / Queue` | Stack/Queue luôn duy trì thứ tự tăng hoặc giảm dần để tìm phần tử lớn hơn/nhỏ hơn gần nhất trong O(n) | `Monotonic Stack Monotonic Queue next greater element` |
| Dynamic Programming | `Memoization vs Tabulation` | Hai phương pháp DP: Top-down (đệ quy có nhớ) vs Bottom-up (lập bảng tính toán từ cơ sở) | `Dynamic Programming Memoization vs Tabulation` |
| Graph Algorithm | `Topological Sort (Kahn's Algorithm)` | Sắp xếp thứ tự phụ thuộc của các đỉnh trong đồ thị có hướng không chu trình (DAG) bằng BFS/Queue | `Topological Sort Kahns Algorithm DAG dependency` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Two Pointers Mechanics**: Dùng 2 con trỏ duy chuyển từ 2 đầu hoặc cùng chiều để giảm độ phức tạp từ O(n^2) xuống O(n).
- **Fast & Slow Pointers (Floyd's Cycle)**: Phát hiện chu trình trong Linked List bằng 2 con trỏ rùa và thỏ.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **In-place Array Mutation**: Tối ưu dung lượng bộ nhớ O(1) space bằng cách hoán đổi trực tiếp trên mảng đầu vào.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Time & Space Complexity Trade-off**: Phân tích khi nào nên đánh đổi bộ nhớ (dùng Hash Map O(n) space) lấy thời gian thực thi O(n) time.

