# 🗂️ State Hooks — `useState` & `useReducer`

> Nhóm hook dùng để **quản lý trạng thái cục bộ** bên trong component.

---

## Mục Lục

1. [useState](#1-usestate)
   - [Cú pháp](#11-cú-pháp)
   - [Use Cases](#12-use-cases)
   - [Code Examples](#13-code-examples)
   - [Gotchas & Pitfalls](#14-gotchas--pitfalls)
   - [Câu Hỏi Phỏng Vấn](#15-câu-hỏi-phỏng-vấn)
2. [useReducer](#2-usereducer)
   - [Cú pháp](#21-cú-pháp)
   - [Use Cases](#22-use-cases)
   - [Code Examples](#23-code-examples)
   - [useState vs useReducer](#24-usestate-vs-usereducer)
   - [Câu Hỏi Phỏng Vấn](#25-câu-hỏi-phỏng-vấn)

---

## 1. `useState`

### 1.1 Cú Pháp

```jsx
const [state, setState] = useState(initialValue);
// hoặc lazy initialization:
const [state, setState] = useState(() => computeExpensiveValue());
```

- `state` — Giá trị hiện tại.
- `setState` — Hàm cập nhật state, trigger re-render.
- `initialValue` — Giá trị khởi tạo (chỉ dùng lần đầu render).

---

### 1.2 Use Cases

| Use Case | Mô tả |
|----------|-------|
| Toggle UI | Mở/đóng modal, dropdown, accordion |
| Form fields | Kiểm soát input, validation |
| Counter / step | Pagination, quantity selector |
| Fetch status | `loading`, `error`, `data` |
| Derived UI state | Tab active, selected item |

---

### 1.3 Code Examples

#### ✅ Cơ Bản — Counter

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  );
}
```

#### ✅ Functional Update — Đúng Cách

```jsx
// ❌ SAI: dễ bị stale closure
setCount(count + 1);

// ✅ ĐÚNG: luôn nhận giá trị mới nhất
setCount(prev => prev + 1);
```

> **Khi nào dùng functional update?** Khi gọi setState nhiều lần liên tiếp trong một handler, hoặc trong async callbacks.

#### ✅ Lazy Initialization

```jsx
// Hàm khởi tạo chỉ chạy 1 lần (lần đầu mount)
const [data, setData] = useState(() => {
  const saved = localStorage.getItem('data');
  return saved ? JSON.parse(saved) : [];
});
```

#### ✅ Object State

```jsx
function UserForm() {
  const [form, setForm] = useState({ name: '', email: '' });

  const handleChange = (e) => {
    // Luôn spread state cũ để tránh mất field
    setForm(prev => ({ ...prev, [e.target.name]: e.target.value }));
  };

  return (
    <form>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
    </form>
  );
}
```

#### ✅ Toggle State

```jsx
function Modal() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(prev => !prev)}>
        {isOpen ? 'Đóng' : 'Mở'} Modal
      </button>
      {isOpen && <div className="modal">Nội dung modal</div>}
    </>
  );
}
```

---

### 1.4 Gotchas & Pitfalls

#### ⚠️ State là Snapshot

```jsx
function handleClick() {
  setCount(count + 1); // count = 0 → state = 1
  setCount(count + 1); // count vẫn = 0 (snapshot) → state = 1, không phải 2!
  setCount(count + 1); // tương tự

  // ✅ Dùng functional update để cộng dồn:
  setCount(prev => prev + 1); // 0 → 1
  setCount(prev => prev + 1); // 1 → 2
  setCount(prev => prev + 1); // 2 → 3
}
```

#### ⚠️ State Update là Bất Đồng Bộ

```jsx
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount(1);
  console.log(count); // vẫn in ra 0! (chưa re-render)
};
```

#### ⚠️ Object/Array State — Không Mutate Trực Tiếp

```jsx
// ❌ SAI: Mutate trực tiếp, React không detect được
state.items.push(newItem);
setState(state);

// ✅ ĐÚNG: Tạo reference mới
setState(prev => ({ ...prev, items: [...prev.items, newItem] }));
```

---

### 1.5 Câu Hỏi Phỏng Vấn

**Q1: Tại sao `setState` không cập nhật ngay lập tức?**

> React batches (gom) các state updates lại để tối ưu hiệu năng. Sau khi event handler hoàn tất (hoặc một đơn vị bất đồng bộ xong), React mới re-render component với state mới nhất. Đây là cơ chế **batching** — mặc định trong React 18 với Automatic Batching.

---

**Q2: Sự khác biệt giữa `setState(value)` và `setState(prev => value)` là gì?**

> - `setState(value)`: capture `value` tại thời điểm gọi (có thể là stale nếu trong closure cũ).
> - `setState(prev => ...)`: React đảm bảo `prev` là state **mới nhất** tại thời điểm update được áp dụng.
> 
> **Rule of thumb:** Khi state mới phụ thuộc vào state cũ → luôn dùng functional form.

---

**Q3: Lazy initialization trong `useState` hoạt động như thế nào và khi nào nên dùng?**

> Khi truyền một **function** thay vì value vào `useState`, React chỉ gọi function đó **một lần duy nhất** (lần render đầu tiên). Dùng khi:
> - Tính toán initialState tốn kém (đọc localStorage, parse JSON lớn...).
> - Tránh tính lại mỗi lần re-render.

---

**Q4: React có re-render không nếu `setState` được gọi với cùng giá trị?**

> **Không** (từ React 16.8+). React dùng `Object.is` để so sánh. Nếu giá trị mới bằng giá trị cũ, React sẽ bail out và không re-render.

---

**Q5 (Tình huống):** Component của bạn render quá nhiều lần. Bạn nghi ngờ `useState` là nguyên nhân. Bạn debug như thế nào?

> 1. Dùng **React DevTools Profiler** để xem component nào re-render và lý do.
> 2. Kiểm tra các `setState` call — có đang set object mới mỗi lần không (dù giá trị giống nhau)?
> 3. Xem xét có đang tạo inline object/array trong JSX không (reference mới mỗi render).
> 4. Cân nhắc dùng `useMemo` hoặc `useCallback` cho derived values và handlers.

---

## 2. `useReducer`

### 2.1 Cú Pháp

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
// hoặc lazy initialization:
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

```jsx
// Reducer function
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT': return { ...state, count: state.count + 1 };
    case 'DECREMENT': return { ...state, count: state.count - 1 };
    case 'RESET':     return initialState;
    default: throw new Error(`Unknown action: ${action.type}`);
  }
}
```

---

### 2.2 Use Cases

| Use Case | Mô tả |
|----------|-------|
| State phức tạp | Nhiều sub-values liên quan nhau |
| Multiple actions | Logic update state có nhiều "nhánh" |
| Chia sẻ logic | Dễ test reducer function độc lập |
| Redux-like pattern | Khi không muốn dùng Redux |
| Undo / Redo | Lưu history của state |

---

### 2.3 Code Examples

#### ✅ Kinh Điển — Counter với Reducer

```jsx
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    case 'reset':     return initialState;
    default:          throw new Error('Unknown action');
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

#### ✅ Shopping Cart — State Phức Tạp

```jsx
const initialCart = { items: [], total: 0, isLoading: false };

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const exists = state.items.find(i => i.id === action.payload.id);
      const items = exists
        ? state.items.map(i =>
            i.id === action.payload.id ? { ...i, qty: i.qty + 1 } : i
          )
        : [...state.items, { ...action.payload, qty: 1 }];
      return {
        ...state,
        items,
        total: items.reduce((sum, i) => sum + i.price * i.qty, 0),
      };
    }
    case 'REMOVE_ITEM': {
      const items = state.items.filter(i => i.id !== action.payload);
      return { ...state, items, total: items.reduce((s, i) => s + i.price * i.qty, 0) };
    }
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    default:
      return state;
  }
}

function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, initialCart);

  return (
    <div>
      <p>Tổng: {cart.total.toLocaleString('vi-VN')}đ</p>
      <button onClick={() => dispatch({ type: 'ADD_ITEM', payload: { id: 1, name: 'Sản phẩm A', price: 99000 } })}>
        Thêm vào giỏ
      </button>
    </div>
  );
}
```

#### ✅ Lazy Initialization với `init` Function

```jsx
function init(initialCount) {
  // Tính toán phức tạp hoặc load từ storage
  return { count: initialCount, history: [] };
}

const [state, dispatch] = useReducer(reducer, 0, init);
// => init(0) chỉ được gọi một lần
```

---

### 2.4 `useState` vs `useReducer`

| Tiêu chí | `useState` | `useReducer` |
|----------|-----------|--------------|
| Độ phức tạp state | Đơn giản, primitive | Phức tạp, nhiều sub-state |
| Số lượng actions | Ít | Nhiều, có tên rõ ràng |
| Testability | Khó test logic trong handler | Dễ unit test reducer |
| Readability | Tốt cho state đơn | Tốt khi có nhiều update cases |
| Performance | Tương đương | Tương đương |

> **Rule of thumb:** Khi có ≥ 3 state liên quan nhau → xem xét `useReducer`. Khi logic update phức tạp hoặc cần share → dùng `useReducer`.

---

### 2.5 Câu Hỏi Phỏng Vấn

**Q1: Khi nào nên dùng `useReducer` thay vì `useState`?**

> - Khi state có **nhiều sub-values** liên quan (ví dụ: `{ loading, data, error }`).
> - Khi có **nhiều loại action** cập nhật state theo cách khác nhau.
> - Khi muốn **logic update có thể test độc lập** — reducer là pure function dễ unit test.
> - Khi state của step sau phụ thuộc vào step trước (multi-step form).

---

**Q2: `useReducer` có tốt hơn Redux không?**

> Không thay thế hoàn toàn. `useReducer`:
> - Chỉ là **local state** — không share được giữa các component (trừ khi kết hợp với Context).
> - Không có middleware (thunk, saga).
> - Không có DevTools tích hợp sẵn.
> 
> Redux phù hợp hơn cho **global state lớn, phức tạp, cần time-travel debugging**.

---

**Q3 (Tình huống):** Bạn có một form đăng ký 5 bước, mỗi bước có validation riêng và data từ bước trước ảnh hưởng bước sau. Bạn dùng hook nào?

> **`useReducer`** là lựa chọn phù hợp vì:
> - State có cấu trúc phức tạp: `{ step, formData, errors, isValid }`.
> - Nhiều action types: `NEXT_STEP`, `PREV_STEP`, `UPDATE_FIELD`, `SET_ERROR`, `SUBMIT`.
> - Logic transition giữa các step có thể viết rõ ràng trong reducer.
> - Dễ test từng action độc lập.

---

**Q4: Reducer có được phép có side effects không?**

> **Không.** Reducer **phải là pure function** — cùng input, luôn cho cùng output, không side effects (không fetch API, không set localStorage trong reducer). Side effects nên đặt trong `useEffect` hoặc event handlers, sau đó `dispatch` action với kết quả.

---

**Q5: Sự khác biệt giữa `dispatch` của `useReducer` và `setState` của `useState` về mặt stability?**

> Cả hai đều **stable** (không thay đổi reference giữa các render) — không cần bọc trong `useCallback`. Đây là lý do `dispatch` an toàn khi truyền xuống child component mà không gây re-render không cần thiết.

---

## 📎 Tổng Kết

```
useState  → State đơn, giá trị primitive hoặc object nhỏ
useReducer → State phức tạp, nhiều actions, logic rõ ràng, dễ test
```

---

🔙 [Quay lại README](./README.md) | ➡️ [Context & Ref Hooks](./02-context-ref-hooks.md)
