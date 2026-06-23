# 🚀 Performance Hooks — `useMemo`, `useCallback`, `useTransition`, `useDeferredValue`

> Nhóm hook dùng để **tối ưu hiệu năng render** — tránh tính toán lại không cần thiết và giữ UI mượt mà.

---

## Mục Lục

1. [useMemo](#1-usememo)
   - [Cú pháp](#11-cú-pháp)
   - [Use Cases](#12-use-cases)
   - [Code Examples](#13-code-examples)
   - [Khi nào KHÔNG nên dùng](#14-khi-nào-không-nên-dùng)
   - [Câu Hỏi Phỏng Vấn](#15-câu-hỏi-phỏng-vấn)
2. [useCallback](#2-usecallback)
   - [Cú pháp](#21-cú-pháp)
   - [Use Cases](#22-use-cases)
   - [Code Examples](#23-code-examples)
   - [Câu Hỏi Phỏng Vấn](#24-câu-hỏi-phỏng-vấn)
3. [useTransition](#3-usetransition)
   - [Cú pháp & Use Cases](#31-cú-pháp--use-cases)
   - [Code Examples](#32-code-examples)
   - [Câu Hỏi Phỏng Vấn](#33-câu-hỏi-phỏng-vấn)
4. [useDeferredValue](#4-usedeferredvalue)
   - [Cú pháp & Use Cases](#41-cú-pháp--use-cases)
   - [Code Examples](#42-code-examples)
   - [Câu Hỏi Phỏng Vấn](#43-câu-hỏi-phỏng-vấn)
5. [So Sánh useTransition vs useDeferredValue](#5-so-sánh-usetransition-vs-usedeferredvalue)

---

## 1. `useMemo`

### 1.1 Cú Pháp

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

- Chỉ tính lại khi `a` hoặc `b` thay đổi.
- **Không** chạy trong render — chạy trong commit phase.

---

### 1.2 Use Cases

| Use Case | Mô tả |
|----------|-------|
| Expensive computation | Filter/sort large arrays, complex math |
| Stable reference | Tránh re-render child có `React.memo` |
| Derived state | Computed value từ props/state |
| Context value | Tránh re-render tất cả consumers |

---

### 1.3 Code Examples

#### ✅ Expensive Computation

```jsx
function ProductList({ products, searchQuery, category }) {
  // Chỉ filter lại khi products, searchQuery, hoặc category thay đổi
  const filteredProducts = useMemo(() => {
    console.log('🔄 Filtering...'); // Để thấy khi nào chạy lại
    return products
      .filter(p => p.category === category)
      .filter(p => p.name.toLowerCase().includes(searchQuery.toLowerCase()))
      .sort((a, b) => a.price - b.price);
  }, [products, searchQuery, category]);

  return (
    <ul>
      {filteredProducts.map(p => <ProductCard key={p.id} product={p} />)}
    </ul>
  );
}
```

#### ✅ Stable Reference cho React.memo Child

```jsx
const MemoizedChart = React.memo(function Chart({ data, config }) {
  // Chỉ re-render khi data hoặc config thực sự thay đổi
  return <canvas data={data} config={config} />;
});

function Dashboard({ rawData }) {
  // ❌ Không có useMemo: config mới mỗi render → Chart luôn re-render
  // const config = { colors: ['#FF6384', '#36A2EB'], animated: true };

  // ✅ config chỉ tạo mới khi rawData thay đổi
  const config = useMemo(() => ({
    colors: ['#FF6384', '#36A2EB'],
    animated: true,
  }), []);

  const processedData = useMemo(() => transformData(rawData), [rawData]);

  return <MemoizedChart data={processedData} config={config} />;
}
```

#### ✅ Context Value Optimization

```jsx
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  // Không có useMemo: new object mỗi render → tất cả consumers re-render
  const value = useMemo(() => ({
    user,
    login: (credentials) => authAPI.login(credentials).then(setUser),
    logout: () => { setUser(null); localStorage.clear(); },
  }), [user]); // Chỉ tạo mới khi user thay đổi

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}
```

#### ✅ Tính Toán Số Học Nặng

```jsx
function PrimeCalculator({ limit }) {
  const primes = useMemo(() => {
    // Sieve of Eratosthenes — O(n log log n)
    const sieve = new Array(limit + 1).fill(true);
    sieve[0] = sieve[1] = false;
    for (let i = 2; i * i <= limit; i++) {
      if (sieve[i]) {
        for (let j = i * i; j <= limit; j += i) sieve[j] = false;
      }
    }
    return sieve.map((isPrime, i) => isPrime ? i : null).filter(Boolean);
  }, [limit]);

  return <p>Có {primes.length} số nguyên tố dưới {limit}</p>;
}
```

---

### 1.4 Khi Nào KHÔNG Nên Dùng

```jsx
// ❌ Over-optimization: tính toán đơn giản không cần memo
const doubled = useMemo(() => count * 2, [count]);
// ✅ Đủ đơn giản: const doubled = count * 2;

// ❌ Không có dependency thay đổi theo ý muốn
const value = useMemo(() => expensiveCalc(), []); // Chạy 1 lần, okay nhưng đọc kỳ lạ

// ❌ Array/Object nhỏ, không truyền xuống memo component
const tags = useMemo(() => ['a', 'b', 'c'], []);
```

> **Rule of thumb:** Profile trước, optimize sau. `useMemo` có chi phí riêng (memory, bookkeeping). Chỉ dùng khi profiler cho thấy có bottleneck thực sự.

---

### 1.5 Câu Hỏi Phỏng Vấn

**Q1: `useMemo` và `React.memo` khác nhau như thế nào?**

| | `useMemo` | `React.memo` |
|---|-----------|--------------|
| Loại | Hook | HOC (Higher-Order Component) |
| Memoize | **Giá trị** (computed value) | **Component** (re-render) |
| Dùng cho | Logic trong component | Component function |

---

**Q2: `useMemo` có đảm bảo không tính lại không?**

> **Không 100%.** React có thể xóa memo cache trong một số trường hợp (ví dụ: offscreen rendering trong React 18). `useMemo` là **optimization hint**, không phải semantic guarantee.

---

**Q3 (Tình huống):** Component render lại nhưng `React.memo` vẫn không ngăn được. Nguyên nhân và fix?

> **Nguyên nhân thường gặp:**
> 1. Props là **object/array/function mới** mỗi render (reference inequality).
> 2. Không wrap function prop bằng `useCallback`.
> 3. Không wrap object/array prop bằng `useMemo`.
> 
> **Fix:** Đảm bảo mọi prop không phải primitive đều stable reference giữa các render.

---

## 2. `useCallback`

### 2.1 Cú Pháp

```jsx
const memoizedFn = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

> `useCallback(fn, deps)` ≡ `useMemo(() => fn, deps)` — chỉ khác ở cú pháp.

---

### 2.2 Use Cases

| Use Case | Mô tả |
|----------|-------|
| Props cho `React.memo` child | Tránh re-render do function prop |
| Dependency của `useEffect` | Tránh effect chạy lại không cần thiết |
| Event handlers trong loops | Stable handlers |
| Custom hooks | Expose stable API |

---

### 2.3 Code Examples

#### ✅ Tránh Re-render Child với React.memo

```jsx
const Button = React.memo(function Button({ onClick, label }) {
  console.log(`Button "${label}" re-rendered`);
  return <button onClick={onClick}>{label}</button>;
});

function Counter() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);

  // ❌ Không có useCallback: new function mỗi render → Button re-render
  // const handleIncrement = () => setCount(c => c + 1);

  // ✅ Stable function reference
  const handleIncrement = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // Không deps vì dùng functional update

  return (
    <div>
      <p>Count: {count} | Other: {other}</p>
      <Button onClick={handleIncrement} label="Increment" />
      <button onClick={() => setOther(o => o + 1)}>Update Other</button>
    </div>
  );
}
```

#### ✅ Stable Dependency trong useEffect

```jsx
function DataFetcher({ userId }) {
  const [data, setData] = useState(null);

  // ✅ fetchData không thay đổi reference khi userId không đổi
  const fetchData = useCallback(async () => {
    const result = await api.getUser(userId);
    setData(result);
  }, [userId]);

  useEffect(() => {
    fetchData();
  }, [fetchData]); // fetchData là stable dep

  return <div>{JSON.stringify(data)}</div>;
}
```

#### ✅ Custom Hook với Stable API

```jsx
function useModal() {
  const [isOpen, setIsOpen] = useState(false);

  // Stable references — consumer không bị re-render khi useModal chạy lại
  const open  = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen(prev => !prev), []);

  return { isOpen, open, close, toggle };
}
```

---

### 2.4 Câu Hỏi Phỏng Vấn

**Q1: Khi nào NÊN và KHÔNG NÊN dùng `useCallback`?**

> **NÊN:**
> - Function được truyền xuống `React.memo` child.
> - Function nằm trong deps của `useEffect` / `useMemo`.
>
> **KHÔNG NÊN:**
> - Handler chỉ dùng trong cùng component (không truyền đi).
> - Tất cả function inline một cách mù quáng — thêm overhead không cần thiết.

---

**Q2: `useCallback` với empty deps `[]` có tạo closure cũ không?**

> Có! Nếu function capture state/props trong closure mà deps không liệt kê chúng, function sẽ luôn thấy giá trị cũ (stale closure).
> 
> ```jsx
> // ❌ Stale closure: count luôn là 0
> const logCount = useCallback(() => {
>   console.log(count); // Capture count tại thời điểm tạo
> }, []); // count không trong deps
>
> // ✅ Fix 1: Add count to deps
> const logCount = useCallback(() => console.log(count), [count]);
>
> // ✅ Fix 2: Functional update (không cần capture count)
> const increment = useCallback(() => setCount(c => c + 1), []);
> ```

---

## 3. `useTransition`

### 3.1 Cú Pháp & Use Cases

```jsx
const [isPending, startTransition] = useTransition();

// Wrap state update ít ưu tiên hơn
startTransition(() => {
  setNonUrgentState(newValue);
});
```

- `isPending` — `true` khi transition đang xử lý.
- Cho phép React **interrupt** update và ưu tiên updates quan trọng hơn (typing, clicking).
- Không áp dụng được cho: input value (cần responsive ngay lập tức).

**Use Cases:**
- Chuyển tab với danh sách lớn
- Search results không cần real-time
- Route navigation với heavy content

---

### 3.2 Code Examples

#### ✅ Tab Switching — Không Block UI

```jsx
function TabContainer() {
  const [activeTab, setActiveTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  const switchTab = (tab) => {
    startTransition(() => {
      setActiveTab(tab); // React có thể delay render tab content
    });
  };

  return (
    <div>
      <nav>
        {['home', 'posts', 'about'].map(tab => (
          <button
            key={tab}
            onClick={() => switchTab(tab)}
            style={{ opacity: isPending ? 0.5 : 1 }}
          >
            {tab}
          </button>
        ))}
      </nav>
      {/* TabContent render nặng nhưng không block UI */}
      <Suspense fallback={<Spinner />}>
        <TabContent tab={activeTab} />
      </Suspense>
    </div>
  );
}
```

#### ✅ Large List Filter

```jsx
function FilterableList({ items }) {
  const [query, setQuery] = useState('');
  const [filteredItems, setFilteredItems] = useState(items);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    const value = e.target.value;
    setQuery(value); // Urgent: update input ngay

    startTransition(() => {
      // Non-urgent: filter có thể delay
      setFilteredItems(
        items.filter(item =>
          item.name.toLowerCase().includes(value.toLowerCase())
        )
      );
    });
  };

  return (
    <div>
      <input value={query} onChange={handleSearch} placeholder="Search..." />
      {isPending && <span>Đang lọc...</span>}
      <VirtualList items={filteredItems} />
    </div>
  );
}
```

---

### 3.3 Câu Hỏi Phỏng Vấn

**Q1: `useTransition` hoạt động như thế nào trong React's Concurrent Mode?**

> React 18's Concurrent Mode cho phép render được **interruptible**. `startTransition` đánh dấu state update là "non-urgent" — nếu có urgent update (user click/type) xảy ra trong lúc đó, React sẽ **bỏ render dở** của transition và xử lý urgent update trước, sau đó quay lại.

---

**Q2: Khác biệt giữa `useTransition` và `setTimeout(() => setState(...), 0)`?**

> - `setTimeout`: hack — delay nhân tạo, React không biết về priority, vẫn block nếu render nặng.
> - `useTransition`: React-aware — React thực sự interrupt và reschedule render theo priority. Cho phép Suspense integration.

---

## 4. `useDeferredValue`

### 4.1 Cú Pháp & Use Cases

```jsx
const deferredValue = useDeferredValue(value);
```

- React sẽ **giữ lại giá trị cũ** và render lại với giá trị mới khi rảnh.
- Trong lúc chờ: `deferredValue !== value` → có thể show stale UI.
- Không cần access to state setter (khác `useTransition`).

**Use Cases:**
- Nhận value từ prop (không control được state update)
- Search input với expensive results render
- Real-time data với heavy visualization

---

### 4.2 Code Examples

#### ✅ Search với Deferred Results

```jsx
function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  // isStale: đang hiển thị kết quả cũ trong khi chờ kết quả mới
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Tìm kiếm..."
      />
      <div style={{ opacity: isStale ? 0.5 : 1, transition: 'opacity 0.2s' }}>
        {/* SearchResults vẫn dùng giá trị cũ khi isStale */}
        <SearchResults query={deferredQuery} />
      </div>
    </div>
  );
}

// SearchResults được bọc memo để React biết có thể reuse render cũ
const SearchResults = React.memo(function SearchResults({ query }) {
  const results = useSearchResults(query); // Heavy computation
  return <ResultList items={results} />;
});
```

#### ✅ Real-time Chart với Deferred Data

```jsx
function Dashboard({ liveData }) {
  const deferredData = useDeferredValue(liveData);
  const isStale = liveData !== deferredData;

  return (
    <div>
      <MetricsBar data={liveData} /> {/* Cập nhật ngay */}
      <HeavyChart
        data={deferredData} {/* Có thể lag 1-2 frame */}
        style={{ filter: isStale ? 'blur(1px)' : 'none' }}
      />
    </div>
  );
}
```

---

### 4.3 Câu Hỏi Phỏng Vấn

**Q1: `useDeferredValue` khác `debounce` như thế nào?**

> - `debounce`: Delay **cố định** (ví dụ: 300ms) — dù machine nhanh hay chậm.
> - `useDeferredValue`: Delay **thích nghi** — trên máy nhanh gần như không delay, trên máy chậm React tự điều chỉnh. Không có "magic number" timeout.

---

**Q2: Khi nào dùng `useDeferredValue` thay vì `useTransition`?**

> - Dùng `useTransition` khi bạn **control được state update** (có access đến setter).
> - Dùng `useDeferredValue` khi value đến từ **prop hoặc context** — bạn không control được update.

---

## 5. So Sánh `useTransition` vs `useDeferredValue`

| | `useTransition` | `useDeferredValue` |
|---|----------------|-------------------|
| Wrap | **State update** | **Value** |
| Control | Bạn **gọi** `startTransition` | React **tự quyết định** |
| Access setter | **Cần** | **Không cần** |
| `isPending` | ✅ Có | ❌ Không (phải so sánh thủ công) |
| Dùng khi | Control state update của mình | Nhận value từ ngoài |

```
useTransition     → Bạn control state setter
useDeferredValue  → Bạn nhận value từ prop/context
```

---

## 📎 Performance Hooks — Khi Nào Dùng Cái Gì?

```
useMemo           → Memoize COMPUTED VALUE tốn kém
useCallback       → Memoize FUNCTION để truyền xuống child / đặt trong deps
useTransition     → DELAY non-urgent state update, có isPending
useDeferredValue  → DEFER giá trị không control được, từ prop/context
```

> **⚠️ Golden Rule:** Đừng optimize sớm. Dùng React DevTools Profiler để xác định bottleneck thực sự trước khi thêm memo/callback.

---

🔙 [Effect Hooks](./03-effect-hooks.md) | 🔙 [README](./README.md) | ➡️ [React 19+ Hooks](./05-new-hooks.md)
