# 🟣 Probabilistic Data Structures — Cấu trúc dữ liệu xác suất

> **Probabilistic DS** = cấu trúc dữ liệu chấp nhận **đánh đổi độ chính xác** (có sai số nhỏ khoảng 1%) để đạt được **tốc độ cực nhanh O(1)** và **bộ nhớ cố định siêu nhỏ** khi xử lý dữ liệu quy mô lớn (Big Data).

Hai đại diện kinh điển nhất ở nhóm này là **HyperLogLog** và **Count-Min Sketch**.

---

## Các cấu trúc trong nhóm

| DS | Câu hỏi cần giải quyết | Naive approach (HashSet/Map) | Probabilistic approach |
|----|-------------------------|------------------------------|------------------------|
| **HyperLogLog** ⭐ | Đếm số lượng phần tử độc nhất (ví dụ: số người click xem bài viết này). | `new Set()` lưu trữ mọi ID -> Tốn **40GB RAM** khi có 1 tỷ user. | Dùng phép toán hash và đếm bit đầu -> Tốn **12KB RAM** cố định (sai số 1%). |
| **Count-Min Sketch** | Ước tính tần suất xuất hiện (ví dụ: đếm xem từ khóa nào được search bao nhiêu lần). | `new Map()` đếm từng key -> Tốn RAM khổng lồ khi số lượng từ khóa vô hạn. | Dùng ma trận bit và nhiều hàm hash song song -> Bộ nhớ cố định nhỏ, không bao giờ đếm thiếu. |

---

# 1. HyperLogLog — Đếm xấp xỉ số lượng phần tử độc nhất

## 🧠 Định nghĩa dễ nhớ

> **HyperLogLog** giống như **tung đồng xu**: Nếu bạn tung đồng xu liên tiếp và thấy xuất hiện chuỗi **3 mặt Ngửa liên tiếp** mới có mặt Sấp -> Bạn đoán bạn đã tung khoảng 2³ = 8 lần. Nếu bạn thấy **20 mặt Ngửa liên tiếp** mới có mặt Sấp -> Bạn đoán bạn phải tung hàng triệu lần mới gặp may thế. HyperLogLog băm (hash) ID của user thành chuỗi bit nhị phân, và ghi lại số lượng số `0` liên tiếp dài nhất ở đầu chuỗi bit đó để ước lượng tổng số lượng user độc nhất.

## 💻 Code JavaScript mô phỏng HyperLogLog

```javascript
class SimpleHyperLogLog {
  constructor(bucketCountLog = 6) {
    this.p = bucketCountLog; // số bit dùng làm chỉ số bucket (ví dụ 6 bit -> 64 buckets)
    this.m = 1 << this.p;    // số lượng buckets (64)
    this.buckets = new Array(this.m).fill(0);
  }

  #hash(str) {
    // Hàm băm sinh ra số nguyên 32-bit ngẫu nhiên đồng đều
    let h = 2166136261;
    for (let i = 0; i < str.length; i++) {
      h ^= str.charCodeAt(i);
      h += (h << 1) + (h << 4) + (h << 7) + (h << 8) + (h << 24);
    }
    return h >>> 0;
  }

  add(item) {
    const val = this.#hash(String(item));
    
    // Lấy p bit đầu làm index cho bucket
    const bucketIndex = val >>> (32 - this.p);
    
    // Lấy 32-p bit còn lại, tìm vị trí số 1 đầu tiên từ phải sang (số số 0 liên tiếp)
    const remainingBits = val & ((1 << (32 - this.p)) - 1);
    const countZeros = this.#countLeadingZeros(remainingBits) + 1;

    // Lưu số số 0 lớn nhất gặp được vào bucket tương ứng
    this.buckets[bucketIndex] = Math.max(this.buckets[bucketIndex], countZeros);
  }

  #countLeadingZeros(num) {
    if (num === 0) return 32 - this.p;
    return Math.clz32(num) - this.p; // Hàm ES6 tìm số bit 0 ở đầu
  }

  count() {
    // Áp dụng trung bình điều hòa (Harmonic Mean) của các bucket để loại bỏ sai lệch
    let sum = 0;
    for (let i = 0; i < this.m; i++) {
      sum += Math.pow(2, -this.buckets[i]);
    }

    // Hằng số hiệu chỉnh alpha của thuật toán Flajolet
    const alpha = 0.7213 / (1 + 1.079 / this.m);
    let estimate = alpha * this.m * this.m / sum;

    return Math.round(estimate);
  }
}

// Chạy thử nghiệm
const hll = new SimpleHyperLogLog(8); // 256 buckets
const uniqueUsers = new Set();

// Giả lập thêm 10,000 users
for (let i = 0; i < 10000; i++) {
  const userId = `user_${Math.floor(Math.random() * 8000)}`; // Có trùng lặp
  uniqueUsers.add(userId);
  hll.add(userId);
}

console.log("Số lượng thực tế (HashSet):", uniqueUsers.size);
console.log("Số lượng ước tính (HyperLogLog):", hll.count());
// Kết quả xấp xỉ nhau với sai số nhỏ, nhưng HLL tốn cực ít RAM!
```

---

# 2. Count-Min Sketch — Ước tính tần suất xuất hiện

## 🧠 Định nghĩa dễ nhớ

