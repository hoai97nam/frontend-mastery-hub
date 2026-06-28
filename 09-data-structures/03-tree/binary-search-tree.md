# 🔍 Binary Search Tree (BST)

## 🧠 Định nghĩa dễ nhớ

> **BST** giống như **tra từ điển**: tất cả từ bắt đầu bằng A-M nằm ở nửa trái, A-G nằm ở phần tư đầu... Mỗi bước tìm kiếm bạn loại bỏ **một nửa** không gian còn lại — đó là lý do O(log n) nhanh đến vậy.

**Quy tắc vàng:**
- **Tất cả node ở cây trái** có giá trị **nhỏ hơn** node hiện tại
- **Tất cả node ở cây phải** có giá trị **lớn hơn** node hiện tại

```
        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4   7 13
```

---

## ⏱️ Big-O Complexity

| Thao tác | Average | Worst | Ghi chú |
|----------|---------|-------|---------|
| Search | **O(log n)** | O(n) | Worst khi skewed (như linked list) |
| Insert | **O(log n)** | O(n) | — |
| Delete | **O(log n)** | O(n) | — |
| Min/Max | O(log n) | O(n) | Leftmost / Rightmost |
| Inorder traversal | O(n) | O(n) | Cho kết quả sorted |

> 💡 **Tại sao worst O(n)?** Nếu insert theo thứ tự [1,2,3,4,5] → cây thành linked list thẳng

---

## 💻 Code JavaScript

### BST Implementation

```javascript
class BSTNode {
  constructor(val) {
    this.val = val;
    this.left = null;
    this.right = null;
  }
}

class BST {
  constructor() { this.root = null; }

  // INSERT — O(log n) average
  insert(val) {
    const node = new BSTNode(val);
    if (!this.root) { this.root = node; return; }

    let current = this.root;
    while (true) {
      if (val < current.val) {
        if (!current.left) { current.left = node; return; }
        current = current.left;
      } else if (val > current.val) {
        if (!current.right) { current.right = node; return; }
        current = current.right;
      } else return; // duplicate, skip
    }
  }

  // SEARCH — O(log n) average
  search(val) {
    let current = this.root;
    while (current) {
      if (val === current.val) return current;
      current = val < current.val ? current.left : current.right;
    }
    return null;
  }

  // DELETE — O(log n) average
  delete(val) {
    this.root = this.#deleteNode(this.root, val);
  }

  #deleteNode(node, val) {
    if (!node) return null;

    if (val < node.val) {
      node.left = this.#deleteNode(node.left, val);
    } else if (val > node.val) {
      node.right = this.#deleteNode(node.right, val);
    } else {
      // Node tìm thấy — 3 cases:
      if (!node.left) return node.right;  // case 1: không có con trái
      if (!node.right) return node.left;  // case 2: không có con phải

      // case 3: có cả 2 con → thay bằng inorder successor (min của right subtree)
      let successor = node.right;
      while (successor.left) successor = successor.left;
      node.val = successor.val;
      node.right = this.#deleteNode(node.right, successor.val);
    }
    return node;
  }

  // INORDER — trả về sorted array
  inorder() {
    const result = [];
    const traverse = (node) => {
      if (!node) return;
      traverse(node.left);
      result.push(node.val);
      traverse(node.right);
    };
    traverse(this.root);
    return result;
  }

  // MIN / MAX
  min() {
    let node = this.root;
    while (node?.left) node = node.left;
    return node?.val;
  }

  max() {
    let node = this.root;
    while (node?.right) node = node.right;
    return node?.val;
  }

  // FLOOR — giá trị lớn nhất ≤ val
  floor(val) {
    let floor = null;
    let node = this.root;
    while (node) {
      if (node.val === val) return val;
      if (node.val < val) { floor = node.val; node = node.right; }
      else node = node.left;
    }
    return floor;
  }

  // CEILING — giá trị nhỏ nhất ≥ val
  ceiling(val) {
    let ceil = null;
    let node = this.root;
    while (node) {
      if (node.val === val) return val;
      if (node.val > val) { ceil = node.val; node = node.left; }
      else node = node.right;
    }
    return ceil;
  }
}

const bst = new BST();
[8, 3, 10, 1, 6, 14, 4, 7, 13].forEach(v => bst.insert(v));
bst.inorder();  // [1, 3, 4, 6, 7, 8, 10, 13, 14]
bst.search(6);  // BSTNode { val: 6, ... }
bst.min();      // 1
bst.max();      // 14
bst.floor(5);   // 4
bst.ceiling(5); // 6
```

### Validate BST

```javascript
// Câu hỏi phỏng vấn: "Cây này có phải BST hợp lệ không?"
// ⚠️ KHÔNG kiểm tra left < root < right cho từng node riêng lẻ!
// VD: này không phải BST nhưng mỗi node đều thỏa local:
//    5
//   / \
//  1   4
//     / \
//    3   6

function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val)
      && isValidBST(root.right, root.val, max);
}
// Truyền range [min, max] cho mỗi node — đây là key insight!
```

### Kth Smallest Element

```javascript
// O(k) với iterative inorder
function kthSmallest(root, k) {
  const stack = [];
  let node = root;

  while (node || stack.length) {
    while (node) { stack.push(node); node = node.left; }
    node = stack.pop();
    if (--k === 0) return node.val;
    node = node.right;
  }
}
```

