# 📚 Stack — Ngăn xếp

## 🧠 Định nghĩa dễ nhớ

> **Stack** giống như một **chồng đĩa** trong nhà bếp: bạn chỉ có thể đặt đĩa lên **trên cùng** và chỉ có thể lấy đĩa từ **trên cùng**. Cái đĩa bạn đặt vào cuối cùng sẽ là cái bạn lấy ra đầu tiên — **LIFO: Last In, First Out**.

```
PUSH → [5]        POP → [4]
       [4]              [3]
       [3]              [2]
       [2]              [1]
       [1]         ← lấy từ đỉnh
```

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Push (thêm vào đỉnh) | **O(1)** | — |
| Pop (lấy từ đỉnh) | **O(1)** | — |
| Peek (xem đỉnh) | **O(1)** | Không xóa |
| Search | O(n) | Không phải điểm mạnh |
| Space | O(n) | — |

---

## 💻 Code JavaScript

### Cách 1: Dùng Array (đơn giản nhất)

```javascript
// JavaScript Array đã là Stack hoàn hảo!
const stack = [];

stack.push(1);     // [1]
stack.push(2);     // [1, 2]
stack.push(3);     // [1, 2, 3]

const top = stack[stack.length - 1]; // peek: 3
const popped = stack.pop();          // pop: 3, stack = [1, 2]

console.log(stack.length === 0); // isEmpty
```

### Cách 2: Class đầy đủ (phỏng vấn)

```javascript
class Stack {
  #data = []; // private field (ES2022)

  push(item)  { this.#data.push(item); }
  pop()       { return this.#data.pop(); }
  peek()      { return this.#data[this.#data.length - 1]; }
  isEmpty()   { return this.#data.length === 0; }
  size()      { return this.#data.length; }
  clear()     { this.#data = []; }

  // Iterator support
  [Symbol.iterator]() {
    let index = this.#data.length - 1;
    const data = this.#data;
    return {
      next() {
        return index >= 0
          ? { value: data[index--], done: false }
          : { done: true };
      }
    };
  }
}
```

### Ứng dụng 1: Kiểm tra dấu ngoặc hợp lệ

```javascript
// Bài toán kinh điển: "()[]{}" → valid, "(]" → invalid
function isValidBrackets(s) {
  const stack = [];
  const map = { ')': '(', '}': '{', ']': '[' };

  for (const char of s) {
    if ('({['.includes(char)) {
      stack.push(char);             // mở ngoặc → push
    } else {
      if (stack.pop() !== map[char]) return false; // đóng ngoặc → kiểm tra
    }
  }
  return stack.length === 0; // stack trống = hợp lệ
}

isValidBrackets("()[]{}");  // true
isValidBrackets("([)]");    // false
isValidBrackets("{[]}");    // true
```

### Ứng dụng 2: Undo/Redo

```javascript
class TextEditor {
  #content = '';
  #undoStack = [];
  #redoStack = [];

  type(text) {
    this.#undoStack.push(this.#content); // lưu trạng thái cũ
    this.#redoStack = [];                // redo stack bị xóa khi có action mới
    this.#content += text;
  }

  undo() {
    if (this.#undoStack.length === 0) return;
    this.#redoStack.push(this.#content);
    this.#content = this.#undoStack.pop();
  }

  redo() {
    if (this.#redoStack.length === 0) return;
    this.#undoStack.push(this.#content);
    this.#content = this.#redoStack.pop();
  }

  get content() { return this.#content; }
}

const editor = new TextEditor();
editor.type('Hello');
editor.type(' World');
console.log(editor.content); // "Hello World"
editor.undo();
console.log(editor.content); // "Hello"
editor.redo();
console.log(editor.content); // "Hello World"
```

### Ứng dụng 3: Evaluate Expression (Monotonic Stack)

```javascript
// Daily Temperatures — tìm ngày ấm hơn gần nhất (LeetCode #739)
function dailyTemperatures(temps) {
  const result = new Array(temps.length).fill(0);
  const stack = []; // lưu index, không phải giá trị

  for (let i = 0; i < temps.length; i++) {
    while (stack.length && temps[i] > temps[stack.at(-1)]) {
      const prevIdx = stack.pop();
      result[prevIdx] = i - prevIdx; // số ngày phải chờ
    }
    stack.push(i);
  }
  return result;
}

dailyTemperatures([73,74,75,71,69,72,76,73]);
// [1, 1, 4, 2, 1, 1, 0, 0]
```

---

## 🔧 Trong Framework & Thực tế

### JavaScript Call Stack
```javascript
// Đây là Stack nổi tiếng nhất trong JS!
function a() { b(); }
function b() { c(); }
function c() { console.trace(); }
a();
// Call Stack:
// c ← đỉnh
// b
// a
// main

// Stack Overflow = Call Stack đầy (thường do infinite recursion)
function infinite() { return infinite(); } // RangeError: Maximum call stack size exceeded
```

