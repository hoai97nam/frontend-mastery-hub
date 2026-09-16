# 🔑 Senior Keywords — Modern & Advanced Data Structures

> Từ khóa về các Cấu trúc dữ liệu hiện đại và chuyên sâu.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Caching DS | `LRU Cache (HashMap + Doubly Linked List)` | Cấu trúc dữ liệu Cache loại bỏ phần tử lâu nhất không dùng trong O(1) nhờ kết hợp HashMap và Doubly Linked List | `LRU Cache HashMap Doubly Linked List O1 implementation` |
| Disjoint Set | `Union-Find with Path Compression` | Cấu trúc dữ liệu quản lý các tập hợp rời rạc với thuật toán nén đường đi (Path Compression) và gộp theo cấp (Union by Rank) đạt độ phức tạp O(alpha(N)) gần như hằng số | `Disjoint Set Union Find Path Compression Union by Rank` |
| Skip List | `Skip List Probabilistic Towers` | Thay thế Cây cân bằng bằng danh sách liên kết nhiều tầng ngẫu nhiên hỗ trợ tìm kiếm O(log n) đơn giản (dùng trong Redis Sorted Set) | `Skip List probabilistic index Redis` |
| Immutable DS | `Persistent Data Structure Structural Sharing` | Cấu trúc dữ liệu bất biến lưu lại các phiên bản lịch sử bằng cách chia sẻ lại các nhánh cây không bị sửa đổi (Path Copying) | `Persistent Data Structure Structural Sharing Path Copying` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **HyperLogLog Cardinality Estimation**: Cấu trúc dữ liệu xác suất đếm số phần tử duy nhất (Distinct Count) của hàng tỷ user với dung lượng chỉ vài KB.
- **Rope Data Structure**: Cây nhị phân quản lý chuỗi văn bản cực lớn (dùng trong Text Editors) cho phép chèn/xóa O(log n).

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Path Copying in Persistent DS**: Chỉ tạo node mới trên đường đi từ root đến node bị thay đổi, giữ nguyên các nhánh còn lại.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Implement LRU Cache in Interview**: Viết class LRU Cache đầy đủ phương thức `get` và `put` trong 15 phút phỏng vấn.

