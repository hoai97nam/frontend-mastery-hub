# 🏔️ Heap — Đống

## 🧠 Định nghĩa dễ nhớ

> **Heap** giống như một **ban tổ chức giải đấu**: bạn không biết thứ tự xếp hạng toàn bộ đội — nhưng **luôn biết ngay** ai đang ở vị trí số 1 (đội mạnh nhất hoặc yếu nhất). Bất kể ai thắng thua mới, vị trí số 1 luôn được cập nhật ngay lập tức.

```
MAX-HEAP:           MIN-HEAP:
     100               1
    /   \            /   \
   19    36          5    3
  / \   /           / \
 17  3 25           8   9
```

**Quy tắc:**
- **Max-Heap**: Node cha **luôn ≥** các con → root = maximum
- **Min-Heap**: Node cha **luôn ≤** các con → root = minimum
- Là **Complete Binary Tree** → biểu diễn bằng Array hiệu quả

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Get Max/Min (peek) | **O(1)** | Root luôn là max/min |
| Insert | **O(log n)** | Bubble up |
| Extract Max/Min | **O(log n)** | Sink down |
| Build Heap từ Array | **O(n)** | Heapify — không phải O(n log n)! |
| Search | O(n) | Không phải điểm mạnh |
| Space | O(n) | — |

---

## 💻 Code JavaScript

### Heap dùng Array (chuẩn)

```javascript
// Complete Binary Tree → Array: index i
// Parent:     Math.floor((i-1) / 2)
// Left child: 2*i + 1
// Right child:2*i + 2
//
// Array: [100, 19, 36, 17, 3, 25]
//         0    1   2   3  4   5
// Index 1: parent = 0 (100), left child = 3 (17), right child = 4 (3)

class MinHeap {
  #data = [];

  get size() { return this.#data.length; }
  peek()     { return this.#data[0]; }
  isEmpty()  { return this.#data.length === 0; }

  // INSERT — O(log n)
  push(val) {
    this.#data.push(val);
    this.#bubbleUp(this.#data.length - 1);
  }

  // EXTRACT MIN — O(log n)
  pop() {
    if (this.isEmpty()) return undefined;
    const min = this.#data[0];
    const last = this.#data.pop();
    if (this.#data.length) {
      this.#data[0] = last;    // đưa phần tử cuối lên root
      this.#sinkDown(0);       // sink down để restore heap property
    }
    return min;
  }

  // BUBBLE UP — O(log n)
  #bubbleUp(i) {
    while (i > 0) {
      const parent = Math.floor((i - 1) / 2);
      if (this.#data[parent] <= this.#data[i]) break; // heap property satisfied
      [this.#data[parent], this.#data[i]] = [this.#data[i], this.#data[parent]];
      i = parent;
    }
  }

  // SINK DOWN — O(log n)
  #sinkDown(i) {
    const n = this.#data.length;
    while (true) {
      let smallest = i;
      const left  = 2 * i + 1;
      const right = 2 * i + 2;

      if (left  < n && this.#data[left]  < this.#data[smallest]) smallest = left;
      if (right < n && this.#data[right] < this.#data[smallest]) smallest = right;

      if (smallest === i) break; // đã đúng vị trí
      [this.#data[smallest], this.#data[i]] = [this.#data[i], this.#data[smallest]];
      i = smallest;
    }
  }

  // BUILD HEAP từ array — O(n)
  static buildFrom(arr) {
    const heap = new MinHeap();
    heap.#data = [...arr];
    // Heapify từ cuối lên: chỉ cần xử lý các internal nodes
    for (let i = Math.floor(arr.length / 2) - 1; i >= 0; i--) {
      heap.#sinkDown(i);
    }
    return heap;
  }
}

const heap = new MinHeap();
heap.push(5); heap.push(3); heap.push(8); heap.push(1); heap.push(9);
heap.peek();  // 1 — minimum luôn ở đỉnh
heap.pop();   // 1
heap.peek();  // 3 — minimum mới
```

### Ứng dụng: K Largest Elements — O(n log k)

