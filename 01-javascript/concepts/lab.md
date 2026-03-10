

# `queueMicrotask` trong JavaScript

## 📑 Mục Lục
- [queueMicrotask trong JavaScript](#queuemicrotask-trong-javascript)
- [JavaScript Task Queues — Danh sách đầy đủ](#javascript-task-queues--danh-sách-đầy-đủ)
  - [1. Synchronous (Call Stack)](#1-synchronous-call-stack)
  - [2. Microtask Queue](#2-microtask-queue)
  - [3. Macrotask Queue (Task Queue)](#3-macrotask-queue-task-queue)
  - [Thứ tự thực thi tóm tắt](#thứ-tự-thực-thi-tóm-tắt)
- [Tại sao cập nhật UI mỗi 16.6ms để đạt 60fps?](#tại-sao-cập-nhật-ui-mỗi-166ms-để-đạt-60fps)
- [Ví dụ phức tạp: Event Loop + process.nextTick + Promise lồng nhau](#ví-dụ-phức-tạp-event-loop--processnexttick--promise-lồng-nhau)

# `queueMicrotask` trong JavaScript

```javascript
console.log("1. Bắt đầu");

setTimeout(() => {
  console.log("4. setTimeout (macrotask)");
}, 0);

queueMicrotask(() => {
  console.log("3. queueMicrotask chạy sau sync, trước setTimeout");
});

console.log("2. Kết thúc sync");

// Output:
// 1. Bắt đầu
// 2. Kết thúc sync
// 3. queueMicrotask chạy sau sync, trước setTimeout
// 4. setTimeout (macrotask)
```

**Điểm mấu chốt:** `queueMicrotask` đẩy callback vào **microtask queue** — chạy sau khi call stack hiện tại xong, nhưng **trước** bất kỳ macrotask nào (setTimeout, setInterval, I/O).

Thứ tự ưu tiên: `sync code` → `microtasks` → `macrotasks`

Dùng khi nào? Khi muốn defer một tác vụ nhỏ mà không muốn nó bị "đẩy lùi" quá xa như `setTimeout(fn, 0)`.

# JavaScript Task Queues — Danh sách đầy đủ

---

## 1. Synchronous (Call Stack)

Chạy ngay lập tức, theo thứ tự, block mọi thứ phía sau.

| Hàm / API | Ghi chú |
|---|---|
| Tất cả code thông thường | Khai báo biến, gán giá trị, toán tử... |
| Hàm tự định nghĩa (`function`, arrow fn) | Nếu không có `await`/callback async |
| `console.log/warn/error/...` | |
| `JSON.parse()` / `JSON.stringify()` | |
| `Math.*` | `Math.random()`, `Math.floor()`,... |
| `Array.*` | `.map()`, `.filter()`, `.reduce()`,... |
| `Object.*` | `Object.keys()`, `Object.assign()`,... |
| `String.*` | `.split()`, `.replace()`, `.trim()`,... |
| `parseInt()` / `parseFloat()` | |
| `try / catch / finally` | Phần sync bên trong |
| `throw new Error()` | |
| `new Promise(executor)` | Executor fn chạy **sync** ngay lập tức |
| `localStorage.getItem/setItem` | Blocking I/O (sync) |
| `XMLHttpRequest` (sync mode) | `xhr.open("GET", url, false)` — deprecated |
| `alert()` / `confirm()` / `prompt()` | Blocking UI (browser) |
| `crypto.getRandomValues()` | |
| `structuredClone()` | |

---

## 2. Microtask Queue

Chạy **ngay sau khi call stack rỗng**, trước bất kỳ macrotask nào.  
Nếu microtask tạo ra microtask mới → chạy hết trước khi render/macrotask.

| Hàm / API | Ghi chú |
|---|---|
| `Promise.then()` | |
| `Promise.catch()` | |
| `Promise.finally()` | |
| `Promise.resolve().then(...)` | |
| `Promise.reject().catch(...)` | |
| `Promise.all().then(...)` | |
| `Promise.allSettled().then(...)` | |
| `Promise.any().then(...)` | |
| `Promise.race().then(...)` | |
| `async/await` (phần sau `await`) | Mỗi `await` tạo ra một microtask checkpoint |
| `queueMicrotask(fn)` | API tường minh để queue microtask |
| `MutationObserver` callback | Theo dõi thay đổi DOM |
| `IntersectionObserver` callback | Một số trình duyệt xử lý như microtask |
| `.then()` trên Thenable object | Bất kỳ object nào có `.then()` |

---

## 3. Macrotask Queue (Task Queue)

Chạy **sau khi call stack rỗng VÀ microtask queue đã hết**.  
Mỗi lần event loop chỉ lấy **một** macrotask, sau đó flush hết microtask rồi mới lấy tiếp.

### Timer
| Hàm / API | Ghi chú |
|---|---|
| `setTimeout(fn, delay)` | delay tối thiểu ~4ms trong browser |
| `setInterval(fn, delay)` | Lặp lại định kỳ |
| `clearTimeout()` / `clearInterval()` | Hủy timer |
| `setImmediate(fn)` | **Node.js only** — sau I/O callbacks |
| `clearImmediate()` | **Node.js only** |

### I/O & Network
| Hàm / API | Ghi chú |
|---|---|
| `XMLHttpRequest` callback (async) | `onload`, `onerror`, `onreadystatechange` |
| `fetch()` → Response callback | Resolve của fetch là microtask, nhưng network I/O là macrotask |
| `WebSocket` events | `onmessage`, `onopen`, `onclose` |
| `FileReader` callbacks | `onload`, `onerror` |
| `IndexedDB` callbacks | `onsuccess`, `onerror` |
| Node.js `fs.readFile()` callback | |
| Node.js `http.request()` callback | |
| Node.js `net.connect()` callback | |

### User Interaction & Browser Events
| Hàm / API | Ghi chú |
|---|---|
| `addEventListener` callbacks | `click`, `keydown`, `mousemove`,... |
| `onclick`, `onkeyup`, `oninput`,... | Inline event handlers |
| `dispatchEvent()` (async context) | |
| Drag & Drop events | |
| Clipboard events | `oncopy`, `onpaste` |
| Focus/Blur events | `onfocus`, `onblur` |

### Rendering & Animation
| Hàm / API | Ghi chú |
|---|---|
| `requestAnimationFrame(fn)` | Thực ra là **rendering task** — trước paint, sau microtask |
| `requestIdleCallback(fn)` | Chạy khi browser rảnh |
| `cancelAnimationFrame()` | |
| `cancelIdleCallback()` | |

### MessageChannel & Worker
| Hàm / API | Ghi chú |
|---|---|
| `MessageChannel.port.onmessage` | |
| `postMessage()` callback | Giữa window, iframe, worker |
| `Worker` message events | `onmessage` từ Web Worker |
| `BroadcastChannel.onmessage` | |
| `ServiceWorker` events | `fetch`, `push`, `sync` |

### Node.js Specific
| Hàm / API | Ghi chú |
|---|---|
| `process.nextTick(fn)` | ⚠️ Thực ra là **microtask** trong Node.js, ưu tiên cao hơn cả Promise |
| `setImmediate(fn)` | Macrotask — sau I/O, trước `setTimeout` |
| `EventEmitter.emit()` callbacks | Sync nếu listener sync, async nếu async |

---

## Thứ tự thực thi tóm tắt

```
Sync Code (Call Stack)
    ↓
Microtasks (Promise.then, queueMicrotask, MutationObserver)
    ↓
Rendering (requestAnimationFrame, layout, paint)
    ↓
Macrotask (setTimeout, setInterval, I/O, Events)
    ↓
Microtasks (flush lại sau mỗi macrotask)
    ↓
... lặp lại
```

> **Lưu ý Node.js:** `process.nextTick` chạy trước cả `Promise.then` trong cùng microtask phase.


# Tại sao cập nhật UI mỗi 16.6ms để đạt 60fps?

## Công thức cơ bản

**60fps = 60 frames/giây**

Mỗi frame tồn tại trong: `1000ms / 60 = 16.67ms`

Tức là cứ mỗi **16.6ms**, trình duyệt có một "cửa sổ thời gian" để:

```
│ JS execution │ Style calc │ Layout │ Paint │ Composite │
└──────────────────────────────────────────────────────┘
        Tất cả phải xong trong 16.6ms
```

---

## Tại sao 60fps là ngưỡng "mượt mà"?

Mắt người bắt đầu cảm nhận chuyển động mượt từ ~24fps (phim ảnh dùng 24fps).  
60fps là ngưỡng mà não người **không còn phân biệt được** sự rời rạc giữa các frame — cảm giác là liền mạch hoàn toàn.  
Trên 60fps vẫn tốt hơn, nhưng lợi ích giảm dần rõ rệt.

---

## Hệ quả thực tế với JavaScript

Nếu một đoạn JS chạy **hơn 16.6ms** → trình duyệt bỏ lỡ frame đó → **jank** (giật lag).

```javascript
// Nguy hiểm: block render
for (let i = 0; i < 1_000_000_000; i++) {} // chạy ~500ms → drop hàng chục frames

// An toàn hơn: chia nhỏ qua requestAnimationFrame
function processChunk() {
  // xử lý một phần nhỏ
  requestAnimationFrame(processChunk); // nhường cho render mỗi frame
}
```

---

## Tóm lại

16.6ms không phải con số tùy ý — nó là **deadline vật lý** của một frame 60fps.  
Event Loop phải đủ "nhường chỗ" cho bước Render trong cửa sổ đó.

---

> **Câu hỏi mở:** Nếu Microtask Queue không bao giờ trống (microtask liên tục tạo microtask mới), điều gì xảy ra với bước Render?

---

## 📚 Tài nguyên bổ sung

- [Event Loop Visualizer](https://event-loop-visualizer-ruby.vercel.app/)

---

# Ví dụ phức tạp: Event Loop + process.nextTick + Promise lồng nhau

> ⚠️ Ví dụ này chạy trong **Node.js** (có `process.nextTick`). Thứ tự ưu tiên: `Sync` → `process.nextTick` → `Promise.then` → `setTimeout`

## Code

```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout 1');
  Promise.resolve().then(() => {
    console.log('Promise inside Timeout');
  });
  process.nextTick(() => {
    console.log('Next Tick inside Timeout');
  });
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 1');
  setTimeout(() => {
    console.log('Timeout inside Promise');
  }, 0);
});

console.log('End');
```

## Output

```
Start
End
Promise 1
Timeout 1
Next Tick inside Timeout
Promise inside Timeout
Timeout inside Promise
```

## Giải thích từng bước

### Phase 1 — Synchronous (Call Stack)

| Bước | Code chạy | Output | Ghi chú |
|------|-----------|--------|---------|
| 1 | `console.log('Start')` | `Start` | Sync, chạy ngay |
| 2 | `setTimeout(Timeout1, 0)` | — | Đưa vào **Macrotask Queue** |
| 3 | `Promise.resolve().then(Promise1)` | — | Đưa vào **Microtask Queue** |
| 4 | `console.log('End')` | `End` | Sync, chạy ngay |

> Call stack trống → bắt đầu flush **Microtask Queue**

---

### Phase 2 — Microtask Queue (lần 1)

| Bước | Code chạy | Output | Ghi chú |
|------|-----------|--------|---------|
| 5 | `Promise1` callback chạy | `Promise 1` | Microtask đầu tiên |
| 5b | `setTimeout(Timeout2, 0)` bên trong | — | Đưa thêm vào **Macrotask Queue** |

> Microtask queue rỗng → Event Loop lấy macrotask đầu tiên

---

### Phase 3 — Macrotask #1: `Timeout 1`

| Bước | Code chạy | Output | Ghi chú |
|------|-----------|--------|---------|
| 6 | `console.log('Timeout 1')` | `Timeout 1` | |
| 6b | `Promise.resolve().then(PromiseInsideTimeout)` | — | Đưa vào **Microtask Queue** |
| 6c | `process.nextTick(NextTickInsideTimeout)` | — | Đưa vào **nextTick Queue** (ưu tiên cao hơn Promise!) |

> Macrotask xong → flush **nextTick Queue** trước, rồi mới **Microtask Queue**

---

### Phase 4 — nextTick Queue (sau Macrotask #1)

| Bước | Code chạy | Output | Ghi chú |
|------|-----------|--------|---------|
| 7 | `NextTickInsideTimeout` callback | `Next Tick inside Timeout` | `process.nextTick` chạy trước `Promise.then` |

---

### Phase 5 — Microtask Queue (sau nextTick)

| Bước | Code chạy | Output | Ghi chú |
|------|-----------|--------|---------|
| 8 | `PromiseInsideTimeout` callback | `Promise inside Timeout` | |

> Microtask rỗng → Event Loop lấy macrotask tiếp theo

---

### Phase 6 — Macrotask #2: `Timeout inside Promise`

| Bước | Code chạy | Output | Ghi chú |
|------|-----------|--------|---------|
| 9 | `console.log('Timeout inside Promise')` | `Timeout inside Promise` | Macrotask này được tạo trong Phase 2 |

---

## Sơ đồ tổng quan

```
Call Stack (Sync)
  ├─ Start
  ├─ setTimeout(Timeout1) ──────────────────► [Macrotask Queue]
  ├─ Promise.resolve().then(Promise1) ──────► [Microtask Queue]
  └─ End
         │
         ▼ (Call Stack rỗng)
  Microtask Queue
  └─ Promise1 ──► "Promise 1"
                  └─ setTimeout(Timeout2) ──► [Macrotask Queue]
         │
         ▼ (Macrotask #1)
  Timeout1 ──► "Timeout 1"
               ├─ Promise.resolve().then() ─► [Microtask Queue]
               └─ process.nextTick() ───────► [nextTick Queue]
         │
         ▼ (nextTick trước Promise!)
  "Next Tick inside Timeout"
  "Promise inside Timeout"
         │
         ▼ (Macrotask #2)
  Timeout2 ──► "Timeout inside Promise"
```

## Điểm mấu chốt

- **`process.nextTick`** không phải microtask thông thường — nó có **hàng đợi riêng** (`nextTick Queue`) và luôn chạy **trước** `Promise.then` sau mỗi lần Call Stack trống.
- Mỗi macrotask hoàn thành → flush **toàn bộ** nextTick Queue → flush **toàn bộ** Microtask Queue → mới lấy macrotask tiếp.
- `setTimeout` được tạo ra trong microtask (Phase 2) vẫn chạy **sau** `setTimeout` được đăng ký từ đầu — vì nó được thêm vào macrotask queue **muộn hơn**.