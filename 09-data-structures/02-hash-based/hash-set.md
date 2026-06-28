# 🔘 HashSet — Tập hợp

## 🧠 Định nghĩa dễ nhớ

> **HashSet** giống như một **danh sách khách mời VIP**: mỗi người chỉ xuất hiện **một lần**, và bảo vệ chỉ mất **một giây** để kiểm tra "người này có trong danh sách không?" — dù danh sách có 10 hay 10 triệu người.

HashSet = HashMap chỉ có key, không có value. Sức mạnh duy nhất: **membership check O(1)** + **no duplicates**.

---

## ⏱️ Big-O Complexity

| Thao tác | Average | Worst |
|----------|---------|-------|
| Add | O(1) | O(n) |
| Has / Contains | **O(1)** | O(n) |
| Delete | O(1) | O(n) |
| Set operations (union, intersect) | O(n) | O(n) |

---

## 💻 Code JavaScript

### Dùng `Set` (built-in)

```javascript
const set = new Set();

// ADD
set.add(1);
set.add(2);
set.add(2); // duplicate → bị bỏ qua
set.add('hello');
set.add({id: 1}); // ⚠️ objects so sánh by reference

// HAS
set.has(2);      // true — O(1) ✅
set.has(99);     // false

// DELETE
set.delete(2);

// SIZE
set.size; // 3 (1, 'hello', {id:1})

// ITERATE
for (const item of set) console.log(item);
[...set]; // convert to array
```

### Set Operations (Union, Intersection, Difference)

```javascript
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

// UNION — tất cả phần tử của cả hai
const union = new Set([...a, ...b]);          // {1,2,3,4,5,6}

// INTERSECTION — phần tử chung
const intersection = new Set([...a].filter(x => b.has(x))); // {3,4}

// DIFFERENCE — phần tử trong A nhưng không trong B
const difference = new Set([...a].filter(x => !b.has(x)));  // {1,2}

// SYMMETRIC DIFFERENCE — phần tử chỉ có trong 1 trong 2
const symDiff = new Set([
  ...[...a].filter(x => !b.has(x)),
  ...[...b].filter(x => !a.has(x))
]); // {1,2,5,6}

// ES2025: Set methods built-in!
a.union(b);               // {1,2,3,4,5,6}
a.intersection(b);        // {3,4}
a.difference(b);          // {1,2}
a.symmetricDifference(b); // {1,2,5,6}
a.isSubsetOf(b);          // false
a.isSupersetOf(b);        // false
```

### Deduplicate Array

```javascript
// Cách phổ biến nhất để remove duplicates
const arr = [1, 2, 2, 3, 3, 3, 4];
const unique = [...new Set(arr)]; // [1, 2, 3, 4]

// Với objects (cần custom key)
function deduplicateBy(arr, keyFn) {
  const seen = new Set();
  return arr.filter(item => {
    const key = keyFn(item);
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
}

const users = [
  {id: 1, name: 'Alice'},
  {id: 2, name: 'Bob'},
  {id: 1, name: 'Alice (duplicate)'},
];
deduplicateBy(users, u => u.id); // [{id:1, name:'Alice'}, {id:2, name:'Bob'}]
```

### Ứng dụng: Longest Consecutive Sequence O(n)

```javascript
// Tìm chuỗi số liên tiếp dài nhất: [100,4,200,1,3,2] → 4 (1,2,3,4)
function longestConsecutive(nums) {
  const set = new Set(nums);
  let longest = 0;

  for (const num of set) {
    if (!set.has(num - 1)) { // chỉ bắt đầu từ đầu chuỗi
      let current = num;
      let length = 1;
      while (set.has(current + 1)) { current++; length++; }
      longest = Math.max(longest, length);
    }
  }
  return longest;
}

longestConsecutive([100,4,200,1,3,2]); // 4
```

---

## 🔧 Trong Framework & Thực tế

### React — Dependency Tracking
```javascript
// React dùng Set để track unique dependencies
// useEffect deps array — React internally convert sang Set-like structure
// để check "có dep nào thay đổi không?"

// React DevTools dùng Set để track mounted components
const mountedComponents = new Set();
// component mount → mountedComponents.add(fiber)
// component unmount → mountedComponents.delete(fiber)
```

