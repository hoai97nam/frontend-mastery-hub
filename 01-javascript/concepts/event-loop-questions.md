# Câu Hỏi Phỏng Vấn Senior Frontend

> **Chủ đề:** Event Loop, Microtask vs. Macrotask & UI Performance

---

## Mục Lục

**Nhóm 1 — Tư duy nền tảng**
- [Câu 1: Promise chain dài và frame drop](#câu-1-promise-chain-dài-và-frame-drop)
- [Câu 2: setTimeout vs. queueMicrotask](#câu-2-settimeout-vs-queuemicrotask)

**Nhóm 2 — Debug & Phân tích thực tế**
- [Câu 3: Debug trang web bị giật khi scroll](#câu-3-debug-trang-web-bị-giật-khi-scroll)
- [Câu 4: Phát hiện vấn đề trong scroll handler](#câu-4-phát-hiện-vấn-đề-trong-scroll-handler)

**Nhóm 3 — Tối ưu nâng cao**
- [Câu 5: Render 100,000 items không làm đơ trình duyệt](#câu-5-render-100000-items-không-làm-đơ-trình-duyệt)
- [Câu 6: React Batching & Microtask Queue](#câu-6-react-batching--microtask-queue)
- [Câu 7: Khi nào không nên dùng requestAnimationFrame?](#câu-7-khi-nào-không-nên-dùng-requestanimationframe)

**Nhóm 4 — Câu hỏi bẫy (Trap Questions)**
- [Câu 8: Đoán output — nextTick, Promise, setTimeout](#câu-8-đoán-output--nexttick-promise-settimeout)
- [Câu 9: Vì sao code này crash tab trình duyệt?](#câu-9-vì-sao-code-này-crash-tab-trình-duyệt)

**[Thang đánh giá](#thang-đánh-giá)**

---

## Nhóm 1: Tư duy nền tảng

### Câu 1: Promise chain dài và frame drop

> Giải thích tại sao một Promise chain dài (`.then().then().then()...`) có thể gây ra **frame drop** dù mỗi `.then()` chỉ làm việc rất nhẹ?

**🎯 Điểm cần nghe ở senior:**
- Hiểu rằng microtask flush hết trước khi Render chạy
- Một chuỗi microtask đủ dài có thể vượt 16.6ms → bỏ lỡ frame
- Biết cách đo bằng **Performance tab** trong DevTools

---

### Câu 2: setTimeout vs. queueMicrotask

> `setTimeout(fn, 0)` và `queueMicrotask(fn)` đều defer việc thực thi — vậy khi nào bạn chọn cái nào? Cho ví dụ use case cụ thể.

**🎯 Điểm cần nghe ở senior:**
- `queueMicrotask` — khi cần chạy **trước render** (cập nhật state nội bộ, batching)
- `setTimeout` — khi muốn **nhường render trước** rồi mới xử lý tiếp
- Hiểu rằng `setTimeout` có minimum delay ~4ms và không đảm bảo chính xác

---

## Nhóm 2: Debug & Phân tích thực tế

### Câu 3: Debug trang web bị giật khi scroll

> Bạn nhận được bug report: *"Trang web bị giật khi user scroll nhanh, dù không có animation nào"*. Quy trình debug của bạn là gì? Bạn nghi ngờ Event Loop liên quan như thế nào?

**🎯 Điểm cần nghe ở senior:**
- Dùng Performance tab → tìm **Long Tasks** (>50ms)
- Nghi ngờ scroll event listener gọi heavy sync code hoặc layout thrashing
- Biết đến `passive: true` trong `addEventListener` để không block scroll
- Biết debounce/throttle nhưng hiểu **sự khác biệt** giữa hai cái

---

### Câu 4: Phát hiện vấn đề trong scroll handler

> Đoạn code sau có vấn đề gì về performance?

```javascript
document.addEventListener('scroll', () => {
  const height = document.body.scrollHeight;
  const elements = document.querySelectorAll('.item');
  elements.forEach(el => {
    el.style.transform = `translateY(${window.scrollY}px)`;
  });
});
```

**🎯 Điểm cần nghe ở senior:**
- `scrollHeight` + `querySelectorAll` trong scroll handler → **forced synchronous layout** (layout thrashing)
- Mỗi scroll event đọc layout rồi ghi style → trình duyệt phải recalculate liên tục
- **Fix:** dùng `requestAnimationFrame` để batch reads/writes, cache giá trị ra ngoài handler
- **Bonus:** dùng `passive: true`, hoặc CSS `will-change: transform`

---

## Nhóm 3: Tối ưu nâng cao

### Câu 5: Render 100,000 items không làm đơ trình duyệt

> Bạn cần xử lý một danh sách 100,000 items — render ra DOM mà không làm đơ trình duyệt. Bạn sẽ thiết kế giải pháp như thế nào từ góc độ Event Loop?

**🎯 Điểm cần nghe ở senior:**
- Chia nhỏ công việc thành chunks, dùng `requestAnimationFrame` hoặc `scheduler.postTask`
- Biết đến **Virtual Scrolling / Windowing** (chỉ render items đang visible)
- Nhắc đến `requestIdleCallback` cho công việc không urgent
- Hiểu trade-off giữa chunk size và số frame bị "nặng"

---

### Câu 6: React Batching & Microtask Queue

> React batch state updates như thế nào liên quan đến Microtask Queue? Sự khác biệt giữa React 17 và React 18 trong vấn đề này là gì?

**🎯 Điểm cần nghe ở senior:**
- **React 17:** chỉ batch updates trong event handlers gốc (synthetic events)
- **React 18:** Automatic Batching — batch tất cả updates kể cả trong `setTimeout`, Promise, native events
- React 18 dùng `queueMicrotask` (hoặc `MessageChannel`) để schedule re-render
- Biết khi nào cần `flushSync` để opt-out khỏi batching

---

### Câu 7: Khi nào không nên dùng requestAnimationFrame?

> `requestAnimationFrame` có phải lúc nào cũng là lựa chọn tốt nhất để tối ưu animation không? Khi nào thì **không nên** dùng nó?

**🎯 Điểm cần nghe ở senior:**
- rAF tốt cho animation liên quan đến DOM/canvas
- Không nên dùng cho logic không liên quan đến visual (lãng phí, misleading)
- Với animation đơn giản: CSS `transition`/`animation` thường tốt hơn (chạy trên compositor thread, không block JS)
- `requestIdleCallback` phù hợp hơn cho background tasks
- rAF bị **throttle** khi tab ẩn (inactive tab)

---

## Nhóm 4: Câu hỏi bẫy (Trap Questions)

### Câu 8: Đoán output — nextTick, Promise, setTimeout

> Đoạn code này output gì? Giải thích tại sao.

```javascript
console.log('A');

setTimeout(() => console.log('B'), 0);

Promise.resolve()
  .then(() => {
    console.log('C');
    process.nextTick(() => console.log('D'));
  });

process.nextTick(() => console.log('E'));

console.log('F');
```

**✅ Đáp án:**
```
A → F → E → C → D → B
```

**🎯 Điểm cần nghe ở senior:**
- Phân biệt được: sync → nextTick → Promise.then → setTimeout
- Hiểu rằng `process.nextTick` bên trong `.then()` vẫn chạy **trước** `.then()` tiếp theo
- Không bị bẫy bởi thứ tự khai báo

---

### Câu 9: Vì sao code này crash tab trình duyệt?

> Tại sao đoạn code này có thể **crash tab trình duyệt**?

```javascript
function infiniteMicrotask() {
  Promise.resolve().then(infiniteMicrotask);
}
infiniteMicrotask();
```

**🎯 Điểm cần nghe ở senior:**
- Microtask queue không bao giờ trống → Render không bao giờ chạy → **tab freeze**
- Khác với `while(true)`: không block call stack ngay lập tức nhưng kết quả tương đương
- Biết cách phát hiện qua Performance tab (microtask duration kéo dài vô tận)

---

## Thang đánh giá

| Mức độ | Dấu hiệu |
|:---|:---|
| 🟢 **Junior** | Trả lời đúng output, nhưng giải thích theo kiểu thuộc lòng |
| 🔵 **Mid** | Hiểu cơ chế, nhưng chưa liên kết được với vấn đề performance thực tế |
| 🟠 **Senior** | Tự đặt thêm điều kiện, nhắc đến trade-off, biết đo lường bằng DevTools |
| 🔴 **Staff+** | Đề xuất cải tiến kiến trúc, biết khi nào **không cần** tối ưu |