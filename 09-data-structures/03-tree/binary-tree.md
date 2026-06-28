# 🌲 Binary Tree — Cây nhị phân

## 🧠 Định nghĩa dễ nhớ

> **Binary Tree** giống như một **sơ đồ gia đình** trong đó mỗi người có **tối đa 2 người con** (con trái và con phải). Không có quy tắc về thứ tự — chỉ là cấu trúc cha-con đơn thuần.

```
        10          ← root (gốc)
       /  \
      5    15       ← internal nodes
     / \     \
    3   7     20   ← leaf nodes (lá)
```

**Các dạng:**
- **Full**: mỗi node có 0 hoặc 2 con
- **Complete**: tất cả levels đầy, level cuối fill từ trái
- **Perfect**: tất cả levels đầy hoàn toàn
- **Balanced**: chiều cao trái và phải chênh nhau ≤ 1

---

## ⏱️ Big-O Complexity

| Thao tác | Balanced | Unbalanced | Ghi chú |
|----------|---------|------------|---------|
| Access | O(log n) | O(n) | — |
| Search | O(log n)* | O(n) | *Nếu có BST property |
| Insert | O(log n) | O(n) | — |
| Delete | O(log n) | O(n) | — |
| Space | O(n) | O(n) | — |

---

## 💻 Code JavaScript

### Node và Traversals

```javascript
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

// Tạo cây mẫu:
//        1
//       / \
//      2   3
//     / \
//    4   5
const root = new TreeNode(1,
  new TreeNode(2,
    new TreeNode(4),
    new TreeNode(5)
  ),
  new TreeNode(3)
);
```

### 4 Traversal Patterns

```javascript
// ===== INORDER (Left → Root → Right) =====
// BST: cho kết quả sorted!
function inorder(node, result = []) {
  if (!node) return result;
  inorder(node.left, result);
  result.push(node.val);
  inorder(node.right, result);
  return result;
}
inorder(root); // [4, 2, 5, 1, 3]

// ===== PREORDER (Root → Left → Right) =====
// Dùng để: copy tree, serialize tree, prefix expression
function preorder(node, result = []) {
  if (!node) return result;
  result.push(node.val);
  preorder(node.left, result);
  preorder(node.right, result);
  return result;
}
preorder(root); // [1, 2, 4, 5, 3]

// ===== POSTORDER (Left → Right → Root) =====
// Dùng để: delete tree, evaluate expression tree, build bottom-up
function postorder(node, result = []) {
  if (!node) return result;
  postorder(node.left, result);
  postorder(node.right, result);
  result.push(node.val);
  return result;
}
postorder(root); // [4, 5, 2, 3, 1]

// ===== LEVEL ORDER / BFS (dùng Queue) =====
// Dùng để: level-by-level processing, find minimum depth
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];

  while (queue.length) {
    const levelSize = queue.length;
    const level = [];

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}
levelOrder(root); // [[1], [2,3], [4,5]]
```

### Các bài toán kinh điển

```javascript
// ===== MAX DEPTH =====
function maxDepth(node) {
  if (!node) return 0;
  return 1 + Math.max(maxDepth(node.left), maxDepth(node.right));
}

// ===== INVERT TREE (LeetCode #226) =====
function invertTree(node) {
  if (!node) return null;
  [node.left, node.right] = [invertTree(node.right), invertTree(node.left)];
  return node;
}

// ===== IS SYMMETRIC =====
function isSymmetric(root) {
  function isMirror(left, right) {
    if (!left && !right) return true;
    if (!left || !right) return false;
    return left.val === right.val
      && isMirror(left.left, right.right)
      && isMirror(left.right, right.left);
  }
  return isMirror(root?.left, root?.right);
}

// ===== LOWEST COMMON ANCESTOR =====
function lca(root, p, q) {
  if (!root || root === p || root === q) return root;
  const left = lca(root.left, p, q);
  const right = lca(root.right, p, q);
  if (left && right) return root; // p và q ở 2 phía
  return left || right;           // cả hai ở cùng phía
}

// ===== SERIALIZE / DESERIALIZE =====
// Dùng preorder + null markers
function serialize(root) {
  if (!root) return 'null';
  return `${root.val},${serialize(root.left)},${serialize(root.right)}`;
}

function deserialize(data) {
  const nodes = data.split(',');
  let idx = 0;

  function build() {
    if (nodes[idx] === 'null') { idx++; return null; }
    const node = new TreeNode(parseInt(nodes[idx++]));
    node.left = build();
    node.right = build();
    return node;
  }
  return build();
}

const serialized = serialize(root); // "1,2,4,null,null,5,null,null,3,null,null"
const restored = deserialize(serialized);
```

---

## 🔧 Trong Framework & Thực tế

### React — Virtual DOM
```javascript
// JSX compile thành React.createElement calls → cây Virtual DOM
// <App>           →  { type: App,
//   <Header />   →    props: { children: [
//   <Main />     →      { type: Header, ... },
//   <Footer />   →      { type: Main, ... },
// </App>         →      { type: Footer, ... }
//                →    ]}}

// React reconciler duyệt 2 cây (old và new) để tìm diff
// Dùng DFS (preorder) để so sánh node-by-node
// Kết quả: minimal set of DOM mutations
```

