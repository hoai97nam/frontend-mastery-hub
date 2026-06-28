# 🗺️ HashMap — Bảng băm

## 🧠 Định nghĩa dễ nhớ

> **HashMap** giống như một **cuốn sổ điện thoại thần kỳ**: thay vì lật từng trang để tìm tên, bạn chỉ cần nói "tên ai?" và **ngay lập tức** biết số điện thoại. Đổi lấy tốc độ đó, bạn cần tốn thêm một chút bộ nhớ để lưu cái "sổ thần kỳ" đó.

```
"Alice" → hash(42) → [slot 42]: "0123456789"
"Bob"   → hash(17) → [slot 17]: "0987654321"
"Carol" → hash(42) → [slot 42]: ["Alice:...", "Carol:..."]  ← Collision!
```

---

## ⏱️ Big-O Complexity

| Thao tác | Average | Worst | Ghi chú |
|----------|---------|-------|---------|
| Get | **O(1)** | O(n) | Worst khi hash collision nhiều |
| Set | **O(1)** | O(n) | — |
| Delete | **O(1)** | O(n) | — |
| Has | **O(1)** | O(n) | — |
| Iterate | O(n) | O(n) | — |

> 💡 **Space**: O(n) — thường có load factor ~0.75 để cân bằng speed/memory

---

## 💻 Code JavaScript

### Dùng `Map` (chuẩn — luôn ưu tiên)

```javascript
// Map là HashMap thực thụ trong JS
const map = new Map();

// SET
map.set('name', 'Alice');
map.set(42, 'answer');       // key có thể là bất kỳ type nào!
map.set({id: 1}, 'object');  // ⚠️ object key = by reference

// GET
map.get('name');   // 'Alice'
map.get(99);       // undefined

// HAS
map.has('name');   // true

// DELETE
map.delete('name');

// SIZE
map.size;          // số phần tử thực — không cần đếm

// ITERATE — theo thứ tự chèn vào
for (const [key, value] of map) {
  console.log(key, '→', value);
}

map.keys();        // Iterator của keys
map.values();      // Iterator của values
map.entries();     // Iterator của [key, value] pairs
```

### Dùng Object (khi key là string — common pattern)

```javascript
const cache = {};

// Kiểm tra key tồn tại — CÁCH ĐÚNG
if (Object.prototype.hasOwnProperty.call(cache, 'key')) { }
// Hoặc ngắn gọn hơn (ES2022):
if (Object.hasOwn(cache, 'key')) { }

// ⚠️ NGUY HIỂM — prototype pollution
const obj = {};
obj['__proto__'] = { isAdmin: true }; // thay đổi prototype của tất cả object!
// Giải pháp: dùng Map hoặc Object.create(null)
const safeObj = Object.create(null); // không có prototype
```

### Implement HashMap từ đầu (phỏng vấn)

```javascript
class HashMap {
  #size;
  #buckets;
  #count = 0;
  static LOAD_FACTOR = 0.75;

  constructor(size = 16) {
    this.#size = size;
    this.#buckets = new Array(size).fill(null).map(() => []);
  }

  #hash(key) {
    let hash = 0;
    const str = String(key);
    for (let i = 0; i < str.length; i++) {
      hash = (hash * 31 + str.charCodeAt(i)) % this.#size;
    }
    return hash;
  }

  set(key, value) {
    const index = this.#hash(key);
    const bucket = this.#buckets[index];
    const existing = bucket.find(([k]) => k === key);

    if (existing) {
      existing[1] = value; // update
    } else {
      bucket.push([key, value]); // thêm mới
      this.#count++;
    }

    // Resize nếu load factor vượt ngưỡng
    if (this.#count / this.#size > HashMap.LOAD_FACTOR) this.#resize();
  }

  get(key) {
    const bucket = this.#buckets[this.#hash(key)];
    return bucket.find(([k]) => k === key)?.[1];
  }

  has(key) { return this.get(key) !== undefined; }

  delete(key) {
    const index = this.#hash(key);
    const bucket = this.#buckets[index];
    const idx = bucket.findIndex(([k]) => k === key);
    if (idx !== -1) { bucket.splice(idx, 1); this.#count--; return true; }
    return false;
  }

  #resize() {
    const old = this.#buckets;
    this.#size *= 2;
    this.#buckets = new Array(this.#size).fill(null).map(() => []);
    this.#count = 0;
    for (const bucket of old)
      for (const [k, v] of bucket) this.set(k, v);
  }
}
```

### Các pattern thực dụng

```javascript
// ===== COUNTING / FREQUENCY =====
function frequency(arr) {
  return arr.reduce((map, item) => {
    map.set(item, (map.get(item) || 0) + 1);
    return map;
  }, new Map());
}
frequency(['a','b','a','c','b','b']); // Map{ a:2, b:3, c:1 }

// ===== GROUPING =====
function groupBy(arr, fn) {
  return arr.reduce((map, item) => {
    const key = fn(item);
    if (!map.has(key)) map.set(key, []);
    map.get(key).push(item);
    return map;
  }, new Map());
}

const people = [{name:'Alice',dept:'Eng'},{name:'Bob',dept:'HR'},{name:'Carol',dept:'Eng'}];
groupBy(people, p => p.dept);
// Map{ 'Eng': [Alice, Carol], 'HR': [Bob] }

// ===== MEMOIZATION =====
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const memoFib = memoize(function fib(n) {
  if (n <= 1) return n;
  return memoFib(n-1) + memoFib(n-2);
});
```

