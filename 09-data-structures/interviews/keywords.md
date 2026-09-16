# 🔑 Senior Keywords — Data Structures Interview Strategy & Trade-offs

> Từ khóa tổng hợp chiến lược phỏng vấn Data Structures.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Analysis Framework | `Big-O / Big-Omega / Big-Theta` | Phân tích độ phức tạp thuật toán trong trường hợp xấu nhất (Big-O), tốt nhất (Big-Omega) và trung bình (Big-Theta) | `Algorithm complexity Big O Big Theta Big Omega` |
| Performance Factor | `CPU Cache Locality Impact` | Ảnh hưởng của bộ nhớ đệm CPU đến hiệu năng thực tế của cấu trúc dữ liệu (Array nạp đệm vượt trội hơn Linked List) | `CPU Cache Locality Data Structures performance` |
| Trade-off | `In-place vs Out-of-place Trade-off` | Đánh đổi giữa giải pháp xử lý trực tiếp trên mảng gốc O(1) memory vs tạo mảng mới O(n) memory để giữ immutability | `In-place vs Out-of-place Algorithm Space Trade-off` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Amortized Time Complexity**: Độ phức tạp trung bình tích lũy qua chuỗi N thao tác (ví dụ push vào mảng động).
- **Space Complexity Overhead**: Tính toán dung lượng bộ nhớ phụ trợ (Auxiliary Space) bao gồm Stack đệ quy.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Choosing the Right Data Structure**: Ma trận quyết định chọn DS phù hợp với tần suất đọc/ghi/tìm kiếm của bài toán.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Why JS Arrays are not pure C-arrays**: Giải thích V8 Array có thể chuyển đổi linh hoạt giữa Fast Elements (mảng đệm) và Dictionary Elements (Sparse Array).

