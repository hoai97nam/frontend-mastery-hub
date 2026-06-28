# 📊 Segment Tree — Cây đoạn

## 🧠 Định nghĩa dễ nhớ

> **Segment Tree** giống như **bảng thống kê phân cấp của một giải bóng đá**: bạn có thể hỏi "trong vòng 3-7, ai ghi nhiều bàn nhất?" và nhận được câu trả lời **ngay lập tức** mà không cần xem lại từng trận. Bí quyết: **mỗi segment (đoạn) đã được tính trước kết quả** và lưu trong node tương ứng.

```
Array:  [1, 3, 5, 7, 9, 11]
         0  1  2  3  4   5

Segment Tree (sum):
              [36]          ← range [0,5]
            /      \
         [9]        [27]    ← range [0,2] và [3,5]
        /   \       /   \
      [4]  [5]  [16]  [11]  ← range [0,1],[2,2],[3,4],[5,5]
      / \         / \
    [1] [3]     [7] [9]     ← range [0,0],[1,1],[3,3],[4,4]
```

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Build | **O(n)** | Xây dựng cây |
| Range Query | **O(log n)** | sum/min/max/gcd trong range |
| Point Update | **O(log n)** | Cập nhật 1 phần tử |
| Range Update | **O(log n)** | Với Lazy Propagation |
| Space | O(4n) | Thường cấp phát 4×n |

---

## 💻 Code JavaScript

### Segment Tree — Range Sum & Point Update

```javascript
class SegmentTree {
  #tree;
  #n;

  constructor(arr) {
    this.#n = arr.length;
    this.#tree = new Array(4 * arr.length).fill(0);
    this.#build(arr, 0, 0, arr.length - 1);
  }

  // BUILD — O(n)
  #build(arr, node, start, end) {
    if (start === end) {
      this.#tree[node] = arr[start];
      return;
    }
    const mid = Math.floor((start + end) / 2);
    this.#build(arr, 2*node+1, start, mid);
    this.#build(arr, 2*node+2, mid+1, end);
    this.#tree[node] = this.#tree[2*node+1] + this.#tree[2*node+2];
  }

  // RANGE SUM QUERY — O(log n)
  query(left, right) {
    return this.#queryHelper(0, 0, this.#n-1, left, right);
  }

  #queryHelper(node, start, end, left, right) {
    if (right < start || end < left) return 0; // completely outside
    if (left <= start && end <= right) return this.#tree[node]; // completely inside

    const mid = Math.floor((start + end) / 2);
    return this.#queryHelper(2*node+1, start, mid, left, right)
         + this.#queryHelper(2*node+2, mid+1, end, left, right);
  }

  // POINT UPDATE — O(log n)
  update(index, val) {
    this.#updateHelper(0, 0, this.#n-1, index, val);
  }

  #updateHelper(node, start, end, index, val) {
    if (start === end) {
      this.#tree[node] = val;
      return;
    }
    const mid = Math.floor((start + end) / 2);
    if (index <= mid) this.#updateHelper(2*node+1, start, mid, index, val);
    else              this.#updateHelper(2*node+2, mid+1, end, index, val);
    this.#tree[node] = this.#tree[2*node+1] + this.#tree[2*node+2];
  }
}

const arr = [1, 3, 5, 7, 9, 11];
const st = new SegmentTree(arr);

st.query(1, 3);  // 3+5+7 = 15
st.query(0, 5);  // 1+3+5+7+9+11 = 36
st.update(1, 10); // arr[1] = 10
st.query(1, 3);  // 10+5+7 = 22
```

### Segment Tree — Range Min Query

```javascript
class RangeMinQuery {
  #tree;
  #n;

  constructor(arr) {
    this.#n = arr.length;
    this.#tree = new Array(4 * arr.length).fill(Infinity);
    this.#build(arr, 0, 0, arr.length - 1);
  }

  #build(arr, node, start, end) {
    if (start === end) { this.#tree[node] = arr[start]; return; }
    const mid = Math.floor((start + end) / 2);
    this.#build(arr, 2*node+1, start, mid);
    this.#build(arr, 2*node+2, mid+1, end);
    this.#tree[node] = Math.min(this.#tree[2*node+1], this.#tree[2*node+2]); // MIN!
  }

  query(left, right) {
    return this.#q(0, 0, this.#n-1, left, right);
  }

  #q(node, start, end, left, right) {
    if (right < start || end < left) return Infinity;
    if (left <= start && end <= right) return this.#tree[node];
    const mid = Math.floor((start + end) / 2);
    return Math.min(
      this.#q(2*node+1, start, mid, left, right),
      this.#q(2*node+2, mid+1, end, left, right)
    );
  }
}
```