### Vue 3 — Effect Tracking
```javascript
// Vue 3 Reactivity: mỗi reactive property có một Set của effects
// dep = new Set() // chứa các effect function cần re-run
// Khi value thay đổi → dep.forEach(effect => effect())

// Simplified:
const depsMap = new Map();  // property → Set<Effect>

function track(target, key) {
  if (!depsMap.has(key)) depsMap.set(key, new Set());
  depsMap.get(key).add(currentEffect); // thêm effect đang chạy
}

function trigger(target, key) {
  depsMap.get(key)?.forEach(effect => effect()); // run tất cả effects
}
```

### Angular — Change Detection
```javascript
// Angular dùng Set để track dirty components
const dirtyComponents = new Set();
// Khi binding thay đổi → dirtyComponents.add(component)
// Change detection cycle → dirtyComponents.forEach(c => c.detectChanges())
// → Chỉ update component thực sự thay đổi
```

### Webpack/Vite — Module Graph
```javascript
// Webpack track visited modules bằng Set khi build dependency graph
// Tránh vòng lặp vô tận khi có circular dependencies
const visited = new Set();

function processModule(modulePath) {
  if (visited.has(modulePath)) return; // đã xử lý rồi, skip
  visited.add(modulePath);
  // process module...
}
```

### CSS-in-JS — Style Deduplication
```javascript
// styled-components, Emotion dùng Set để track inserted styles
const insertedStyles = new Set();

function injectStyle(cssRule) {
  if (insertedStyles.has(cssRule)) return; // đừng insert trùng
  document.styleSheet.insertRule(cssRule);
  insertedStyles.add(cssRule);
}
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. WeakSet — Set không ngăn GC
```javascript
// WeakSet: chỉ lưu objects, tự cleanup khi object bị GC
const weakSet = new WeakSet();
let element = document.getElementById('btn');
weakSet.add(element);

// Khi element bị remove khỏi DOM và không có reference khác
element = null; // → weakSet entry tự xóa!

// Use case: track processed items mà không cần cleanup
function processOnce(items) {
  const processed = new WeakSet();
  return (item) => {
    if (processed.has(item)) return 'already done';
    // do work...
    processed.add(item);
    return 'done';
  };
}
```

### 2. Set như "Visited" trong Graph Traversal
```javascript
// BFS/DFS LUÔN cần Set để tránh visit lại node
function bfs(graph, start) {
  const visited = new Set([start]); // O(1) check!
  const queue = [start];

  while (queue.length) {
    const node = queue.shift();
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) { // O(1) — không phải O(n) như Array.includes()
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
}

// ⚠️ Sai lầm phổ biến: dùng Array thay Set → O(n) mỗi lần check → O(n²) tổng
const visited = []; // ❌
if (!visited.includes(node)) { } // O(n) mỗi lần!

const visited = new Set(); // ✅
if (!visited.has(node)) { }  // O(1) mỗi lần!
```

### 3. Set để check anagram
```javascript
// Nhanh hơn sort: O(n) thay vì O(n log n)
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  const count = new Map();
  for (const c of s) count.set(c, (count.get(c)||0) + 1);
  for (const c of t) {
    if (!count.get(c)) return false;
    count.set(c, count.get(c) - 1);
  }
  return true;
}
```

---

## ❓ Bẫy phỏng vấn

### Q1: Set vs Array — khi nào dùng cái nào?
```
Set:   membership check O(1), no duplicates, no random access by index
Array: ordered, random access O(1), có duplicates, map/filter/reduce

→ Nếu cần check "item này có tồn tại không?" nhiều lần → Set
→ Nếu cần giữ thứ tự và truy cập theo index → Array
```

### Q2: Tìm phần tử xuất hiện lẻ số lần (XOR trick)
```javascript
// Không cần Set! Dùng XOR bit
// XOR: a^a = 0, 0^a = a
function findOdd(arr) {
  return arr.reduce((xor, n) => xor ^ n, 0);
  // Mọi số xuất hiện chẵn lần → triệt tiêu nhau
  // Số xuất hiện lẻ lần → còn lại
}
findOdd([1,2,3,2,1,3,5]); // 5
```

### Q3: Tại sao Set nhanh hơn Array.includes()?
```
Array.includes(): O(n) — phải so sánh từng phần tử
Set.has():        O(1) — hash key → nhảy trực tiếp đến bucket

Với 1 triệu phần tử và 1000 lần check:
Array: 1,000,000,000 phép so sánh
Set:           1,000 phép hash
```

---

## 🔗 Xem thêm
- [HashMap](./hash-map.md) — Set với value
- [Bloom Filter](./bloom-filter.md) — Set tiết kiệm bộ nhớ (probabilistic)
