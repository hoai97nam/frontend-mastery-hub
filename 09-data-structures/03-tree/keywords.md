# 🔑 Senior Keywords — Tree Data Structures & Priority Queues

> Từ khóa về Cây nhị phân, Heap và Trie.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Binary Tree | `Binary Search Tree (BST) Balancing` | Cây tìm kiếm nhị phân duy trì tính chất node trái < node cha < node phải, yêu cầu cân bằng (AVL/Red-Black) để tránh suy biến thành O(n) | `Binary Search Tree BST balancing AVL Red Black Tree` |
| Priority Queue | `Binary Min/Max Heap` | Cây nhị phân gần hoàn chỉnh biểu diễn dưới dạng mảng (`parent = (i-1)/2`), lấy min/max O(1) và Heapify O(log n) | `Binary Min Max Heap array representation Priority Queue` |
| Prefix Tree | `Trie (Prefix Tree)` | Cây tiền tố tối ưu cho bài toán gõ từ gợi ý (Autocomplete), tìm kiếm chuỗi độ dài k trong O(k) | `Trie Prefix Tree autocomplete search` |
| Range Query | `Segment Tree Lazy Propagation` | Cây quản lý khoảng hỗ trợ truy vấn (Range Sum/Min/Max) và cập nhật khoảng O(log n) nhờ kỹ thuật trì hoãn tính toán (Lazy Propagation) | `Segment Tree Lazy Propagation range queries` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Heapify Algorithm**: Quá trình vun đống lại cây từ dưới lên với độ phức tạp O(n) để tạo Heap ban đầu.
- **Trie Node Array vs Map**: Lưu con trỏ con trong Trie Node bằng mảng cố định 26 ký tự hoặc HashMap.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Segment Tree Array Allocation**: Cấp phát mảng kích thước `4 * N` để biểu diễn Segment Tree an toàn.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Top K Frequent Elements**: Dùng Min Heap kích thước K để giải bài toán Top K phần tử lớn nhất với thời gian O(n log k).

