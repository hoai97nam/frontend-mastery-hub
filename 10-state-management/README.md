# 📦 State Management — Quản Lý Trạng Thái Trong Frontend Hiện Đại

> "State management is not about writing state to a global object, it's about managing the flow of data through time and space in a predictable way."

---

## 📂 Cấu Trúc Thư Mục

| File | Nội dung |
| :--- | :--- |
| 📑 [`README.md`](./README.md) | Tổng quan các trường phái quản lý trạng thái, So sánh paradigms, Ma trận quyết định & Mapping liên framework |
| ├── 📑 [`interviews.md`](./interviews.md) | Tổng hợp câu hỏi phỏng vấn nâng cao (Concept & Tình huống thiết kế thực tế cho Senior) |

---

## 1. Khái Niệm Chính & Phân Loại State

Trong lập trình frontend hiện đại, **State (Trạng thái)** là nguồn dữ liệu duy nhất đại diện cho giao diện người dùng tại một thời điểm cụ thể (`UI = f(State)`). Để thiết kế hệ thống quản lý state hiệu quả, trước hết cần phân loại rõ ràng:

### 1.1 Phân Loại Theo Phạm Vi (Scope)
- **Local State:** Trạng thái chỉ dùng trong một component hoặc cây con nhỏ (ví dụ: `isOpen` của Modal, giá trị input tạm thời).
- **Global State:** Trạng thái dùng chung bởi nhiều component ở các nhánh khác nhau (ví dụ: thông tin `userProfile`, `theme`, `shoppingCart`).

### 1.2 Phân Loại Theo Mục Đích (Purpose)
- **UI State (Client State):** Trạng thái phục vụ giao diện người dùng (ví dụ: loading spinners, active tabs, bộ lọc tìm kiếm). Thường biến mất khi reload trang.
- **Server Cache State:** Trạng thái đồng bộ từ server (ví dụ: danh sách sản phẩm, chi tiết bài viết). Có tính chất không đáng tin cậy (stale) và cần cơ chế cache, re-fetch, và sync.
- **Form State:** Trạng thái quản lý dữ liệu nhập liệu từ người dùng (validation, touched fields, values). Cần phản hồi nhanh nhạy.

---

## 2. 5 Trường Phái Kiến Trúc State Management Phổ Biến

Mỗi thư viện hoặc cơ chế quản lý state đều đi theo một trong các triết lý thiết kế (paradigms) dưới đây:

### 2.1 Flux / Unidirectional Data Flow (Luồng dữ liệu đơn hướng)
- **Triết lý:** Dữ liệu di chuyển theo một chiều duy nhất và State là bất biến (**Immutable**). Bất kỳ sự thay đổi trạng thái nào cũng phải được mô tả thông qua một sự kiện rõ ràng gọi là **Action**.
- **Cơ chế hoạt động sâu:** 
  - Thay vì sửa đổi trực tiếp (mutation), khi một Action được gửi đi (**Dispatch**), Store/Reducer nhận state hiện tại và action, sau đó trả về một state hoàn toàn mới (sử dụng toán tử spread `{...state}` hoặc thư viện Immutability như Immer).
  - Trình duyệt/Thư viện UI so sánh địa chỉ bộ nhớ (reference pointer) của state cũ và mới thông qua phép so sánh nhanh $O(1)$ (`oldState !== newState`). Nếu khác, component được cập nhật.
- **Khái niệm cốt lõi:**
  - **Store**: Nơi lưu trữ trạng thái tập trung.
  - **Action**: Một plain object có thuộc tính `type` và dữ liệu đính kèm `payload`.
  - **Reducer / Mutator**: Hàm thuần khiết (pure function) nhận vào state hiện tại + action và tính toán ra state mới.
- **Ưu & Nhược điểm:**
  - *Ưu điểm:* Cực kỳ dễ tiên đoán (predictable), hỗ trợ tốt Time-Travel Debugging (quay ngược thời gian bằng cách replay các Action), dễ viết Unit Test cho Reducer.
  - *Nhược điểm:* Viết nhiều code boilerplate (đặc biệt là Redux cổ điển), có thể gây suy giảm hiệu năng render nếu không viết các selector tối ưu (khiến cả component tree re-render do thay đổi reference của store cha).
- **Đại diện:** Redux, Zustand (React); Pinia, Vuex (Vue); NgRx (Angular).
- **Luồng dữ liệu:**
  ```mermaid
  graph LR
      View[Component / View] -->|Dispatch| Action[Action]
      Action -->|Payload| Store[Store / Reducer]
      Store -->|State update| View
  ```

