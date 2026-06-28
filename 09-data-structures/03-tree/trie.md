# 🔤 Trie — Cây tiền tố

## 🧠 Định nghĩa dễ nhớ

> **Trie** (đọc là "try") giống như một **cây danh sách điện thoại**: từng ký tự của cái tên được lưu theo từng nhánh. Gõ "Al" → ngay lập tức bạn thấy tất cả tên bắt đầu bằng "Al": Alice, Alex, Albert... mà không cần tìm kiếm trong toàn bộ danh sách.

```
Root
 ├── a
 │   ├── p
 │   │   └── p → [isEnd: "app"]
 │   │       └── l → [isEnd: "apple"]
 │   └── n
 │       └── d → [isEnd: "and"]
 └── b
     └── a
         └── t → [isEnd: "bat"]
```

**Điểm mạnh**: Tìm kiếm theo prefix trong O(L) — L là độ dài chuỗi. Không phụ thuộc n (số từ trong dictionary)!

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Insert | **O(L)** | L = độ dài chuỗi |
| Search (exact) | **O(L)** | — |
| StartsWith (prefix) | **O(L)** | — |
| Delete | O(L) | — |
| Space | O(ALPHABET_SIZE × L × N) | Tốn memory |

> 💡 **L** = key length, không phụ thuộc **N** (số keys). Tìm kiếm không chậm dần khi thêm từ!

---

## 💻 Code JavaScript

### Trie cơ bản

```javascript
class TrieNode {
  constructor() {
    this.children = {};  // character → TrieNode
    this.isEnd = false;  // đánh dấu kết thúc một từ
    this.word = null;    // lưu từ đầy đủ (optional, tiện lợi)
  }
}

class Trie {
  constructor() { this.root = new TrieNode(); }

  // INSERT — O(L)
  insert(word) {
    let node = this.root;
    for (const char of word) {
      if (!node.children[char]) {
        node.children[char] = new TrieNode();
      }
      node = node.children[char];
    }
    node.isEnd = true;
    node.word = word;
  }

  // SEARCH exact — O(L)
  search(word) {
    const node = this.#traverse(word);
    return node?.isEnd === true;
  }

  // STARTS WITH prefix — O(L)
  startsWith(prefix) {
    return this.#traverse(prefix) !== null;
  }

  // AUTOCOMPLETE — trả về tất cả từ có prefix
  autocomplete(prefix, limit = 10) {
    const node = this.#traverse(prefix);
    if (!node) return [];

    const results = [];
    this.#dfs(node, results, limit);
    return results;
  }

  #traverse(str) {
    let node = this.root;
    for (const char of str) {
      if (!node.children[char]) return null;
      node = node.children[char];
    }
    return node;
  }

  #dfs(node, results, limit) {
    if (results.length >= limit) return;
    if (node.isEnd) results.push(node.word);
    for (const child of Object.values(node.children)) {
      this.#dfs(child, results, limit);
    }
  }

  // DELETE — O(L)
  delete(word) {
    this.#deleteHelper(this.root, word, 0);
  }

  #deleteHelper(node, word, depth) {
    if (!node) return false;
    if (depth === word.length) {
      if (!node.isEnd) return false;
      node.isEnd = false;
      node.word = null;
      return Object.keys(node.children).length === 0; // true nếu có thể xóa node
    }
    const char = word[depth];
    if (this.#deleteHelper(node.children[char], word, depth + 1)) {
      delete node.children[char]; // xóa con nếu không cần nữa
      return !node.isEnd && Object.keys(node.children).length === 0;
    }
    return false;
  }
}

// Demo
const trie = new Trie();
['apple', 'app', 'application', 'apply', 'apt', 'bat', 'ball'].forEach(w => trie.insert(w));

trie.search('app');        // true
trie.search('ap');         // false (chưa được insert)
trie.startsWith('app');    // true
trie.autocomplete('app');  // ['app', 'apple', 'application', 'apply']
trie.autocomplete('b');    // ['bat', 'ball']
```

### Word Search II (Trie + DFS)

