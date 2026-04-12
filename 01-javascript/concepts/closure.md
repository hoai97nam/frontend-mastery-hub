# Closures & Memory Leak — Câu hỏi phỏng vấn Senior Frontend

---

## Mục lục

1. [Nền tảng lý thuyết](#1-nền-tảng-lý-thuyết)
2. [Debug & Nhận diện leak](#2-debug--nhận-diện-leak)
3. [Tình huống giả định](#3-tình-huống-giả-định)
4. [Câu hỏi bẫy (Trap Questions)](#4-câu-hỏi-bẫy-trap-questions)
5. [Framework-specific (React / Vue)](#5-framework-specific-react--vue)
6. [Cách đo bằng DevTools](#6-cách-đo-bằng-devtools-memory-tab)
7. [Thang đánh giá](#7-thang-đánh-giá)

---

## 1. Nền tảng lý thuyết

### Câu 1.1
> Closure là gì? Tại sao closure có thể là nguyên nhân gây memory leak?

**Điểm cần nghe ở senior:**
- Closure = function giữ tham chiếu đến lexical scope nơi nó được tạo ra
- Garbage Collector (GC) không thể thu hồi object nếu còn ít nhất một tham chiếu đang sống
- Leak xảy ra khi closure bị giữ sống lâu hơn cần thiết (global var, listener, timer)

**Ví dụ cơ bản:**
```javascript
function createLeak() {
  const hugeData = new Array(1_000_000).fill('🔥'); // 8MB+

  return function() {
    // closure giữ hugeData trong scope
    console.log('data length:', hugeData.length);
  };
}

// leak: fn giữ hugeData sống mãi
const fn = createLeak();

// fix: giải phóng khi không cần
// fn = null;
```

---

### Câu 1.2
> Garbage Collector hoạt động như thế nào trong V8? Tại sao `null` lại giúp giải phóng bộ nhớ?

**Điểm cần nghe ở senior:**
- V8 dùng **Mark-and-Sweep**: bắt đầu từ GC roots (global, stack), đánh dấu tất cả object có thể reach được
- Object không được đánh dấu → bị thu hồi
- Gán `null` → cắt đứt tham chiếu → object trở thành unreachable → GC thu hồi được

```javascript
let cache = { data: new Array(500_000).fill(0) };

// Dùng xong
cache = null; // GC root không còn trỏ đến object → eligible for collection
```

---

### Câu 1.3
> Phân biệt **memory leak** và **memory bloat**. Cả hai đều nguy hiểm không?

**Điểm cần nghe ở senior:**

| | Memory Leak | Memory Bloat |
|---|---|---|
| **Định nghĩa** | Bộ nhớ cấp phát nhưng không bao giờ được giải phóng | Dùng nhiều bộ nhớ hơn cần thiết nhưng vẫn được giải phóng |
| **Xu hướng** | Tăng dần theo thời gian | Tăng giảm theo chu kỳ |
| **Triệu chứng** | Tab crash sau vài giờ | Chậm, nhưng ổn định |
| **Ví dụ** | Listener không remove | Cache quá lớn, image không tối ưu |
| **Mức độ nguy hiểm** | Nghiêm trọng hơn | Có thể chấp nhận nếu kiểm soát |

---

## 2. Debug & Nhận diện Leak

### Câu 2.1
> Đoạn code sau có memory leak không? Nếu có, fix như thế nào?

```javascript
class EventManager {
  constructor() {
    this.data = new Array(100_000).fill('data');
    this.handler = this.handleClick.bind(this);
    document.addEventListener('click', this.handler);
  }

  handleClick() {
    console.log('clicked, data size:', this.data.length);
  }

  destroy() {
    // Quên remove listener
  }
}

// Dùng nhiều lần trong SPA
const managers = [];
for (let i = 0; i < 100; i++) {
  managers.push(new EventManager());
}
managers.length = 0; // tưởng đã dọn sạch
```

**Đáp án:**
- **Có leak**: `document` giữ tham chiếu đến `this.handler` → giữ instance sống → giữ `this.data` sống
- `managers.length = 0` chỉ xóa tham chiếu trong array, không remove listener

**Fix:**
```javascript
destroy() {
  document.removeEventListener('click', this.handler);
  this.data = null;
  this.handler = null;
}
```

---

### Câu 2.2
> `WeakMap` và `WeakRef` giải quyết vấn đề gì mà `Map` thông thường không làm được?

**Điểm cần nghe ở senior:**
- `Map` giữ **strong reference** → object không bao giờ bị GC dù không còn dùng
- `WeakMap` giữ **weak reference** → GC có thể thu hồi key object bất cứ lúc nào
- Dùng `WeakMap` để gắn metadata vào DOM node mà không lo leak

```javascript
// BAD: Map giữ DOM node sống mãi dù đã remove khỏi DOM
const cache = new Map();
const node = document.createElement('div');
cache.set(node, { meta: 'data' });
// node bị remove khỏi DOM nhưng Map vẫn giữ nó

// GOOD: WeakMap → GC tự thu hồi node khi không còn ai dùng
const weakCache = new WeakMap();
weakCache.set(node, { meta: 'data' });
// node bị remove → WeakMap entry tự biến mất
```

---

### Câu 2.3
> Timer-based leak là gì? Tại sao `setInterval` đặc biệt nguy hiểm hơn `setTimeout`?

```javascript
function startPolling() {
  const hugeState = { data: new Array(500_000).fill(0) };

  // setInterval giữ closure sống mãi mãi
  setInterval(() => {
    console.log(hugeState.data.length); // hugeState không bao giờ được GC
  }, 1000);
}

startPolling(); // gọi nhiều lần = nhiều interval = leak tích lũy
```

**Fix:**
```javascript
function startPolling() {
  const hugeState = { data: new Array(500_000).fill(0) };
  let intervalId;

  intervalId = setInterval(() => {
    console.log(hugeState.data.length);
  }, 1000);

  // Trả về cleanup function
  return () => {
    clearInterval(intervalId);
    hugeState.data = null;
  };
}

const stop = startPolling();
// Khi không cần:
stop();
```

---

## 3. Tình huống giả định

### Tình huống 3.1 — SPA Navigation
> Sau mỗi lần user navigate giữa các trang trong SPA (React/Vue), bộ nhớ tăng dần và không giảm. Dù component đã unmount, Memory tab vẫn thấy heap tăng. Bạn debug như thế nào?

**Hướng tiếp cận cần nghe:**
1. Mở DevTools Memory → Take Heap Snapshot trước khi navigate
2. Navigate đi về 5 lần → Take Snapshot lần 2
3. So sánh: filter **"Objects allocated between snapshots"**
4. Tìm các object của component cũ còn tồn tại → trace back xem ai đang giữ tham chiếu

**Nguyên nhân thường gặp:**
```javascript
// Leak trong React component
useEffect(() => {
  window.addEventListener('resize', handleResize);
  // Quên cleanup → listener sống dù component unmount
}, []);

// Fix
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

---

### Tình huống 3.2 — Global Event Bus
> Team dùng một EventEmitter global để giao tiếp giữa các module. Sau 2 tuần production, user báo tab chậm dần theo thời gian. Bạn nghi ngờ gì?

**Điểm cần nghe ở senior:**
- EventEmitter global = GC root → mọi listener đăng ký đều được giữ sống
- Component subscribe nhưng không unsubscribe khi destroy → accumulate theo thời gian

```javascript
// BAD: mỗi lần mount component đăng ký thêm 1 listener
class UserDashboard {
  mounted() {
    eventBus.on('data-update', this.refresh.bind(this));
  }
  // Không có beforeDestroy/unmount cleanup
}

// GOOD
class UserDashboard {
  mounted() {
    this._handler = this.refresh.bind(this);
    eventBus.on('data-update', this._handler);
  }
  beforeDestroy() {
    eventBus.off('data-update', this._handler);
    this._handler = null;
  }
}
```

---

### Tình huống 3.3 — Infinite Scroll Cache
> Bạn implement infinite scroll — load thêm item khi user scroll xuống. Sau 30 phút dùng, tab bắt đầu giật. Nguyên nhân có thể là gì?

**Điểm cần nghe ở senior:**
- Mỗi batch load thêm DOM node + JS object vào memory, không bao giờ remove
- Sau 30 phút có thể có hàng nghìn DOM node trong cây

**Giải pháp:**
```javascript
// Virtual scrolling: chỉ render items trong viewport
// Ví dụ cơ chế:
function renderVisibleItems(scrollTop, containerHeight, items, itemHeight) {
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.ceil((scrollTop + containerHeight) / itemHeight);

  // Chỉ render [startIndex, endIndex]
  // Remove/recycle DOM node ngoài range này
  return items.slice(startIndex, endIndex);
}
```

---

### Tình huống 3.4 — Closure trong Loop
> Junior trong team viết đoạn code sau và thắc mắc tại sao tất cả button đều log ra `5`:

```javascript
for (var i = 0; i < 5; i++) {
  document.querySelectorAll('.btn')[i]
    .addEventListener('click', function() {
      console.log(i); // luôn log 5
    });
}
```

> Giải thích vấn đề và đưa ra 3 cách fix khác nhau.

**Đáp án:**
```javascript
// Vấn đề: var không có block scope → tất cả closure share cùng 1 biến i
// Khi click, loop đã chạy xong → i = 5

// Fix 1: dùng let (block scope)
for (let i = 0; i < 5; i++) {
  document.querySelectorAll('.btn')[i]
    .addEventListener('click', () => console.log(i));
}

// Fix 2: IIFE tạo scope riêng
for (var i = 0; i < 5; i++) {
  (function(j) {
    document.querySelectorAll('.btn')[j]
      .addEventListener('click', () => console.log(j));
  })(i);
}

// Fix 3: bind
for (var i = 0; i < 5; i++) {
  document.querySelectorAll('.btn')[i]
    .addEventListener('click', console.log.bind(null, i));
}
```

---

## 4. Câu hỏi bẫy (Trap Questions)

### Câu 4.1
> Đoạn code này có leak không?

```javascript
function setup() {
  const el = document.getElementById('btn');
  const data = { value: new Array(100_000).fill(0) };

  el.addEventListener('click', function handler() {
    console.log(data.value.length);
    el.removeEventListener('click', handler); // tự remove sau lần đầu
  });
}
```

**Đáp án:** Không leak — handler tự remove sau click đầu tiên → closure được giải phóng.  
Nhưng nếu button không bao giờ được click → `data` sống đến khi `el` bị remove khỏi DOM.

---

### Câu 4.2
> `console.log` có thể gây memory leak trong production không?

**Đáp án: Có** — ít người biết điều này.

```javascript
const hugeObject = { data: new Array(1_000_000).fill(0) };
console.log(hugeObject); // DevTools giữ reference để hiển thị
// hugeObject không bị GC khi DevTools đang mở
```

- Khi DevTools mở: browser giữ tham chiếu đến object đã log để có thể expand trong console
- Production code nên strip `console.log` qua build tool (webpack/vite plugin)

---

### Câu 4.3
> Hai đoạn code này có hành vi GC khác nhau không?

```javascript
// A
function a() {
  const big = new Array(1_000_000).fill(0);
  return big.length;
}

// B
function b() {
  const big = new Array(1_000_000).fill(0);
  const len = big.length;
  return len;
}
```

**Đáp án:** Về mặt lý thuyết giống nhau — `big` là local variable, bị thu hồi sau khi function return. Tuy nhiên V8 có thể optimize khác nhau. Điểm quan trọng hơn: nếu `big` bị capture vào closure trước khi return thì khác.

---

## 5. Framework-specific (React / Vue)

### Câu 5.1 — React
> Những pattern nào trong React dễ gây memory leak nhất?

```javascript
// Pattern 1: setState sau khi component unmount
useEffect(() => {
  fetchData().then(data => {
    setState(data); // component đã unmount → React warning + potential leak
  });
}, []);

// Fix: AbortController hoặc flag
useEffect(() => {
  let cancelled = false;
  fetchData().then(data => {
    if (!cancelled) setState(data);
  });
  return () => { cancelled = true; };
}, []);

// Pattern 2: Subscription không cleanup
useEffect(() => {
  const sub = observable.subscribe(handler);
  // Thiếu: return () => sub.unsubscribe();
}, []);

// Pattern 3: Ref giữ stale closure
const countRef = useRef(count);
// Không update ref → closure cũ giữ data cũ
```

---

### Câu 5.2 — Vue
> `onUnmounted` vs `onBeforeUnmount` — cái nào phù hợp hơn để cleanup listener? Tại sao?

**Điểm cần nghe ở senior:**
- `onBeforeUnmount`: component vẫn còn fully functional → phù hợp cleanup subscriptions, timers, listeners
- `onUnmounted`: DOM đã bị detach → không nên access DOM ở đây
- Dùng `onBeforeUnmount` cho cleanup là safer

```javascript
// Vue 3 Composition API
onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
  clearInterval(pollingId);
  subscription.unsubscribe();
});
```

---

## 6. Cách đo bằng DevTools Memory Tab

### Quy trình chuẩn

```
Bước 1: Mở DevTools → tab Memory
Bước 2: Chọn "Heap snapshot" → Take snapshot (baseline)
Bước 3: Thực hiện hành động nghi ngờ leak (navigate, click, scroll...)
Bước 4: Trigger GC thủ công (nút thùng rác trong Memory tab)
Bước 5: Take snapshot lần 2
Bước 6: Chọn snapshot 2 → Dropdown chọn "Comparison"
Bước 7: Sort by "Size Delta" (giảm dần) → tìm object tăng bất thường
Bước 8: Click vào object → xem "Retainers" → trace xem ai đang giữ nó
```

### Dấu hiệu leak trong Performance tab
```
- Heap size (đường xanh) tăng đều, không giảm sau GC
- Nhiều GC events liên tiếp (đường răng cưa ngày càng cao)
- Major GC (stop-the-world) xuất hiện thường xuyên
```

### Dùng `performance.measureUserAgentSpecificMemory()`
```javascript
// Chrome 89+, chỉ dùng trong cross-origin isolated context
const result = await performance.measureUserAgentSpecificMemory();
console.log(result.bytes); // tổng bytes đang dùng
```

---

## 7. Thang đánh giá

| Mức độ | Dấu hiệu nhận biết |
|---|---|
| **Junior** | Biết closure là gì, biết `null` giải phóng bộ nhớ, nhưng không giải thích được tại sao |
| **Mid** | Debug được leak bằng DevTools, biết `removeEventListener`, hiểu `WeakMap` cơ bản |
| **Senior** | Tự phát hiện leak tiềm ẩn trong code review, biết đo heap snapshot, hiểu GC internals, đề xuất kiến trúc tránh leak từ đầu |
| **Staff+** | Thiết lập monitoring memory trong production (`performance.memory`), có chiến lược cảnh báo tự động khi heap vượt ngưỡng |

---

> **Câu hỏi mở để đào sâu:** Trong codebase hiện tại của bạn, làm thế nào để đảm bảo mọi `addEventListener` đều có cặp `removeEventListener` tương ứng — bạn có linting rule hoặc convention nào cho việc này không?