### 2.2 Proxy-based / Mutable Reactivity (Phản xạ dựa trên Proxy)
- **Triết lý:** Cho phép lập trình viên thao tác và thay đổi trực tiếp (mutate) trạng thái một cách tự nhiên như biến JS thông thường (`state.count++`), nhưng dưới sự giám sát tự động của ES6 **Proxy**.
- **Cơ chế hoạt động sâu:**
  - Khi khởi tạo store, toàn bộ object trạng thái được bọc trong một Proxy. Proxy này cài đặt các trap `get` và `set`.
  - **Dependency Tracking (Thu thập phụ thuộc):** Khi component chạy hàm render, nó truy cập thuộc tính của state (kích hoạt trap `get`). Lúc này, Proxy ghi nhận component hiện tại là một "phụ thuộc" (dependency).
  - **Change Propagation (Lan truyền thay đổi):** Khi state bị sửa đổi trực tiếp (kích hoạt trap `set`), Proxy lập tức tra cứu danh sách các component phụ thuộc đã lưu trước đó và trigger re-render cho đúng các component đó.
- **Khái niệm cốt lõi:**
  - **Proxy/Observable State**: Trạng thái được bọc để tự theo dõi các hành vi get/set.
  - **Derivations/Computed**: Các giá trị được tính toán tự động dựa trên các thuộc tính reactive.
  - **Actions**: Các hàm thực hiện việc đột biến trạng thái (giúp gom nhóm các thay đổi để re-render một lần duy nhất - batching updates).
- **Ưu & Nhược điểm:**
  - *Ưu điểm:* Code ngắn gọn, không Boilerplate, tự động tối ưu hóa re-render ở mức chi tiết nhất mà không cần viết Selector thủ công.
  - *Nhược điểm:* State mutable làm mất tính năng Time-travel mặc định (muốn làm phải tự quản lý bản chụp snapshot), khó theo dõi dòng dữ liệu (data flow) vì đột biến có thể xảy ra ở bất kỳ đâu nếu không có quy chuẩn nghiêm ngặt.
- **Đại diện:** MobX, Valtio (React); Vue 3 Reactivity (ref, reactive).
- **Luồng dữ liệu:**
  ```mermaid
  graph LR
      Component[Component / View] -->|Read property| Proxy[Proxy State]
      Proxy -->|Auto-track dependency| Component
      Action[Action / Direct Mutation] -->|Modify| Proxy
      Proxy -->|Auto trigger re-render| Component
  ```

### 2.3 Atomic State (Trạng thái dạng nguyên tử)
- **Triết lý:** Đi ngược lại với cách quản lý tập trung (Single Store) của Flux. Atomic chia nhỏ toàn bộ trạng thái hệ thống thành các nút độc lập có kích thước nhỏ nhất gọi là **Atom** (Nguyên tử), sau đó kết hợp chúng lại thành đồ thị trạng thái (State Graph).
- **Cơ chế hoạt động sâu:**
  - Mỗi Atom đại diện cho một phần trạng thái cụ thể. Thay vì lưu trữ tập trung tại cây component, các giá trị của Atom được lưu trong một cấu trúc giống như một Dictionary (bản đồ Key-Value) nằm bên ngoài cây React.
  - Khi component sử dụng một Atom, nó sẽ đăng ký lắng nghe (subscribe) trực tiếp vào key của Atom đó. Khi giá trị Atom cập nhật, chỉ có các component đăng ký đúng key đó mới render lại, loại bỏ hoàn toàn hiện tượng re-render của Context.
  - Hỗ trợ các derived atom (computed atom) tính toán dựa trên các atom khác. Khi một atom cha thay đổi, nó sẽ kích hoạt đồ thị phụ thuộc để cập nhật các derived atom liên quan.
- **Khái niệm cốt lõi:**
  - **Atom**: Đơn vị trạng thái nhỏ nhất có thể đọc/ghi.
  - **Selector / Derived Atom**: Atom phụ thuộc tính toán từ một hoặc nhiều atom khác.
- **Ưu & Nhược điểm:**
  - *Ưu điểm:* Thiết kế từ dưới lên (Bottom-up) cực kỳ linh hoạt, giải quyết triệt để bài toán render cục bộ mà không cần cấu hình phức tạp, lý tưởng cho các ứng dụng có nhiều component độc lập tương tác phức tạp (như ứng dụng vẽ Canvas, Dashboard đồ thị).
  - *Nhược điểm:* Khó quản lý luồng dữ liệu trên quy mô hệ thống cực lớn do các atom nằm rải cờc, khó áp dụng các quy chuẩn kiểm soát bảo mật dữ liệu tập trung.
- **Đại diện:** Recoil, Jotai (React).
- **Luồng dữ liệu:**
  ```mermaid
  graph TD
      AtomA[Atom A: count] --> DerivedAtom[Derived Atom: doubleCount]
      AtomB[Atom B: text]
      Component1[Component 1] -->|Subscribe| AtomA
      Component2[Component 2] -->|Subscribe| DerivedAtom
      Component3[Component 3] -->|Subscribe| AtomB
  ```