### Browser — History API
```javascript
// history.back() / history.forward() hoạt động như 2 Stacks
// Khi navigate forward → push vào "Forward Stack"
// Khi nhấn Back      → pop từ "Forward Stack", push vào "Back Stack"

// React Router dùng history object bên dưới
import { useNavigate } from 'react-router-dom';
const navigate = useNavigate();
navigate(-1); // tương đương history.back()
```

### React — useState Batching
```javascript
// React 18 dùng stack để batch updates
// Khi bạn gọi nhiều setState trong một event handler,
// React push tất cả vào một queue/stack và xử lý một lần
setState(1);
setState(2);
setState(3);
// Chỉ render 1 lần với giá trị cuối cùng
```

### Angular — Dependency Injection
```javascript
// Angular dùng Stack để resolve dependencies
// Khi inject ServiceA cần ServiceB cần ServiceC
// Angular push từng dependency lên stack theo thứ tự, pop khi done
// → Phát hiện circular dependency khi thấy item đã có trong stack
```

### Webpack — Module Resolution
```javascript
// Webpack build dependency graph bằng DFS + Stack
// Stack chứa các module đang được xử lý
// Nếu module A đã trong stack mà được request lại → circular dependency!
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Convert Recursion → Iteration bằng Stack
```javascript
// Recursion = dùng Call Stack ngầm
// Có thể luôn convert sang explicit Stack để tránh stack overflow

// ❌ Recursion (risk: stack overflow với cây lớn)
function dfs(node) {
  if (!node) return;
  console.log(node.val);
  dfs(node.left);
  dfs(node.right);
}

// ✅ Iteration với explicit Stack
function dfsIterative(root) {
  const stack = [root];
  while (stack.length) {
    const node = stack.pop();
    if (!node) continue;
    console.log(node.val);
    stack.push(node.right); // right trước vì LIFO
    stack.push(node.left);  // left được xử lý trước
  }
}
```

### 2. Monotonic Stack — pattern cực kỳ mạnh
```javascript
// Monotonic Stack = Stack luôn giữ thứ tự tăng (hoặc giảm)
// Giải quyết "tìm phần tử lớn/nhỏ hơn gần nhất" trong O(n)

// Next Greater Element — O(n) thay vì O(n²)
function nextGreaterElement(nums) {
  const result = new Array(nums.length).fill(-1);
  const stack = [];

  for (let i = 0; i < nums.length; i++) {
    while (stack.length && nums[i] > nums[stack.at(-1)]) {
      result[stack.pop()] = nums[i];
    }
    stack.push(i);
  }
  return result;
}
nextGreaterElement([2, 1, 2, 4, 3]); // [4, 2, 4, -1, -1]
```

### 3. Stack dùng trong CSS specificity
```javascript
// Browsers dùng stack-like approach để tính CSS specificity
// và resolve cascade — inline > id > class > element
// Đây là lý do devtools hiển thị "struck through" styles
```

---

## ❓ Bẫy phỏng vấn

### Q1: Implement Stack chỉ dùng Queue
```javascript
// Trick: Dùng 2 queues hoặc 1 queue và đảo ngược
class StackUsingQueue {
  #q = [];

  push(x) {
    this.#q.push(x);
    // Xoay tất cả phần tử cũ ra sau phần tử mới
    for (let i = 0; i < this.#q.length - 1; i++) {
      this.#q.push(this.#q.shift());
    }
  }
  pop()  { return this.#q.shift(); }
  top()  { return this.#q[0]; }
  empty(){ return this.#q.length === 0; }
}
```

### Q2: Min Stack — O(1) cho cả push, pop, và getMin
```javascript
class MinStack {
  #stack = [];
  #minStack = [];

  push(val) {
    this.#stack.push(val);
    // minStack luôn giữ min tính đến thời điểm hiện tại
    const currentMin = this.#minStack.length
      ? Math.min(val, this.#minStack.at(-1))
      : val;
    this.#minStack.push(currentMin);
  }

  pop() {
    this.#stack.pop();
    this.#minStack.pop();
  }

  getMin() { return this.#minStack.at(-1); } // O(1)!
}
```

### Q3: Tại sao async/await không làm "block" Call Stack?
```
- async function trả về Promise ngay lập tức → không block stack
- await suspends function, trả control về caller
- Khi Promise resolve → callback được đặt vào Microtask Queue
- Event Loop lấy từ Microtask Queue và push lên Call Stack khi stack rỗng
```

---

## 🔗 Xem thêm
- [Queue](./queue.md) — FIFO, đối nghịch với Stack
- [Linked List](./linked-list.md) — implement Stack bằng Linked List
- [Tree](../03-tree/binary-tree.md) — DFS dùng Stack
