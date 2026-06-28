# ↔️ Deque — Double-Ended Queue

## 🧠 Định nghĩa dễ nhớ

> **Deque** (đọc là "deck") giống như một **đường hầm 2 đầu**: bạn có thể cho người vào/ra từ **cả hai phía** — không bị giới hạn như Queue (một chiều) hay Stack (một đầu).

```
addFront ← [Front] ↔ [A] ↔ [B] ↔ [C] [Back] → addBack
removeFront → ...                          ... ← removeBack
```

**Deque = Stack + Queue**: Nó thực hiện được chức năng của cả hai!

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| addFront / addBack | **O(1)** | — |
| removeFront / removeBack | **O(1)** | — |
| peekFront / peekBack | **O(1)** | — |
| Search | O(n) | — |
| Space | O(n) | — |

---

## 💻 Code JavaScript

### Implementation

```javascript
class Deque {
  #data = new Map();
  #front = 0;
  #back = 0;

  addBack(val) {
    this.#data.set(this.#back++, val);
  }

  addFront(val) {
    this.#data.set(--this.#front, val);
  }

  removeBack() {
    if (this.isEmpty()) return undefined;
    const val = this.#data.get(--this.#back);
    this.#data.delete(this.#back);
    return val;
  }

  removeFront() {
    if (this.isEmpty()) return undefined;
    const val = this.#data.get(this.#front);
    this.#data.delete(this.#front++);
    return val;
  }

  peekFront() { return this.#data.get(this.#front); }
  peekBack()  { return this.#data.get(this.#back - 1); }
  isEmpty()   { return this.#front === this.#back; }
  size()      { return this.#back - this.#front; }
}

const dq = new Deque();
dq.addBack(1);
dq.addBack(2);
dq.addFront(0);
console.log(dq.peekFront()); // 0
console.log(dq.peekBack());  // 2
dq.removeFront(); // 0
dq.removeBack();  // 2
```

### Ứng dụng kinh điển: Sliding Window Maximum

```javascript
// Bài toán: Tìm giá trị lớn nhất trong mỗi cửa sổ kích thước k — O(n)
// Đây là ứng dụng chuẩn bài nhất của Deque!
function maxSlidingWindow(nums, k) {
  const result = [];
  const deque = []; // chứa INDEX, không phải giá trị
                    // Deque luôn giữ thứ tự giảm dần theo giá trị

  for (let i = 0; i < nums.length; i++) {
    // Xóa index cũ ra khỏi window
    if (deque.length && deque[0] <= i - k) deque.shift();

    // Xóa các phần tử nhỏ hơn phần tử hiện tại (vô dụng rồi)
    while (deque.length && nums[deque.at(-1)] <= nums[i]) deque.pop();

    deque.push(i);

    // Bắt đầu có kết quả sau k phần tử đầu tiên
    if (i >= k - 1) result.push(nums[deque[0]]);
  }
  return result;
}

maxSlidingWindow([1,3,-1,-3,5,3,6,7], 3); // [3,3,5,5,6,7]
// Window:  [1,3,-1] [3,-1,-3] [-1,-3,5] [-3,5,3] [5,3,6] [3,6,7]
// Max:         3        3         5         5       6       7
```

### Deque dùng như Stack và Queue cùng lúc

```javascript
// Palindrome check — dùng cả 2 đầu
function isPalindrome(str) {
  const deque = [...str];
  while (deque.length > 1) {
    if (deque.shift() !== deque.pop()) return false;
  }
  return true;
}

isPalindrome('racecar'); // true
isPalindrome('hello');   // false
```

---

## 🔧 Trong Framework & Thực tế

### React Scheduler — Task Scheduling
```javascript
// React Scheduler (trong react-scheduler package) dùng
// Min-heap + deque-like structure để manage tasks
// High priority tasks được thêm vào đầu
// Normal priority tasks được thêm vào cuối
```

### Browser — Render Pipeline
```javascript
// Chrome's compositor thread dùng deque cho tile processing
// Tiles được added từ cả 2 đầu dựa trên visibility
// → Render những gì người dùng đang nhìn thấy trước
```

### Animation Libraries
```javascript
// GSAP, Framer Motion dùng deque-like structure
// để manage animation queue với khả năng insert từ cả 2 đầu
// pauseAll() / resumeAll() hoạt động trên toàn bộ deque
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Work-Stealing Algorithm (Node.js Libuv)
```javascript
// Libuv (underlying Node.js I/O) dùng work-stealing deque
// Thread pool: mỗi thread có một deque của tasks
// Khi một thread nhàn rỗi → "steal" task từ ĐUÔI deque của thread khác
// → Minimize synchronization, maximize CPU utilization
// Đây là lý do Node.js xử lý I/O hiệu quả!
```

### 2. Browser Back/Forward Navigation
```javascript
// Thực ra browser dùng Deque, không phải 2 Stack
// Current page = giữa deque
// Back = removeFront từ phía "future"
// Forward = removeBack từ phía "past"
// New navigation = clear phần future, append vào past

// Simulation:
const history = new Deque();
history.addBack('page1');
history.addBack('page2'); // navigate to page2
history.addBack('page3'); // navigate to page3
// current = page3 (back)
history.removeBack(); // go back to page2
```

### 3. Deque trong Game Development (Input Buffer)
```javascript
// Fighting games dùng Deque cho input buffer
// Khi player nhấn phím → addBack vào deque
// Game đọc từ front để process inputs theo thứ tự
// Nhưng khi buffer đầy → removeBack (bỏ input cũ nhất từ sau)
// hoặc removeFront (bỏ input cũ nhất từ trước)
```

---

## ❓ Bẫy phỏng vấn

### Q1: Deque vs Queue vs Stack — khác nhau chỗ nào?
```
Stack: thêm/xóa ở 1 đầu  (top)           — LIFO
Queue: thêm ở cuối, xóa ở đầu           — FIFO
Deque: thêm/xóa ở CẢ HAI đầu           — Linh hoạt nhất
```

### Q2: Khi nào dùng Deque thay vì Queue?
```
→ Cần undo/redo + forward navigation (2 chiều)
→ Sliding window maximum/minimum problems
→ Khi cần insert từ cả 2 đầu với O(1)
→ Implement Deque-based priority với 2 levels
```

### Q3: Implement bài Sliding Window Maximum — tại sao Deque?
```
Brute force: O(n*k) — mỗi window tìm max trong k phần tử
Deque:       O(n)   — mỗi phần tử được add/remove tối đa 1 lần

Deque giữ: index các phần tử "có thể là max"
- Loại bỏ phần tử nào nhỏ hơn phần tử mới (không bao giờ là max nữa)
- Front luôn là index của max trong window hiện tại
```

---

## 🔗 Xem thêm
- [Stack](./stack.md) — LIFO
- [Queue](./queue.md) — FIFO  
- [Segment Tree](../03-tree/segment-tree.md) — cũng giải được Sliding Window nhưng phức tạp hơn