### 2.4 Reactive / Observable Streams (Dòng dữ liệu phản ứng)
- **Triết lý:** Dựa trên mô hình lập trình hướng sự kiện (**Event-driven**) và **Lập trình hàm phản ứng (FRP)**. Mọi thứ từ click chuột, phản hồi từ API đến sự thay đổi trạng thái đều được trừu tượng hóa thành các luồng dữ liệu liên tục theo thời gian (Streams).
- **Cơ chế hoạt động sâu:**
  - Sử dụng mẫu thiết kế **Observer Pattern** cải tiến kết hợp với **Iterator Pattern**. Dữ liệu được đẩy từ nguồn phát (**Observable**) tới bên nhận (**Observer**).
  - Sử dụng các toán tử thuần khiết (**Operators**) để xử lý dòng dữ liệu như `map`, `filter`, `debounceTime`, `switchMap`... trước khi nó được hiển thị lên màn hình.
  - Phù hợp với kiến trúc Push-based: UI là bên thụ động đăng ký nhận tin và cập nhật bất cứ khi nào dòng dữ liệu bắn ra giá trị mới.
- **Khái niệm cốt lõi:**
  - **Observable**: Luồng dữ liệu có thể lắng nghe.
  - **Observer**: Bộ lắng nghe nhận các thông báo `next`, `error`, và `complete`.
  - **Operators**: Hàm chuyển đổi luồng dữ liệu đầu vào thành luồng dữ liệu đầu ra.
- **Ưu & Nhược điểm:**
  - *Ưu điểm:* Cực kỳ mạnh mẽ khi xử lý bất đồng bộ phức tạp, xử lý đồng thời (concurrency), các bài toán real-time, WebSocket, gom nhóm sự kiện hoặc hủy bỏ tác vụ (cancellation).
  - *Nhược điểm:* Độ dốc học tập (learning curve) cực kỳ cao, code dễ trở nên rối rắm nếu lạm dụng (được gọi là "RxJS spaghetti"), nguy cơ rò rỉ bộ nhớ rất lớn nếu quên đóng kết nối (`unsubscribe`).
- **Đại diện:** RxJS (mặc định trong Angular), Akita, Elf.
- **Luồng dữ liệu:**
  ```mermaid
  graph LR
      Event[User Click / API Response] -->|Stream| Observable[Observable Stream]
      Observable -->|Pipe / Map / Filter| Operators[Operators]
      Operators -->|Emit value| Subscription[Subscription / View]
  ```

### 2.5 Signals (Phản ứng hạt mịn - Fine-grained Reactivity)
- **Triết lý:** Là đỉnh cao của tối ưu hóa hiệu năng render trong thời đại mới. Triết lý của Signals là bỏ qua hoàn toàn Virtual DOM hoặc quá trình chạy lại (re-running) của component cha để cập nhật trực tiếp tại đúng vị trí nút DOM cần thay đổi.
- **Cơ chế hoạt động sâu:**
  - Một Signal là một object đóng gói một giá trị kèm hàm getter/setter.
  - Khi một Signal được gọi (`count()`) bên trong phần hiển thị của component (template / JSX), bộ biên dịch (compiler) hoặc runtime sẽ ghi nhận mối liên kết trực tiếp giữa Signal này với nút DOM thực tế hiển thị chữ số đó.
  - Khi Signal được cập nhật giá trị mới (`setCount(1)`), nó không thông qua bộ so sánh ảo (Virtual DOM Diffing) của component cha, mà trực tiếp cập nhật nút DOM thực tế đó thông qua tham chiếu trỏ thẳng (`domNode.textContent = 1`).
- **Khái niệm cốt lõi:**
  - **Signal**: Trạng thái cơ bản (chứa getter và setter).
  - **Computed / Memoized**: Giá trị phái sinh tự động cập nhật khi Signal phụ thuộc thay đổi.
  - **Effect**: Tác vụ phụ chạy tự động khi bất kỳ Signal nào được gọi trong nó thay đổi.
- **Ưu & Nhược điểm:**
  - *Ưu điểm:* Hiệu năng tiệm cận Javascript thuần (Vanilla JS), không có chi phí dư thừa cho việc re-run hàm component hay diffing DOM ảo, code tự nhiên không cần khai báo dependencies.
  - *Nhược điểm:* Cần compiler hoặc runtime hỗ trợ sâu, mất tính năng reactive nếu phá cấu trúc (destructuring) object chứa signal mà không có hàm bọc.
