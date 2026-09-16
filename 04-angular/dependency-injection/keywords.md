# 🔑 Senior Keywords — Angular Dependency Injection & Hierarchical Injector

> Từ khóa về cơ chế DI chuyên sâu của Angular.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| DI Architecture | `Hierarchical Injector Tree` | Cấu trúc cây Injector phân cấp: ElementInjector -> EnvironmentInjector -> Root Injector | `Angular Hierarchical Injector Tree resolution` |
| DI Token | `InjectionToken & Tree-shakable Providers` | Tạo token duy nhất cho interface/primitive value và khai báo `providedIn: 'root'` để hỗ trợ tree-shaking | `Angular InjectionToken providedIn root` |
| Resolution Modifier | `@SkipSelf & @Host Modifiers` | Các decorator điều chỉnh hướng tìm kiếm dependency trong cây Injector (`@Optional`, `@Self`, `@SkipSelf`, `@Host`) | `Angular DI resolution modifiers SkipSelf Host` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Injector Resolution Algorithm**: Thuật toán tìm kiếm dependency từ ElementInjector hiện tại ngược lên cha cho tới Root Injector.
- **Multi-provider Tokens**: Đăng ký nhiều provider cho cùng 1 `InjectionToken` (`multi: true`) phục vụ plugin system (như `HTTP_INTERCEPTORS`).

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Tree-shakable Services**: Tránh khai báo service trong `providers` array của NgModule để compiler xóa service nếu không dùng.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Singletons in Lazy Loaded Modules**: Bẫy tạo nhiều instance service khác nhau khi provider service trong Lazy Loaded Modules thay vì `root`.

