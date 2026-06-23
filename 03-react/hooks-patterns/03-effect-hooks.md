# ⚡ Effect Hooks — `useEffect`, `useLayoutEffect`, `useInsertionEffect`

> Nhóm hook dùng để **đồng bộ component với hệ thống bên ngoài** — API, DOM, timer, subscriptions.

---

## Mục Lục

1. [useEffect](#1-useeffect)
   - [Cú pháp & Lifecycle](#11-cú-pháp--lifecycle)
   - [Dependency Array](#12-dependency-array)
   - [Use Cases](#13-use-cases)
   - [Code Examples](#14-code-examples)
   - [Gotchas & Pitfalls](#15-gotchas--pitfalls)
   - [Câu Hỏi Phỏng Vấn](#16-câu-hỏi-phỏng-vấn)
2. [useLayoutEffect](#2-uselayouteffect)
   - [Cú pháp & Timing](#21-cú-pháp--timing)
   - [Use Cases & Code Examples](#22-use-cases--code-examples)
   - [Câu Hỏi Phỏng Vấn](#23-câu-hỏi-phỏng-vấn)
3. [useInsertionEffect](#3-useinsertioneffect)
   - [Use Cases & Code Examples](#31-use-cases--code-examples)
4. [So Sánh 3 Effect Hooks](#4-so-sánh-3-effect-hooks)

---

## 1. `useEffect`

### 1.1 Cú Pháp & Lifecycle

```jsx
useEffect(() => {
  // Setup: Chạy sau khi render
  const subscription = subscribe();

  // Cleanup: Chạy trước khi effect tiếp theo / trước khi unmount
  return () => {
    subscription.unsubscribe();
  };
}, [dependencies]); // Dependency array
```

**Lifecycle mapping:**

| Class Component | `useEffect` |
|-----------------|-------------|
| `componentDidMount` | `useEffect(() => {...}, [])` |
| `componentDidUpdate` | `useEffect(() => {...}, [dep])` |
| `componentWillUnmount` | `useEffect(() => { return () => {...} }, [])` |

---

### 1.2 Dependency Array

| Dạng | Hành vi |
|------|---------|
| `useEffect(fn)` | Chạy sau **mỗi** render |
| `useEffect(fn, [])` | Chạy **một lần** sau mount |
| `useEffect(fn, [a, b])` | Chạy khi `a` hoặc `b` thay đổi |

---

### 1.3 Use Cases

| Use Case | Pattern |
|----------|---------|
| Data fetching | Fetch API khi component mount hoặc params thay đổi |
| Event listeners | Đăng ký / hủy đăng ký |
| Subscriptions | WebSocket, observable, store |
| Timer | `setInterval` / `setTimeout` |
| Sync với external libs | D3, Google Maps, v.v. |
| Document title | `document.title = ...` |

---

### 1.4 Code Examples

#### ✅ Kinh Điển — Data Fetching

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false; // Tránh race condition

    setLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(data => {
        if (!cancelled) {
          setUser(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err.message);
          setLoading(false);
        }
      });

    return () => { cancelled = true; }; // Cleanup khi userId thay đổi
  }, [userId]);

  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  return <UserCard user={user} />;
}
```

#### ✅ Event Listener

```jsx
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handler = () => setSize({
      width: window.innerWidth,
      height: window.innerHeight,
    });

    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler); // Cleanup!
  }, []); // Chỉ register 1 lần

  return size;
}
```

#### ✅ WebSocket Subscription

```jsx
function LiveChat({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const ws = new WebSocket(`wss://api.example.com/chat/${roomId}`);

    ws.onmessage = (event) => {
      setMessages(prev => [...prev, JSON.parse(event.data)]);
    };

    return () => ws.close(); // Đóng kết nối khi unmount hoặc roomId thay đổi

  }, [roomId]); // Re-connect khi roomId thay đổi

  return <MessageList messages={messages} />;
}
```

#### ✅ Document Title Sync

```jsx
function useDocumentTitle(title) {
  useEffect(() => {
    const prevTitle = document.title;
    document.title = title;
    return () => { document.title = prevTitle; }; // Restore khi unmount
  }, [title]);
}

// Dùng:
function ProductPage({ product }) {
  useDocumentTitle(`${product.name} | MyShop`);
  return <div>...</div>;
}
```

#### ✅ AbortController — Fetch Cancellation (Best Practice)

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    if (!query) return;

    const controller = new AbortController();

    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then(res => res.json())
      .then(setResults)
      .catch(err => {
        if (err.name !== 'AbortError') console.error(err);
      });

    return () => controller.abort(); // Cancel request khi query thay đổi
  }, [query]);

  return <List items={results} />;
}
```

---

### 1.5 Gotchas & Pitfalls

#### ⚠️ Infinite Loop — Thiếu hoặc Sai Dependencies

```jsx
// ❌ Vòng lặp vô hạn: object mới mỗi render
function Bad() {
  const [data, setData] = useState([]);
  const options = { page: 1 }; // New reference mỗi render!

  useEffect(() => {
    fetchData(options).then(setData);
  }, [options]); // options luôn "thay đổi"
}

// ✅ Fix: Move object inside effect hoặc dùng useMemo
function Good() {
  const [data, setData] = useState([]);

  useEffect(() => {
    const options = { page: 1 }; // Stable reference bên trong effect
    fetchData(options).then(setData);
  }, []);
}
```

#### ⚠️ Race Condition — Kết Quả Cũ Ghi Đè Kết Quả Mới

```jsx
// ❌ Nếu userId thay đổi nhanh: response của userId cũ có thể về sau
useEffect(() => {
  fetchUser(userId).then(setUser); // Không có cleanup!
}, [userId]);

// ✅ Fix: Cleanup flag hoặc AbortController
useEffect(() => {
  let active = true;
  fetchUser(userId).then(data => {
    if (active) setUser(data);
  });
  return () => { active = false; };
}, [userId]);
```

#### ⚠️ Missing Cleanup — Memory Leak

```jsx
// ❌ Subscription không được hủy → memory leak
useEffect(() => {
  const subscription = store.subscribe(handleChange);
  // Quên return cleanup!
}, []);

// ✅ Luôn return cleanup function
useEffect(() => {
  const subscription = store.subscribe(handleChange);
  return () => subscription.unsubscribe();
}, []);
```

#### ⚠️ Không Dùng async Trực Tiếp

```jsx
// ❌ SẼ LỖI: useEffect callback không được là async
useEffect(async () => {
  const data = await fetchData(); // Warning: Effect returns a Promise
}, []);

// ✅ ĐÚNG: Wrap trong async IIFE
useEffect(() => {
  (async () => {
    const data = await fetchData();
    setData(data);
  })();
}, []);

// ✅ Hoặc tách thành hàm riêng
useEffect(() => {
  const load = async () => {
    const data = await fetchData();
    setData(data);
  };
  load();
}, []);
```

---

### 1.6 Câu Hỏi Phỏng Vấn

**Q1: `useEffect` chạy khi nào — trước hay sau khi browser paint?**

> **Sau** khi browser paint (hiển thị UI lên màn hình). Đây là lý do `useEffect` không gây block UI. Đây cũng là điểm khác biệt với `useLayoutEffect`.

---

**Q2: Tại sao `useEffect` không được phép là `async` function trực tiếp?**

> Vì `useEffect` kỳ vọng callback **return `undefined` hoặc một cleanup function**. Async function luôn return một `Promise` — điều này sẽ khiến React không biết khi nào cleanup cần chạy và gây memory leak.

---

**Q3: Giải thích race condition trong `useEffect` và cách xử lý.**

> **Race condition:** Khi `userId` thay đổi nhanh (A → B), request của A có thể về sau B. Kết quả: UI hiển thị data của A nhưng `userId` đang là B.
> 
> **Fix:** Dùng cleanup flag hoặc `AbortController` để cancel request cũ khi dependency thay đổi.

---

**Q4: Cleanup function trong `useEffect` chạy khi nào?**

> - **Trước mỗi lần effect chạy lại** (khi dependencies thay đổi).
> - **Khi component unmount**.
> - **Không chạy** sau render đầu tiên.

---

**Q5 (Tình huống):** User báo cáo app bị chậm, nghi ngờ memory leak. Bạn kiểm tra `useEffect` như thế nào?

> 1. Tìm các `useEffect` **không có cleanup function** — đặc biệt là event listeners, subscriptions, timers.
> 2. Dùng **Chrome DevTools → Memory tab** → Take Heap Snapshot → so sánh trước/sau navigate.
> 3. Kiểm tra **Detached DOM nodes** — thường xuất hiện khi timer/listener giữ reference đến element đã unmount.
> 4. Dùng **React DevTools Profiler** để xem components mount/unmount patterns.

---

**Q6: Sự khác biệt giữa `useEffect(() => fn, [])` và `componentDidMount`?**

> Về cơ bản tương đương về timing (sau mount). Nhưng trong **React Strict Mode** (development), React sẽ mount → unmount → remount để phát hiện cleanup issues — nghĩa là `useEffect` với `[]` sẽ chạy **2 lần** trong dev mode.

---

## 2. `useLayoutEffect`

### 2.1 Cú Pháp & Timing

```jsx
useLayoutEffect(() => {
  // Chạy SYNCHRONOUSLY sau DOM mutation, TRƯỚC khi browser paint
  const height = element.current.getBoundingClientRect().height;
  // ...
  return () => cleanup();
}, [deps]);
```

**Timeline:**

```
React renders → DOM updated → useLayoutEffect runs → Browser paints → useEffect runs
```

---

### 2.2 Use Cases & Code Examples

#### ✅ DOM Measurement — Tooltip Position

```jsx
function Tooltip({ targetRef, content }) {
  const tooltipRef = useRef(null);
  const [position, setPosition] = useState({ top: 0, left: 0 });

  useLayoutEffect(() => {
    // Đọc DOM TRƯỚC khi user thấy
    const targetRect = targetRef.current.getBoundingClientRect();
    const tooltipRect = tooltipRef.current.getBoundingClientRect();

    setPosition({
      top: targetRect.bottom + window.scrollY,
      left: targetRect.left + (targetRect.width - tooltipRect.width) / 2,
    });
  }, [targetRef]);

  return (
    <div
      ref={tooltipRef}
      style={{ position: 'absolute', top: position.top, left: position.left }}
    >
      {content}
    </div>
  );
}
```

> Dùng `useEffect` sẽ thấy **flash** — tooltip xuất hiện ở vị trí sai rồi nhảy sang vị trí đúng. `useLayoutEffect` đảm bảo position được tính trước khi browser paint.

#### ✅ Animation — FLIP Technique

```jsx
function AnimatedList({ items }) {
  const listRef = useRef(null);

  useLayoutEffect(() => {
    const nodes = listRef.current.querySelectorAll('.item');
    nodes.forEach(node => {
      // Đọc vị trí trước khi re-layout
      const rect = node.getBoundingClientRect();
      node.dataset.prevTop = rect.top;
    });
  });

  // Xử lý FLIP animation sau đây...
}
```

---

### 2.3 Câu Hỏi Phỏng Vấn

**Q1: Khi nào dùng `useLayoutEffect` thay vì `useEffect`?**

> Khi cần **đọc layout từ DOM và thực hiện thay đổi ngay lập tức** trước khi browser paint — như tính toán vị trí, kích thước, tránh visual flash. **Rule:** Bắt đầu bằng `useEffect`, chỉ chuyển sang `useLayoutEffect` khi thấy flickering/flash.

---

**Q2: `useLayoutEffect` có nhược điểm gì không?**

> Vì chạy **synchronously** (blocking), nếu effect tốn nhiều thời gian sẽ **block browser paint** → UI bị đơ. Không nên dùng cho logic nặng như API call hay heavy computation.
>
> Ngoài ra, `useLayoutEffect` **không chạy trên server** (SSR) — sẽ gây warning trong Next.js. Cần dùng `useEffect` hoặc check `typeof window !== 'undefined'`.

---

## 3. `useInsertionEffect`

### 3.1 Use Cases & Code Examples

> ⚠️ Hook **nội bộ** — dành cho tác giả CSS-in-JS libraries (styled-components, emotion). Không dùng trong code ứng dụng thông thường.

**Timing:** Chạy trước `useLayoutEffect`, trước khi React commit DOM.

```jsx
// Dùng trong CSS-in-JS library để inject styles trước khi layout đọc
function useCSS(rule) {
  useInsertionEffect(() => {
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}
```

---

## 4. So Sánh 3 Effect Hooks

```
DOM Update
    ↓
useInsertionEffect  → Inject CSS (TRƯỚC layout đọc DOM)
    ↓
useLayoutEffect     → Đọc/ghi DOM layout (TRƯỚC paint, BLOCKING)
    ↓
[Browser paints UI]
    ↓
useEffect           → Side effects chung (SAU paint, NON-BLOCKING)
```

| Hook | Thời điểm | Blocking? | Dùng cho |
|------|-----------|-----------|---------|
| `useInsertionEffect` | Trước layout | Có | CSS-in-JS injection |
| `useLayoutEffect` | Sau DOM, trước paint | Có | DOM measurement, animation |
| `useEffect` | Sau paint | Không | API, events, subscriptions |

---

🔙 [Context & Ref Hooks](./02-context-ref-hooks.md) | 🔙 [README](./README.md) | ➡️ [Performance Hooks](./04-performance-hooks.md)