---

## 🔧 Trong Framework & Thực tế

### V8 Engine — TreeMap / OrderedHashMap
```javascript
// V8 (Node.js / Chrome JS engine) dùng BST variants để implement:
// - Object property access order (cho integer keys)
// - Map iteration order (trước ES6, sau đó là insertion order)
// - Set ordering

// Khi bạn có object với numeric keys:
const obj = {};
obj[3] = 'three';
obj[1] = 'one';
obj[2] = 'two';
Object.keys(obj); // ['1', '2', '3'] — sorted! V8 dùng BST internally
```

### JavaScript Engine — Scope Chain
```javascript
// Scope lookups hoạt động như BST search qua scope chain
// Không phải BST chính xác, nhưng hierarchical search tương tự

function outer() {
  const x = 1;        // scope level 1
  function inner() {
    const y = 2;       // scope level 2
    console.log(x);    // tìm x: inner scope → outer scope → global
    // = traversal từ leaf lên root của scope tree
  }
}
```

### Database Indexing — B-Tree
```javascript
// SQL databases dùng B-Tree (generalized BST) cho indexes
// MySQL InnoDB, PostgreSQL → B+ Tree cho clustered indexes

// Khi bạn tạo index:
// CREATE INDEX idx_name ON users(email);
// → Database build BST-like structure
// → SELECT * WHERE email = 'x' → O(log n) thay vì O(n)

// Mongoose (MongoDB ODM):
const userSchema = new Schema({
  email: { type: String, index: true } // tạo index → B-tree internally
});
```

### React — Component Tree Search
```javascript
// React DevTools tìm component bằng DFS trên fiber tree
// Fiber tree = BST-like với left/right pointers

// Context API propagates như tree traversal
const ThemeContext = React.createContext('light');
// Khi Provider value thay đổi → React duyệt cây con → update consumers
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. BST Iterator — O(h) space thay vì O(n)
```javascript
// Khi cần iterate BST mà không phải load toàn bộ vào array
class BSTIterator {
  #stack = [];

  constructor(root) {
    this.#pushLeft(root);
  }

  #pushLeft(node) {
    while (node) {
      this.#stack.push(node);
      node = node.left;
    }
  }

  hasNext() { return this.#stack.length > 0; }

  next() {
    const node = this.#stack.pop();
    this.#pushLeft(node.right); // push left spine của right subtree
    return node.val;
  }
}
// Space: O(h) — chỉ lưu đường từ root đến leftmost node!
// Time: O(1) amortized mỗi lần gọi next()
```

### 2. BST để build Sorted Map hiệu quả
```javascript
// BST cho phép: floor, ceiling, range queries, rank
// Những thứ HashMap KHÔNG làm được

class SortedMap extends BST {
  // Đếm số phần tử trong range [lo, hi]
  countRange(lo, hi) {
    let count = 0;
    const traverse = (node) => {
      if (!node) return;
      if (node.val > lo) traverse(node.left);   // có thể có phần tử trong left
      if (node.val >= lo && node.val <= hi) count++;
      if (node.val < hi) traverse(node.right);  // có thể có phần tử trong right
    };
    traverse(this.root);
    return count;
  }
}
```

### 3. Convert Sorted Array → Balanced BST
```javascript
// Tạo balanced BST từ sorted array — O(n)
// Luôn chọn middle element làm root
function sortedArrayToBST(nums) {
  if (!nums.length) return null;
  const mid = Math.floor(nums.length / 2);
  const node = new BSTNode(nums[mid]);
  node.left  = sortedArrayToBST(nums.slice(0, mid));
  node.right = sortedArrayToBST(nums.slice(mid + 1));
  return node;
}
// [1,2,3,4,5,6,7] → balanced BST với height 3
```

---

## ❓ Bẫy phỏng vấn

### Q1: Tại sao BST thực tế ít dùng mà hay dùng Red-Black Tree?
```
BST: O(log n) trung bình, O(n) worst (khi insert sorted data)
Red-Black Tree: O(log n) GUARANTEED — self-balancing
→ Production code cần guarantee, không chấp nhận worst case O(n)
→ Java TreeMap/TreeSet dùng Red-Black Tree
→ C++ std::map/std::set dùng Red-Black Tree
```

### Q2: BST vs Binary Search trên Array?
```
Array Binary Search: O(log n) search, O(1) space, nhưng O(n) insert/delete
BST:                 O(log n) search + insert + delete, O(n) space

→ Dữ liệu static (không thay đổi): Array + Binary Search
→ Dữ liệu dynamic (insert/delete nhiều): BST / Red-Black Tree
```

### Q3: Inorder traversal BST cho kết quả gì?
```
Luôn cho kết quả theo thứ tự TĂNG DẦN
→ Ứng dụng: "Kth smallest", "Sort" (đọc BST inorder)
→ Ứng dụng: Validate BST (inorder result phải strictly increasing)
```

---

## 🔗 Xem thêm
- [Red-Black Tree](./red-black-tree.md) — Self-balancing BST
- [Segment Tree](./segment-tree.md) — Range queries
- [HashMap](../02-hash-based/hash-map.md) — O(1) lookup nhưng không có ordering
