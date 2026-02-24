1. Cơ chế vận hành (The Engine & Runtime)
Event Loop: Hiểu sâu về Microtasks (Promises, queueMicrotask) vs Macrotasks (setTimeout, I/O). Cách trình duyệt render giữa các task.

🏎️ Bản chất của Event Loop: Micro vs. Macro
JavaScript là đơn luồng (Single-threaded), nhưng trình duyệt thì không. Event Loop giúp JS "có vẻ" như chạy đa nhiệm bằng cách quản lý các hàng đợi (queues).

# Phân loại Task trong JavaScript Event Loop

## 1. Phân loại Task

| Loại Task      | Các API điển hình                          | Ưu tiên                  |
|----------------|--------------------------------------------|--------------------------|
| **Synchronous** | Code chạy trực tiếp trong Call Stack      | Cao nhất (Chặn mọi thứ) |
| **Microtasks** | Promise.then/catch/finally, queueMicrotask, MutationObserver | Trung bình (Chạy ngay sau Stack) |
| **Macrotasks** | setTimeout, setInterval, setImmediate (Node), I/O, UI Rendering | Thấp nhất (Chạy từng cái một) |

## 🔄 Luồng thực thi chi tiết (The Cycle)

Vòng lặp của Event Loop diễn ra theo các bước nghiêm ngặt sau:

1. **Execute Stack**: Chạy toàn bộ code đồng bộ trong Call Stack cho đến khi trống rỗng.
2. **Process Microtasks**: 
   - Kiểm tra Microtask Queue.
   - **Quan trọng**: Chạy tất cả các task đang có trong hàng đợi này. Nếu một Microtask lại tạo ra một Microtask mới, nó cũng sẽ được chạy luôn trong vòng này.
3. **Render (Nếu cần)**: Trình duyệt kiểm tra xem có cần cập nhật UI không (thường là mỗi 16.6ms để đạt 60fps).
4. **Execute ONE Macrotask**: Lấy duy nhất một task từ Macrotask Queue ra chạy. Sau đó quay lại Bước 2.

## 💡 Lưu ý cho Senior

- Nếu bạn viết một hàm **đệ quy bằng Promise (Microtask)**, trình duyệt sẽ bị treo (UI Freeze) vì nó mãi quanh quẩn ở bước 2 mà không bao giờ sang được bước 3 (Render).
- Ngược lại, nếu dùng **setTimeout (Macrotask)**, trình duyệt vẫn có khe hở để Render giữa các lần thực thi.

## 🎨 Cách trình duyệt Render giữa các Task

Đây là phần mà UI Developer cần cực kỳ quan tâm để tối ưu Performance:

- **Thời điểm Render**: Trình duyệt thường cố gắng render sau khi Microtask Queue đã trống. Tuy nhiên, nó không render sau mỗi Macrotask nếu tốc độ xử lý quá nhanh.
- **RequestAnimationFrame (rAF)**: Đây là một "hàng đợi đặc biệt". rAF chạy ngay trước khi trình duyệt thực hiện bước Paint (vẽ lên màn hình). Nó tối ưu hơn setTimeout cho animation vì nó đồng bộ với tần số quét của màn hình.

## 🧪 Code thí nghiệm thực tế

Hãy thử đoạn code này trong Console để thấy sự "thiên vị" của Microtask:

```javascript
console.log('1. Start');

setTimeout(() => {
  console.log('2. Timeout (Macrotask)');
}, 0);

Promise.resolve().then(() => {
  console.log('3. Promise 1 (Microtask)');
}).then(() => {
  console.log('4. Promise 2 (Microtask)');
});

queueMicrotask(() => {
  console.log('5. queueMicrotask (Microtask)');
});

console.log('6. End');
```

**Kết quả**: `1 → 6 → 3 → 4 → 5 → 2`

## 🚩 Dấu hiệu của một Senior khi trả lời phỏng vấn

Khi nói về chủ đề này, hãy chủ động đề cập đến:

- **Starvation**: Hiện trạng Microtask quá nhiều làm nghẽn Macrotask và Rendering.
- **Zero-delay timeout**: Giải thích tại sao `setTimeout(..., 0)` thực tế vẫn mất khoảng 4ms (theo spec) và không chạy ngay lập tức.
- **Performance Optimization**: Cách dùng `requestIdleCallback` để chạy các task vụn vặt khi trình duyệt đang rảnh rỗi, tránh làm lag Frame.

Memory Management: Cách Garbage Collection hoạt động (Mark-and-Sweep), nhận diện Memory Leaks (closures, global variables, forgotten timers).

Execution Context & Lexical Environment: Hiểu về Creation Phase và Execution Phase.

2. Functional & OOP Patterns
Prototypes: Sự khác biệt giữa __proto__ và prototype. Tại sao Class trong JS chỉ là "Syntactic Sugar"?

Functional Programming: Pure functions, Higher-order functions, Currying, và tính Immutability (bất biến).

This Keyword: Quy tắc binding (Default, Implicit, Explicit, New) và cách nó hoạt động trong Arrow functions.

3. Asynchronous & Modern APIs
Concurrency: Cách xử lý nhiều Promise cùng lúc (all, allSettled, race, any).

Web APIs nâng cao: Intersection Observer (tối ưu lazy load), Web Workers (xử lý đa luồng), Proxy & Reflect (nền tảng của Vue 3 reactivity).