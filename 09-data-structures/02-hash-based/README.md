# 🟢 Hash-based Data Structures

> **Hash-based** = dùng **hàm hash** để ánh xạ key sang vị trí lưu trữ, cho phép tra cứu **O(1)** trung bình — nhanh hơn bất kỳ cấu trúc so sánh nào.

---

## Tại sao Hash nhanh đến vậy?

```
Không dùng hash: tìm kiếm trong 1 triệu phần tử → duyệt từng cái → O(n)
Dùng hash:       hash("username") = 42703 → đi thẳng đến ô 42703 → O(1)

Giống như: biết chính xác ngăn trong tủ thay vì mở từng ngăn một
```

---

## Các cấu trúc trong nhóm

| DS | Mô tả | Khi nào dùng |
|----|-------|--------------|
| [HashMap](./hash-map.md) | Key → Value, O(1) lookup | Cache, index, counting |
| [HashSet](./hash-set.md) | Unique values, O(1) lookup | Dedup, membership check |
| [Bloom Filter](./bloom-filter.md) ⭐ | Probabilistic set — "maybe yes, definitely no" | CDN cache, spam filter |

---

## Nguyên lý hoạt động của Hash Function

```javascript
// Hash function: chuyển bất kỳ key nào thành số nguyên
// Số nguyên đó = index trong array nội bộ
function simpleHash(key, tableSize) {
  let hash = 0;
  for (const char of key) {
    hash = (hash * 31 + char.charCodeAt(0)) % tableSize;
  }
  return hash;
}

simpleHash('name', 100);    // 65
simpleHash('username', 100); // 17

// Collision: 2 key khác nhau có cùng hash → xử lý bằng Chaining hoặc Open Addressing
```

---

## So sánh nhanh

| | HashMap | HashSet | Bloom Filter |
|--|---------|---------|-------------|
| Lưu value | ✅ | ❌ | ❌ |
| False positive | ❌ | ❌ | ✅ có thể |
| False negative | ❌ | ❌ | ❌ không bao giờ |
| Memory | Nhiều | Ít hơn | **Rất ít** |
| Delete | ✅ | ✅ | ❌ (standard) |

---

## Trong JavaScript

```javascript
// Tất cả đều built-in!
const map = new Map();     // HashMap
const set = new Set();     // HashSet
const obj = {};            // "HashMap" — nhưng có gotchas!

// Object vs Map:
// Object: key chỉ là string/symbol, prototype pollution risk
// Map:    key là bất kỳ type nào, maintain insertion order, .size property
```
