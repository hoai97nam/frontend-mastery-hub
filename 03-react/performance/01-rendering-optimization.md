# 🔄 Rendering Optimization — Tối Ưu Re-render

> Hiểu sâu về **cơ chế render của React**, phát hiện và loại bỏ re-render không cần thiết.

---

## Mục Lục

1. [React Render Pipeline](#1-react-render-pipeline)
2. [Khi Nào Component Re-render?](#2-khi-nào-component-re-render)
3. [React.memo — Memoize Component](#3-reactmemo--memoize-component)
4. [useMemo & useCallback — Ổn Định Props](#4-usememo--usecallback--ổn-định-props)
5. [Key Prop & Reconciliation](#5-key-prop--reconciliation)
6. [Virtualization — Render Danh Sách Lớn](#6-virtualization--render-danh-sách-lớn)
7. [React DevTools Profiler](#7-react-devtools-profiler)
8. [Concurrent Features](#8-concurrent-features)

---

## 1. React Render Pipeline

### Ba Giai Đoạn

```
Trigger → Render → Commit
```

| Giai đoạn | Mô tả | Xảy ra ở đâu |
|-----------|-------|--------------|
| **Trigger** | setState, context change, parent re-render | React |
| **Render** | Gọi component function, tạo Virtual DOM mới | JS Thread |
| **Commit** | So sánh (diffing) + cập nhật DOM thật | DOM |

### Điều Quan Trọng Cần Nhớ

- **Render ≠ DOM update** — Render phase chỉ tạo Virtual DOM (VDOM), chưa chạm DOM thật.
- **Commit phase mới chạm DOM** — Chỉ cập nhật những node thực sự thay đổi.
- **Render phase có thể bị discard** (React 18 Concurrent Mode) — Không nên có side effects trong render.

```jsx
// ⚠️ Render phase phải pure — không có side effects
function Component({ items }) {
  // ❌ KHÔNG làm thế này trong render
  // fetchData(); // Side effect trong render!
  // localStorage.setItem(...); // Side effect trong render!

  // ✅ Chỉ compute, không có side effects
  const sorted = [...items].sort((a, b) => a.name.localeCompare(b.name));
  return <List items={sorted} />;
}
```

---

## 2. Khi Nào Component Re-render?

### 4 Nguyên Nhân Re-render

```
1. State thay đổi (useState / useReducer)
2. Props thay đổi
3. Parent re-render → tất cả children mặc định re-render
4. Context thay đổi (useContext)
```

### Ví Dụ — Re-render Lan Rộng (Cascade)

```jsx
function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      {/* ❌ Mỗi lần count thay đổi, TẤT CẢ re-render */}
      <Header />          {/* Re-render dù không cần count */}
      <Sidebar />         {/* Re-render dù không cần count */}
      <Counter count={count} /> {/* Cần count → OK */}
      <Footer />          {/* Re-render dù không cần count */}
    </div>
  );
}
```

### Fix — State Colocation

```jsx
// ✅ Đưa state xuống gần nơi dùng nhất
function CounterSection() {
  const [count, setCount] = useState(0); // State ở đây, không phải App
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <Counter count={count} />
    </div>
  );
}

function App() {
  return (
    <div>
      <Header />     {/* Không bao giờ re-render do count */}
      <Sidebar />    {/* Không bao giờ re-render do count */}
      <CounterSection /> {/* Chỉ section này re-render */}
      <Footer />     {/* Không bao giờ re-render do count */}
    </div>
  );
}
```

---

## 3. `React.memo` — Memoize Component

### Cách Hoạt Động

```jsx
const MemoComponent = React.memo(function MyComponent(props) {
  // Chỉ re-render khi props thay đổi (shallow comparison)
  return <div>{props.value}</div>;
});
```

React dùng **shallow comparison** (`Object.is`) để so sánh từng prop. Nếu tất cả props giống nhau → **skip render**.

### ✅ Khi Nào Nên Dùng `React.memo`

```jsx
// 1. Component render tốn kém
const HeavyChart = React.memo(function Chart({ data, config }) {
  // D3 rendering, complex calculations...
  const processedData = expensiveDataTransform(data);
  return <canvas ref={canvasRef} />;
});

// 2. Component render thường xuyên do parent update
const ListItem = React.memo(function ListItem({ item, onSelect }) {
  console.log('ListItem render:', item.id);
  return (
    <div onClick={() => onSelect(item.id)}>
      {item.name}
    </div>
  );
});

// 3. Component con trong danh sách lớn
const VirtualRow = React.memo(function Row({ index, data }) {
  return <div>{data[index].name}</div>;
});
```

### ❌ Khi Nào KHÔNG Cần `React.memo`

```jsx
// ❌ Component quá đơn giản — overhead của memo > lợi ích
const SimpleText = React.memo(({ text }) => <span>{text}</span>);

// ❌ Props thay đổi hầu hết mỗi render — memo vô ích
const AlwaysChanging = React.memo(({ data }) => <div>{data.value}</div>);
// Nếu data object mới mỗi render → memo không có tác dụng

// ❌ Component không tốn kém render
const Badge = React.memo(({ count }) => <span>{count}</span>);
```

### Custom Comparison Function

```jsx
const UserCard = React.memo(
  function UserCard({ user, onEdit }) {
    return (
      <div>
        <img src={user.avatar} alt={user.name} />
        <h3>{user.name}</h3>
        <button onClick={() => onEdit(user.id)}>Edit</button>
      </div>
    );
  },
  // Custom comparison — chỉ so sánh những gì quan trọng
  (prevProps, nextProps) => {
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.name === nextProps.user.name &&
      prevProps.user.avatar === nextProps.user.avatar
      // Không so sánh onEdit — dùng useCallback bên ngoài
    );
  }
);
```

---

## 4. `useMemo` & `useCallback` — Ổn Định Props

### Vấn Đề: Reference Inequality

```jsx
// ❌ Pattern này làm React.memo vô dụng
function Parent() {
  const [count, setCount] = useState(0);

  // Mỗi render → config mới → MemoChild luôn re-render!
  const config = { theme: 'dark', size: 'large' };
  const handleClick = () => console.log('clicked');

  return <MemoChild config={config} onClick={handleClick} />;
}
```

### Fix — Ổn Định Reference

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [theme, setTheme] = useState('dark');

  // ✅ config chỉ tạo mới khi theme thay đổi
  const config = useMemo(() => ({
    theme,
    size: 'large',
  }), [theme]);

  // ✅ handler stable — không bao giờ thay đổi
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      {/* MemoChild không re-render khi count thay đổi */}
      <MemoChild config={config} onClick={handleClick} />
    </div>
  );
}
```

### Checklist Khi Dùng `React.memo`

```
□ Component được bọc React.memo?
□ TẤT CẢ props object/array được useMemo?
□ TẤT CẢ props function được useCallback?
□ Props primitive (string, number, boolean) → OK không cần memo
□ Đã verify bằng DevTools Profiler?
```

---

## 5. Key Prop & Reconciliation

### React Dùng `key` Như Thế Nào

```
Render lần 1:    [A(key=1), B(key=2), C(key=3)]
Render lần 2:    [A(key=1), C(key=3)]           ← B bị xóa

React: key=1 → reuse, key=2 → unmount, key=3 → reuse
```

### ❌ Lỗi `key` Hay Gặp

```jsx
// ❌ Dùng index làm key — gây bug khi reorder/delete
{items.map((item, index) => (
  <Item key={index} data={item} />
))}

// Vấn đề: Khi xóa item đầu tiên:
// Trước: [Item(key=0, data=A), Item(key=1, data=B), Item(key=2, data=C)]
// Sau:   [Item(key=0, data=B), Item(key=1, data=C)]
// React nghĩ key=0 vẫn là A, chỉ cập nhật data → Animation, input state bị lẫn lộn!
```

```jsx
// ✅ Luôn dùng stable ID
{items.map(item => (
  <Item key={item.id} data={item} />
))}
```

### Dùng `key` Để Force Reset

```jsx
// ✅ Trick hay: thay đổi key để reset hoàn toàn component
function ProfilePage({ userId }) {
  return (
    // Khi userId thay đổi → UserDetails unmount và remount (reset mọi state nội bộ)
    <UserDetails key={userId} userId={userId} />
  );
}
```

```jsx
// Ví dụ thực tế: Reset form khi chuyển user
function EditUserForm({ user }) {
  const [name, setName] = useState(user.name);
  // Vấn đề: useState không reset khi user thay đổi!
  // Fix 1: useEffect để sync
  // Fix 2 (cleaner): dùng key
  return <input value={name} onChange={e => setName(e.target.value)} />;
}

// Parent:
<EditUserForm key={selectedUser.id} user={selectedUser} />
// Khi selectedUser.id thay đổi → form reset hoàn toàn
```

### Reconciliation Algorithm (Diffing)

| Trường hợp | React làm gì |
|------------|--------------|
| Cùng type, cùng key | Update props, giữ instance |
| Khác type | Unmount cũ, mount mới (kể cả children) |
| Thêm/xóa key | Unmount/mount tương ứng |

```jsx
// ⚠️ Render conditional sai cách → reset state bất ngờ
function Form() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  // ❌ Switch giữa 2 type khác nhau → React unmount/remount → reset state
  return isLoggedIn ? <UserForm /> : <GuestForm />;
  // Nếu UserForm và GuestForm có cùng structure nhưng khác tên class
  // → React sẽ unmount toàn bộ và mount lại

  // ✅ Giữ cùng type, dùng CSS hoặc conditional rendering trong component
}
```

---

## 6. Virtualization — Render Danh Sách Lớn

### Vấn Đề — 10,000 Rows

```jsx
// ❌ Render 10,000 DOM nodes → chậm, tốn bộ nhớ
function HugeList({ items }) {
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

### Giải Pháp — Chỉ Render Những Gì User Thấy

```
Viewport:  [Item 50] [Item 51] [Item 52] [Item 53] [Item 54]
           ← Chỉ 5 items được render trong DOM
           
Scroll buffer (trên):  [Item 48] [Item 49]
Scroll buffer (dưới):  [Item 55] [Item 56]
```

### Với `react-window` (Phổ Biến Nhất)

```jsx
import { FixedSizeList } from 'react-window';

// ✅ Chỉ render ~10 items tại một thời điểm, dù list có 100,000 items
function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style} className="list-item">
      {items[index].name}
    </div>
  );

  return (
    <FixedSizeList
      height={600}        // Chiều cao container (px)
      width="100%"
      itemCount={items.length}
      itemSize={50}       // Chiều cao mỗi row (px)
    >
      {Row}
    </FixedSizeList>
  );
}
```

### Với `@tanstack/react-virtual` (Mạnh Hơn)

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,         // Ước tính chiều cao
    overscan: 5,                    // Buffer items ngoài viewport
  });

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      {/* Container với total height để scrollbar đúng */}
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: virtualRow.start,
              left: 0,
              width: '100%',
              height: virtualRow.size,
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Khi Nào Cần Virtualization?

| Số lượng items | Recommendation |
|---------------|----------------|
| < 100 | Không cần |
| 100 – 500 | Cân nhắc nếu items phức tạp |
| 500 – 1,000 | Nên dùng |
| > 1,000 | **Bắt buộc** |

---

## 7. React DevTools Profiler

### Cách Đọc Flame Graph

```
Flame Graph:
┌──────────────────────────────────────┐
│            App (2.3ms)               │  ← Tổng thời gian render
├──────────┬───────────┬───────────────┤
│Header    │  Main     │    Footer     │
│(0.1ms)   │  (2.1ms)  │    (0.1ms)   │
│          ├─────┬─────┤               │
│          │List │Aside│               │
│          │(1.9ms)(0.2ms)             │
└──────────┴─────┴─────┴───────────────┘

→ List (1.9ms) là bottleneck → optimize ở đây
```

### Workflow Debug Re-render

```
1. Mở React DevTools → Profiler tab
2. Click "Record" → Thực hiện hành động chậm
3. Click "Stop"
4. Xem Flame Graph:
   - Màu vàng/đỏ → render lâu
   - Màu xanh nhạt → render nhanh
   - Màu xám (striped) → không re-render (memoized)
5. Click vào component → xem "Why did this render?"
6. Fix bottleneck → Record lại để so sánh
```

### "Why did this render?" — 3 Lý Do Phổ Biến

```
1. "Props changed" → Check xem prop nào thay đổi
   → Dùng useMemo/useCallback để stabilize

2. "State changed" → State nào trigger?
   → Cân nhắc colocate state

3. "Parent rendered" → Parent không có memo
   → Thêm React.memo hoặc colocate state trong parent
```

---

## 8. Concurrent Features

### `useTransition` — Không Freeze UI

```jsx
function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    const value = e.target.value;

    // URGENT: cập nhật input ngay lập tức
    setQuery(value);

    // NON-URGENT: React có thể delay nếu cần render urgent update
    startTransition(() => {
      const filtered = hugeDataset.filter(item =>
        item.name.toLowerCase().includes(value.toLowerCase())
      );
      setResults(filtered);
    });
  };

  return (
    <>
      <input value={query} onChange={handleSearch} />
      {isPending && <LoadingSpinner />}
      <ResultList items={results} style={{ opacity: isPending ? 0.5 : 1 }} />
    </>
  );
}
```

### `useDeferredValue` — Defer Props

```jsx
function ProductSearch({ searchQuery }) {
  // Defer searchQuery → giữ UI responsive khi type nhanh
  const deferredQuery = useDeferredValue(searchQuery);
  const isStale = searchQuery !== deferredQuery;

  return (
    <div style={{ opacity: isStale ? 0.6 : 1, transition: 'opacity 200ms' }}>
      <ProductGrid query={deferredQuery} /> {/* Render với giá trị cũ nếu đang type */}
    </div>
  );
}
```

---

## 📎 Tổng Kết Checklist

```
Re-render quá nhiều?
  □ Dùng DevTools Profiler để xác định component
  □ State colocation — đưa state xuống gần nơi dùng
  □ React.memo + stable props (useMemo/useCallback)
  □ Key prop đúng cách

List lớn chậm?
  □ Virtualization (react-window hoặc @tanstack/react-virtual)
  □ Memoize ListItem component

UI freeze khi compute?
  □ useTransition cho non-urgent updates
  □ useDeferredValue cho deferred props
  □ Web Worker cho heavy computation
```

---

🔙 [README](./README.md) | ➡️ [Code Splitting](./02-code-splitting.md)