### Browser — DOM Tree
```javascript
// Toàn bộ HTML page = một N-ary Tree (cây n nhánh)
// querySelector dùng DFS để tìm element đầu tiên
// querySelectorAll dùng DFS để tìm tất cả

// Ví dụ: duyệt DOM như Tree
function walkDOM(node, callback) {
  callback(node);
  for (const child of node.childNodes) {
    walkDOM(child, callback); // DFS preorder
  }
}

// Shadow DOM = separate tree attached to element
const host = document.getElementById('app');
const shadow = host.attachShadow({ mode: 'open' }); // new tree!
```

### Babel / EST — Abstract Syntax Tree
```javascript
// JavaScript code được parse thành AST (Binary/N-ary Tree)
// const x = 1 + 2 trở thành:
// VariableDeclaration
//   └── VariableDeclarator
//         ├── Identifier: x
//         └── BinaryExpression (+)
//               ├── NumericLiteral: 1
//               └── NumericLiteral: 2

// ESLint, Babel, TypeScript đều dùng AST để:
// - Lint: traverse tree, check rules
// - Transform: modify nodes, generate new code
// - Type check: infer types từ cấu trúc tree

// Visitor pattern trên AST:
import traverse from '@babel/traverse';
traverse(ast, {
  Identifier(path) {
    console.log('Found identifier:', path.node.name);
  },
  BinaryExpression(path) {
    // transform: a + b → add(a, b)
  }
});
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Iterative Traversal tránh Stack Overflow
```javascript
// Recursion với cây sâu → Stack Overflow!
// Convert sang iterative dùng explicit Stack

function inorderIterative(root) {
  const result = [];
  const stack = [];
  let current = root;

  while (current || stack.length) {
    // Đi hết sang trái
    while (current) {
      stack.push(current);
      current = current.left;
    }
    // Lấy node từ stack (đây là leftmost chưa visit)
    current = stack.pop();
    result.push(current.val);
    // Chuyển sang right subtree
    current = current.right;
  }
  return result;
}
```

### 2. Morris Traversal — O(1) Space!
```javascript
// Traversal KHÔNG dùng stack, KHÔNG dùng recursion → O(1) extra space
// Dùng "threaded binary tree" technique — temporary modify tree
function morrisInorder(root) {
  const result = [];
  let current = root;

  while (current) {
    if (!current.left) {
      result.push(current.val);
      current = current.right;
    } else {
      // Tìm inorder predecessor (rightmost của left subtree)
      let predecessor = current.left;
      while (predecessor.right && predecessor.right !== current) {
        predecessor = predecessor.right;
      }

      if (!predecessor.right) {
        // Lần đầu: tạo thread (link tạm thời)
        predecessor.right = current;
        current = current.left;
      } else {
        // Lần hai: visit và xóa thread
        predecessor.right = null;
        result.push(current.val);
        current = current.right;
      }
    }
  }
  return result;
}
```

### 3. Tree DP — Dynamic Programming trên cây
```javascript
// Nhiều bài toán cây giải bằng DP bottom-up
// Tính diameter (đường kính) của cây:
function diameter(root) {
  let maxDiam = 0;

  function height(node) {
    if (!node) return 0;
    const leftH = height(node.left);
    const rightH = height(node.right);
    maxDiam = Math.max(maxDiam, leftH + rightH); // cập nhật diameter tại mỗi node
    return 1 + Math.max(leftH, rightH); // trả về height cho parent
  }

  height(root);
  return maxDiam;
}
```

---

## ❓ Bẫy phỏng vấn

### Q1: DFS vs BFS — khi nào dùng cái nào?
```
DFS (Stack/Recursion):
  + Ít memory nếu cây balanced (O(log n) vs O(n))
  + Tìm path, check connectivity, topological sort
  - Memory O(n) nếu cây skewed
  - Không tìm được đường ngắn nhất

BFS (Queue):
  + Tìm đường ngắn nhất (unweighted)
  + Level-by-level processing
  + Tìm node gần nhất với root
  - Memory O(n) cho queue (worst = last level = n/2 nodes)
```

### Q2: Serialize binary tree cần gì?
```
Preorder + Inorder: có thể reconstruct nếu không có duplicates
Preorder + null markers: có thể reconstruct với duplicates (dùng cách trên)
Level order + null markers: cũng đủ để reconstruct
Inorder alone: KHÔNG đủ (không biết root)
```

### Q3: Tại sao React cần key khi render list?
```
Khi list thay đổi, React phải match old virtual DOM nodes với new ones
Không có key: React match theo index → wrong element reuse
Có key: React match theo key → correct element reuse/unmount

Ví dụ: [A,B,C] → [B,C] 
Không key: A→B update, B→C update, C remove (3 operations)
Có key:    A remove, B stays, C stays (1 operation)
```

---

## 🔗 Xem thêm
- [BST](./binary-search-tree.md) — Binary Tree với ordering property
- [Heap](./heap.md) — Complete Binary Tree với heap property
- [Segment Tree](./segment-tree.md) — Binary Tree cho range queries
