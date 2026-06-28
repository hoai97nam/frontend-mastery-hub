# 🌸 Bloom Filter — Bộ lọc xác suất

## 🧠 Định nghĩa dễ nhớ

> **Bloom Filter** giống như một **nhân viên bảo vệ cực nhanh** nhưng đôi khi nhầm: nếu anh ta nói "**Người này KHÔNG có trong danh sách**" → **chắc chắn 100%** là đúng. Nhưng nếu anh ta nói "Người này **có thể** trong danh sách" → cần kiểm tra thêm vì **có thể anh ta nhầm**.

```
"Definitely NOT in set" → sai thì không bao giờ  (no false negative)
"Probably in set"       → đôi khi nhầm           (false positive có thể xảy ra)
```

**Đánh đổi**: Tốn **ít bộ nhớ hơn HashSet 100x-1000x** nhưng không chắc 100%.

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Add | O(k) | k = số hash functions |
| Lookup | O(k) | Constant — không phụ thuộc n |
| Delete | ❌ Không hỗ trợ | Standard Bloom Filter |
| Space | **O(m)** | m = số bits — không tỷ lệ với n |

> k thường là 3-7 → thực tế O(1) | **Space tiết kiệm cực kỳ**: 10 triệu URLs chỉ cần ~10MB

---

## 💻 Code JavaScript

### Bloom Filter từ đầu

```javascript
class BloomFilter {
  #bitArray;
  #size;
  #hashFunctions;

  constructor(size = 1000, numHashFunctions = 3) {
    this.#size = size;
    this.#bitArray = new Uint8Array(size); // 0 hoặc 1
    this.#hashFunctions = this.#generateHashFunctions(numHashFunctions);
  }

  // Tạo k hash functions khác nhau dùng different seeds
  #generateHashFunctions(k) {
    return Array.from({length: k}, (_, seed) =>
      (str) => {
        let hash = seed * 2654435761; // Golden ratio prime seed
        for (let i = 0; i < str.length; i++) {
          hash = ((hash << 5) + hash) + str.charCodeAt(i);
          hash = hash & hash; // Convert to 32bit integer
        }
        return Math.abs(hash) % this.#size;
      }
    );
  }

  // ADD: set bit tại mọi hash position
  add(item) {
    const str = String(item);
    for (const hashFn of this.#hashFunctions) {
      this.#bitArray[hashFn(str)] = 1;
    }
  }

  // LOOKUP: kiểm tra TẤT CẢ bit positions
  mightContain(item) {
    const str = String(item);
    return this.#hashFunctions.every(hashFn => this.#bitArray[hashFn(str)] === 1);
    // Nếu BẤT KỲ bit nào = 0 → chắc chắn KHÔNG có
    // Nếu TẤT CẢ bit = 1 → CÓ THỂ có (false positive)
  }

  // Ước tính false positive rate
  get falsePositiveRate() {
    const setBits = this.#bitArray.reduce((sum, bit) => sum + bit, 0);
    return Math.pow(setBits / this.#size, this.#hashFunctions.length);
  }
}

// Demo
const filter = new BloomFilter(1000, 3);

// Thêm các URL đã thăm
filter.add('https://google.com');
filter.add('https://github.com');
filter.add('https://example.com');

console.log(filter.mightContain('https://google.com')); // true (đã thêm)
console.log(filter.mightContain('https://evil.com'));   // false (chắc chắn chưa thêm)
console.log(filter.mightContain('https://test.com'));   // false hoặc true (nếu false positive)

console.log(`False positive rate: ${(filter.falsePositiveRate * 100).toFixed(4)}%`);
```

### Counting Bloom Filter (hỗ trợ Delete)

