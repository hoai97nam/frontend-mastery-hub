# 📦 Pattern 1: Compound Component

> Mẫu thiết kế **Compound Component (Component Hỗn Hợp)** cho phép các component cha và component con hợp tác làm việc cùng nhau thông qua một luồng dữ liệu ngầm (implicit state sharing) để tạo ra các cụm UI linh hoạt và dễ tái sử dụng.

---

## 1. Khái niệm & Mục đích

Thay vì tạo một component duy nhất chứa vô số props cấu hình (ví dụ: `<Select options={...} defaultValue={...} onChange={...} />`), **Compound Component** chia nhỏ component lớn thành các component con riêng biệt liên kết chặt chẽ với nhau (ví dụ: `<Select><Select.Option>...</Select.Option></Select>`).

### Ưu điểm:
- **Tùy biến giao diện (UI Flexibility):** Consumer có toàn quyền sắp xếp thứ tự hiển thị, chèn thêm các thẻ HTML hoặc các component khác vào giữa các component con mà không làm hỏng logic.
- **Tránh Prop Drilling:** Toàn bộ việc chia sẻ state (ví dụ: item nào đang mở, active tab nào) được xử lý tự động ngầm.
- **API Sạch và Tự nhiên:** Khai báo cấu trúc component giống như viết thẻ HTML thuần túy.

---

## 2. Cách hiện thực hóa (Implementation)

Có 2 phương pháp phổ biến để phát triển Compound Component trong React:

### 2.1 Sử dụng React Context API (Khuyên dùng - Modern)
Đây là giải pháp tốt nhất vì nó cho phép các component con nằm ở bất cứ độ sâu nào trong cây component con (nested DOM structure) mà vẫn nhận được state từ component cha.

### 2.2 Sử dụng `React.Children.map` & `React.cloneElement` (Truyền thống)
Component cha duyệt qua các con trực tiếp của nó và clone chúng để bổ sung thêm các props liên quan đến state. Phương pháp này có hạn chế là chỉ áp dụng được cho **con trực tiếp** (direct children). Nếu bọc component con trong một thẻ `<div>`, nó sẽ bị mất các props truyền thêm.

---

## 3. Code Example: Component `<Accordion>` thực chiến

Dưới đây là cách xây dựng một Accordion hoạt động đa năng bằng **Context API**:

```jsx
import React, { createContext, useContext, useState, useCallback } from 'react';

// 1. Tạo Context để lưu trữ state dùng chung ngầm
const AccordionContext = createContext(null);

// 2. Component Cha (Accordion Container)
export function Accordion({ children, defaultIndex = null }) {
  const [openIndex, setOpenIndex] = useState(defaultIndex);

  const toggleIndex = useCallback((index) => {
    setOpenIndex((prevIndex) => (prevIndex === index ? null : index));
  }, []);

  return (
    <AccordionContext.Provider value={{ openIndex, toggleIndex }}>
      <div className="accordion-container" style={{ border: '1px solid #ccc', borderRadius: '4px' }}>
        {children}
      </div>
    </AccordionContext.Provider>
  );
}

// 3. Component Con: Accordion.Item (Định nghĩa ngữ cảnh cho từng dòng)
const AccordionItemContext = createContext(null);

Accordion.Item = function AccordionItem({ children, index }) {
  return (
    <AccordionItemContext.Provider value={{ index }}>
      <div className="accordion-item" style={{ borderBottom: '1px solid #eee' }}>
        {children}
      </div>
    </AccordionItemContext.Provider>
  );
};

// 4. Component Con: Accordion.Header (Phần bấm nút)
Accordion.Header = function AccordionHeader({ children }) {
  const { openIndex, toggleIndex } = useContext(AccordionContext);
  const { index } = useContext(AccordionItemContext);
  const isOpen = openIndex === index;

  return (
    <button
      onClick={() => toggleIndex(index)}
      style={{
        width: '100%',
        padding: '12px',
        textAlign: 'left',
        background: isOpen ? '#f0f0f0' : '#fff',
        border: 'none',
        cursor: 'pointer',
        fontWeight: 'bold',
        display: 'flex',
        justifyContent: 'space-between'
      }}
    >
      <span>{children}</span>
      <span>{isOpen ? '▲' : '▼'}</span>
    </button>
  );
};

// 5. Component Con: Accordion.Body (Phần hiển thị nội dung)
Accordion.Body = function AccordionBody({ children }) {
  const { openIndex } = useContext(AccordionContext);
  const { index } = useContext(AccordionItemContext);
  const isOpen = openIndex === index;

  if (!isOpen) return null;

  return (
    <div className="accordion-body" style={{ padding: '12px', background: '#fafafa' }}>
      {children}
    </div>
  );
};
```

### Cách sử dụng (Usage):

```jsx
function App() {
  return (
    <Accordion defaultIndex={0}>
      <Accordion.Item index={0}>
        <Accordion.Header>Khái niệm Compound Component là gì?</Accordion.Header>
        <Accordion.Body>
          Là kỹ thuật kết hợp nhiều component con lại dưới một component cha để quản lý state ngầm.
        </Accordion.Body>
      </Accordion.Item>

      <Accordion.Item index={1}>
        <Accordion.Header>Tại sao nên dùng Context thay vì cloneElement?</Accordion.Header>
        <Accordion.Body>
          Context cho phép lồng các component con bên trong các thẻ HTML tùy ý khác (ví dụ: bọc item trong thẻ div để style) mà không bị mất state.
        </Accordion.Body>
      </Accordion.Item>
    </Accordion>
  );
}
```

---

## 4. Gotchas & Pitfalls (Lưu ý quan trọng)

### ⚠️ Lỗi phá vỡ phân cấp khi dùng `React.cloneElement`
Nếu bạn sử dụng phương pháp `React.Children.map` để clone props, người dùng chỉ có thể viết:
```jsx
<Accordion>
  <Accordion.Item>...</Accordion.Item> {/* Hợp lệ */}
</Accordion>
```
Nhưng nếu người dùng muốn bọc thêm style:
```jsx
<Accordion>
  <div className="wrapper">
    <Accordion.Item>...</Accordion.Item> {/* ❌ LỖI: Accordion.Item sẽ không nhận được state! */}
  </div>
</Accordion>
```
> **Giải pháp:** Luôn ưu tiên dùng **React Context** để tránh hạn chế này.

### ⚠️ Vấn đề tối ưu hóa render (Context Performance)
Mỗi lần state của component cha thay đổi, tất cả các component con gọi `useContext` đều sẽ bị re-render, kể cả khi chúng không thực sự cần cập nhật UI (ví dụ: một Accordion.Header khác không đổi trạng thái).

**Cách khắc phục:**
1. Chia tách Context thành 2 phần: `AccordionStateContext` (lưu data) và `AccordionDispatchContext` (lưu hàm thay đổi state được bọc bằng `useCallback`).
2. Sử dụng `React.memo` cho các component con hoặc tận dụng prop `children` để React bỏ qua quá trình render lại của DOM nhánh nếu nội dung không đổi.