### Lazy Propagation — Range Update O(log n)

```javascript
class LazySegmentTree {
  #tree;
  #lazy; // pending updates
  #n;

  constructor(arr) {
    this.#n = arr.length;
    this.#tree = new Array(4 * arr.length).fill(0);
    this.#lazy = new Array(4 * arr.length).fill(0);
    this.#build(arr, 0, 0, arr.length - 1);
  }

  #build(arr, node, s, e) {
    if (s === e) { this.#tree[node] = arr[s]; return; }
    const m = Math.floor((s+e)/2);
    this.#build(arr, 2*node+1, s, m);
    this.#build(arr, 2*node+2, m+1, e);
    this.#tree[node] = this.#tree[2*node+1] + this.#tree[2*node+2];
  }

  // Áp dụng lazy update xuống con
  #pushDown(node, s, e) {
    if (this.#lazy[node] !== 0) {
      const m = Math.floor((s+e)/2);
      const left = 2*node+1, right = 2*node+2;
      this.#tree[left]  += this.#lazy[node] * (m - s + 1);
      this.#tree[right] += this.#lazy[node] * (e - m);
      this.#lazy[left]  += this.#lazy[node];
      this.#lazy[right] += this.#lazy[node];
      this.#lazy[node] = 0; // reset
    }
  }

  // RANGE UPDATE: thêm val vào tất cả phần tử trong [l, r] — O(log n)
  rangeUpdate(l, r, val) {
    this.#update(0, 0, this.#n-1, l, r, val);
  }

  #update(node, s, e, l, r, val) {
    if (r < s || e < l) return;
    if (l <= s && e <= r) {
      this.#tree[node] += val * (e - s + 1);
      this.#lazy[node] += val;
      return;
    }
    this.#pushDown(node, s, e);
    const m = Math.floor((s+e)/2);
    this.#update(2*node+1, s, m, l, r, val);
    this.#update(2*node+2, m+1, e, l, r, val);
    this.#tree[node] = this.#tree[2*node+1] + this.#tree[2*node+2];
  }

  query(l, r) { return this.#q(0, 0, this.#n-1, l, r); }

  #q(node, s, e, l, r) {
    if (r < s || e < l) return 0;
    if (l <= s && e <= r) return this.#tree[node];
    this.#pushDown(node, s, e);
    const m = Math.floor((s+e)/2);
    return this.#q(2*node+1, s, m, l, r)
         + this.#q(2*node+2, m+1, e, l, r);
  }
}

const lst = new LazySegmentTree([1, 2, 3, 4, 5]);
lst.query(0, 4);       // 15
lst.rangeUpdate(1, 3, 2); // thêm 2 vào index 1,2,3
lst.query(0, 4);       // 1+4+5+6+5 = 21
lst.query(1, 3);       // 4+5+6 = 15
```

---

## 🔧 Trong Framework & Thực tế

### Video Player — Timeline Scrubbing
```javascript
// YouTube / Netflix: hiển thị thumbnail khi hover timeline
// Data: mảng thumbnails theo timestamp
// Query: "thumbnail nào gần timestamp T nhất?"

class VideoTimeline {
  #segTree; // RangeMinQuery
  #timestamps;
  #thumbnails;

  constructor(data) {
    this.#timestamps = data.map(d => d.time);
    this.#thumbnails = data.map(d => d.thumbnail);
    // Segment tree trên timestamps để binary search range
    this.#segTree = new SegmentTree(this.#timestamps);
  }

  getThumbnailAt(seekTime) {
    // Tìm index gần nhất với seekTime
    let lo = 0, hi = this.#timestamps.length - 1;
    while (lo < hi) {
      const mid = Math.floor((lo + hi) / 2);
      if (this.#timestamps[mid] < seekTime) lo = mid + 1;
      else hi = mid;
    }
    return this.#thumbnails[lo];
  }
}
```

### Data Analytics — Range Aggregation
```javascript
// Dashboard analytics: "sum views từ ngày 5 đến ngày 20 tháng này"
// Dữ liệu thay đổi mỗi giây (realtime) → cần update nhanh

class AnalyticsDashboard {
  #dailyViews;
  #segTree;

  constructor(days) {
    this.#dailyViews = new Array(days).fill(0);
    this.#segTree = new SegmentTree(this.#dailyViews);
  }

  recordView(day) {
    this.#dailyViews[day]++;
    this.#segTree.update(day, this.#dailyViews[day]); // O(log n)
  }

  getTotalViews(fromDay, toDay) {
    return this.#segTree.query(fromDay, toDay); // O(log n)
  }
}

// Không dùng Segment Tree: query phải iterate O(n)
// Với 365 ngày: mỗi query ~365 operations
// Với Segment Tree: mỗi query ~log(365) ≈ 8 operations
```