```javascript
// Thay mỗi bit bằng counter — có thể delete
class CountingBloomFilter {
  #counters;
  #size;
  #hashFns;

  constructor(size = 1000, k = 3) {
    this.#size = size;
    this.#counters = new Uint16Array(size); // đếm số lần set
    this.#hashFns = this.#buildHashFns(k);
  }

  #buildHashFns(k) {
    return Array.from({length: k}, (_, seed) =>
      (str) => {
        let h = seed * 0x9e3779b9;
        for (let i = 0; i < str.length; i++) h = h * 31 + str.charCodeAt(i);
        return Math.abs(h) % this.#size;
      }
    );
  }

  add(item) {
    for (const fn of this.#hashFns)
      this.#counters[fn(String(item))]++;
  }

  // DELETE được support!
  remove(item) {
    if (!this.mightContain(item)) return; // chắc chắn không có
    for (const fn of this.#hashFns)
      this.#counters[fn(String(item))]--;
  }

  mightContain(item) {
    return this.#hashFns.every(fn => this.#counters[fn(String(item))] > 0);
  }
}
```

---

## 🔧 Trong Framework & Thực tế

### Chrome — Safe Browsing
```javascript
// Chrome dùng Bloom Filter để check URL có trong danh sách độc hại không
// Trước khi query server (chậm), check filter local (nhanh)
//
// Flow:
// 1. Browser nhận danh sách ~2 triệu URL nguy hiểm từ Google
// 2. Build Bloom Filter local (~2MB thay vì ~100MB full list)
// 3. Khi user navigate: check filter O(1)
//    - Nếu "definitely not" → cho phép ngay
//    - Nếu "maybe" → query server để verify
// → 99%+ request được trả lời locally, cực kỳ nhanh
```

### Redis — HyperLogLog & Bloom Module
```javascript
// Redis có sẵn Bloom Filter module
// Dùng trong: cache stampede prevention, dedup events

// Redis commands:
// BF.ADD myfilter "item1"
// BF.EXISTS myfilter "item1"  → 1 (probably exists)
// BF.EXISTS myfilter "item99" → 0 (definitely not exists)

// Node.js client:
import { createClient } from 'redis';
const redis = createClient();
await redis.bf.add('visited_urls', 'https://example.com');
await redis.bf.exists('visited_urls', 'https://example.com'); // true
await redis.bf.exists('visited_urls', 'https://other.com');  // false (or rare false positive)
```

### Next.js / CDN — Cache Negative
```javascript
// CDN dùng Bloom Filter để tránh "cache negative" stampede
// Vấn đề: cache miss → tất cả requests đổ vào origin server cùng lúc
//
// Giải pháp với Bloom Filter:
const urlFilter = new BloomFilter(100000, 7);

async function getCachedResponse(url) {
  // Bước 1: Check Bloom Filter — O(1), không cần query cache
  if (!urlFilter.mightContain(url)) {
    // Chắc chắn cache miss → fetch và cache
    const data = await fetchFromOrigin(url);
    await cache.set(url, data);
    urlFilter.add(url);
    return data;
  }

  // Bước 2: Check actual cache
  const cached = await cache.get(url);
  if (cached) return cached;

  // False positive → fetch anyway
  return fetchFromOrigin(url);
}
```

### Email Spam Filter
```javascript
// Spam filter dùng Bloom Filter để check domain/IP blacklist
// Ví dụ: SpamAssassin, Postfix dùng Bloom-based structures

const spamDomains = new BloomFilter(500000, 5);
// Load blacklist
for (const domain of blacklistedDomains) spamDomains.add(domain);

function isSpam(email) {
  const domain = email.split('@')[1];
  return spamDomains.mightContain(domain);
}
```

### Distributed Systems — Cassandra, Bigtable
```javascript
// Apache Cassandra dùng Bloom Filter per SSTable
// Khi query: check Bloom Filter trước → tránh disk I/O không cần thiết
//
// Vấn đề: Cassandra có thể có hàng nghìn SSTable files
// Không dùng Bloom Filter: mỗi query = đọc nhiều file → slow
// Với Bloom Filter: skip files chắc chắn không có data → fast
//
// Cassandra config:
// bloom_filter_fp_chance: 0.1  → 10% false positive rate
// bloom_filter_fp_chance: 0.01 → 1% false positive rate (tốn bộ nhớ hơn)
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Optimal Parameters Calculator
```javascript
// Công thức toán học để tính số bit và hash functions tối ưu
function bloomFilterOptimalParams(n, fpRate) {
  // n = expected number of elements
  // fpRate = desired false positive rate (e.g., 0.01 for 1%)

  // Số bits tối ưu:
  const m = Math.ceil(-n * Math.log(fpRate) / (Math.log(2) ** 2));

  // Số hash functions tối ưu:
  const k = Math.round((m / n) * Math.log(2));

  const memoryKB = m / 8 / 1024;

  return {
    bits: m,
    hashFunctions: k,
    memoryKB: memoryKB.toFixed(2),
  };
}

