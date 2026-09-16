# 🔑 Senior Keywords — Design Patterns & Software Architecture (C# / TS)

> Từ khóa về SOLID principles và 23 Gang of Four Design Patterns.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Architecture Principle | `SOLID Principles` | 5 nguyên lý hướng đối tượng: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion | `SOLID Principles Software Architecture` |
| Creational Pattern | `Abstract Factory & Builder` | Khởi tạo họ đối tượng liên quan (Abstract Factory) hoặc xây dựng đối tượng phức tạp từng bước (Builder) | `Abstract Factory Builder Pattern GoF` |
| Structural Pattern | `Decorator & Adapter & Facade` | Bọc đối tượng bổ sung chức năng mới (Decorator), chuyển đổi interface tương thích (Adapter), cung cấp interface đơn giản hóa cho hệ thống lớn (Facade) | `Decorator Adapter Facade Design Patterns` |
| Behavioral Pattern | `Strategy & Observer & Command` | Đóng gói thuật toán linh hoạt (Strategy), mô hình đăng ký/thông báo thay đổi (Observer), biến request thành đối tượng độc lập (Command) | `Strategy Observer Command Design Patterns` |
| Enterprise Pattern | `Repository & Unit of Work` | Tách biệt lớp truy cập dữ liệu (Repository) và quản lý giao dịch lưu thay đổi hàng loạt (Unit of Work) | `Repository Unit of Work Pattern Enterprise Architecture` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Dependency Inversion vs Dependency Injection**: Principle phụ thuộc vào Abstraction không phụ thuộc Detail (DIP) được thực thi bởi kỹ thuật Inject dependencies từ bên ngoài (DI).
- **Specification Pattern**: Đóng gói các quy tắc kinh doanh (Business Rules) thành các đối tượng có thể kết hợp bằng phép logic AND/OR.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Flyweight Pattern**: Chia sẻ trạng thái chung (Intrinsic State) giữa hàng ngàn đối tượng để tiết kiệm bộ nhớ RAM.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Singleton Pattern Thread Safety & Anti-pattern Trap**: Tại sao Singleton bị coi là Anti-pattern nếu lạm dụng (gây ra Global State, khó Unit Test) và cách triển khai Thread-safe trong C#/TS.

