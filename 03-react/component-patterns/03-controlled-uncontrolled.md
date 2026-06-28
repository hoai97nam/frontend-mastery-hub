# 📦 Pattern 3: Controlled & Uncontrolled Components

> Khái niệm **Controlled (Kiểm soát)** và **Uncontrolled (Không kiểm soát)** đề cập đến cách quản lý nguồn dữ liệu (Single Source of Truth) trong các form HTML hoặc các component React có trạng thái thay đổi.

---

## 1. Bản chất & Sự khác biệt

| Đặc điểm | Controlled Component | Uncontrolled Component |
|----------|----------------------|------------------------|
| **Nguồn lưu trữ dữ liệu** | Trạng thái của React (`useState`, `useReducer` hoặc Redux/Zustand) | Bản thân nút DOM trong trình duyệt (truy xuất qua `useRef`) |
| **Giá trị hiển thị** | Nhận từ prop `value` | Đặt ban đầu bằng `defaultValue` |
| **Cơ chế cập nhật** | Gọi hàm callback (như `onChange`) để ghi đè state | Trình duyệt tự cập nhật, React đọc trực tiếp khi cần |
| **Re-render** | Re-render component trên mỗi ký tự nhập vào | Không trigger re-render khi gõ chữ (tối ưu hiệu năng) |
| **Kiểm tra dữ liệu (Validation)** | Dễ dàng kiểm tra theo thời gian thực (Real-time Validation) | Thường chỉ kiểm tra khi submit form |

---

## 2. Code Example thực chiến

Dưới đây là sự đối lập về cách triển khai Form trong React:

### 2.1 Controlled Form (Kiểm soát hoàn toàn)
Phù hợp khi cần validate độ dài mật khẩu ngay khi gõ, format định dạng số điện thoại, hoặc khóa nút Submit nếu form chưa hợp lệ.

```jsx
import React, { useState } from 'react';

function ControlledForm() {
  const [formData, setFormData] = useState({ username: '', email: '' });
  const [error, setError] = useState('');

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));

    // Real-time validation
    if (name === 'username' && value.length < 3) {
      setError('Tên đăng nhập phải chứa ít nhất 3 ký tự');
    } else {
      setError('');
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submit dữ liệu kiểm soát:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username:</label>
        <input name="username" value={formData.username} onChange={handleChange} />
        {error && <span style={{ color: 'red' }}>{error}</span>}
      </div>
      <div>
        <label>Email:</label>
        <input name="email" value={formData.email} onChange={handleChange} />
      </div>
      <button type="submit" disabled={!!error || !formData.username}>Submit</button>
    </form>
  );
}
```

### 2.2 Uncontrolled Form (Không kiểm soát)
Phù hợp cho form cực lớn (tránh lag khi gõ), form đăng nhập đơn giản không cần validate động, hoặc khi tích hợp với các thư viện DOM ngoài.

```jsx
import React, { useRef } from 'react';

function UncontrolledForm() {
  // Tạo Ref để tham chiếu trực tiếp đến nút DOM
  const usernameRef = useRef(null);
  const emailRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    // Đọc trực tiếp value từ DOM khi submit
    const username = usernameRef.current.value;
    const email = emailRef.current.value;
    console.log('Submit dữ liệu không kiểm soát:', { username, email });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username:</label>
        {/* Dùng defaultValue thay vì value để gán giá trị khởi tạo */}
        <input name="username" defaultValue="admin" ref={usernameRef} />
      </div>
      <div>
        <label>Email:</label>
        <input name="email" defaultValue="" ref={emailRef} />
      </div>
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## 3. Các bẫy phỏng vấn & Lỗi phổ biến (Gotchas)

### ⚠️ Lỗi: "A component is changing an uncontrolled input to be controlled"
Lỗi này xảy ra khi bạn gán giá trị ban đầu của state là `undefined` hoặc `null`, sau đó cập nhật nó thành một chuỗi (string) khi có sự kiện nhập liệu.

#### Code lỗi:
```jsx
const [name, setName] = useState(); // Trạng thái ban đầu là undefined (uncontrolled)
...
<input value={name} onChange={(e) => setName(e.target.value)} />
```
Khi React chạy lần đầu, nó thấy `value={undefined}` nên coi input này là Uncontrolled. Sau khi người dùng gõ, state chuyển sang dạng chuỗi, React đột ngột chuyển nó sang Controlled và quăng lỗi cảnh báo ở console.

#### Giải pháp: Luôn khởi tạo giá trị rõ ràng (đặc biệt là chuỗi rỗng `""`):
```jsx
const [name, setName] = useState(""); // ✅ Luôn là string
```

---

### ⚠️ Anti-pattern: Đồng bộ props vào state nội bộ để render
Nhiều lập trình viên có thói quen copy prop nhận được từ cha vào state của con để chỉnh sửa độc lập:

#### Code lỗi:
```jsx
function Child({ initialUser }) {
  const [user, setUser] = useState(initialUser); // ❌ Sai lầm tai hại
  return <div>{user.name}</div>;
}
```
**Tại sao đây là sai lầm?**
Khi component cha thay đổi prop `initialUser` (ví dụ: chuyển từ User A sang User B), component con sẽ **không** được cập nhật lại vì `useState` chỉ khởi tạo một lần duy nhất khi component mount. Con vẫn sẽ hiển thị User A cũ.

#### Giải pháp 1: Chuyển hoàn toàn sang Controlled
Đẩy toàn bộ state lên cha quản lý (Lifting State Up), con chỉ nhận prop và gọi callback thay đổi.

#### Giải pháp 2: Sử dụng khóa `key` để reset trạng thái
Nếu con bắt buộc phải tự quản lý state nhưng cần reset khi cha thay đổi dữ liệu, hãy gán thuộc tính `key` cho component con:
```jsx
// Ở component cha:
<Child initialUser={currentUser} key={currentUser.id} />
```
Khi `currentUser.id` thay đổi, React sẽ hủy (unmount) instance cũ của component `Child` và khởi tạo lại toàn bộ từ đầu, chạy lại `useState` với dữ liệu mới.
