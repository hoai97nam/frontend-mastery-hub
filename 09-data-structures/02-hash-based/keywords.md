# 🔑 Senior Keywords — Hash-Based Data Structures & Probabilistic DS

> Từ khóa về Bảng băm và Cấu trúc dữ liệu xác suất.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Hash Mechanism | `Load Factor & Rehashing` | Tỷ lệ số phần tử / số bucket (`alpha`), khi vượt quá ngưỡng (thường là 0.75) HashMap sẽ cấp phát bảng mới gấp đôi và rehash lại toàn bộ phần tử | `HashMap Load Factor Rehashing performance` |
| Collision Resolution | `Separate Chaining vs Open Addressing` | Giải quyết đụng độ băm bằng Linked List/Red-Black Tree tại bucket (Chaining) hoặc tìm ô trống tiếp theo (Linear Probing) | `HashMap Collision Resolution Chaining vs Open Addressing` |
| Probabilistic DS | `Bloom Filter Bit Array` | Cấu trúc dữ liệu xác suất tiết kiệm bộ nhớ dùng k hàm băm để kiểm tra sự tồn tại phần tử (Chắc chắn KHÔNG CÓ hoặc CÓ THỂ CÓ) | `Bloom Filter probabilistic data structure false positive rate` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Hash Function Uniformity**: Hàm băm phân phối đều giá trị để giảm thiểu tỉ lệ đụng độ.
- **Java 8 HashMap Treeify**: Chuyển đổi Linked List thành Red-Black Tree khi số đụng độ trong 1 bucket >= 8 để giữ tìm kiếm O(log n).

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Bloom Filter False Positive Control**: Điều chỉnh kích thước Bit Array `m` và số hàm băm `k` để khống chế tỉ lệ báo sai dưới 1%.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Bloom Filter Use Cases in Web**: Dùng Bloom Filter kiểm tra username trùng lặp hoặc chặn URL độc hại trước khi truy vấn Database.