---

## 🔧 Trong Framework & Thực tế

### React — Fiber Map & Hook State
```javascript
// React dùng Map để track fiber nodes
// hookState là linked list, nhưng nhiều lookups dùng Map internally

// useMemo — bản chất là HashMap
const memoized = useMemo(() => expensiveCompute(a, b), [a, b]);
// [a, b] → hash → kiểm tra cache → trả về kết quả cũ nếu deps không đổi

// React Context dùng Map để store consumer updates
const contextMap = new Map(); // context → subscribers
```

### Vue 3 — Reactivity System
```javascript
// Vue 3 dùng WeakMap để track reactive objects
const targetMap = new WeakMap(); // target object → Map(key → Set(effects))

// Khi bạn reactive({ name: 'Alice' })
// targetMap.get(obj)        → dep map
// depMap.get('name')        → set of effects that depend on 'name'
// Khi 'name' thay đổi → lấy Set của effects và run tất cả
```

### Next.js — Router Cache
```javascript
// Next.js App Router lưu RSC payloads trong Map
// Map<url, cachedPayload>
// Mỗi navigation → check cache trước → nếu hit → không fetch lại

// Simplified:
const routerCache = new Map();
function navigate(url) {
  if (routerCache.has(url)) return routerCache.get(url);
  const data = fetchRSC(url);
  routerCache.set(url, data);
  return data;
}
```

### Node.js — Module Cache
```javascript
// require() cache modules trong Map (require.cache)
// Lần đầu: load module, execute, store result
// Lần sau: return cached result → O(1)

console.log(require.cache); // object-like Map của tất cả loaded modules

// Khi bạn mock modules trong Jest:
jest.mock('./myModule'); // Jest replace module trong cache
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. WeakMap — HashMap không ngăn garbage collection
```javascript
// WeakMap: key phải là object, không ngăn GC khi object bị bỏ
const weakMap = new WeakMap();
let obj = { name: 'Alice' };
weakMap.set(obj, 'metadata');

obj = null; // obj bị GC → entry trong WeakMap tự động bị xóa!
// ✅ Không memory leak
// ✅ Không cần manual cleanup

// Use cases:
// 1. Private data của class
const privateData = new WeakMap();
class Person {
  constructor(name) { privateData.set(this, { name }); }
  getName() { return privateData.get(this).name; }
}

// 2. Cache per-object metadata mà không cần cleanup
const cache = new WeakMap();
function getMetadata(element) {
  if (!cache.has(element)) cache.set(element, computeMetadata(element));
  return cache.get(element);
}
```

### 2. Object.create(null) — "Pure" Map
```javascript
// Object bình thường có prototype → có các key ẩn như 'toString', 'hasOwnProperty'
const normal = {};
'toString' in normal; // true ← không mong muốn!

const pure = Object.create(null); // không có prototype
'toString' in pure;   // false ✅
// Dùng khi cần Map từ string → value mà không muốn bị prototype pollution
```

### 3. Map với Object key — Reference vs Value
```javascript
// Map dùng "SameValueZero" để compare key
// Object key → compare by reference!
const map = new Map();
map.set({id: 1}, 'Alice');
map.get({id: 1}); // undefined! ← khác reference dù cùng nội dung

// Giải pháp: dùng primitive key
map.set(JSON.stringify({id: 1}), 'Alice');
map.get(JSON.stringify({id: 1})); // 'Alice' ✅
```

### 4. Map maintain insertion order — O(n) ordered iteration
```javascript
// Khác với Object (không đảm bảo thứ tự với số key)
// Map luôn iterate theo thứ tự chèn vào
const map = new Map();
map.set('c', 3);
map.set('a', 1);
map.set('b', 2);

[...map.keys()]; // ['c', 'a', 'b'] — thứ tự chèn vào
// Hữu ích cho: ordered cache, LRU implementation, step-by-step processing
```

---

## ❓ Bẫy phỏng vấn

### Q1: Object vs Map — khi nào dùng cái nào?
```
Object:  key là string/symbol đã biết trước, JSON serializable, prototype methods cần
Map:     key là bất kỳ type nào, cần .size, cần iterate theo thứ tự, nhiều add/delete

Performance: Map thường nhanh hơn Object với add/delete nhiều (V8 optimization)
```

### Q2: Implement Two Sum O(n)
```javascript
function twoSum(nums, target) {
  const seen = new Map(); // value → index
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (seen.has(complement)) return [seen.get(complement), i];
    seen.set(nums[i], i);
  }
  return null;
}
twoSum([2,7,11,15], 9); // [0, 1]
```

### Q3: HashMap có thể bị worst case O(n) không? Khi nào?
```
Có! Khi hash collision nhiều → tất cả key vào cùng một bucket → duyệt linked list O(n)
Hash DoS attack: gửi nhiều key có cùng hash → server slow down
Giải pháp: modern hashmap dùng random seed để tránh predictable collision
```

---

## 🔗 Xem thêm
- [HashSet](./hash-set.md) — HashMap không có value
- [LRU Cache](../05-modern/lru-cache.md) — HashMap + Doubly Linked List
- [Bloom Filter](./bloom-filter.md) — Probabilistic HashMap