```javascript
// Câu hỏi kinh điển: tìm k phần tử lớn nhất trong array
// Naive: sort O(n log n), take last k
// Optimal: Min-Heap size k — O(n log k) — tốt hơn khi k << n

function kLargest(nums, k) {
  const heap = new MinHeap();

  for (const num of nums) {
    heap.push(num);
    if (heap.size > k) heap.pop(); // bỏ phần tử nhỏ nhất nếu heap quá k
  }

  // Heap giờ chứa k phần tử lớn nhất
  const result = [];
  while (!heap.isEmpty()) result.push(heap.pop());
  return result.reverse(); // nhỏ → lớn
}

kLargest([3,1,4,1,5,9,2,6,5,3], 3); // [6, 9, 5] or [5, 6, 9]
```

### Heap Sort — O(n log n)

```javascript
function heapSort(arr) {
  const n = arr.length;

  // Build max-heap in-place
  for (let i = Math.floor(n/2) - 1; i >= 0; i--) {
    heapify(arr, n, i);
  }

  // Extract elements one by one
  for (let i = n - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]]; // move max to end
    heapify(arr, i, 0); // restore heap property for remaining
  }
  return arr;
}

function heapify(arr, n, i) {
  let largest = i;
  const left = 2*i+1, right = 2*i+2;
  if (left  < n && arr[left]  > arr[largest]) largest = left;
  if (right < n && arr[right] > arr[largest]) largest = right;
  if (largest !== i) {
    [arr[i], arr[largest]] = [arr[largest], arr[i]];
    heapify(arr, n, largest);
  }
}

heapSort([64, 34, 25, 12, 22, 11, 90]); // [11, 12, 22, 25, 34, 64, 90]
```

### Median of Data Stream

```javascript
// Dùng 2 heaps: maxHeap (lower half) + minHeap (upper half)
class MedianFinder {
  #maxHeap = new MaxHeap(); // lower half, root = max of lower
  #minHeap = new MinHeap(); // upper half, root = min of upper

  addNum(num) {
    this.#maxHeap.push(num);
    // Đảm bảo tất cả phần tử lower ≤ tất cả phần tử upper
    this.#minHeap.push(this.#maxHeap.pop());

    // Cân bằng kích thước: maxHeap có thể lớn hơn minHeap 1
    if (this.#maxHeap.size < this.#minHeap.size) {
      this.#maxHeap.push(this.#minHeap.pop());
    }
  }

  findMedian() {
    if (this.#maxHeap.size > this.#minHeap.size) return this.#maxHeap.peek();
    return (this.#maxHeap.peek() + this.#minHeap.peek()) / 2;
  }
}
```

---

## 🔧 Trong Framework & Thực tế

### React Scheduler — Priority Queue
```javascript
// React 18 Scheduler (react-scheduler package)
// Dùng Min-Heap để quản lý task priorities!

// Simplified Scheduler:
const taskQueue = new MinHeap(); // sorted by expirationTime

function scheduleCallback(priority, callback) {
  const task = {
    callback,
    priority,
    expirationTime: getCurrentTime() + priorityToTimeout(priority)
  };
  taskQueue.push(task); // O(log n)
}

// Work loop:
function workLoop() {
  let currentTask = taskQueue.peek(); // O(1)
  while (currentTask) {
    if (currentTask.expirationTime > getCurrentTime() && shouldYield()) break;
    taskQueue.pop(); // O(log n)
    currentTask.callback();
    currentTask = taskQueue.peek();
  }
}

// Priority levels:
// ImmediatePriority: -1ms (sync)
// UserBlockingPriority: 250ms (scroll, click)
// NormalPriority: 5000ms (default)
// LowPriority: 10000ms
// IdlePriority: Infinity
```

### Node.js — Timer Heap
```javascript
// Node.js Libuv dùng Min-Heap cho setTimeout/setInterval
// Heap key = expiration time
// Event loop: peek() → nếu expired → pop() và execute callback

// Khi bạn:
setTimeout(cb1, 1000);  // push vào heap với key = now + 1000
setTimeout(cb2, 500);   // push vào heap với key = now + 500
// Heap: [cb2(500), cb1(1000)]
// → cb2 được execute trước dù được register sau

// setImmediate là KHÁC: dùng Check Queue, không phải Heap
```

### OS — Process Scheduling
```javascript
// Tương tự: OS scheduler dùng Priority Heap
// Kubernetes pod scheduling dùng Priority Queue (Priority Class)
// Điều này ảnh hưởng khi bạn set resource limits trong Docker/K8s
```