```javascript
// Bài toán: Tìm tất cả từ trong grid (ma trận chữ cái)
// Trie cho phép search nhiều từ cùng lúc trong 1 lần DFS!
function findWords(board, words) {
  const trie = new Trie();
  words.forEach(w => trie.insert(w));

  const result = new Set();
  const m = board.length, n = board[0].length;

  function dfs(node, i, j, path) {
    if (node.isEnd) result.add(node.word);
    if (i < 0 || i >= m || j < 0 || j >= n || board[i][j] === '#') return;

    const char = board[i][j];
    if (!node.children[char]) return; // prefix không tồn tại trong Trie → prune

    board[i][j] = '#'; // đánh dấu đã thăm
    const nextNode = node.children[char];
    [[i-1,j],[i+1,j],[i,j-1],[i,j+1]].forEach(([ni, nj]) => dfs(nextNode, ni, nj));
    board[i][j] = char; // restore
  }

  for (let i = 0; i < m; i++)
    for (let j = 0; j < n; j++)
      dfs(trie.root, i, j);

  return [...result];
}
```

### Compressed Trie (Radix Tree)

```javascript
// Trie thường tốn memory — Radix Tree nén các chuỗi ký tự đơn thành 1 node
// 'abc' → 'def' → 3 nodes riêng biệt trong Trie thường
//              → 1 node 'abcdef' trong Radix Tree

// Express.js router dùng Radix Tree!
// 'GET /users'   → 1 node
// 'GET /users/:id' → 1 node thêm, nhánh từ 'users'
// Thay vì match string-by-string, route matching là O(log n)

class RadixNode {
  constructor(label = '') {
    this.label = label;    // chuỗi ký tự (không phải 1 char)
    this.children = new Map();
    this.isEnd = false;
    this.handler = null;   // route handler
  }
}
```

---

## 🔧 Trong Framework & Thực tế

### Express.js / Fastify — Router (Radix Tree)
```javascript
// Express dùng path-to-regexp (không phải Trie)
// Fastify dùng find-my-way (Radix Tree) → nhanh hơn Express nhiều!

// find-my-way internally:
const router = require('find-my-way')();
router.on('GET', '/user/:id', handler1);
router.on('GET', '/user/:id/posts', handler2);
router.on('POST', '/user', handler3);

// Route lookup: O(L) — L = path length
// Radix Tree tối ưu: /user → nhánh → :id → nhánh → /posts
```

### Browser — CSS Selector Matching
```javascript
// Browser engine dùng Trie-like structure để match CSS selectors
// .header .nav a { } → trie của selector parts
// Selector matching từ RIGHT TO LEFT (quirk của CSS engines):
// 1. Tìm tất cả 'a' elements
// 2. Check có ancestor '.nav' không
// 3. Check có ancestor '.header' không
// → Trie của selector tokens giúp tăng tốc

// Đây là lý do:
// .container .list .item span { } → SLOW (nhiều levels)
// .item-span { } → FAST (direct class)
```

### Autocomplete Search
```javascript
// Search box trong ứng dụng — Trie là giải pháp chuẩn
// Google Suggest, IDE intellisense đều dùng Trie variants

class SearchSuggest {
  #trie = new Trie();
  #searchHistory = new Map(); // word → frequency

  addSearchTerm(term) {
    term = term.toLowerCase();
    this.#trie.insert(term);
    this.#searchHistory.set(term, (this.#searchHistory.get(term) || 0) + 1);
  }

  getSuggestions(prefix, limit = 5) {
    const candidates = this.#trie.autocomplete(prefix.toLowerCase(), 50);
    // Sort theo frequency — kết quả phổ biến hơn xuất hiện trước
    return candidates
      .sort((a, b) => (this.#searchHistory.get(b) || 0) - (this.#searchHistory.get(a) || 0))
      .slice(0, limit);
  }
}

const suggest = new SearchSuggest();
['javascript', 'java', 'java spring', 'javascript framework', 'javascript react'].forEach(t => {
  suggest.addSearchTerm(t);
  suggest.addSearchTerm(t); // thêm 2 lần = frequency cao hơn
});
suggest.addSearchTerm('javascript react'); // thêm lần 3

suggest.getSuggestions('java');
// ['javascript react', 'javascript', 'javascript framework', 'java', 'java spring']
```

