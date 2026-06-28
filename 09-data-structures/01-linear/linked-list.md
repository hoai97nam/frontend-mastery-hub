# 🔗 Linked List — Danh sách liên kết

## 🧠 Định nghĩa dễ nhớ

> **Linked List** giống như một **đoàn tàu**: mỗi toa (node) chứa hàng hóa (data) và có móc nối đến toa tiếp theo (next pointer). Muốn đến toa số 5, bạn **phải đi từ đầu tàu** và qua từng toa một — không thể nhảy trực tiếp.

```
[Head]  →  [Node 1]  →  [Node 2]  →  [Node 3]  →  null
           data: 10     data: 20     data: 30
           next: →      next: →      next: null
```

**Hai loại chính:**
- **Singly Linked List**: Mỗi node chỉ biết node *tiếp theo*
- **Doubly Linked List**: Mỗi node biết cả *tiếp theo* và *trước đó*

---

## ⏱️ Big-O Complexity

| Thao tác | Singly | Doubly | Ghi chú |
|----------|--------|--------|---------|
| Access theo index | O(n) | O(n) | Phải duyệt từ đầu |
| Search | O(n) | O(n) | Duyệt tuyến tính |
| Insert at head | **O(1)** | **O(1)** | Điểm mạnh! |
| Insert at tail | O(n)* | **O(1)** | *Nếu có tail pointer thì O(1) |
| Insert at middle | O(n) | O(n) | Tìm vị trí O(n), chèn O(1) |
| Delete at head | **O(1)** | **O(1)** | — |
| Delete at tail | O(n) | **O(1)** | Doubly mạnh hơn đây |

> 💡 **Space**: O(n) — nhưng tốn thêm memory cho pointer (8 bytes mỗi node)

---

## 💻 Code JavaScript

### Singly Linked List — từ đầu

```javascript
class Node {
  constructor(data) {
    this.data = data;
    this.next = null;
  }
}

class SinglyLinkedList {
  constructor() {
    this.head = null;
    this.size = 0;
  }

  // Thêm vào đầu — O(1) ✅
  prepend(data) {
    const node = new Node(data);
    node.next = this.head;
    this.head = node;
    this.size++;
  }

  // Thêm vào cuối — O(n)
  append(data) {
    const node = new Node(data);
    if (!this.head) { this.head = node; this.size++; return; }

    let current = this.head;
    while (current.next) current = current.next;
    current.next = node;
    this.size++;
  }

  // Xóa theo giá trị — O(n)
  delete(data) {
    if (!this.head) return;
    if (this.head.data === data) {
      this.head = this.head.next;
      this.size--;
      return;
    }
    let current = this.head;
    while (current.next) {
      if (current.next.data === data) {
        current.next = current.next.next; // bỏ qua node cần xóa
        this.size--;
        return;
      }
      current = current.next;
    }
  }

  // Duyệt in ra
  print() {
    const result = [];
    let current = this.head;
    while (current) {
      result.push(current.data);
      current = current.next;
    }
    return result.join(' → ');
  }

  // Đảo ngược — O(n) — câu hỏi phỏng vấn kinh điển!
  reverse() {
    let prev = null;
    let current = this.head;
    while (current) {
      const next = current.next; // lưu lại next trước
      current.next = prev;       // đảo chiều
      prev = current;
      current = next;
    }
    this.head = prev;
  }
}

const list = new SinglyLinkedList();
list.append(10);
list.append(20);
list.append(30);
list.prepend(5);
console.log(list.print()); // 5 → 10 → 20 → 30
list.reverse();
console.log(list.print()); // 30 → 20 → 10 → 5
```

### Doubly Linked List

```javascript
class DNode {
  constructor(data) {
    this.data = data;
    this.prev = null;
    this.next = null;
  }
}

class DoublyLinkedList {
  constructor() {
    this.head = null;
    this.tail = null;
    this.size = 0;
  }

  append(data) {
    const node = new DNode(data);
    if (!this.tail) {
      this.head = this.tail = node;
    } else {
      node.prev = this.tail;
      this.tail.next = node;
      this.tail = node;
    }
    this.size++;
  }

  // Xóa node cuối — O(1) ← Doubly mạnh hơn Singly
  removeTail() {
    if (!this.tail) return null;
    const data = this.tail.data;
    if (this.tail === this.head) {
      this.head = this.tail = null;
    } else {
      this.tail = this.tail.prev;
      this.tail.next = null;
    }
    this.size--;
    return data;
  }
}
```

### Kỹ thuật phát hiện vòng (Floyd's Cycle Detection)

