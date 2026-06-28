# 🧩 React Component Design Patterns

> Tổng hợp các mẫu thiết kế (Design Patterns) phổ biến và nâng cao cho React Component, giúp chia sẻ state, tái sử dụng logic UI, và tăng khả năng mở rộng (scalability) của hệ thống Frontend.

---

## 📂 Cấu Trúc Thư Mục

| File | Pattern | Độ khó / Level | Mục đích chính |
|------|---------|----------------|----------------|
| 📦 [01-compound-component.md](./01-compound-component.md) | **Compound Component** | ⭐⭐ Mid | Quản lý state ngầm và xây dựng giao diện có tính phân cấp cao (Tabs, Accordion) |
| 📦 [02-render-props.md](./02-render-props.md) | **Render Props & Children as a Function** | ⭐⭐ Mid | Chia sẻ logic xử lý UI và cung cấp quyền tùy biến cấu trúc render cho component cha |
| 📦 [03-controlled-uncontrolled.md](./03-controlled-uncontrolled.md) | **Controlled & Uncontrolled** | ⭐ Mid | Quản lý nguồn dữ liệu (state vs ref) trong form và các tương tác UI |
| 📦 [04-hoc.md](./04-hoc.md) | **Higher-Order Component (HOC)** | ⭐⭐ Mid | Kế thừa logic UI thông qua composition (Authentication guard, loading wrapper) |
| 📦 [05-state-reducer-control-props.md](./05-state-reducer-control-props.md) | **State Reducer & Control Props** | ⭐⭐⭐ Senior | Cung cấp khả năng can thiệp sâu vào state nội bộ và kiểm soát từ xa (Hybrid state) |
| 🧠 [interviews.md](./interviews.md) | **Q&A & Scenarios** | ⭐⭐⭐ Senior | Tổng hợp câu hỏi phỏng vấn hóc búa và tình huống thiết kế hệ thống thực tế |

---

## 🎯 Bảng Chọn Lựa Pattern (Decision Matrix)

Để lựa chọn pattern phù hợp nhất cho bài toán của bạn, hãy đối chiếu với bảng ma trận quyết định dưới đây:

| Pattern | Chia sẻ State cục bộ? | Tái sử dụng Logic? | Khả năng Custom UI | Độ phức tạp khi code | Thay thế tốt nhất hiện nay |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Compound Component** | ✅ Có (qua Context) | ❌ Không | ⭐⭐⭐ Cao | 🛠️ Trung bình | Vẫn là tiêu chuẩn cho UI Component nhóm (Dropdown, Select, Menu) |
| **Render Props** | ✅ Có | ✅ Có | ⭐⭐⭐ Cao | 🛠️ Trung bình | Custom Hooks (đối với logic không cần render UI) |
| **Controlled/Uncontrolled** | ✅ Có (hoặc qua Ref) | ❌ Không | ⭐⭐ Trung bình | 🟢 Thấp | Phối hợp cả hai để tạo linh hoạt (Control Props) |
| **HOC** | ✅ Có (qua props) | ✅ Có | ⭐ Thấp | 🛠️ Trung bình | Custom Hooks / Component Composition |
| **State Reducer** | ✅ Có | ✅ Có | ⭐⭐⭐ Cao | 🔴 Cao | Custom hooks trả về dispatch custom |
| **Control Props** | ✅ Có (qua props) | ✅ Có | ⭐⭐⭐ Cao | 🔴 Cao | useState + useEffect đồng bộ |

---

## 🗺️ Lộ Trình Học Đề Xuất

Khi nghiên cứu các React Component Patterns, bạn nên bắt đầu từ cơ bản rồi tiến dần lên các mẫu phức tạp:

```
Controlled vs Uncontrolled (Cơ bản)
     ↓
Compound Component (Quản lý Component nhóm)
     ↓
Render Props & HOC (Tái sử dụng logic cũ)
     ↓
Control Props & State Reducer (Tối thượng cho UI Library)
```

---

## 🔗 Liên Kết Chéo Tiêu Biểu

- **01-javascript/concepts/closure.md**: Hiểu sâu về Closure để tránh lỗi *Stale Closure* khi triển khai Render Props và HOC.
- **03-react/hooks-patterns/02-context-ref-hooks.md**: Nền tảng về `useContext` (dùng trong Compound Component) và `useRef` (dùng trong Uncontrolled Component).
- **08-design-patterns/creational-patterns.md**: Đối chiếu triết lý Composition trong React với các OOP Design Patterns kinh điển.
