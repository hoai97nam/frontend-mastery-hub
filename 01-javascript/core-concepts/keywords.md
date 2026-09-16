# 🔑 Senior Keywords — JavaScript Fundamental Concepts

> Từ khóa nền tảng sâu sắc về Prototype, Scope và Execution Model.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| OOP | `Prototype Chain Resolution` | Cơ chế tra cứu thuộc tính ngược lên cây prototype cho tới khi gặp `Object.prototype.__proto__ === null` | `JavaScript Prototype Chain Property Lookup` |
| Scope | `Variable Environment vs Lexical Environment` | Phân biệt nơi lưu trữ biến `var`/function declaration và biến `let`/`const` trong Execution Context | `ECMAScript Variable Environment Lexical Environment` |
| Execution Context | `Implicit `this` Binding` | Quy tắc xác định `this` dựa trên caller tại thời điểm gọi hàm (Call-site evaluation) | `JavaScript explicit implicit binding rules` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Object.create(null)**: Tạo đối tượng thuần túy không kế thừa bất kỳ thuộc tính nào từ `Object.prototype`, thích hợp làm Safe Map Data Structure.
- **Lexical Arrow Function Binding**: Hàm mũi tên không có `this`, `arguments`, `super` hay `new.target` riêng mà bắt cố định từ scope cha.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Prototypal Inheritance vs Class Inheritance**: Tiết kiệm bộ nhớ bằng cách chia sẻ phương thức trên prototype thay vì nhân bản trong mỗi instance.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Hard Binding with `bind`**: Cơ chế tạo wrapper function cố định `this` và currying tham số.