```javascript
// Phỏng vấn kinh điển: "Kiểm tra Linked List có vòng không?"
function hasCycle(head) {
  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next;       // đi 1 bước
    fast = fast.next.next;  // đi 2 bước

    if (slow === fast) return true; // gặp nhau → có vòng
  }
  return false;
}

// Biến thể: Tìm điểm bắt đầu của vòng
function detectCycleStart(head) {
  let slow = head, fast = head;
  while (fast?.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) {
      // Đặt một pointer về head, cùng tốc độ → gặp tại điểm bắt đầu vòng
      slow = head;
      while (slow !== fast) { slow = slow.next; fast = fast.next; }
      return slow;
    }
  }
  return null;
}
```

---

## 🔧 Trong Framework & Thực tế

### React — Fiber Architecture
```javascript
// React Fiber là một Linked List khổng lồ!
// Mỗi component = một Fiber node với .child, .sibling, .return (parent)

// Fiber node structure (simplified)
const fiberNode = {
  type: 'div',
  child: childFiber,      // con đầu tiên
  sibling: nextSibling,   // anh em tiếp theo
  return: parentFiber,    // cha (quay về)
  alternate: workInProgress, // bản sao để reconcile
};
// React duyệt tree này như một Linked List — không dùng recursion
// → Có thể pause, resume, abort render bất cứ lúc nào!
```

### Vue — Dependency Tracking
```javascript
// Vue 2 dùng Linked List để track watchers
// Mỗi reactive property có một "dep" chứa danh sách subscriber
// Khi data thay đổi → duyệt linked list → notify tất cả watchers
```

### Browser — DOM Structure  
```javascript
// DOM là cây, nhưng mỗi cấp là một Linked List!
// element.nextSibling, element.previousSibling
// element.firstChild, element.lastChild

const children = [];
let node = parent.firstChild;
while (node) {
  if (node.nodeType === Node.ELEMENT_NODE) children.push(node);
  node = node.nextSibling; // duyệt như linked list
}
```

### LRU Cache (ứng dụng thực tế nhất)
```javascript
// LRU Cache = HashMap + Doubly Linked List
// HashMap: O(1) tìm kiếm
// Doubly Linked List: O(1) move-to-front và remove-from-tail
// → Xem chi tiết: 05-modern/lru-cache.md
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. JS không có built-in Linked List — nhưng có thể dùng Map như thế
```javascript
// Nhiều người implement LRU Cache bằng Map + object tự làm
// Thực ra: Map trong JavaScript duy trì insertion order
// Nên có thể dùng Map như một "ordered hash" — xem lru-cache.md
```

### 2. Sentinel Node (Dummy Head) — loại bỏ edge case
```javascript
// Thêm một node giả ở đầu → không cần xử lý head === null riêng
class BetterLinkedList {
  constructor() {
    this.dummy = new Node(null); // sentinel
    this.dummy.next = null;
    this.size = 0;
  }

  prepend(data) {
    const node = new Node(data);
    node.next = this.dummy.next;
    this.dummy.next = node; // luôn thao tác với dummy.next, không cần check null
    this.size++;
  }
}
```

### 3. Runner Technique — tìm middle node
```javascript
// Tìm node giữa: slow đi 1 bước, fast đi 2 bước
function findMiddle(head) {
  let slow = head, fast = head;
  while (fast?.next?.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  return slow; // slow đang ở giữa khi fast đến cuối
}
// Dùng để: chia đôi list (Merge Sort), tìm palindrome
```

---

## ❓ Bẫy phỏng vấn

### Q1: Array vs Linked List — chọn cái nào?
```
Array:       Random access O(1), Cache-friendly, ít overhead memory
Linked List: Insert/delete đầu O(1), Dynamic size, không cần contiguous memory

→ Thực tế frontend: Array dùng 90% trường hợp
→ Linked List nổi trội khi: implement Stack/Queue, LRU Cache, undo-redo
```

### Q2: Đảo ngược Linked List (câu hỏi #1 phỏng vấn Big Tech)
```javascript
// Phải làm in-place, không dùng extra array
// Ba biến: prev, current, next
function reverseList(head) {
  let prev = null, curr = head;
  while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }
  return prev; // prev là head mới
}
```

### Q3: Tại sao React Fiber dùng Linked List thay vì Call Stack?
```
Call Stack: Không thể pause/resume — phải chạy đến hết
Linked List: React có thể lưu "con trỏ hiện tại" và dừng lại bất cứ lúc nào
→ Concurrent Mode / Suspense hoạt động nhờ điều này
```

---

## 🔗 Xem thêm
- [Stack](./stack.md) — implement bằng Linked List
- [Queue](./queue.md) — implement bằng Linked List cho hiệu năng tốt hơn Array
- [LRU Cache](../05-modern/lru-cache.md) — ứng dụng thực tế của Doubly Linked List