// 1 triệu URLs, 1% false positive rate:
bloomFilterOptimalParams(1_000_000, 0.01);
// { bits: 9585059, hashFunctions: 7, memoryKB: '1170.04' } → chỉ 1.1MB!
// HashSet tương đương: mỗi URL ~50 bytes → 50MB
```

### 2. Scalable Bloom Filter
```javascript
// Vấn đề: Bloom Filter có kích thước cố định — đầy thì false positive tăng
// Giải pháp: Scalable Bloom Filter = chuỗi các Bloom Filter
// Khi filter đầy → tạo filter mới (lớn hơn)

class ScalableBloomFilter {
  #filters = [];
  #capacity;
  #fpRate;

  constructor(initialCapacity = 1000, fpRate = 0.01) {
    this.#capacity = initialCapacity;
    this.#fpRate = fpRate;
    this.#addFilter();
  }

  #addFilter() {
    // Mỗi filter mới có capacity gấp đôi, fpRate chặt hơn
    const f = new BloomFilter(this.#capacity * this.#filters.length + this.#capacity, 5);
    this.#filters.push({ filter: f, count: 0 });
  }

  add(item) {
    const current = this.#filters.at(-1);
    if (current.count >= this.#capacity) this.#addFilter(); // tạo filter mới khi đầy
    current.filter.add(item);
    current.count++;
  }

  mightContain(item) {
    return this.#filters.some(({ filter }) => filter.mightContain(item));
  }
}
```

### 3. Bloom Filter vs Hash Table — trade-off bộ nhớ

```
1 triệu items, 1% false positive:
Bloom Filter: ~1.2MB  (9.6 bits/item)
HashSet:      ~50MB   (400 bits/item, với string overhead)
→ Tiết kiệm ~40x bộ nhớ!

Nhưng:
- Không thể iterate qua items
- Không thể delete (standard)
- Không thể lấy item ra
→ Chỉ dùng khi: "item này có tồn tại không?" là câu hỏi DUY NHẤT
```

---

## ❓ Bẫy phỏng vấn

### Q1: Bloom Filter có thể có false negative không?
```
KHÔNG BAO GIỜ!
- Khi add item → set các bit tương ứng thành 1
- Khi check → nếu bất kỳ bit nào = 0 → item chắc chắn chưa được add
→ "Definitely not present" luôn đúng 100%
→ "Probably present" có thể sai (false positive)
```

### Q2: Tại sao không dùng HashSet thay thế Bloom Filter?
```
HashSet cần lưu actual value (hoặc hash) của từng item
→ Memory O(n) theo số items

Bloom Filter chỉ cần bit array cố định
→ Memory O(m) không phụ thuộc n
→ 10 triệu items, 1% FP rate → chỉ ~12MB
→ 10 triệu items trong HashSet → ~400MB+

→ Bloom Filter dùng khi: memory là constraint, false positive chấp nhận được
```

### Q3: Thiết kế URL shortener dùng Bloom Filter
```
Vấn đề: "short code XXX đã tồn tại chưa?" → cần check nhanh

Không dùng Bloom Filter: query DB mỗi lần → slow
Dùng Bloom Filter:
1. Khi tạo short code mới → check Bloom Filter
   - "Definitely not" → tạo code đó ngay
   - "Probably yes"   → generate code khác hoặc query DB để verify
2. Khi short code được create → add vào Bloom Filter
3. Memory: 1 triệu links, 0.1% FP → ~1.4MB → fit trong RAM!
```

---

## 🔗 Xem thêm
- [HashSet](./hash-set.md) — Bloom Filter chính xác 100% nhưng tốn memory hơn
- [Probabilistic Structures](../05-modern/probabilistic.md) — HyperLogLog, Count-Min Sketch