> **Count-Min Sketch** giống như một **bảng chấm công**: thay vì ghi tên từng người để đếm số buổi đi làm (Map), ta có một ma trận các ô số và một số hàm băm. Khi một người đến, ta băm tên họ bằng 3 hàm hash khác nhau ra 3 vị trí ô số và cộng 1 vào 3 ô đó. Khi muốn kiểm tra người đó đi làm mấy buổi, ta băm lại tên họ và lấy giá trị **nhỏ nhất** trong 3 ô số đó. Vì có thể có người khác bị băm trùng ô (Collision), việc lấy giá trị nhỏ nhất (Min) sẽ giúp triệt tiêu bớt sai lệch phóng đại.

```
Hàm Hash 1 ──► [ô 2] = 5
Hàm Hash 2 ──► [ô 7] = 8
Hàm Hash 3 ──► [ô 1] = 5
Tần suất ước tính = Min(5, 8, 5) = 5.
```

## 💻 Code JavaScript mô phỏng Count-Min Sketch

```javascript
class CountMinSketch {
  constructor(width = 1000, depth = 4) {
    this.w = width;  // Số lượng cột
    this.d = depth;  // Số lượng hàng (tương đương số hàm hash)
    // Khởi tạo ma trận d x w chứa các giá trị đếm bằng 0
    this.table = Array.from({ length: depth }, () => new Uint32Array(width));
  }

  // Tạo d hàm hash khác nhau bằng thuật toán FNV-1a với seeds
  #hash(str, index) {
    let hash = 2166136261 + index * 16777619;
    for (let i = 0; i < str.length; i++) {
      hash ^= str.charCodeAt(i);
      hash = (hash * 16777619) >>> 0;
    }
    return hash % this.w;
  }

  // Thêm phần tử
  add(item, count = 1) {
    const str = String(item);
    for (let i = 0; i < this.d; i++) {
      const col = this.#hash(str, i);
      this.table[i][col] += count;
    }
  }

  // Ước tính tần suất
  estimate(item) {
    const str = String(item);
    let minCount = Infinity;
    for (let i = 0; i < this.d; i++) {
      const col = this.#hash(str, i);
      minCount = Math.min(minCount, this.table[i][col]);
    }
    return minCount;
  }
}

// Chạy thử nghiệm
const sketch = new CountMinSketch(100, 4);
sketch.add("apple", 15);
sketch.add("banana", 3);
sketch.add("apple", 5);

console.log("Ước tính apple:", sketch.estimate("apple"));   // 20
console.log("Ước tính banana:", sketch.estimate("banana")); // 3
console.log("Ước tính orange:", sketch.estimate("orange")); // 0 (hoặc sai số rất nhỏ nếu trùng)
```

---

## 🔧 Trong Framework & Thực tế

### 1. Redis HyperLogLog (PFADD, PFCOUNT)
```
Redis cung cấp sẵn kiểu dữ liệu HyperLogLog cực mạnh.
- PFADD run_log "user_1"
- PFADD run_log "user_2"
- PFCOUNT run_log  -> Trả về số lượng độc nhất.
Redis sử dụng đúng 12KB bộ nhớ cho mỗi HyperLogLog key bất kể bạn add bao nhiêu phần tử, 
sai số tiêu chuẩn chỉ khoảng 0.81%.
```

### 2. CDN Rate Limiting (Chặn Spam, DDOS)
```
Các nhà cung cấp CDN như Cloudflare cần đếm xem một địa chỉ IP đã gửi bao nhiêu request 
trong 1 phút để quyết định chặn (rate-limit).
Vì có hàng tỷ IP truy cập toàn cầu, việc lưu từng IP vào HashMap trong RAM của máy chủ CDN là bất khả thi.
Họ dùng Count-Min Sketch để ước tính tần suất IP rất nhanh với bộ nhớ RAM cực nhỏ và cố định.
```

### 3. Tìm các phần tử phổ biến nhất (Heavy Hitters)
```
Các mạng xã hội cần xác định hashtag nào đang thịnh hành (Trending topics).
Bằng cách đẩy các hashtag liên tục vào Count-Min Sketch, hệ thống dễ dàng lọc ra 
các key có tần suất vượt ngưỡng quy định mà không cần lưu cơ sở dữ liệu khổng lồ.
```

---

## ❓ Bẫy phỏng vấn

### Q1: Cấu trúc dữ liệu xác suất có bao giờ trả về sai số âm (False Negative) không?
```
KHÔNG BAO GIỜ.
- Với HyperLogLog: Đếm số lượng độc nhất, nó chỉ có sai số xấp xỉ chứ không lưu giá trị cụ thể.
- Với Count-Min Sketch: Vì ta cộng 1 vào các ô, sự trùng lặp (Collision) chỉ có thể làm 
  cho giá trị trong ô LỚN HƠN hoặc BẰNG giá trị thực tế.
  Nó không bao giờ làm giá trị nhỏ đi -> Count-Min Sketch có thể đếm THỪA chứ không bao giờ đếm THIẾU.
```

### Q2: Tại sao không dùng HashMap đếm cho chính xác 100%?
```
Khi kích thước dữ liệu vượt quá giới hạn RAM vật lý (ví dụ: log lưu lượng mạng gigabit mỗi giây).
Việc dùng HashMap sẽ làm tràn bộ nhớ (Out of Memory) và sập hệ thống.
Sử dụng cấu trúc xác suất giúp bảo vệ hệ thống hoạt động ổn định với tài nguyên cố định.
```

---

## 🔗 Xem thêm
- [Bloom Filter](../02-hash-based/bloom-filter.md) — Một cấu trúc xác suất nổi tiếng khác để check membership
- [HashMap](../02-hash-based/hash-map.md) — Khi cần chính xác tuyệt đối
