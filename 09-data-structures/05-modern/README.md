# 🟣 Modern Data Structures — Cấu trúc dữ liệu hiện đại & Nâng cao

> **Modern Data Structures** = các cấu trúc dữ liệu được sinh ra để giải quyết các bài toán quy mô lớn (Scale), tối ưu hóa phần cứng hiện đại hoặc đảm bảo tính bất biến (Immutability).

Mặc dù ít được giảng dạy chính quy, chúng lại là xương sống của các công cụ frontend/backend hiện đại như Redux, VS Code, Redis, Next.js cache.

---

## Các cấu trúc trong nhóm

| File | Data Structure | Ứng dụng thực tế nổi tiếng | Sức mạnh chính |
|------|---------------|----------------------------|----------------|
| [lru-cache.md](./lru-cache.md) | LRU Cache | Next.js Fetch Cache, Memcached | Giới hạn dung lượng bộ nhớ cache, tự động giải phóng phần tử cũ nhất trong O(1). |
| [skip-list.md](./skip-list.md) | Skip List | Redis Sorted Sets, LevelDB | Thay thế cây cân bằng (Red-Black tree) với cấu trúc danh sách liên kết đa tầng dễ lập trình hơn, tìm kiếm trong O(log n). |
| [persistent-data-structure.md](./persistent-data-structure.md) | Persistent DS | Redux, Immutable.js | Tạo bản sao mới (immutable update) cực nhanh bằng cách chia sẻ cấu trúc (structural sharing), tránh deep clone tốn kém. |
| [rope.md](./rope.md) | Rope | VS Code, Monaco Editor | Xử lý, chèn, xóa trên các chuỗi văn bản khổng lồ (hàng triệu dòng) trong O(log n) thay vì O(n) của String truyền thống. |
| [disjoint-set.md](./disjoint-set.md) | Disjoint-Set (Union-Find) | Kruskal's MST, React Fiber reconciliation | Nhóm các phần tử vào tập hợp không giao nhau, kiểm tra liên kết giữa hai phần tử gần như trong O(1). |
| [probabilistic.md](./probabilistic.md) | HyperLogLog & Count-Min Sketch | Redis analytics, CDN rate limiting | Đếm số lượng phần tử độc nhất (count-distinct) hoặc tần suất xuất hiện với bộ nhớ cố định cực nhỏ, chấp nhận sai số nhỏ. |

---

## Tại sao chúng ta cần chúng?

```
Văn bản 10MB:   Dùng String bình thường -> Chèn 1 ký tự ở đầu -> Trình duyệt treo O(n)
                 Dùng Rope -> Chỉ cập nhật vài liên kết cây -> Ổn định O(log n)

Dữ liệu lớn:    HashSet đếm 1 tỷ user độc nhất -> Tốn 40GB RAM
                 HyperLogLog đếm xấp xỉ -> Chỉ tốn 12KB RAM với sai số 1%
```
