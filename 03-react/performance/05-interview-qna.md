# 🎯 Interview Q&A — React Performance (Middle → Senior)

> 25+ câu hỏi tình huống thực chiến, được nhóm theo chủ đề.

---

## Mục Lục

1. [Rendering & Re-render](#1-rendering--re-render)
2. [Memoization](#2-memoization)
3. [State & Data Flow](#3-state--data-flow)
4. [Code Splitting & Loading](#4-code-splitting--loading)
5. [Network & Caching](#5-network--caching)
6. [Tình Huống Thực Chiến](#6-tình-huống-thực-chiến)
7. [Senior-level Deep Dive](#7-senior-level-deep-dive)

---

## 1. Rendering & Re-render

---

**Q1: React re-render component khi nào? Liệt kê đầy đủ các nguyên nhân.**

> **4 nguyên nhân:**
> 1. **State thay đổi** — `useState`, `useReducer` dispatch.
> 2. **Props thay đổi** — kể cả reference equality (object/array/function mới).
> 3. **Parent re-render** — con cái mặc định re-render theo parent.
> 4. **Context thay đổi** — mọi consumer của context đó re-render.
>
> **Bonus (Senior):** Trong React 18+, `useSyncExternalStore` re-render khi external store snapshot thay đổi.

---

**Q2: `React.memo` không ngăn được re-render — nguyên nhân có thể là gì?**

> Nguyên nhân thường gặp (theo thứ tự phổ biến):
> 1. **Props là object/array/function mới** mỗi render — shallow comparison thất bại.
>    - Fix: `useMemo` cho object/array, `useCallback` cho function.
> 2. **Context thay đổi** — `React.memo` không chặn được context re-render.
>    - Fix: Tách context, colocate consumer.
> 3. **Custom comparison sai** — hàm thứ 2 của `React.memo` return `false` không đúng.

---

**Q3: Giải thích "Children as Props" pattern và tại sao nó tránh được re-render.**

> ```jsx
> // Parent re-render → children KHÔNG re-render
> function Counter({ children }) {
>   const [count, setCount] = useState(0);
>   return <div><button onClick={() => setCount(c=>c+1)}>{count}</button>{children}</div>;
> }
>
> function App() {
>   return <Counter><ExpensiveTree /></Counter>; // ExpensiveTree tạo bởi App (không re-render)
> }
> ```
> **Lý do:** `<ExpensiveTree />` được React tạo ra bởi `App` — khi `Counter` re-render, nó chỉ nhận `children` từ props (reference không đổi vì `App` không re-render). React không tạo mới element, do đó skip render.

---

**Q4: `key` prop ảnh hưởng đến performance như thế nào?**

> - **`key` đúng (stable ID):** React reuse DOM node → không tạo mới, chỉ update props → nhanh.
> - **`key` sai (index):** Khi reorder/delete, React nghĩ component vẫn là cũ nhưng data đã khác → re-apply toàn bộ props, có thể gây bug state bên trong component.
> - **Thay đổi `key`:** Buộc React unmount + remount hoàn toàn → dùng như "reset trick" khi cần.

---

**Q5: Trong React 18 Strict Mode, `useEffect` chạy 2 lần trong development. Điều này ảnh hưởng performance production không?**

> **Không.** Strict Mode double-invocation chỉ xảy ra trong **development** để phát hiện side effects không có cleanup. Production build không bị ảnh hưởng.
>
> **Ý nghĩa thực tế:** Nếu code bị lỗi khi effect chạy 2 lần, nghĩa là cleanup function đang thiếu hoặc sai — cần fix.

---

## 2. Memoization

---

**Q6: Khi nào `useMemo` thực sự có lợi ích? Khi nào là over-engineering?**

> **CÓ LỢI ÍCH khi:**
> - Computation nặng: filter/sort 10k+ items, số học phức tạp.
> - Kết quả truyền xuống `React.memo` child (tránh re-render).
> - Kết quả dùng trong deps của `useEffect`/`useMemo` khác.
>
> **OVER-ENGINEERING khi:**
> - Computation đơn giản: `count * 2`, `items.length`.
> - Props không truyền xuống memo component.
> - Memoize mọi thứ "cho chắc" mà không profile.
>
> **Rule:** `useMemo` có chi phí: memory cho cache + bookkeeping mỗi render. Chỉ dùng khi đo được lợi ích thực sự.

---

**Q7: Tại sao `useCallback` không phải lúc nào cũng cần thiết?**

> `useCallback` chỉ hữu ích khi:
> 1. Function truyền xuống `React.memo` child.
> 2. Function là dependency trong `useEffect`/`useMemo`.
>
> Nếu function chỉ dùng trong cùng component (event handler không truyền đi) → `useCallback` chỉ thêm overhead, không có lợi ích gì.

---

**Q8 (Tình huống):** `React.memo` được thêm nhưng component vẫn re-render mỗi giây. Qua DevTools thấy lý do là "Context changed". Bạn xử lý thế nào?

> **Bước 1 — Xác định context nào trigger:**
> Dùng React DevTools → click component → xem "Context" section.
>
> **Bước 2 — Tách context:**
> ```jsx
> // Thay vì 1 context chứa tất cả
> <AppContext.Provider value={{ user, theme, cart, notifications }}>
>
> // Tách theo tần suất thay đổi
> <UserContext.Provider value={user}>        {/* Ít thay đổi */}
>   <ThemeContext.Provider value={theme}>   {/* Ít thay đổi */}
>     <CartContext.Provider value={cart}>   {/* Thay đổi thường */}
> ```
>
> **Bước 3 — Memoize value:**
> ```jsx
> const value = useMemo(() => ({ user, login, logout }), [user]);
> <UserContext.Provider value={value}>
> ```
>
> **Bước 4 — Cân nhắc Zustand** nếu context quá phức tạp.

---

## 3. State & Data Flow

---

**Q9: Giải thích tại sao "State Colocation" là kỹ thuật tối ưu performance quan trọng.**

> Nguyên tắc: **State nên sống gần nhất với nơi nó được dùng.**
>
> Khi state đặt quá cao trong tree, mỗi lần state thay đổi → tất cả component phía dưới (không cần state đó) đều re-render.
>
> ```
> App (state thay đổi) → re-render toàn bộ tree ❌
>
> ComponentA (state thay đổi) → chỉ ComponentA và con của nó re-render ✅
> ```
>
> **Thực tế:** Trước khi thêm `React.memo` hay `useMemo`, hãy hỏi: "State này có thể đặt thấp hơn không?"

---

**Q10: Normalized state là gì? Tại sao nó tốt cho performance?**

> Normalized state = lưu data theo dạng `{ byId: {}, allIds: [] }` giống database table.
>
> **Lợi ích:**
> - **Update O(1):** `state.products.byId[id] = newData` thay vì duyệt array.
> - **Ít re-render:** Thay đổi 1 product không tạo mới toàn bộ array.
> - **Selector đơn giản:** `state.products.byId[productId]` — không cần `find()`.
>
> **Trade-off:** Phức tạp hơn khi setup, cần join data khi render.

---

**Q11: Tại sao không nên lưu derived state vào `useState`?**

> 1. **Sync bug:** Dễ quên update derived state khi source thay đổi.
> 2. **Thừa re-render:** Cập nhật source + derived state → 2 lần setState → có thể 2 lần render (trước React 18 batching).
> 3. **Lãng phí memory:** Lưu data có thể tính lại được.
>
> **Fix:** Tính derived state trong render (hoặc `useMemo` nếu nặng).

---

**Q12 (Tình huống):** App dùng Zustand nhưng có component re-render liên tục dù chỉ cần đọc 1 field. Nguyên nhân và fix?

> **Nguyên nhân:** Subscribe toàn bộ store thay vì dùng selector.
> ```jsx
> // ❌ Re-render khi bất kỳ field nào thay đổi
> const store = useStore();
> const name = store.user.name;
>
> // ✅ Chỉ re-render khi user.name thay đổi
> const name = useStore(state => state.user.name);
>
> // ✅ Nhiều fields — dùng useShallow để so sánh nông
> import { useShallow } from 'zustand/react/shallow';
> const { name, email } = useStore(
>   useShallow(state => ({ name: state.user.name, email: state.user.email }))
> );
> ```

---

## 4. Code Splitting & Loading

---

**Q13: Route-based code splitting khác component-level splitting như thế nào? Khi nào dùng cái nào?**

> | | Route-based | Component-level |
> |---|------------|----------------|
> | Split theo | Trang (URL) | Component nặng |
> | Khi load | Navigate đến route | Component render lần đầu |
> | Ví dụ | `/admin`, `/checkout` | `<RichTextEditor>`, `<HeavyChart>` |
>
> **Route-based:** Default choice — giảm initial bundle rõ rệt.
> **Component-level:** Khi component dùng thư viện nặng (monaco-editor, D3, PDF library) và không phải lúc nào cũng hiển thị.

---

**Q14: `React.lazy` chỉ hỗ trợ default export. Làm thế nào để lazy load named export?**

> ```jsx
> // Wrap named export thành default export "giả"
> const UserModal = lazy(() =>
>   import('./Modals').then(module => ({ default: module.UserModal }))
> );
>
> // Hoặc tạo file riêng re-export default
> // UserModal.js: export { UserModal as default } from './Modals';
> const UserModal = lazy(() => import('./UserModal'));
> ```

---

**Q15 (Tình huống):** User báo cáo trang admin load chậm (8 giây). Bundle là 3MB. Quy trình debug và fix?

> **Bước 1 — Analyze:**
> ```bash
> npm run build
> npx source-map-explorer 'build/static/js/*.js'
> # Hoặc: rollup-plugin-visualizer với Vite
> ```
>
> **Bước 2 — Tìm nguyên nhân trong report:**
> - Thư viện lớn không cần cho initial load (chart library, PDF, rich editor).
> - Module import sai (lodash thay vì lodash-es).
> - Chunk admin chưa được split.
>
> **Bước 3 — Fix:**
> ```jsx
> // Split route admin
> const AdminPage = lazy(() => import('./pages/AdminPage'));
>
> // Lazy load chart chỉ khi cần
> const Chart = lazy(() => import('./components/Chart'));
>
> // Import đúng cách
> import { debounce } from 'lodash-es'; // Thay vì import _ from 'lodash'
> ```
>
> **Kết quả kỳ vọng:** Initial bundle < 500KB, TTI < 2 giây.

---

## 5. Network & Caching

---

**Q16: `staleTime` và `gcTime` trong React Query khác nhau như thế nào?**

> - **`staleTime`:** Thời gian data được coi là "fresh" — trong thời gian này, React Query **không refetch** dù component re-mount hoặc window focus.
> - **`gcTime` (cacheTime):** Thời gian cache được giữ **sau khi tất cả subscribers unmount**. Khi hết thời gian này, cache bị xóa → lần fetch sau sẽ thấy loading.
>
> ```
> staleTime = 5 phút: Trong 5 phút, dùng data cũ không hỏi server
> gcTime = 10 phút:  Sau unmount 10 phút, xóa cache khỏi memory
> ```

---

**Q17: Request waterfall là gì? Làm thế nào để detect và fix?**

> **Waterfall:** Các request chạy tuần tự — request sau chờ request trước xong mới bắt đầu. Thường xảy ra khi dùng nhiều `useEffect` lồng nhau hoặc component cần data từ component cha.
>
> **Detect:** Chrome DevTools → Network tab → Waterfall column. Nếu thấy các request không overlap → waterfall.
>
> **Fix:**
> ```jsx
> // ✅ Promise.all cho parallel fetch
> const [user, posts, followers] = await Promise.all([
>   fetchUser(id), fetchPosts(id), fetchFollowers(id)
> ]);
>
> // ✅ React Query useQueries
> const results = useQueries({ queries: [q1, q2, q3] });
>
> // ✅ Server-side: fetch all data trong Server Component
> ```

---

**Q18: Optimistic UI là gì? Khi nào nên và không nên dùng?**

> **Optimistic UI:** Cập nhật UI **trước** khi server confirm, assume API sẽ thành công. Nếu thất bại → rollback.
>
> **Nên dùng khi:**
> - Thao tác thường xuyên: like, follow, reorder.
> - API có success rate cao (> 95%).
> - Lỗi rollback không gây confusion.
>
> **Không nên dùng khi:**
> - Thanh toán, đặt hàng — user cần xác nhận thực sự.
> - Thao tác có side effects nghiêm trọng không rollback được.
> - API có thể conflict (race condition với dữ liệu realtime).

---

**Q19 (Tình huống):** Sau mỗi lần user submit form, list data hiển thị vẫn là data cũ dù server đã update. Fix thế nào?

> **Nguyên nhân:** Cache React Query chưa bị invalidate sau mutation.
>
> **Fix:**
> ```jsx
> const mutation = useMutation({
>   mutationFn: updateData,
>   onSuccess: () => {
>     // Option 1: Invalidate → trigger refetch
>     queryClient.invalidateQueries({ queryKey: ['items'] });
>
>     // Option 2: Update cache trực tiếp (không cần refetch)
>     queryClient.setQueryData(['items'], (old) =>
>       old.map(item => item.id === updatedId ? newData : item)
>     );
>   }
> });
> ```

---

## 6. Tình Huống Thực Chiến

---

**Q20 (Tình huống):** Dashboard với 50+ widget, mỗi widget fetch data riêng. Trang load 12 giây, user phàn nàn. Bạn tiếp cận thế nào?

> **Phase 1 — Measure (không đoán mò):**
> - Lighthouse audit → xác định metric nào kém (LCP, TBT, FID).
> - Network waterfall → có 50 sequential requests không?
> - React Profiler → widget nào render lâu nhất?
>
> **Phase 2 — Fix theo impact:**
> 1. **Code splitting:** Lazy load các widget không visible ban đầu.
> 2. **Parallel fetch:** Thay 50 request tuần tự bằng batch API hoặc `useQueries`.
> 3. **Prioritize above-the-fold:** Load 5 widget đầu tiên trước, lazy load phần còn lại.
> 4. **Virtualization:** Nếu user scroll qua nhiều widget, chỉ render widget trong viewport.
> 5. **Skeleton UI:** Cải thiện perceived performance trong khi data load.
>
> **Phase 3 — Monitor:** Set up performance budget, alert khi regression.

---

**Q21 (Tình huống):** Có một `<DataTable>` với 5,000 rows. Filter theo search query bị lag rõ rệt khi typing. Giải pháp?

> **Giải pháp kết hợp:**
>
> ```jsx
> function DataTable({ data }) {
>   const [query, setQuery] = useState('');
>   const [isPending, startTransition] = useTransition();
>   const deferredQuery = useDeferredValue(query);
>
>   // 1. useMemo: Chỉ filter lại khi deferredQuery thay đổi
>   const filtered = useMemo(() =>
>     data.filter(row => row.name.toLowerCase().includes(deferredQuery.toLowerCase())),
>     [data, deferredQuery]
>   );
>
>   return (
>     <>
>       {/* Input cập nhật ngay lập tức */}
>       <input value={query} onChange={e => setQuery(e.target.value)} />
>
>       {/* 2. Virtualization: chỉ render rows visible */}
>       <VirtualList items={filtered} itemHeight={40} />
>
>       {/* 3. Visual feedback khi đang filter */}
>       {isPending && <span>Đang lọc...</span>}
>     </>
>   );
> }
> ```
>
> **Kết quả:** Input responsive ngay lập tức, filter chạy nền, list không render 5000 DOM nodes.

---

**Q22 (Tình huống):** App React có memory leak — dùng lâu browser tab bị chậm dần, memory tăng liên tục. Debug như thế nào?

> **Bước 1 — Confirm memory leak:**
> Chrome DevTools → Memory → Take Heap Snapshot → Thực hiện action → Take snapshot lại → So sánh.
>
> **Bước 2 — Tìm nguyên nhân phổ biến:**
> ```jsx
> // ❌ Event listener không cleanup
> useEffect(() => {
>   window.addEventListener('resize', handler);
>   // Quên return cleanup!
> }, []);
>
> // ❌ setInterval không clear
> useEffect(() => {
>   const id = setInterval(tick, 1000);
>   // Quên clearInterval!
> }, []);
>
> // ❌ setState sau unmount
> useEffect(() => {
>   fetchData().then(data => setState(data)); // Component có thể đã unmount!
> }, []);
>
> // ❌ WebSocket không close
> useEffect(() => {
>   const ws = new WebSocket(url);
>   // Quên ws.close()!
> }, []);
> ```
>
> **Bước 3 — Fix:** Luôn return cleanup function trong `useEffect`.
>
> **Bước 4 — Verify:** Memory snapshot trước và sau fix — heap không tăng sau fix.

---

**Q23 (Tình huống):** Product yêu cầu thêm "Like" button — phải instant, không thể có loading state. Implement thế nào?

> **Optimistic Update với rollback:**
> ```jsx
> function LikeButton({ postId, initialLiked, initialCount }) {
>   const [liked, setLiked] = useState(initialLiked);
>   const [count, setCount] = useState(initialCount);
>
>   const handleLike = async () => {
>     // Snapshot để rollback
>     const prevLiked = liked;
>     const prevCount = count;
>
>     // Cập nhật UI NGAY (optimistic)
>     setLiked(!liked);
>     setCount(c => liked ? c - 1 : c + 1);
>
>     try {
>       await api.toggleLike(postId);
>     } catch {
>       // Rollback nếu API fail
>       setLiked(prevLiked);
>       setCount(prevCount);
>       toast.error('Không thể thực hiện. Thử lại sau.');
>     }
>   };
>
>   return (
>     <button onClick={handleLike}>
>       {liked ? '❤️' : '🤍'} {count}
>     </button>
>   );
> }
> ```

---

## 7. Senior-level Deep Dive

---

**Q24: Giải thích React Fiber và tại sao nó quan trọng cho performance.**

> **React Fiber** (React 16+) là rewrite của reconciliation algorithm, cho phép:
>
> 1. **Incremental rendering:** Chia nhỏ render work thành đơn vị nhỏ (fibers), có thể pause/resume.
> 2. **Priority scheduling:** Assign độ ưu tiên khác nhau cho updates (urgent vs non-urgent).
> 3. **Concurrency (React 18):** Nhiều updates có thể được chuẩn bị song song.
>
> **Trước Fiber (Stack Reconciler):** Render là synchronous, không thể interrupt → UI freeze khi render nặng.
>
> **Sau Fiber:** React có thể interrupt low-priority render để xử lý urgent updates (user input) trước → `useTransition`, `useDeferredValue` khả thi.

---

**Q25: `useSyncExternalStore` dùng để làm gì? Khi nào cần dùng thay vì `useEffect` + `useState`?**

> `useSyncExternalStore` là hook để subscribe vào external store (bên ngoài React state) một cách an toàn với Concurrent Mode.
>
> **Vấn đề với `useEffect + useState`:**
> ```jsx
> // ❌ Có thể gây "tearing" — UI inconsistent trong Concurrent Mode
> function useWindowSize() {
>   const [size, setSize] = useState(getWindowSize());
>   useEffect(() => {
>     const handler = () => setSize(getWindowSize());
>     window.addEventListener('resize', handler);
>     return () => window.removeEventListener('resize', handler);
>   }, []);
>   return size;
> }
> ```
>
> **Fix với `useSyncExternalStore`:**
> ```jsx
> // ✅ Concurrent Mode safe
> function useWindowSize() {
>   return useSyncExternalStore(
>     (callback) => {
>       window.addEventListener('resize', callback);
>       return () => window.removeEventListener('resize', callback);
>     },
>     () => ({ width: window.innerWidth, height: window.innerHeight }),
>     () => ({ width: 0, height: 0 }), // SSR fallback
>   );
> }
> ```
>
> **Khi dùng:** Viết custom hooks subscribe vào browser APIs (scroll, resize, online status), Redux store, hoặc custom event emitters.

---

**Q26: Core Web Vitals là gì? React ảnh hưởng đến chúng như thế nào?**

> | Metric | Đo lường | React Impact |
> |--------|----------|-------------|
> | **LCP** (Largest Contentful Paint) | Thời gian render phần tử lớn nhất | Code splitting, SSR, image optimization |
> | **INP** (Interaction to Next Paint) | Responsive khi user interact | `useTransition`, tránh long tasks |
> | **CLS** (Cumulative Layout Shift) | Độ dịch chuyển layout | Skeleton UI, image dimensions, `Suspense` fallback |
>
> **Practical tips:**
> - **LCP:** SSR/SSG với Next.js, `priority` image, preload fonts.
> - **INP:** `startTransition` cho non-urgent updates, code split để giảm JS parse time.
> - **CLS:** Luôn set `width`/`height` cho images, skeleton giữ đúng layout.

---

**Q27 (Senior Tình Huống):** Team đang migrate một legacy class component lớn (500 lines) sang functional component. Performance sau migrate tệ hơn. Bạn điều tra thế nào?

> **Checklist điều tra:**
>
> 1. **`shouldComponentUpdate` / `PureComponent` bị mất:**
>    Class component có thể đã có `shouldComponentUpdate` custom. Functional component cần `React.memo` + custom comparison.
>
> 2. **Instance variables bị thay bằng state:**
>    ```jsx
>    // Class: this.timerId không trigger re-render
>    // Function: useState(timerId) sẽ trigger re-render!
>    // Fix: useRef(null) cho mutable values không cần re-render
>    ```
>
> 3. **Event handler mất reference stability:**
>    Class methods (`this.handleClick`) luôn stable reference. Inline arrow functions trong functional component tạo reference mới mỗi render → cần `useCallback`.
>
> 4. **State update không được batch đúng cách:**
>    Kiểm tra nếu có nhiều `setState` calls trong async context — React 18 auto-batching fix vấn đề này nhưng cần verify.
>
> **Quy trình:** Profile trước/sau migration → So sánh render count và duration → Identify regression → Fix từng nguyên nhân.

---

## 📎 Tổng Hợp — "Red Flags" Trong Code Review

```
🔴 CRITICAL:
  - useEffect không có cleanup (timer, listener, subscription)
  - setState sau unmount (không có AbortController/cancel flag)
  - key={index} trong list có thể reorder/delete

🟡 WARNING:
  - Inline object/array/function trong JSX truyền xuống memo component
  - Context value không được memoize
  - State lưu derived values có thể tính từ source
  - Fetch sequential thay vì parallel (waterfall)

🟢 NICE TO HAVE:
  - useMemo cho expensive computation
  - Code splitting cho heavy routes
  - Virtualization cho list > 500 items
  - Prefetch khi hover
```

---

🔙 [Network & Data Fetching](./04-network-data.md) | 🔙 [README](./README.md)
