# 🟣 LRU Cache — Least Recently Used Cache

## 🧠 Định nghĩa dễ nhớ

> **LRU Cache** giống như **bàn làm việc của bạn**: diện tích có hạn (dung lượng tối đa). Tài liệu nào bạn mới đọc xong sẽ để lên trên cùng (Recently Used). Khi bàn quá đầy mà bạn cần đặt tài liệu mới vào, bạn sẽ **dọn bỏ tài liệu nằm ở dưới cùng** — cái đã lâu nhất không có ai sờ tới (Least Recently Used).

Bản chất kỹ thuật: Sự kết hợp hoàn hảo giữa **HashMap** (để truy cập O(1)) và **Doubly Linked List** (để quản lý thứ tự sử dụng và di chuyển phần tử trong O(1)).

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Get | **O(1)** | Nhờ HashMap tra cứu trực tiếp node |
| Put (Insert / Update) | **O(1)** | Thêm node mới vào đầu danh sách liên kết |
| Evict (Giải phóng phần tử cũ) | **O(1)** | Xóa node ở đuôi danh sách liên kết |
| Space | **O(Capacity)** | Phụ thuộc vào giới hạn bộ nhớ được cấu hình |

---

## 💻 Code JavaScript

### LRU Cache sử dụng Map (Quirk đặc biệt của ES6)

Trong JavaScript ES6, `Map` duy trì **thứ tự chèn** của các key. Ta có thể tận dụng điều này để viết một LRU Cache siêu ngắn gọn:

```javascript
class SimpleLRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return -1;
    
    // Đọc ra giá trị, xóa đi rồi set lại để đưa nó lên làm "mới nhất" (cuối Map)
    const val = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, val);
    return val;
  }

  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // Đầu của Map (keys().next().value) là phần tử cũ nhất -> Xóa đi
      const oldestKey = this.cache.keys().next().value;
      this.cache.delete(oldestKey);
    }
    this.cache.set(key, value);
  }
}
```

### LRU Cache chuẩn bằng HashMap + Doubly Linked List (Phỏng vấn)

```javascript
class LRUNode {
  constructor(key, val) {
    this.key = key;
    this.val = val;
    this.prev = null;
    this.next = null;
  }
}

class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map(); // key -> LRUNode
    
    // Sentinel nodes (giả) để tránh kiểm tra null
    this.head = new LRUNode(0, 0);
    this.tail = new LRUNode(0, 0);
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }

  // Thêm node vào ngay sau head (vị trí mới nhất)
  #addNode(node) {
    node.prev = this.head;
    node.next = this.head.next;
    this.head.next.prev = node;
    this.head.next = node;
  }

  // Xóa liên kết của một node bất kỳ
  #removeNode(node) {
    const prev = node.prev;
    const next = node.next;
    prev.next = next;
    next.prev = prev;
  }

  // Đưa node hiện tại lên đầu (mới sử dụng)
  #moveToHead(node) {
    this.#removeNode(node);
    this.#addNode(node);
  }

  // Xóa phần tử cũ nhất ở sát tail
  #popTail() {
    const res = this.tail.prev;
    this.#removeNode(res);
    return res;
  }

  get(key) {
    const node = this.map.get(key);
    if (!node) return -1;
    
    this.#moveToHead(node); // Đánh dấu mới sử dụng
    return node.val;
  }

  put(key, value) {
    const node = this.map.get(key);
    
    if (node) {
      node.val = value; // Cập nhật giá trị
      this.#moveToHead(node);
    } else {
      const newNode = new LRUNode(key, value);
      this.map.set(key, newNode);
      this.#addNode(newNode);
      
      if (this.map.size > this.capacity) {
        // Vượt quá capacity -> Xóa phần tử cũ nhất
        const tailNode = this.#popTail();
        this.map.delete(tailNode.key);
      }
    }
  }
}

// Chạy thử nghiệm
const lru = new LRUCache(2);
lru.put(1, 1); // cache is {1=1}
lru.put(2, 2); // cache is {1=1, 2=2}
console.log(lru.get(1));    // returns 1, cache is {2=2, 1=1}
lru.put(3, 3); // evicts key 2, cache is {1=1, 3=3}
console.log(lru.get(2));    // returns -1 (not found)
```

---

## 🔧 Trong Framework & Thực tế

### 1. Next.js Fetch Cache (Data Cache)
```javascript
// Next.js ghi đè hàm `fetch` mặc định để hỗ trợ cache
// Nếu cache lưu trong RAM quá nhiều -> Tràn RAM
// Next.js (hoặc các thư viện caching như lru-cache) quản lý dung lượng RAM
import { LRUCache as ExternalLRU } from 'lru-cache';

const options = {
  max: 500, // Tối đa 500 API responses
  maxSize: 5000 * 1024, // Hoặc tối đa 5MB dữ liệu
  sizeCalculation: (value, key) => value.length,
  ttl: 1000 * 60 * 5, // Time to live: 5 phút
};

const apiCache = new ExternalLRU(options);
```

### 2. Vue `keep-alive` Component Cache
```javascript
// Vue.js dùng LRU Cache để lưu trữ trạng thái của các component được bọc trong <keep-alive>
// Nhờ đó, các component cũ không bị hủy hoàn toàn (destroy), khi quay lại trạng thái cũ không cần render lại.
// Cấu hình max prop của keep-alive hoạt động chính xác theo cơ chế LRU:
// <keep-alive :max="10">
//   <component :is="activeComponent"></component>
// </keep-alive>
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### LFU (Least Frequently Used) vs LRU
- **LRU** quan tâm đến **thời gian gần nhất** (Recency). Node truy cập cách đây 1 giây luôn quan trọng hơn node truy cập cách đây 1 giờ.
- **LFU** quan tâm đến **tần suất sử dụng** (Frequency). Node được đọc 100 lần trong quá khứ quan trọng hơn node vừa được tạo và chỉ đọc 1 lần.
- **Thực tế**: LRU được dùng nhiều hơn vì LFU tốn thêm bộ nhớ lưu biến đếm (counter) và thuật toán phức tạp hơn nhiều.

---

## ❓ Bẫy phỏng vấn

### Q1: Tại sao không dùng Array đơn thuần để làm LRU Cache?
```
Nếu dùng Array:
- Mỗi khi `get` hoặc `put`, ta phải tìm phần tử đó trong mảng: O(n).
- Khi đưa nó lên đầu mảng (hoặc cuối mảng), ta phải dịch chuyển toàn bộ các phần tử còn lại: O(n).
-> Hiệu năng giảm cực nghiêm trọng khi kích thước cache lớn.
-> HashMap + Doubly Linked List là giải pháp duy nhất đạt O(1) cho mọi thao tác.
```

### Q2: Quirk của JS Map: Tại sao `keys().next().value` lấy được key cũ nhất?
```
JavaScript ES6 Map lưu các entry theo thứ tự chèn (insertion order).
Khi ta chèn phần tử mới, nó nằm ở cuối Map.
Khi ta lặp qua Map, phần tử đầu tiên lấy ra chính là phần tử cũ nhất được chèn vào mà chưa bị xóa.
Phương thức `.keys()` trả về một Iterator, và `.next().value` lấy phần tử đầu tiên của Iterator đó trong O(1).
```

---

## 🔗 Xem thêm
- [HashMap](../02-hash-based/hash-map.md) — Cơ sở của LRU Map lookup
- [Linked List](../01-linear/linked-list.md) — Chi tiết về Doubly Linked List