### Game Development — Collision Detection
```javascript
// 2D game: "có object nào trong rect [x1,x2] × [y1,y2] không?"
// 2D Segment Tree (Segment Tree of Segment Trees)
// hoặc dùng Interval Tree cho 1D, kết hợp cho 2D
```

### Stock Price Analysis
```javascript
// "Giá cao nhất/thấp nhất trong khoảng ngày X đến Y?"
// Với millions of updates mỗi ngày

class StockPriceTracker {
  #highTree; // Range Max
  #lowTree;  // Range Min

  constructor(days) {
    this.#highTree = new SegmentTree(new Array(days).fill(-Infinity));
    this.#lowTree  = new SegmentTree(new Array(days).fill(Infinity));
  }

  recordPrice(day, high, low) {
    this.#highTree.update(day, high);
    this.#lowTree.update(day, low);
  }

  getRangeHigh(from, to) { return this.#highTree.query(from, to); }
  getRangeLow(from, to)  { return this.#lowTree.query(from, to); }
}
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Fenwick Tree (BIT) — nhẹ hơn cho sum queries
```javascript
// Fenwick Tree (Binary Indexed Tree) = Segment Tree đơn giản hóa
// Chỉ hỗ trợ: prefix sum, point update
// Nhưng: code ngắn hơn, nhanh hơn trong practice

class FenwickTree {
  #tree;
  #n;

  constructor(n) {
    this.#n = n;
    this.#tree = new Array(n + 1).fill(0);
  }

  // UPDATE — O(log n)
  update(i, delta) {
    for (i++; i <= this.#n; i += i & (-i)) // i & (-i) = last set bit
      this.#tree[i] += delta;
  }

  // PREFIX SUM [0, i] — O(log n)
  prefixSum(i) {
    let sum = 0;
    for (i++; i > 0; i -= i & (-i))
      sum += this.#tree[i];
    return sum;
  }

  // RANGE SUM [l, r] — O(log n)
  rangeSum(l, r) {
    return this.prefixSum(r) - (l > 0 ? this.prefixSum(l-1) : 0);
  }
}

const bit = new FenwickTree(6);
[1,3,5,7,9,11].forEach((v,i) => bit.update(i, v));
bit.rangeSum(1, 3); // 3+5+7 = 15
bit.update(1, 7);   // arr[1] thay đổi từ 3 → 10 (delta = 7)
bit.rangeSum(1, 3); // 10+5+7 = 22
```

### 2. Persistent Segment Tree
```javascript
// Mỗi update tạo ra một "version" mới của cây
// Cho phép query bất kỳ version nào — O(log n) space per update
// Dùng trong: offline range queries, version control of data
// Xem: ../05-modern/persistent-data-structure.md
```

### 3. 2D Segment Tree / Merge Sort Tree
```javascript
// Segment Tree of sorted arrays
// Mỗi node lưu sorted array của range
// Cho phép: "có bao nhiêu số trong range [l,r] lớn hơn x?"
// Space: O(n log n), Query: O(log²n)
```

---

## ❓ Bẫy phỏng vấn

### Q1: Segment Tree vs Fenwick Tree — khi nào dùng cái nào?
```
Fenwick Tree (BIT):
+ Code đơn giản hơn nhiều
+ Nhanh hơn trong practice (ít overhead)
- Chỉ support prefix queries (sum, min/max với trick)
- Khó extend

Segment Tree:
+ Linh hoạt hơn: min, max, gcd, sum, lazy propagation
+ Support range update
- Code phức tạp hơn

→ Cần sum đơn giản: Fenwick Tree
→ Cần range update / min / max: Segment Tree
```

### Q2: Sparse Table — khi nào dùng thay Segment Tree?
```
Sparse Table:
- Build: O(n log n)
- Query (range min/max): O(1) — không cần update!
- Space: O(n log n)

Segment Tree:
- Build: O(n)
- Query: O(log n)
- Update: O(log n)

→ Dữ liệu STATIC (không update), cần query nhiều → Sparse Table
→ Dữ liệu DYNAMIC (có update) → Segment Tree
```

### Q3: Sliding Window Maximum — Segment Tree hay Deque?
```
Deque:      O(n) — tốt nhất
Segment Tree: O(n log n) — không cần thiết

→ Sliding Window Maximum → Deque (xem deque.md)
→ Segment Tree khi: window size thay đổi, queries không tuần tự
```

---

## 🔗 Xem thêm
- [Deque](../01-linear/deque.md) — Sliding window maximum O(n)
- [Heap](./heap.md) — Priority queries
- [Persistent Data Structure](../05-modern/persistent-data-structure.md) — Persistent Segment Tree
