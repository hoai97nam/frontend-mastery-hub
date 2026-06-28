# 📦 Pattern 2: Render Props

> Mẫu thiết kế **Render Props** là kỹ thuật chia sẻ mã nguồn giữa các React Component bằng cách sử dụng một prop nhận giá trị là một **hàm (function)**. Hàm này chịu trách nhiệm trả về React element để render UI.

---

## 1. Khái niệm & Mục đích

Trước khi React Hooks ra đời ở phiên bản 16.8, việc chia sẻ logic có trạng thái (stateful logic) giữa các component rất khó khăn. HOC và **Render Props** là hai giải pháp cứu cánh hàng đầu. 

Triết lý của Render Props là: *"Component nhận logic và dữ liệu, còn cấu trúc giao diện (UI) sẽ do nơi gọi (consumer) quyết định thông qua một hàm render."*

### Cú pháp cơ bản:
```jsx
<DataProvider render={(data) => <h1>Hello {data.name}</h1>} />
```

### Children as a Function (Biến thể phổ biến):
Thay vì dùng prop tên là `render`, ta sử dụng chính prop `children` như một hàm:
```jsx
<DataProvider>
  {(data) => <h1>Hello {data.name}</h1>}
</DataProvider>
```

---

## 2. Code Example: Component `<MouseTracker>` thực chiến

Một ví dụ kinh điển là theo dõi tọa độ con trỏ chuột trên màn hình để tái sử dụng ở nhiều giao diện khác nhau (hiển thị tọa độ dạng text, vẽ một hình tròn chạy theo chuột, v.v.).

```jsx
import React, { useState, useEffect } from 'react';

// 1. Component chứa logic theo dõi chuột
function MouseTracker({ children }) {
  const [coords, setCoords] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (event) => {
      setCoords({
        x: event.clientX,
        y: event.clientY
      });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);

  // Gọi children như một hàm và truyền dữ liệu tọa độ vào
  return children(coords);
}

// 2. Cách sử dụng 1: Hiển thị tọa độ text đơn giản
function CoordinatesDisplay() {
  return (
    <MouseTracker>
      {({ x, y }) => (
        <div style={{ padding: '20px', border: '1px solid #ccc' }}>
          <h3>Tọa độ chuột hiện tại:</h3>
          <p>X: {x}, Y: {y}</p>
        </div>
      )}
    </MouseTracker>
  );
}

// 3. Cách sử dụng 2: Vẽ vòng tròn di chuyển theo chuột
function CircleFollower() {
  return (
    <MouseTracker>
      {({ x, y }) => (
        <div
          style={{
            position: 'absolute',
            left: x - 15,
            top: y - 15,
            width: '30px',
            height: '30px',
            borderRadius: '50%',
            backgroundColor: 'rgba(255, 99, 71, 0.6)',
            pointerEvents: 'none' // Tránh chặn sự kiện click chuột
          }}
        />
      )}
    </MouseTracker>
  );
}
```

---

## 3. So Sánh: Render Props vs React Hooks

| Tiêu chí | Render Props | React Hooks (Modern) |
|----------|--------------|----------------------|
| **Cú pháp** | Lồng nhau nhiều tầng (Callback Hell / Wrapper Hell) | Phẳng, dễ đọc, gọi trực tiếp ở top-level |
| **Chia sẻ logic** | Thông qua Component bọc | Thông qua Custom Hook |
| **Độ sâu cây DOM** | Làm tăng độ sâu cây component | Không ảnh hưởng đến cấu trúc DOM |
| **Kiểm thử (Test)** | Cần render component để kiểm tra | Có thể kiểm thử logic độc lập |

### Khi nào Render Props vẫn hữu ích trong React hiện đại?
Dù React Hooks đã thay thế 90% trường hợp sử dụng Render Props (như data fetching, event handling), Render Props vẫn cực kỳ mạnh mẽ trong các trường hợp:
1. **Thiết kế UI Layouts & Components Library:** Khi bạn muốn cung cấp các khe cắm (slots) tùy biến cao cho các thư viện component (ví dụ: `List` component cần consumer định nghĩa cách render từng item).
2. **Virtualized Lists (như `react-window`):**
   ```jsx
   <FixedSizeList
     height={150}
     itemCount={1000}
     itemSize={35}
     width={300}
   >
     {({ index, style }) => <div style={style}>Row {index}</div>}
   </FixedSizeList>
   ```

---

## 4. Tối ưu hiệu năng (Performance Optimization)

Một lỗi phổ biến khiến Render Props gây re-render lãng phí là khai báo hàm inline trực tiếp trong render.

### ⚠️ Anti-pattern (Gây re-render không cần thiết):
```jsx
function Parent() {
  return (
    // Mỗi lần Parent render, một hàm mới được tạo ra, khiến MouseTracker
    // luôn coi prop children là thay đổi và tự re-render lại.
    <MouseTracker>
      {(coords) => <Display coordinates={coords} />}
    </MouseTracker>
  );
}
```

### ✅ Cách khắc phục:
Lưu trữ hàm render bên ngoài component hoặc sử dụng `useCallback`:

```jsx
import { useCallback } from 'react';

function Parent() {
  // Memoize hàm render để giữ nguyên tham chiếu giữa các lần render
  const renderDisplay = useCallback((coords) => (
    <Display coordinates={coords} />
  ), []);

  return <MouseTracker>{renderDisplay}</MouseTracker>;
}
```
