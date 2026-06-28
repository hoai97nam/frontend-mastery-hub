# 📦 Array — Mảng

## 🧠 Định nghĩa dễ nhớ

> **Array** giống như một **dãy ghế đánh số trong rạp chiếu phim**. Muốn ngồi ghế số 5 thì đi thẳng đến đó — không cần đi qua ghế 1, 2, 3, 4. Tốc độ truy cập **không phụ thuộc vào kích thước** của rạp.

Về mặt kỹ thuật: Array là **dãy ô nhớ liên tiếp** trong RAM. Vì địa chỉ ô nhớ = `base_address + index * element_size`, nên truy cập bất kỳ phần tử nào đều **O(1)**.

---

## ⏱️ Big-O Complexity

| Thao tác | Average | Worst | Ghi chú |
|----------|---------|-------|---------|
| Access (đọc theo index) | **O(1)** | O(1) | Điểm mạnh nhất |
| Search (tìm kiếm) | O(n) | O(n) | Phải duyệt từng phần tử |
| Insert at end | **O(1)**† | O(n) | †Amortized — đôi khi resize |
| Insert at middle/front | O(n) | O(n) | Phải dịch chuyển các phần tử |
| Delete at end | O(1) | O(1) | — |
| Delete at middle/front | O(n) | O(n) | Phải dịch chuyển |

> 💡 **Space**: O(n)

---

## 💻 Code JavaScript

### Các thao tác cơ bản

```javascript
// ===== TẠO ARRAY =====
const arr = [1, 2, 3, 4, 5];
const matrix = [[1,2],[3,4]];       // 2D array
const filled = new Array(5).fill(0); // [0,0,0,0,0]

// ===== TRUY CẬP O(1) =====
console.log(arr[2]);        // 3
console.log(arr.at(-1));    // 5 (ES2022: index âm)
console.log(arr.at(-2));    // 4

// ===== THÊM / XÓA =====
arr.push(6);         // thêm cuối  — O(1) amortized
arr.pop();           // xóa cuối   — O(1)
arr.unshift(0);      // thêm đầu   — O(n) ⚠️
arr.shift();         // xóa đầu    — O(n) ⚠️

// ===== TÌM KIẾM =====
arr.indexOf(3);          // 2 — O(n)
arr.includes(3);         // true — O(n)
arr.findIndex(x => x > 3); // 3 — O(n)

// ===== BIẾN ĐỔI =====
const doubled = arr.map(x => x * 2);      // [2,4,6,8,10]
const evens   = arr.filter(x => x % 2 === 0); // [2,4]
const sum     = arr.reduce((acc, x) => acc + x, 0); // 15

// ===== SẮP XẾP =====
[3,1,4,1,5].sort((a,b) => a - b); // [1,1,3,4,5]
// ⚠️ sort() mặc định so sánh STRING, không phải số!
[10, 9, 2].sort();         // [10, 2, 9] ← SAI!
[10, 9, 2].sort((a,b) => a-b); // [2, 9, 10] ← ĐÚNG
```

### Kỹ thuật thực dụng

```javascript
// ===== SPREAD & DESTRUCTURING =====
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first=1, second=2, rest=[3,4,5]

const copy = [...arr];           // shallow copy
const merged = [...arr1, ...arr2]; // merge

// ===== SLIDING WINDOW — O(n) thay vì O(n²) =====
function maxSumSubarray(arr, k) {
  let windowSum = arr.slice(0, k).reduce((a, b) => a + b, 0);
  let maxSum = windowSum;

  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k]; // trượt cửa sổ
    maxSum = Math.max(maxSum, windowSum);
  }
  return maxSum;
}
maxSumSubarray([2, 1, 5, 1, 3, 2], 3); // 9

// ===== TWO POINTERS — tìm cặp tổng bằng target =====
function twoSum(sortedArr, target) {
  let left = 0, right = sortedArr.length - 1;
  while (left < right) {
    const sum = sortedArr[left] + sortedArr[right];
    if (sum === target) return [left, right];
    if (sum < target) left++;
    else right--;
  }
  return null;
}

// ===== FLATTEN NESTED ARRAY =====
[1, [2, [3, [4]]]].flat(Infinity); // [1,2,3,4]
```

---

## 🔧 Trong Framework & Thực tế

### React
```javascript
// useState với array — React dùng array để biểu diễn state list
const [items, setItems] = useState([]);

// ĐÚNG: tạo array mới (immutable update)
setItems(prev => [...prev, newItem]);       // thêm
setItems(prev => prev.filter(i => i.id !== id)); // xóa
setItems(prev => prev.map(i => i.id === id ? {...i, done:true} : i)); // update

// Hooks API chính là array destructuring!
const [state, setState] = useState(0); // trả về [value, setter]
const [ref]             = useRef();    // trả về [ref]
```