### Dijkstra — Shortest Path
```javascript
// Dijkstra's algorithm cần Priority Queue (Min-Heap)
// Dùng trong: routing, network topology, A* search

function dijkstra(graph, start) {
  const distances = {};
  const heap = new MinHeap();  // [distance, node]
  
  for (const node in graph) distances[node] = Infinity;
  distances[start] = 0;
  heap.push([0, start]);

  while (!heap.isEmpty()) {
    const [dist, node] = heap.pop();
    if (dist > distances[node]) continue; // stale entry

    for (const [neighbor, weight] of graph[node] || []) {
      const newDist = dist + weight;
      if (newDist < distances[neighbor]) {
        distances[neighbor] = newDist;
        heap.push([newDist, neighbor]);
      }
    }
  }
  return distances;
}
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Build Heap là O(n), KHÔNG phải O(n log n)!
```javascript
// Nhiều người nghĩ: n inserts × O(log n) = O(n log n)
// THỰC TẾ: Heapify từ cuối = O(n)
// Proof: hầu hết nodes ở leaf levels → sink down ít bước
// → Tổng số operations = O(n) (toán học: sum của geometric series)

const arr = [3, 1, 4, 1, 5, 9, 2, 6];
const heap = MinHeap.buildFrom(arr); // O(n)!
```

### 2. Lazy Deletion — xóa phần tử bất kỳ trong heap
```javascript
// Heap không hỗ trợ delete arbitrary element hiệu quả
// Kỹ thuật: đánh dấu "deleted" thay vì xóa thực sự

class LazyHeap {
  #heap = new MinHeap();
  #deleted = new Set();

  push(val) { this.#heap.push(val); }

  remove(val) { this.#deleted.add(val); } // O(1)!

  pop() {
    // Bỏ qua các phần tử đã đánh dấu xóa
    while (!this.#heap.isEmpty() && this.#deleted.has(this.#heap.peek())) {
      this.#deleted.delete(this.#heap.peek());
      this.#heap.pop();
    }
    return this.#heap.pop();
  }
}
// Use case: event cancellation trong scheduler
```

### 3. D-ary Heap — tối ưu cho modern hardware
```javascript
// Binary heap = 2-ary heap
// 4-ary hoặc 8-ary heap: ít levels hơn → ít cache misses
// → Nhanh hơn trong practice dù O() giống nhau
// Fibonacci Heap: O(1) amortized decrease-key → tốt cho Dijkstra trên sparse graph
```

---

## ❓ Bẫy phỏng vấn

### Q1: Heap vs BST — khi nào dùng cái nào?
```
Heap:  Get min/max O(1), Insert/Extract O(log n), KHÔNG support arbitrary search
BST:   Search/Insert/Delete O(log n), support range queries, ordered traversal

→ Chỉ cần min/max nhanh → Heap
→ Cần search, floor, ceiling, kth → BST / Red-Black Tree
```

### Q2: Tại sao Heap Sort không được dùng trong practice?
```
Heap Sort: O(n log n) worst, O(1) space
Quick Sort: O(n log n) average, O(n²) worst — nhưng nhanh hơn trong practice

Lý do Heap Sort chậm hơn:
- Cache unfriendly: truy cập memory không tuần tự (jump giữa parent-child)
- Quick Sort có tính locality tốt hơn → ít cache miss
→ V8 (JavaScript) dùng TimSort (hybrid MergeSort + InsertionSort)
```

### Q3: Merge K sorted arrays — dùng Heap như thế nào?
```javascript
function mergeKSorted(arrays) {
  const heap = new MinHeap(); // [value, arrayIndex, elementIndex]
  const result = [];

  // Init: lấy phần tử đầu tiên của mỗi array
  for (let i = 0; i < arrays.length; i++) {
    if (arrays[i].length) heap.push([arrays[i][0], i, 0]);
  }

  while (!heap.isEmpty()) {
    const [val, arrIdx, elemIdx] = heap.pop();
    result.push(val);
    // Thêm phần tử tiếp theo từ cùng array
    if (elemIdx + 1 < arrays[arrIdx].length) {
      heap.push([arrays[arrIdx][elemIdx+1], arrIdx, elemIdx+1]);
    }
  }
  return result;
}
// O(n log k) — n = tổng phần tử, k = số arrays
```

---

## 🔗 Xem thêm
- [Queue](../01-linear/queue.md) — Priority Queue implement bằng Heap
- [Graph](../04-graph/graph.md) — Dijkstra dùng Heap
- [Binary Tree](./binary-tree.md) — Heap là Complete Binary Tree
