# 📦 Pattern 4: Higher-Order Component (HOC)

> **Higher-Order Component (HOC - Component Bọc Cấp Cao)** là một mẫu thiết kế nâng cao trong React dùng để tái sử dụng logic của component. Bản chất HOC không phải là một React API, mà là một mẫu thiết kế hình thành từ tính chất functional của React.

---

## 1. Khái niệm & Định nghĩa

HOC là một **hàm (function)** nhận tham số đầu vào là một component và trả về một component mới đã được nâng cấp hoặc bổ sung thêm tính năng.

### Công thức:
```javascript
const EnhancedComponent = withFeature(WrappedComponent);
```

### Ưu điểm:
- **Tái sử dụng logic chung:** Tách biệt các tác vụ lặp đi lặp lại như: kiểm tra đăng nhập (authorization), hiển thị màn hình chờ (loading animation), gửi log theo dõi (analytics), v.v.
- **Giữ component con sạch sẽ:** Component con chỉ tập trung vào việc hiển thị giao diện dựa trên props nhận được.

---

## 2. Code Example thực chiến: `withAuth` & `withLoading`

Dưới đây là cách xây dựng bộ đôi HOC phổ biến trong các dự án thực tế:

```jsx
import React, { useState, useEffect } from 'react';

// Giả lập kiểm tra quyền đăng nhập
const isAuthenticated = () => !!localStorage.getItem('token');

// 1. HOC: withAuth - Bảo vệ router / trang chỉ dành cho user đã đăng nhập
export function withAuth(WrappedComponent) {
  return function WithAuthComponent(props) {
    const [auth, setAuth] = useState(false);
    const [checking, setChecking] = useState(true);

    useEffect(() => {
      // Giả lập gọi API check token
      setTimeout(() => {
        if (isAuthenticated()) {
          setAuth(true);
        } else {
          setAuth(false);
          // Redirect về login ở đây nếu cần
          window.location.href = '/login';
        }
        setChecking(false);
      }, 500);
    }, []);

    if (checking) return <div>Đang xác thực tài khoản...</div>;
    if (!auth) return null;

    return <WrappedComponent {...props} />;
  };
}

// 2. HOC: withLoading - Hiển thị spinner khi đang tải dữ liệu
export function withLoading(WrappedComponent) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return (
        <div className="spinner-container" style={{ textAlign: 'center', padding: '20px' }}>
          <div className="spinner" style={{ border: '4px solid #f3f3f3', borderTop: '4px solid #3498db', borderRadius: '50%', width: '30px', height: '30px', animation: 'spin 1s linear infinite', margin: '0 auto' }} />
          <p>Đang tải dữ liệu...</p>
        </div>
      );
    }
    return <WrappedComponent {...props} />;
  };
}

// 3. Component gốc hiển thị thông tin Dashboard
function Dashboard({ user }) {
  return (
    <div style={{ padding: '20px', border: '1px solid green' }}>
      <h1>Chào mừng Admin {user} đã quay trở lại!</h1>
      <p>Nội dung nhạy cảm của hệ thống nằm ở đây.</p>
    </div>
  );
}

// 4. Áp dụng HOC lồng nhau (Composition)
// Vừa bắt đăng nhập, vừa hỗ trợ hiển thị loading khi nạp dữ liệu
const SecureDashboard = withAuth(withLoading(Dashboard));

export default SecureDashboard;
```

---

## 3. Các quy tắc & Bẫy kinh điển (HOC Pitfalls)

### ⚠️ Lỗi: Không chuyển tiếp Refs (Refs Forwarding)
Theo mặc định, refs sẽ không được truyền xuyên qua HOC vì `ref` không phải là một prop bình thường, nó được React xử lý đặc biệt giống như `key`. Nếu bạn gán `ref` cho một Component đã bọc qua HOC, `ref` đó sẽ trỏ vào Component bọc ngoài cùng thay vì trỏ vào component gốc bên trong.

#### Giải pháp: Sử dụng `React.forwardRef`
```jsx
export function withLog(WrappedComponent) {
  class LogProps extends React.Component {
    render() {
      const { forwardedRef, ...rest } = this.props;
      // Trả lại ref gốc cho WrappedComponent
      return <WrappedComponent ref={forwardedRef} {...rest} />;
    }
  }

  // Sử dụng React.forwardRef để bắt lấy ref và truyền qua prop forwardedRef
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```

### ⚠️ Lỗi: Gọi HOC bên trong hàm render
Không bao giờ được tạo hoặc gọi HOC bên trong phương thức render của một component khác.

#### Code lỗi:
```jsx
function App() {
  // ❌ SAI: Mỗi lần App render, SecureDashboard sẽ được tạo mới hoàn toàn
  const SecureDashboard = withAuth(Dashboard);
  return <SecureDashboard />;
}
```
**Hậu quả:** React sẽ hủy bỏ hoàn toàn component cũ và mount component mới trên mỗi lượt render. Điều này làm mất toàn bộ state nội bộ của cây component con đó, đồng thời làm giảm hiệu năng nghiêm trọng (gây giật lag UI).

#### Giải pháp: Luôn định nghĩa HOC ở bên ngoài component:
```jsx
// ✅ ĐÚNG: Chỉ định nghĩa 1 lần duy nhất
const SecureDashboard = withAuth(Dashboard);

function App() {
  return <SecureDashboard />;
}
```

### ⚠️ Lỗi: Mất các Static Methods của Component gốc
Khi bọc một Component bằng HOC, component mới trả về sẽ không có các phương thức static được định nghĩa ở component gốc.
```jsx
Dashboard.someStaticMethod = () => { ... };
const SecureDashboard = withAuth(Dashboard);
console.log(SecureDashboard.someStaticMethod); // ❌ undefined!
```
#### Giải pháp:
Bạn phải tự copy các phương thức đó sang component mới trước khi return, hoặc sử dụng thư viện `hoist-non-react-statics`.

---

## 4. HOC vs Custom Hooks

Dù Custom Hooks đã trở thành tiêu chuẩn vàng để tái sử dụng logic trong React hiện đại, HOC vẫn giữ vị trí độc tôn trong các tình huống:
- **Tách biệt hoàn toàn (Zero component modification):** Bọc trang bằng Guard (như `withAuth`) mà không cần thay đổi bất cứ dòng code nào bên trong file Component gốc.
- **Hoạt động với Class Components:** Nếu dự án của bạn có các component cũ viết bằng class (Class Components), bạn không thể dùng Custom Hooks (chỉ dùng được cho functional components), bắt buộc phải dùng HOC để tái sử dụng logic.
- **Quản lý middleware cho page render (ví dụ Next.js cũ):** Bọc các trang để tiêm dữ liệu cấu hình trước khi nạp.