### Spell Checker / Typo Correction
```javascript
// Trie + Levenshtein distance cho spell check
function spellCheck(trie, word, maxDistance = 2) {
  const results = [];
  const n = word.length;

  // BFS qua Trie với edit distance tracking
  function dfs(node, current, row) {
    if (row[n] <= maxDistance && node.isEnd) {
      results.push({ word: node.word, distance: row[n] });
    }

    for (const [char, child] of Object.entries(node.children)) {
      const newRow = [current.length + 1];
      for (let i = 1; i <= n; i++) {
        const cost = word[i-1] === char ? 0 : 1;
        newRow[i] = Math.min(newRow[i-1]+1, row[i]+1, row[i-1]+cost);
      }
      if (Math.min(...newRow) <= maxDistance) {
        dfs(child, current + char, newRow);
      }
    }
  }

  const initialRow = Array.from({length: n+1}, (_, i) => i);
  dfs(trie.root, '', initialRow);
  return results.sort((a,b) => a.distance - b.distance);
}
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Trie dùng Array thay Map — nhanh hơn cho ASCII
```javascript
// Dùng Array[26] thay Map cho lowercase alphabet
// Array access O(1) và cache-friendly hơn Map
class FastTrieNode {
  constructor() {
    this.children = new Array(26).fill(null); // 'a'=0, 'b'=1,...
    this.isEnd = false;
  }
}

class FastTrie {
  insert(word) {
    let node = this.root;
    for (const char of word) {
      const idx = char.charCodeAt(0) - 97; // 'a' = 0
      if (!node.children[idx]) node.children[idx] = new FastTrieNode();
      node = node.children[idx];
    }
    node.isEnd = true;
  }
}
```

### 2. Trie số nguyên — XOR problems
```javascript
// Binary Trie (0/1 trie) — mỗi node là bit 0 hoặc 1
// Dùng để tìm maximum XOR của 2 số trong array — O(n × 32)

class BinaryTrie {
  root = { children: [null, null] };

  insert(num) {
    let node = this.root;
    for (let i = 31; i >= 0; i--) {
      const bit = (num >> i) & 1;
      if (!node.children[bit]) node.children[bit] = { children: [null, null] };
      node = node.children[bit];
    }
  }

  maxXOR(num) {
    let node = this.root;
    let result = 0;
    for (let i = 31; i >= 0; i--) {
      const bit = (num >> i) & 1;
      const want = 1 - bit; // muốn bit ngược lại để XOR = 1
      if (node.children[want]) {
        result |= (1 << i);
        node = node.children[want];
      } else {
        node = node.children[bit];
      }
    }
    return result;
  }
}
```

### 3. Trie cho IP Routing
```javascript
// Router (network) dùng Binary Trie để lookup IP prefix
// IP: 192.168.1.1 → binary → tìm longest matching prefix
// → Quyết định route packet đi đâu
// Đây là tại sao mạng lớn vẫn route packet nhanh!
```

---

## ❓ Bẫy phỏng vấn

### Q1: Trie vs HashMap cho word search?
```
HashMap: search exact word O(1), nhưng KHÔNG support prefix search
Trie:    search exact O(L), search prefix O(L) → linh hoạt hơn nhiều

Với 1 triệu từ:
HashMap.has("apple"): O(1) — rất nhanh
HashMap prefix search: phải duyệt tất cả keys O(n) — chậm
Trie prefix search: O(L) — không phụ thuộc số từ
```

### Q2: Memory của Trie vs HashMap?
```
HashMap: lưu mỗi string nguyên vẹn → thường hiệu quả hơn
Trie: mỗi node = object + Map/Array → overhead nhiều hơn
     Nhưng: share prefix → tiết kiệm khi nhiều từ chung prefix
     
Ví dụ: 1 triệu URLs bắt đầu bằng "https://api.example.com"
Trie: path chung chỉ lưu 1 lần
HashMap: mỗi URL lưu full string
```

### Q3: Khi nào dùng Trie vs Aho-Corasick?
```
Trie:            search nhiều pattern trong 1 text lần lượt
Aho-Corasick:    search nhiều pattern trong 1 text CÙNG LÚC — O(n + m + z)
                 (n = text length, m = total pattern length, z = matches)

Aho-Corasick dùng trong:
- Anti-virus scanning (tìm nhiều virus signatures cùng lúc)
- Spam filter (tìm nhiều keywords)
- grep với nhiều patterns
```

---

## 🔗 Xem thêm
- [HashMap](../02-hash-based/hash-map.md) — Trie khi không cần prefix search
- [Graph](../04-graph/graph.md) — Trie là Directed Acyclic Graph đặc biệt
- [Segment Tree](./segment-tree.md) — Tree khác cho range queries
