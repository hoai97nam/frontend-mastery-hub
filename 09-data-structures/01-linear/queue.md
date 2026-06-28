# 🚶 Queue — Hàng đợi

## 🧠 Định nghĩa dễ nhớ

> **Queue** giống như **hàng chờ mua vé**: ai đến trước được phục vụ trước, ai đến sau đứng cuối hàng. **FIFO: First In, First Out**.

```
ENQUEUE →  [Front] 1 ← 2 ← 3 ← 4 [Back] ← ENQUEUE
           DEQUEUE ↑
```

**Các biến thể:**
- **Simple Queue**: FIFO cơ bản
- **Circular Queue**: Queue vòng tròn — buffer tái sử dụng bộ nhớ
- **Priority Queue**: Phần tử nào ưu tiên cao hơn ra trước (dùng Heap bên dưới)

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Enqueue (thêm vào cuối) | **O(1)** | — |
| Dequeue (lấy từ đầu) | **O(1)**† | †Nếu dùng Linked List |
| Peek (xem đầu) | **O(1)** | — |
| Search | O(n) | — |

> ⚠️ **Dùng Array làm Queue**: `shift()` là **O(n)** vì phải dịch chuyển tất cả phần tử!  
> ✅ **Giải pháp**: Dùng Linked List hoặc chỉ số `head` pointer

---

## 💻 Code JavaScript

### Cách 1: Array (đơn giản, nhưng dequeue chậm)

```javascript
const queue = [];
queue.push('A');    // enqueue
queue.push('B');
queue.push('C');
const front = queue.shift(); // dequeue: 'A' — O(n) ⚠️
```

### Cách 2: Queue hiệu quả với head pointer

```javascript
// Tránh shift() — dùng index để "giả vờ" xóa phần tử đầu
class EfficientQueue {
  #data = {};
  #head = 0;
  #tail = 0;

  enqueue(item) { this.#data[this.#tail++] = item; }

  dequeue() {
    if (this.isEmpty()) return undefined;
    const item = this.#data[this.#head];
    delete this.#data[this.#head++];
    return item;
  }

  peek()    { return this.#data[this.#head]; }
  isEmpty() { return this.#head === this.#tail; }
  size()    { return this.#tail - this.#head; }
}

const q = new EfficientQueue();
q.enqueue(1); q.enqueue(2); q.enqueue(3);
console.log(q.dequeue()); // 1
console.log(q.size());    // 2
```

### Cách 3: Queue bằng Doubly Linked List (optimal)

```javascript
class QueueNode { constructor(v) { this.val = v; this.next = null; } }

class LinkedQueue {
  #head = null;
  #tail = null;
  #size = 0;

  enqueue(val) {
    const node = new QueueNode(val);
    if (!this.#tail) { this.#head = this.#tail = node; }
    else { this.#tail.next = node; this.#tail = node; }
    this.#size++;
  }

  dequeue() {
    if (!this.#head) return undefined;
    const val = this.#head.val;
    this.#head = this.#head.next;
    if (!this.#head) this.#tail = null;
    this.#size--;
    return val;
  }

  peek()    { return this.#head?.val; }
  isEmpty() { return this.#size === 0; }
  size()    { return this.#size; }
}
```

### Ứng dụng: BFS — Tìm đường ngắn nhất

```javascript
// BFS = Breadth-First Search — luôn dùng Queue!
function bfs(graph, start) {
  const visited = new Set([start]);
  const queue = [start];
  const order = [];

  while (queue.length) {
    const node = queue.shift(); // ⚠️ O(n) — chấp nhận được nếu không quá lớn
    order.push(node);

    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
  return order;
}

const graph = { A: ['B','C'], B: ['D','E'], C: ['F'], D: [], E: [], F: [] };
bfs(graph, 'A'); // ['A', 'B', 'C', 'D', 'E', 'F']
```

### Priority Queue (Min-Heap based)

```javascript
// Priority Queue không phải FIFO — phần tử ưu tiên cao ra trước
// Xem: 03-tree/heap.md để hiểu Heap là gì
class MinPriorityQueue {
  #heap = [];

  enqueue(item, priority) {
    this.#heap.push({ item, priority });
    this.#bubbleUp();
  }

  dequeue() {
    if (this.#heap.length === 1) return this.#heap.pop().item;
    const min = this.#heap[0];
    this.#heap[0] = this.#heap.pop();
    this.#sinkDown(0);
    return min.item;
  }

  #bubbleUp() {
    let i = this.#heap.length - 1;
    while (i > 0) {
      const parent = Math.floor((i - 1) / 2);
      if (this.#heap[parent].priority <= this.#heap[i].priority) break;
      [this.#heap[parent], this.#heap[i]] = [this.#heap[i], this.#heap[parent]];
      i = parent;
    }
  }

  #sinkDown(i) {
    const n = this.#heap.length;
    while (true) {
      let smallest = i;
      const left = 2 * i + 1, right = 2 * i + 2;
      if (left < n && this.#heap[left].priority < this.#heap[smallest].priority) smallest = left;
      if (right < n && this.#heap[right].priority < this.#heap[smallest].priority) smallest = right;
      if (smallest === i) break;
      [this.#heap[smallest], this.#heap[i]] = [this.#heap[i], this.#heap[smallest]];
      i = smallest;
    }
  }

  isEmpty() { return this.#heap.length === 0; }
}

const pq = new MinPriorityQueue();
pq.enqueue('low priority task', 10);
pq.enqueue('urgent task', 1);
pq.enqueue('normal task', 5);
console.log(pq.dequeue()); // 'urgent task' (priority 1)
console.log(pq.dequeue()); // 'normal task' (priority 5)
```

