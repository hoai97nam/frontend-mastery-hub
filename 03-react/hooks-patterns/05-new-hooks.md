# 🆕 React 19+ Hooks — `use`, `useOptimistic`, `useActionState`, `useFormStatus`

> Nhóm hook mới trong **React 19** — hỗ trợ **Server Actions**, async rendering, và Optimistic UI.

---

## Mục Lục

1. [use](#1-use)
   - [Cú pháp & Use Cases](#11-cú-pháp--use-cases)
   - [Code Examples](#12-code-examples)
   - [Câu Hỏi Phỏng Vấn](#13-câu-hỏi-phỏng-vấn)
2. [useOptimistic](#2-useoptimistic)
   - [Cú pháp & Use Cases](#21-cú-pháp--use-cases)
   - [Code Examples](#22-code-examples)
   - [Câu Hỏi Phỏng Vấn](#23-câu-hỏi-phỏng-vấn)
3. [useActionState](#3-useactionstate)
   - [Cú pháp & Use Cases](#31-cú-pháp--use-cases)
   - [Code Examples](#32-code-examples)
   - [Câu Hỏi Phỏng Vấn](#33-câu-hỏi-phỏng-vấn)
4. [useFormStatus](#4-useformstatus)
   - [Cú pháp & Use Cases](#41-cú-pháp--use-cases)
   - [Code Examples](#42-code-examples)
   - [Câu Hỏi Phỏng Vấn](#43-câu-hỏi-phỏng-vấn)
5. [Tổng Kết & Mental Model](#5-tổng-kết--mental-model)

---

> ⚠️ **Yêu cầu:** React 19+. Các hook này được thiết kế để dùng với **React Server Components** và **Server Actions** (Next.js App Router), nhưng cũng hoạt động trong Client Components.

---

## 1. `use`

### 1.1 Cú Pháp & Use Cases

```jsx
// Đọc Promise (trong Suspense boundary)
const data = use(promise);

// Đọc Context (linh hoạt hơn useContext — có thể trong điều kiện)
const theme = use(ThemeContext);
```

**Điểm đặc biệt:** Không giống các hook khác, `use` **có thể được gọi trong điều kiện** (if/else, loop) — nhưng **không thể gọi sau early return** (vẫn phải theo Rules of Hooks ở mức component).

**Use Cases:**
- Đọc Promise được truyền từ Server Component
- Conditional context reading
- Streaming data với Suspense

---

### 1.2 Code Examples

#### ✅ Đọc Promise với Suspense

```jsx
// Server Component — tạo promise và truyền xuống
async function ProductPage({ id }) {
  const productPromise = fetchProduct(id); // Không await!
  return (
    <Suspense fallback={<Skeleton />}>
      <ProductDetails promise={productPromise} />
    </Suspense>
  );
}

// Client Component — đọc promise bằng `use`
'use client';
function ProductDetails({ promise }) {
  const product = use(promise); // Suspend cho đến khi resolve

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price.toLocaleString('vi-VN')}đ</p>
    </div>
  );
}
```

#### ✅ Đọc Context có Điều Kiện

```jsx
function Greeting({ showUsername }) {
  // ✅ useContext không được phép trong if
  // ❌ if (showUsername) { const user = useContext(UserContext); }

  // ✅ use() có thể trong điều kiện
  if (showUsername) {
    const user = use(UserContext);
    return <h1>Xin chào, {user.name}!</h1>;
  }
  return <h1>Xin chào!</h1>;
}
```

#### ✅ Xử Lý Error với ErrorBoundary

```jsx
function UserProfile({ userPromise }) {
  // Nếu Promise reject → ErrorBoundary bắt, không crash app
  const user = use(userPromise);
  return <ProfileCard user={user} />;
}

function Page({ id }) {
  const userPromise = fetchUser(id);
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<Skeleton />}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

### 1.3 Câu Hỏi Phỏng Vấn

**Q1: `use` khác `useEffect` để fetch data như thế nào?**

> - `useEffect`: Fetch **sau khi render** (client-side), cần quản lý loading/error state thủ công.
> - `use`: Fetch **trong render** với Suspense — component tự suspend cho đến khi data sẵn sàng. Không cần `loading` state, Suspense fallback lo phần đó.

---

**Q2: Tại sao `use` có thể gọi trong điều kiện còn các hook khác thì không?**

> React hooks thông thường phụ thuộc vào **thứ tự gọi** để tracking state giữa các render. `use` không lưu state — nó là cơ chế "suspend and resume" thuần túy, nên không cần maintain call order.

---

**Q3 (Tình huống):** Bạn đang xây dựng trang sản phẩm với Next.js App Router. Data fetch nên xảy ra ở Server hay Client? Dùng hook nào?

> **Server Component + `use`:**
> ```jsx
> // Fetch ở Server Component (không block, không bundle size)
> async function Page({ id }) {
>   const productPromise = getProduct(id); // Không await
>   return (
>     <Suspense fallback={<Loading />}>
>       <ProductDetail promise={productPromise} />
>     </Suspense>
>   );
> }
> // Client Component đọc bằng use()
> ```
> Ưu điểm: Không gửi fetch logic xuống client, streaming tự nhiên với Suspense.

---

## 2. `useOptimistic`

### 2.1 Cú Pháp & Use Cases

```jsx
const [optimisticState, addOptimistic] = useOptimistic(
  actualState,
  (currentState, optimisticValue) => {
    // Trả về state "lạc quan" — assume thành công trước
    return { ...currentState, ...optimisticValue };
  }
);
```

- `optimisticState` — State hiển thị cho user (có thể là state "giả").
- `addOptimistic(value)` — Cập nhật optimistic state ngay lập tức.
- Khi async action hoàn tất → tự động revert về `actualState` hoặc giữ state mới.

**Use Cases:**
- Like / unlike button
- Comment / delete
- Reorder list
- Toggle settings

---

### 2.2 Code Examples

#### ✅ Like Button — Optimistic UI

```jsx
function LikeButton({ postId, initialLikes, initialLiked }) {
  const [liked, setLiked] = useState(initialLiked);
  const [likes, setLikes] = useState(initialLikes);

  const [optimisticLiked, addOptimisticLiked] = useOptimistic(
    liked,
    (_, newLiked) => newLiked
  );

  const [optimisticCount, addOptimisticCount] = useOptimistic(
    likes,
    (_, newCount) => newCount
  );

  async function handleLike() {
    const newLiked = !optimisticLiked;
    const newCount = newLiked ? optimisticCount + 1 : optimisticCount - 1;

    // Cập nhật UI ngay lập tức (optimistic)
    addOptimisticLiked(newLiked);
    addOptimisticCount(newCount);

    try {
      // Gọi API thực sự
      await toggleLike(postId);
      // Nếu thành công: cập nhật actual state
      setLiked(newLiked);
      setLikes(newCount);
    } catch {
      // Nếu thất bại: useOptimistic tự revert về liked/likes cũ
    }
  }

  return (
    <button onClick={handleLike}>
      {optimisticLiked ? '❤️' : '🤍'} {optimisticCount}
    </button>
  );
}
```

#### ✅ Todo List — Optimistic Add/Delete

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const [optimisticTodos, addOptimistic] = useOptimistic(
    todos,
    (state, action) => {
      switch (action.type) {
        case 'ADD':
          return [...state, { ...action.todo, pending: true }];
        case 'DELETE':
          return state.filter(t => t.id !== action.id);
        default:
          return state;
      }
    }
  );

  async function addTodo(text) {
    const tempTodo = { id: Date.now(), text };
    addOptimistic({ type: 'ADD', todo: tempTodo }); // Hiện ngay

    const savedTodo = await api.createTodo(text);
    setTodos(prev => [...prev, savedTodo]); // Cập nhật với ID thật từ server
  }

  async function deleteTodo(id) {
    addOptimistic({ type: 'DELETE', id }); // Xóa ngay trên UI

    await api.deleteTodo(id);
    setTodos(prev => prev.filter(t => t.id !== id));
  }

  return (
    <ul>
      {optimisticTodos.map(todo => (
        <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
          {todo.text}
          <button onClick={() => deleteTodo(todo.id)}>Xóa</button>
        </li>
      ))}
    </ul>
  );
}
```

---

### 2.3 Câu Hỏi Phỏng Vấn

**Q1: `useOptimistic` xử lý lỗi như thế nào khi API fail?**

> Khi async action kết thúc (dù thành công hay thất bại), `useOptimistic` **tự động revert** `optimisticState` về `actualState`. Vì vậy, nếu API fail và bạn không `setActualState`, UI tự động trở về trạng thái trước — không cần try/catch phức tạp.

---

**Q2: Sự khác biệt giữa `useOptimistic` và chỉ `setState` ngay lập tức?**

> - `setState` thủ công: Bạn phải tự rollback nếu fail → logic phức tạp.
> - `useOptimistic`: React tự động quản lý revert — optimistic state là "tạm thời" cho đến khi actual state thay đổi.

---

## 3. `useActionState`

### 3.1 Cú Pháp & Use Cases

```jsx
const [state, dispatch, isPending] = useActionState(
  async (prevState, formData) => {
    // Server Action hoặc async function
    // Trả về state mới
    return newState;
  },
  initialState
);
```

- `state` — State hiện tại (bắt đầu từ `initialState`).
- `dispatch` — Dùng như form `action` hoặc gọi trực tiếp.
- `isPending` — `true` khi action đang chạy.

**Use Cases:**
- Form submission với Server Actions
- Inline editing
- Multi-step actions

---

### 3.2 Code Examples

#### ✅ Form Đăng Nhập với Server Action

```jsx
// actions.js (Server Action)
'use server';
import { redirect } from 'next/navigation';

export async function loginAction(prevState, formData) {
  const email = formData.get('email');
  const password = formData.get('password');
  let isSuccess = false;

  try {
    const user = await auth.signIn(email, password);
    isSuccess = true;
  } catch (error) {
    return { error: 'Email hoặc mật khẩu không đúng', email };
  }

  if (isSuccess) {
    redirect('/dashboard'); // Gọi ngoài try/catch để tránh catch lỗi redirect nội bộ
  }
}

// LoginForm.jsx (Client Component)
'use client';
import { useActionState } from 'react';
import { loginAction } from './actions';

function LoginForm() {
  const [state, dispatch, isPending] = useActionState(loginAction, null);

  return (
    <form action={dispatch}>
      <div>
        <label>Email</label>
        <input
          type="email"
          name="email"
          defaultValue={state?.email} {/* Giữ lại email nếu lỗi */}
          required
        />
      </div>
      <div>
        <label>Mật khẩu</label>
        <input type="password" name="password" required />
      </div>

      {state?.error && (
        <p style={{ color: 'red' }}>{state.error}</p>
      )}

      <button type="submit" disabled={isPending}>
        {isPending ? 'Đang đăng nhập...' : 'Đăng nhập'}
      </button>
    </form>
  );
}
```

#### ✅ Counter với useActionState

```jsx
function Counter() {
  const [count, increment] = useActionState(
    async (prevCount) => {
      await new Promise(r => setTimeout(r, 500)); // Giả lập async
      return prevCount + 1;
    },
    0
  );

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => increment()}>+1</button>
    </div>
  );
}
```

#### ✅ Validation Phức Tạp

```jsx
function RegisterForm() {
  const [state, formAction, isPending] = useActionState(
    async (prevState, formData) => {
      const name = formData.get('name');
      const email = formData.get('email');

      // Client-side validation trong action
      const errors = {};
      if (!name || name.length < 2) errors.name = 'Tên phải có ít nhất 2 ký tự';
      if (!email?.includes('@')) errors.email = 'Email không hợp lệ';

      if (Object.keys(errors).length > 0) {
        return { errors, values: { name, email } };
      }

      try {
        await api.register({ name, email });
        return { success: true };
      } catch (err) {
        return { errors: { general: err.message }, values: { name, email } };
      }
    },
    { errors: null, success: false }
  );

  if (state.success) return <p>🎉 Đăng ký thành công!</p>;

  return (
    <form action={formAction}>
      <input name="name" defaultValue={state.values?.name} />
      {state.errors?.name && <span>{state.errors.name}</span>}

      <input name="email" defaultValue={state.values?.email} />
      {state.errors?.email && <span>{state.errors.email}</span>}

      {state.errors?.general && <p>{state.errors.general}</p>}

      <button type="submit" disabled={isPending}>
        {isPending ? 'Đang xử lý...' : 'Đăng ký'}
      </button>
    </form>
  );
}
```

---

### 3.3 Câu Hỏi Phỏng Vấn

**Q1: `useActionState` thay thế `useState` + `useEffect` + `fetch` như thế nào?**

> Trước React 19, fetch trong form thường là:
> ```jsx
> const [loading, setLoading] = useState(false);
> const [error, setError] = useState(null);
> const handleSubmit = async (e) => {
>   e.preventDefault();
>   setLoading(true);
>   try { await submitForm(data); } catch { setError(...); }
>   setLoading(false);
> };
> ```
> `useActionState` gộp tất cả vào một API: không cần `e.preventDefault()`, không cần manage loading/error thủ công, progressive enhancement tốt hơn.

---

**Q2: `useActionState` có chạy được mà không cần Server Actions không?**

> Có — action function có thể là **bất kỳ async function nào**, không nhất thiết phải là Server Action. Nó hoạt động tốt trong thuần Client Components.

---

## 4. `useFormStatus`

### 4.1 Cú Pháp & Use Cases

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Đang gửi...' : 'Gửi'}
    </button>
  );
}
```

- `pending` — `true` khi form đang submit.
- `data` — `FormData` của lần submit đó.
- Phải là **child component** của `<form>` — không gọi trong chính component chứa form.

**Use Cases:**
- Submit button với loading state
- Disable form fields trong khi submit
- Loading spinner trong form

---

### 4.2 Code Examples

#### ✅ Submit Button Component Tái Sử Dụng

```jsx
// Reusable component — đặt trong bất kỳ form nào
function SubmitButton({ label = 'Gửi', loadingLabel = 'Đang gửi...' }) {
  const { pending } = useFormStatus();

  return (
    <button
      type="submit"
      disabled={pending}
      style={{
        opacity: pending ? 0.7 : 1,
        cursor: pending ? 'not-allowed' : 'pointer',
      }}
    >
      {pending ? (
        <>
          <Spinner size={16} />
          {loadingLabel}
        </>
      ) : label}
    </button>
  );
}

// Dùng trong form
function ContactForm() {
  const [state, action] = useActionState(submitContact, null);
  return (
    <form action={action}>
      <input name="email" type="email" />
      <textarea name="message" />
      <SubmitButton label="Gửi liên hệ" loadingLabel="Đang gửi..." />
    </form>
  );
}
```

#### ✅ Disable Toàn Bộ Form Khi Submitting

```jsx
function FormFields() {
  const { pending } = useFormStatus();
  return (
    <fieldset disabled={pending}>
      <input name="name" />
      <input name="email" />
      <select name="role">
        <option value="user">User</option>
        <option value="admin">Admin</option>
      </select>
    </fieldset>
  );
}

function UserForm() {
  const [state, action] = useActionState(updateUser, null);
  return (
    <form action={action}>
      <FormFields /> {/* Tự disable khi pending */}
      <SubmitButton />
    </form>
  );
}
```

#### ✅ Optimistic Preview khi Submit

```jsx
function MessageForm() {
  const { pending, data } = useFormStatus();
  const pendingMessage = data?.get('message');

  return (
    <>
      {pending && (
        <div style={{ opacity: 0.5, fontStyle: 'italic' }}>
          Đang gửi: {pendingMessage}...
        </div>
      )}
      {/* Input field */}
      <input name="message" />
    </>
  );
}
```

---

### 4.3 Câu Hỏi Phỏng Vấn

**Q1: Tại sao `useFormStatus` phải gọi trong child component, không phải chính component chứa form?**

> Vì `useFormStatus` cần được đặt trong **context của form**. React tìm form ancestor gần nhất trong component tree. Nếu gọi trong cùng component chứa `<form>`, React không thể biết form nào bạn đang refer đến (đặc biệt khi có nhiều form).

---

**Q2: Kết hợp `useFormStatus` và `useActionState` như thế nào?**

> - `useActionState` → ở component chứa form — quản lý state và action.
> - `useFormStatus` → ở child components (button, fieldset) — đọc trạng thái pending.
> - Hai hook độc lập nhau, phối hợp qua form context.

---

## 5. Tổng Kết & Mental Model

### Bản Đồ React 19+ Hooks

```
                    ┌─────────────────────┐
                    │    Server Actions    │
                    └──────────┬──────────┘
                               │
            ┌──────────────────┼──────────────────┐
            ▼                  ▼                  ▼
    useActionState        useFormStatus      useOptimistic
   (State + dispatch)   (Pending trong      (Optimistic UI
                          child form)        Update)
            │
            └── Streaming / Suspense
                        │
                    use(promise)
                  (Đọc async data)
```

### Khi Nào Dùng Hook Nào

| Hook | Dùng khi |
|------|----------|
| `use` | Đọc Promise từ Server Component, conditional context |
| `useOptimistic` | Muốn UI phản hồi ngay trước khi server confirm |
| `useActionState` | Form submission với Server Actions / async logic |
| `useFormStatus` | Button/field cần biết form đang submit không |

### So Sánh Cách Xử Lý Form Trước/Sau React 19

**Trước React 19:**
```jsx
function OldForm() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    try {
      await submitData(new FormData(e.target));
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" />
      <button type="submit" disabled={loading}>
        {loading ? 'Loading...' : 'Submit'}
      </button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

**React 19:**
```jsx
function NewForm() {
  const [state, action, isPending] = useActionState(submitAction, null);

  return (
    <form action={action}>
      <input name="email" />
      <SubmitButton /> {/* useFormStatus bên trong */}
      {state?.error && <p>{state.error}</p>}
    </form>
  );
}
```

---

🔙 [Performance Hooks](./04-performance-hooks.md) | 🔙 [README](./README.md)