### Virtual DOM Diffing
```javascript
// React so sánh 2 arrays con để tính toán update tối thiểu
// Đó là lý do key prop quan trọng — giúp identify element O(1)
// Không có key → React dùng index → bug khi reorder
<ul>
  {items.map(item => (
    <li key={item.id}>{item.name}</li> // key cho phép O(n) diff
  ))}
</ul>
```

### Angular
```javascript
// NgFor với trackBy — tương tự React key
@Component({
  template: `<div *ngFor="let item of items; trackBy: trackById">{{item.name}}</div>`
})
export class MyComponent {
  trackById(index: number, item: Item) { return item.id; }
}
```

### Node.js / Express
```javascript
// Middleware stack là một Array!
const middlewares = [cors(), helmet(), express.json(), authMiddleware];
app.use(...middlewares);

// Express xử lý request bằng cách duyệt qua array middleware
// next() = đi đến middleware tiếp theo trong array
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. JavaScript Array là Dynamic Array
```javascript
// JS Array không phải array thuần túy như C
// Nó là hash map + array hybrid — "array-like object"
const a = [];
a[0] = 'zero';
a[100] = 'hundred'; // sparse array — không allocate 100 ô
a.length; // 101, nhưng chỉ có 2 phần tử thực

// V8 Engine có nhiều "ElementKind" để optimize:
// SMI_ELEMENTS    → array chỉ có small integers [1,2,3]
// DOUBLE_ELEMENTS → array chỉ có float [1.1, 2.2]
// FAST_ELEMENTS   → mixed types
// HOLEY_ELEMENTS  → sparse array (chậm hơn!)
const fast = [1, 2, 3];     // SMI_ELEMENTS → nhanh nhất
const slow = [1, , 3];      // HOLEY → chậm hơn
```

### 2. sort() không ổn định trước ES2019
```javascript
// Trước Chrome 70 / Node 11: sort() có thể không stable
// Từ V8 7.0+: TimSort (stable sort)
// Bây giờ an toàn, nhưng vẫn cần biết khi làm việc với môi trường cũ
```

### 3. Array.from() cho array-like & iterables
```javascript
Array.from('hello');           // ['h','e','l','l','o']
Array.from({length:5}, (_,i) => i*2); // [0,2,4,6,8]
Array.from(new Set([1,1,2,2])); // [1,2] — deduplicate
Array.from(document.querySelectorAll('div')); // NodeList → Array
```

### 4. TypedArray — khi cần hiệu năng cao
```javascript
// Thường dùng trong Canvas, WebGL, Audio Processing
const buffer = new Int32Array([1, 2, 3, 4]);
const floats  = new Float32Array(1024);  // WebAudio buffer

// Nhanh hơn Array thường vì fixed type, contiguous memory
// Không có thao tác như push/pop — fixed size
```

---

## ❓ Bẫy phỏng vấn

### Q1: `arr.sort()` có sắp xếp số đúng không?
```javascript
[10, 9, 100, 2].sort()         // ❌ [10, 100, 2, 9] — sort theo string!
[10, 9, 100, 2].sort((a,b)=>a-b) // ✅ [2, 9, 10, 100]
```

### Q2: Copy mảng như thế nào?
```javascript
const a = [[1,2],[3,4]];
const b = [...a];        // shallow copy — b[0] === a[0]!
const c = JSON.parse(JSON.stringify(a)); // deep copy — chậm
const d = structuredClone(a);           // ✅ ES2022 deep clone

// Thay đổi nested object:
b[0].push(99); // a[0] cũng thay đổi! ← BUG phổ biến
```

### Q3: Tại sao React yêu cầu tạo array mới thay vì mutate?
```javascript
// ❌ SAI — React không detect thay đổi này
items.push(newItem);
setItems(items); // items là cùng reference → no re-render!

// ✅ ĐÚNG — reference mới → React biết phải re-render
setItems([...items, newItem]);
```

### Q4: Xóa phần tử không dùng delete
```javascript
const arr = [1, 2, 3, 4];
delete arr[1]; // ❌ [1, empty, 3, 4] — tạo hole!
arr.splice(1, 1); // ✅ [1, 3, 4]
```

---

## 🔗 Xem thêm
- [Linked List](./linked-list.md) — khi chèn/xóa ở đầu quá nhiều
- [Stack](./stack.md) — dùng Array làm Stack
- [Queue](./queue.md) — dùng Array làm Queue (cẩn thận hiệu năng!)
