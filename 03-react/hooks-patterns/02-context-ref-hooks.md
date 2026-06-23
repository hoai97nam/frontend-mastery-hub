# 🔗 Context & Ref Hooks — `useContext`, `useRef`, `useImperativeHandle`

> Nhóm hook dùng để **chia sẻ dữ liệu** qua component tree và **thao tác trực tiếp với DOM/instances**.

---

## Mục Lục

1. [useContext](#1-usecontext)
   - [Cú pháp & Setup](#11-cú-pháp--setup)
   - [Use Cases](#12-use-cases)
   - [Code Examples](#13-code-examples)
   - [Gotchas & Pitfalls](#14-gotchas--pitfalls)
   - [Câu Hỏi Phỏng Vấn](#15-câu-hỏi-phỏng-vấn)
2. [useRef](#2-useref)
   - [Cú pháp](#21-cú-pháp)
   - [Use Cases](#22-use-cases)
   - [Code Examples](#23-code-examples)
   - [Câu Hỏi Phỏng Vấn](#24-câu-hỏi-phỏng-vấn)
3. [useImperativeHandle](#3-useimperativehandle)
   - [Cú pháp](#31-cú-pháp)
   - [Use Cases & Code Examples](#32-use-cases--code-examples)
   - [Câu Hỏi Phỏng Vấn](#33-câu-hỏi-phỏng-vấn)

---

## 1. `useContext`

### 1.1 Cú Pháp & Setup

```jsx
// Bước 1: Tạo Context
const ThemeContext = createContext(defaultValue);

// Bước 2: Cung cấp giá trị qua Provider
function App() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <ComponentTree />
    </ThemeContext.Provider>
  );
}

// Bước 3: Consume trong bất kỳ component con nào
function Button() {
  const { theme, setTheme } = useContext(ThemeContext);
  return <button className={theme}>Click me</button>;
}
```

---

### 1.2 Use Cases

| Use Case | Mô tả |
|----------|-------|
| Theme / Dark mode | Áp dụng theme toàn app |
| Auth / User session | Lưu thông tin user đăng nhập |
| Language / i18n | Ngôn ngữ hiện tại |
| Toast / Notification | Global notification system |
| Tránh prop drilling | Truyền data qua nhiều cấp |

---

### 1.3 Code Examples

#### ✅ Pattern Kinh Điển — Theme Context

```jsx
// theme-context.js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggle = () => setTheme(prev => prev === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook để dễ dùng + kiểm soát lỗi
export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
}
```

```jsx
// App.jsx
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

// Bất kỳ component con nào
function Header() {
  const { theme, toggle } = useTheme();
  return (
    <header className={`header--${theme}`}>
      <button onClick={toggle}>Toggle Theme</button>
    </header>
  );
}
```

#### ✅ Auth Context — Pattern Thực Tế

```jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Check existing session
    const token = localStorage.getItem('token');
    if (token) {
      validateToken(token)
        .then(setUser)
        .finally(() => setIsLoading(false));
    } else {
      setIsLoading(false);
    }
  }, []);

  const login = async (credentials) => {
    const userData = await authAPI.login(credentials);
    localStorage.setItem('token', userData.token);
    setUser(userData.user);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  if (isLoading) return <Spinner />;

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be inside AuthProvider');
  return ctx;
};
```

#### ✅ Context + Reducer — Pattern Mạnh Nhất

```jsx
const CartContext = createContext(null);
const CartDispatchContext = createContext(null);

// Tách Context cho value và dispatch để tránh re-render không cần thiết
export function CartProvider({ children }) {
  const [cart, dispatch] = useReducer(cartReducer, []);

  return (
    <CartContext.Provider value={cart}>
      <CartDispatchContext.Provider value={dispatch}>
        {children}
      </CartDispatchContext.Provider>
    </CartContext.Provider>
  );
}

export const useCart = () => useContext(CartContext);
export const useCartDispatch = () => useContext(CartDispatchContext);
```

---

### 1.4 Gotchas & Pitfalls

#### ⚠️ Context Re-render All Consumers

```jsx
// ❌ Mỗi lần App re-render, object mới được tạo → tất cả consumers re-render
function App() {
  const [count, setCount] = useState(0);
  return (
    <MyContext.Provider value={{ theme: 'dark', count }}>
      {/* tất cả consumers đều re-render khi count thay đổi */}
    </MyContext.Provider>
  );
}

// ✅ Tách context nếu data độc lập nhau
<ThemeContext.Provider value={themeValue}>
  <CountContext.Provider value={countValue}>
    ...
  </CountContext.Provider>
</ThemeContext.Provider>

// ✅ Hoặc memo hóa value
const value = useMemo(() => ({ theme, setTheme }), [theme]);
<ThemeContext.Provider value={value}>
```

---

### 1.5 Câu Hỏi Phỏng Vấn

**Q1: `useContext` có khiến component re-render không? Khi nào?**

> Có — bất cứ khi nào giá trị `value` trong `Provider` thay đổi (theo `Object.is`), **tất cả** component đang dùng `useContext` với context đó sẽ re-render, bất kể component đó có `React.memo` hay không.

---

**Q2: Làm thế nào để tránh re-render không cần thiết khi dùng Context?**

> 3 cách chính:
> 1. **Tách Context** — Chia nhỏ thành nhiều context (state + dispatch riêng).
> 2. **Memoize value** — Dùng `useMemo` cho value truyền vào Provider.
> 3. **Colocate Provider** — Đặt Provider gần nhất với nơi cần, không nhất thiết phải ở root.

---

**Q3: Khi nào không nên dùng Context?**

> - Khi data thay đổi rất thường xuyên (mỗi keystroke) → gây re-render toàn bộ consumers.
> - Khi chỉ cần truyền props qua 1-2 cấp → prop drilling vẫn ổn.
> - Khi cần caching, mutation, subscriptions phức tạp → dùng React Query, Zustand, hoặc Redux.

---

**Q4 (Tình huống):** App bị chậm, bạn phát hiện UserContext re-render toàn bộ component tree mỗi giây. Nguyên nhân và cách fix?

> **Nguyên nhân:** Value truyền vào Provider là inline object `{ user, setUser }` → tạo reference mới mỗi render.
> 
> **Fix:**
> ```jsx
> const value = useMemo(() => ({ user, setUser }), [user]);
> <UserContext.Provider value={value}>
> ```
> Hoặc tách thành `UserContext` (data) và `UserActionsContext` (dispatch) riêng biệt.

---

## 2. `useRef`

### 2.1 Cú Pháp

```jsx
const ref = useRef(initialValue);
// ref.current → giá trị có thể mutate tự do, không gây re-render
```

---

### 2.2 Use Cases

| Use Case | Mô tả |
|----------|-------|
| DOM access | Focus input, scroll, measure element |
| Mutable value | Timer ID, previous value, subscription |
| Tránh stale closure | Giữ giá trị mới nhất trong callback |
| Tracking mount | `isMounted` flag để tránh memory leak |
| Lưu instance | WebSocket, IntersectionObserver, v.v. |

---

### 2.3 Code Examples

#### ✅ DOM Access — Auto Focus

```jsx
function SearchBar() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} placeholder="Tìm kiếm..." />;
}
```

#### ✅ Timer / Interval Management

```jsx
function Stopwatch() {
  const [time, setTime] = useState(0);
  const intervalRef = useRef(null);

  const start = () => {
    if (intervalRef.current) return;
    intervalRef.current = setInterval(() => {
      setTime(prev => prev + 1);
    }, 1000);
  };

  const stop = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };

  useEffect(() => () => clearInterval(intervalRef.current), []);

  return (
    <div>
      <p>Time: {time}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

#### ✅ Previous Value Pattern

```jsx
function usePrevious(value) {
  const prevRef = useRef();
  useEffect(() => {
    prevRef.current = value;
  }); // Chạy sau mỗi render — lưu giá trị trước đó

  return prevRef.current;
}

function Component({ count }) {
  const prevCount = usePrevious(count);
  return <p>Trước: {prevCount} | Hiện tại: {count}</p>;
}
```

#### ✅ Tránh Stale Closure trong Event Listener

```jsx
function EventLogger() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  // Luôn sync ref với state mới nhất
  useEffect(() => {
    countRef.current = count;
  }, [count]);

  useEffect(() => {
    const handler = () => {
      // countRef.current luôn là giá trị mới nhất
      console.log('Current count:', countRef.current);
    };
    window.addEventListener('keypress', handler);
    return () => window.removeEventListener('keypress', handler);
  }, []); // deps rỗng — handler không bị stale

  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}
```

#### ✅ Scroll to Bottom

```jsx
function ChatBox({ messages }) {
  const bottomRef = useRef(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  return (
    <div className="chat-box">
      {messages.map(msg => <Message key={msg.id} {...msg} />)}
      <div ref={bottomRef} />
    </div>
  );
}
```

---

### 2.4 Câu Hỏi Phỏng Vấn

**Q1: Sự khác biệt giữa `useRef` và `useState` là gì?**

| | `useRef` | `useState` |
|---|----------|------------|
| Re-render khi thay đổi? | **Không** | **Có** |
| Persist giữa renders? | **Có** | **Có** |
| Mutation | Trực tiếp (`ref.current = x`) | Qua `setState` |
| Dùng cho | DOM, mutable vars | UI state |

---

**Q2: Tại sao không nên dùng `useRef` để lưu UI state?**

> Vì thay đổi `ref.current` **không trigger re-render** — UI sẽ không cập nhật. `useRef` phù hợp cho các giá trị "ngầm" không cần hiển thị trực tiếp lên màn hình.

---

**Q3 (Tình huống):** Bạn có một event listener đăng ký trong `useEffect` với deps `[]`. Bên trong handler cần dùng state, nhưng bạn thấy state luôn là giá trị ban đầu (stale closure). Bạn fix như thế nào?

> Dùng **ref pattern**:
> ```jsx
> const stateRef = useRef(state);
> useEffect(() => { stateRef.current = state; }, [state]);
> // Trong handler: stateRef.current luôn là giá trị mới nhất
> ```

---

**Q4: `ref` có thể dùng với `forwardRef` như thế nào?**

> `forwardRef` cho phép component cha truyền `ref` xuống để truy cập DOM node hoặc instance của component con:
> ```jsx
> const Input = forwardRef((props, ref) => (
>   <input ref={ref} {...props} />
> ));
> // Parent:
> const inputRef = useRef();
> <Input ref={inputRef} />
> inputRef.current.focus(); // Truy cập DOM trực tiếp
> ```

---

## 3. `useImperativeHandle`

### 3.1 Cú Pháp

```jsx
useImperativeHandle(ref, () => ({
  // Expose API tùy chỉnh thay vì toàn bộ DOM node
  focus() { ... },
  scrollTo(position) { ... },
}), [deps]);
```

> Phải dùng cùng `forwardRef`.

---

### 3.2 Use Cases & Code Examples

#### ✅ Custom Input với Exposed API

```jsx
const FancyInput = forwardRef(function FancyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current.focus();
    },
    clear() {
      inputRef.current.value = '';
    },
    getValue() {
      return inputRef.current.value;
    },
  }));

  return <input ref={inputRef} className="fancy-input" {...props} />;
});

// Parent sử dụng
function Form() {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    const value = inputRef.current.getValue();
    console.log('Submitted:', value);
    inputRef.current.clear();
  };

  return (
    <div>
      <FancyInput ref={inputRef} placeholder="Nhập tên..." />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

#### ✅ Video Player với Imperative API

```jsx
const VideoPlayer = forwardRef(function VideoPlayer({ src }, ref) {
  const videoRef = useRef(null);

  useImperativeHandle(ref, () => ({
    play()  { videoRef.current.play(); },
    pause() { videoRef.current.pause(); },
    seek(time) { videoRef.current.currentTime = time; },
  }));

  return <video ref={videoRef} src={src} />;
});
```

---

### 3.3 Câu Hỏi Phỏng Vấn

**Q1: Khi nào dùng `useImperativeHandle` thay vì chỉ dùng `forwardRef`?**

> - Khi muốn **giới hạn API** expose ra ngoài (không để parent tự ý truy cập toàn bộ DOM).
> - Khi muốn expose **các method tùy chỉnh** (không phải DOM methods).
> - **Nguyên tắc:** Ưu tiên props và state trước. `useImperativeHandle` chỉ dùng khi thực sự cần imperative interaction (focus management, animation libraries, v.v.).

---

**Q2 (Tình huống):** Team muốn xây dựng component `<DatePicker>` tái sử dụng, cho phép parent gọi `open()` và `close()` từ bên ngoài. Bạn implement như thế nào?

> Dùng `forwardRef` + `useImperativeHandle`:
> ```jsx
> const DatePicker = forwardRef((props, ref) => {
>   const [isOpen, setIsOpen] = useState(false);
>
>   useImperativeHandle(ref, () => ({
>     open:  () => setIsOpen(true),
>     close: () => setIsOpen(false),
>   }));
>
>   return <div>{isOpen && <Calendar />}</div>;
> });
> ```

---

## 📎 Tổng Kết

```
useContext          → Chia sẻ data qua nhiều cấp, tránh prop drilling
useRef             → DOM access, mutable vars, tránh stale closure
useImperativeHandle → Expose custom API cho component con (dùng khi thực sự cần)
```

---

🔙 [State Hooks](./01-state-hooks.md) | 🔙 [README](./README.md) | ➡️ [Effect Hooks](./03-effect-hooks.md)