- **Đại diện:** SolidJS Signals, Preact Signals, Angular Signals, Vue Ref.
- **Luồng dữ liệu:**
  ```mermaid
  graph LR
      Signal[Signal / State] -->|Track dependency| Effect[Computed / Effect]
      Signal -->|Direct DOM Bind| DOMNode[DOM Text Node]
      Write[Write Signal] -->|Update value| Signal
      Signal -->|Auto update directly| DOMNode
  ```

---

## 3. Ma Trận Quyết Định & Đối Chiếu Thư Viện (Decision Matrix)

### 3.1 Ma Trận Quyết Định Chọn Paradigm

| Tiêu chí | Flux (Redux, Zustand) | Proxy (MobX) | Atomic (Jotai) | Reactive (RxJS) | Signals |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Học tập (Learning Curve)** | ⚠️ Trung bình - Khó | ⭐ Dễ tiếp cận | 🚀 Dễ nhất | ❌ Cực khó | ⭐ Dễ tiếp cận |
| **Boilerplate Code** | ❌ Nhiều (Redux) / Ít (Zustand) | ⭐ Ít | ⭐ Rất ít | ⚠️ Trung bình | 🚀 Ít nhất |
| **Hiệu năng mặc định** | ⚠️ Cần tối ưu Selector | 🚀 Cực tốt (Auto-track) | ⚡ Tốt | ⚡ Tốt | 🚀 Cực định (Fine-grained) |
| **Khả năng Debug (DevTools)** | 🚀 Cực tốt (Time-travel) | ⚠️ Khó trace | ⚠️ Khó trace | ❌ Rất khó trace | ⚠️ Đang phát triển |
| **Phù hợp nhất với** | App lớn, luồng nghiệp vụ phức tạp, cần lịch sử | App cần hiệu năng cao, viết code nhanh | App dạng dashboard, widget độc lập | App real-time, xử lý luồng sự kiện liên tục | SPA hiệu năng cao, tương tác DOM dày đặc |

### 3.2 Bảng Mapping Thư Viện Giữa Các Framework

| Thư viện / Paradigm | React | Vue | Angular | Svelte |
| :--- | :--- | :--- | :--- | :--- |
| **Flux / Unidirectional** | **Redux Toolkit**, **Zustand** | Pinia, Vuex | NgRx Store | (Thường dùng Svelte Store) |
| **Proxy / Mutable** | MobX, Valtio | Vue Reactivity (mặc định) | - | - |
| **Atomic** | Jotai, Recoil | - | - | Svelte Stores (tương đồng) |
| **Reactive Streams** | RxJS (kết hợp useEffect) | RxJS | **RxJS** (Mặc định trong Angular) | RxJS |
| **Signals** | `@preact/signals-react` | Vue `ref()` / `computed()` | **Angular Signals** (v16+) | Svelte 5 Runes (`$state`) |

---

## 4. Code Ví Dụ So Sánh Cú Pháp (React vs Vue vs Angular)

Hãy xem cách tăng một biến đếm (`count`) và tính toán giá trị nhân đôi (`doubleCount`) qua các kiến trúc khác nhau:

### 4.1 React: Zustand (Flux Paradigm)
```typescript
import { create } from 'zustand';

interface CounterState {
  count: number;
  doubleCount: () => number;
  increment: () => void;
}

export const useCounterStore = create<CounterState>((set, get) => ({
  count: 0,
  doubleCount: () => get().count * 2,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// Sử dụng trong Component
function CounterComponent() {
  const count = useCounterStore((state) => state.count);
  const doubleCount = useCounterStore((state) => state.doubleCount());
  const increment = useCounterStore((state) => state.increment);

  return <button onClick={increment}>{count} (Double: {doubleCount})</button>;
}
```

### 4.2 Vue: Pinia (Flux-Proxy Hybrid)
```typescript
import { defineStore } from 'pinia';
import { computed, ref } from 'vue';

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0);
  const doubleCount = computed(() => count.value * 2);
  function increment() {
    count.value++;
  }

  return { count, doubleCount, increment };
});
```

### 4.3 Angular: Signals (Signals Paradigm)
```typescript
import { Component, computed, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <button (click)="increment()">
      {{ count() }} (Double: {{ doubleCount() }})
    </button>
  `
})
export class CounterComponent {
  // Định nghĩa Signal
  count = signal(0);
  
  // Derived state tự động tracking
  doubleCount = computed(() => this.count() * 2);

  increment() {
    this.count.update(val => val + 1);
  }
}
```

---

> [!TIP]
> Để hiểu sâu hơn các cạm bẫy thiết kế và chuẩn bị tốt nhất cho phỏng vấn Senior, hãy chuyển sang tài liệu chuyên đề **[Q&A Phỏng Vấn State Management](./interviews.md)**.
