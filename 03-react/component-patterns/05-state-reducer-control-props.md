# 📦 Pattern 5: State Reducer & Control Props

> Đây là các **mẫu thiết kế nâng cao (Advanced Patterns)** thường được sử dụng trong các thư viện UI Component phức tạp (như Downshift, Formik) nhằm trao cho lập trình viên sử dụng (consumer) quyền kiểm soát tuyệt đối đối với hành vi và trạng thái bên trong component.

---

## 1. State Reducer Pattern

### 1.1 Khái niệm
**State Reducer Pattern** đảo ngược quyền kiểm soát (Inversion of Control) bằng cách cho phép consumer truyền vào một hàm reducer tùy chỉnh. Khi có sự kiện xảy ra bên trong component (như Click, KeyDown), component sẽ gọi hàm reducer này của consumer để xác định trạng thái tiếp theo thay vì tự ý cập nhật theo logic mặc định của nó.

### 1.2 Code Example thực chiến: Component `<Toggle>` giới hạn số lần click
Chúng ta xây dựng một component `<Toggle>` thông thường, nhưng muốn consumer có khả năng ngăn cản việc bật/tắt (toggle) nếu họ đã click quá 4 lần.

```jsx
import React, { useReducer } from 'react';

// 1. Khai báo các Action Types nội bộ của Component
const TOGGLE_ACTION_TYPES = {
  toggle: 'TOGGLE',
  reset: 'RESET'
};

// 2. Reducer mặc định của component
function defaultToggleReducer(state, action) {
  switch (action.type) {
    case TOGGLE_ACTION_TYPES.toggle:
      return { on: !state.on };
    case TOGGLE_ACTION_TYPES.reset:
      return { on: false };
    default:
      throw new Error(`Action type ${action.type} không được hỗ trợ`);
  }
}

// 3. Component Toggle áp dụng State Reducer
export function Toggle({ customReducer = defaultToggleReducer }) {
  // Sử dụng customReducer nhận được từ props thay vì dùng reducer mặc định cố định
  const [state, dispatch] = useReducer(customReducer, { on: false });

  const toggle = () => dispatch({ type: TOGGLE_ACTION_TYPES.toggle });
  const reset = () => dispatch({ type: TOGGLE_ACTION_TYPES.reset });

  return (
    <div style={{ padding: '20px', border: '1px dashed #666', margin: '10px 0' }}>
      <p>Trạng thái: <strong>{state.on ? 'BẬT 🟢' : 'TẮT 🔴'}</strong></p>
      <button onClick={toggle} style={{ marginRight: '8px' }}>Toggle</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// 4. Cách sử dụng (Usage): Consumer chặn không cho toggle nếu click quá 4 lần
export function App() {
  const [clickCount, setClickCount] = React.useState(0);

  const myCustomReducer = (state, action) => {
    // Nếu click quá 4 lần, chặn không cho toggle nữa
    if (action.type === TOGGLE_ACTION_TYPES.toggle && clickCount >= 4) {
      alert('⚠️ Bạn đã vượt quá giới hạn 4 lần click! Vui lòng reset.');
      return state; // Giữ nguyên state cũ, không cho cập nhật
    }
    
    // Tăng biến đếm click
    if (action.type === TOGGLE_ACTION_TYPES.toggle) {
      setClickCount(c => c + 1);
    } else if (action.type === TOGGLE_ACTION_TYPES.reset) {
      setClickCount(0);
    }

    // Chạy logic reducer mặc định của component
    return defaultToggleReducer(state, action);
  };

  return (
    <div>
      <h2>Ví dụ State Reducer</h2>
      <p>Số lần đã click: {clickCount}</p>
      <Toggle customReducer={myCustomReducer} />
    </div>
  );
}
```

---

## 2. Control Props Pattern

### 2.1 Khái niệm
**Control Props Pattern** biến đổi một component hoạt động ở chế độ "Hybrid" (Lai):
- Mặc định: Component tự quản lý state nội bộ của nó (giống Uncontrolled).
- Khi có props truyền vào: Component chuyển sang chế độ được kiểm soát hoàn toàn bởi cha (giống Controlled).

Điều này giúp component vừa dễ dùng (không cần khai báo state ở cha cho trường hợp đơn giản), vừa cực kỳ linh hoạt (khi cha cần can thiệp đồng bộ).

### 2.2 Code Example thực chiến: Custom `<Switch>` hoạt động linh hoạt
Chúng ta thiết kế Switch có thể tự chạy hoặc có thể bị ép buộc trạng thái từ bên ngoài.

```jsx
import React, { useState } from 'react';

export function Switch({ value, onChange }) {
  // Kiểm tra xem prop 'value' có được truyền vào từ bên ngoài hay không
  const isControlled = value !== undefined;

  // State nội bộ chỉ dùng khi ở chế độ Uncontrolled (không truyền prop value)
  const [internalState, setInternalState] = useState(false);

  // Lấy ra giá trị thực tế đang hiển thị dựa trên chế độ hoạt động
  const stateValue = isControlled ? value : internalState;

  const handleToggle = () => {
    if (isControlled) {
      // Ở chế độ Controlled: Chỉ gửi sự kiện lên cha, không tự ý đổi state
      if (onChange) onChange(!value);
    } else {
      // Ở chế độ Uncontrolled: Tự thay đổi state nội bộ và báo cáo
      setInternalState(!internalState);
      if (onChange) onChange(!internalState);
    }
  };

  return (
    <div 
      onClick={handleToggle}
      style={{
        width: '50px',
        height: '25px',
        borderRadius: '15px',
        backgroundColor: stateValue ? '#4caf50' : '#ccc',
        position: 'relative',
        cursor: 'pointer',
        transition: 'background-color 0.3s'
      }}
    >
      <div 
        style={{
          width: '21px',
          height: '21px',
          borderRadius: '50%',
          backgroundColor: '#fff',
          position: 'absolute',
          top: '2px',
          left: stateValue ? '27px' : '2px',
          transition: 'left 0.3s'
        }}
      />
    </div>
  );
}

// --- CÁCH SỬ DỤNG ---

export function AppDemo() {
  // Case 1: Chạy độc lập (Uncontrolled) - Cực kỳ tiện lợi cho UI tĩnh
  // Case 2: Kiểm soát từ bên ngoài (Controlled) - Thích hợp khi lưu vào Database
  const [isLocked, setIsLocked] = useState(false);

  return (
    <div>
      <h3>1. Switch chạy độc lập (Uncontrolled):</h3>
      <Switch onChange={(val) => console.log('Switch 1:', val)} />

      <h3>2. Switch bị kiểm soát từ bên ngoài (Controlled):</h3>
      <p>Trạng thái khóa: {isLocked ? 'ĐÃ KHÓA 🔒' : 'ĐANG MỞ 🔓'}</p>
      <Switch value={isLocked} onChange={(val) => setIsLocked(val)} />
      
      <button onClick={() => setIsLocked(false)} style={{ marginTop: '10px' }}>
        Ép mở khóa từ xa
      </button>
    </div>
  );
}
```

---

## 3. Tổng kết so sánh hai Pattern

- **State Reducer:** Đóng vai trò cấu hình **Cách thức hoạt động (How the state changes)**. Phù hợp khi muốn thay đổi luật xử lý sự kiện trong component mà không muốn phá vỡ cấu trúc render.
- **Control Props:** Đóng vai trò quyết định **Ai sở hữu dữ liệu (Who owns the state value)**. Phù hợp khi muốn tích hợp component vào một hệ thống quản lý state lớn hơn của toàn bộ màn hình (như Redux, React Hook Form).