---

## 🔧 Trong Framework & Thực tế

### Node.js — Event Loop
```javascript
// Node.js có nhiều Queue xếp hàng:
// 1. Microtask Queue: Promise.then, queueMicrotask (cao nhất)
// 2. Timer Queue: setTimeout, setInterval
// 3. I/O Queue: fs.readFile callback
// 4. Check Queue: setImmediate

// Thứ tự ưu tiên: Microtask > Timer > I/O > Check
setTimeout(() => console.log('timer'), 0);
Promise.resolve().then(() => console.log('microtask'));
// Output: 'microtask' trước, sau đó 'timer'
```

### React — Concurrent Mode Task Queue
```javascript
// React 18 có Scheduler package với Priority Queue
// Mỗi update có priority: ImmediatePriority, UserBlockingPriority, NormalPriority...
// React dequeue task theo priority, không phải FIFO thuần túy

// Khi bạn dùng:
startTransition(() => setState(newState));
// React đặt update này vào queue với priority thấp hơn
// → UI không bị block, user vẫn có thể tương tác
```

### Angular — Zone.js Change Detection Queue
```javascript
// Angular dùng Queue để batch change detection
// zone.js intercept async operations (setTimeout, Promise, XHR)
// Mỗi async operation completion → đẩy change detection vào queue
// → Angular process tất cả rồi mới update DOM một lần
```

### Redux — Action Queue (Middleware)
```javascript
// Redux Saga dùng Channel (= Queue) để process actions
import { channel } from 'redux-saga';
const chan = channel();

// put vào channel = enqueue
// take từ channel = dequeue
// → Kiểm soát concurrency, tránh race condition
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Web Workers dùng MessageChannel (Queue)
```javascript
// Giao tiếp giữa main thread và Web Worker là Queue-based
const worker = new Worker('worker.js');
worker.postMessage({ data: 'process this' }); // enqueue message
worker.onmessage = (e) => console.log(e.data); // dequeue khi có kết quả

// MessageChannel cho phép 2 chiều
const { port1, port2 } = new MessageChannel();
port1.onmessage = (e) => console.log('received:', e.data);
port2.postMessage('hello from port2');
```

### 2. requestAnimationFrame Queue
```javascript
// rAF callbacks được đặt vào một Queue đặc biệt
// Chỉ được xử lý trước mỗi frame render (~60fps)
// → Batching DOM updates hiệu quả

function animate() {
  updateDOM(); // update state
  requestAnimationFrame(animate); // enqueue lần tiếp theo
}
requestAnimationFrame(animate);
```

### 3. Double Buffer Queue (Producer-Consumer)
```javascript
// Kỹ thuật: 2 queues để tránh lock contention
class DoubleBufferQueue {
  #active = [];
  #standby = [];

  produce(item) { this.#active.push(item); }

  // Swap buffers và process tất cả cùng lúc
  consumeAll() {
    [this.#active, this.#standby] = [this.#standby, this.#active];
    const items = this.#standby.splice(0);
    return items;
  }
}
// Dùng trong game loops, batch processing
```

---

## ❓ Bẫy phỏng vấn

### Q1: Tại sao dùng Array.shift() cho Queue là vấn đề?
```javascript
// shift() phải di chuyển TẤT CẢ phần tử còn lại sang trái → O(n)
// Với 1 triệu phần tử: 1 triệu operations mỗi lần dequeue!

// ✅ Giải pháp: Dùng head pointer hoặc Linked List
const q = new EfficientQueue(); // O(1) dequeue
```

### Q2: Implement Queue chỉ dùng 2 Stacks
```javascript
// Trick phỏng vấn kinh điển — Amortized O(1) dequeue
class QueueFromStacks {
  #inbox = [];   // dùng để enqueue
  #outbox = [];  // dùng để dequeue

  enqueue(x) { this.#inbox.push(x); }

  dequeue() {
    if (this.#outbox.length === 0) {
      // Đổ tất cả từ inbox sang outbox (đảo ngược thứ tự)
      while (this.#inbox.length) {
        this.#outbox.push(this.#inbox.pop());
      }
    }
    return this.#outbox.pop();
  }
}
// Mỗi phần tử được move 1 lần → Amortized O(1)
```

### Q3: Circular Queue — tại sao cần?
```javascript
// Circular Queue tái sử dụng memory đã được giải phóng
// Dùng trong: Audio/Video streaming buffer, Ring buffer

class CircularQueue {
  #data;
  #head = 0;
  #tail = 0;
  #size = 0;
  #capacity;

  constructor(capacity) {
    this.#capacity = capacity;
    this.#data = new Array(capacity);
  }

  enqueue(val) {
    if (this.isFull()) return false;
    this.#data[this.#tail] = val;
    this.#tail = (this.#tail + 1) % this.#capacity; // wrap around!
    this.#size++;
    return true;
  }

  dequeue() {
    if (this.isEmpty()) return false;
    this.#head = (this.#head + 1) % this.#capacity; // wrap around!
    this.#size--;
    return true;
  }

  isFull()  { return this.#size === this.#capacity; }
  isEmpty() { return this.#size === 0; }
}
```

---

## 🔗 Xem thêm
- [Stack](./stack.md) — LIFO, ngược với Queue
- [Deque](./deque.md) — Queue 2 đầu
- [Heap](../03-tree/heap.md) — Priority Queue implementation
