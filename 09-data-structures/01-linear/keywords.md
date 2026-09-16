# 🔑 Senior Keywords — Linear Data Structures & Memory Layout

> Từ khóa về các cấu trúc dữ liệu tuyến tính.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Memory Layout | `Contiguous Memory & Dynamic Array` | Mảng lưu trữ trên ô nhớ liên tục hỗ trợ truy xuất O(1) theo index, tự động gấp đôi dung lượng (Amortized O(1) insertion) | `Contiguous memory dynamic array amortized analysis` |
| Node-based DS | `Doubly Linked List` | Danh sách liên kết đôi hỗ trợ thêm/xóa O(1) ở hai đầu nhưng tốn dung lượng lưu con trỏ `prev`/`next` | `Doubly Linked List memory overhead pointer` |
| LIFO / FIFO | `Circular Ring Buffer Queue` | Hàng đợi vòng lặp sử dụng mảng tĩnh cố định với con trỏ `head`/`tail` tối ưu bộ nhớ không cần dịch chuyển phần tử | `Circular Ring Buffer Queue array implementation` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Cache Locality Advantage**: Array có cache locality cực cao giúp CPU L1/L2 Cache nạp dữ liệu nhanh hơn hẳn Linked List.
- **Deque (Double-Ended Queue)**: Cấu trúc dữ liệu cho phép Push/Pop O(1) ở cả 2 đầu.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Array Resizing Overhead**: Chi phí copy toàn bộ mảng cũ sang mảng mới khi vượt quá dung lượng allocated.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Stack vs Queue in JS Engine**: Call Stack xử lý LIFO cho hàm thực thi, Task Queue xử lý FIFO cho sự kiện.

