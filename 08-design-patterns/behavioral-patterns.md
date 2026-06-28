# 🧠 Behavioral Patterns (Mẫu Hành Vi)

> **Mẫu Hành Vi (Behavioral Patterns)** tập trung vào thuật toán và sự phân bổ trách nhiệm giữa các đối tượng. Chúng không chỉ mô tả các mẫu đối tượng hoặc lớp mà còn cả các mẫu giao tiếp giữa chúng.

---

## 📂 Các Pattern Trong Nhóm

1.  **[Strategy](#1-strategy)** — Thay đổi linh hoạt thuật toán ở runtime.
2.  **[Observer](#2-observer)** — Lắng nghe và tự động cập nhật khi có thay đổi.
3.  **[Command](#3-command)** — Đóng gói yêu cầu thành một đối tượng độc lập.
4.  **[Iterator](#4-iterator)** — Duyệt qua các phần tử của tập hợp mà không lộ cấu trúc ngầm.
5.  **[State](#5-state)** — Thay đổi hành vi theo trạng thái nội bộ.
6.  **[Template Method](#6-template-method)** — Định nghĩa khung thuật toán, để lớp con tùy biến các bước cụ thể.
7.  **[Mediator](#7-mediator)** — Điều phối giao tiếp tập trung giữa các class.
8.  **[Chain of Responsibility](#8-chain-of-responsibility)** — Chuỗi các bộ lọc xử lý tuần tự.
9.  **[Memento](#9-memento)** — Chụp lại và khôi phục trạng thái đối tượng.
10. **[Visitor](#10-visitor)** — Thêm thao tác mới vào một nhóm đối tượng mà không sửa class của chúng.
11. **[Interpreter](#11-interpreter)** — Xây dựng bộ phân tích cú pháp cho ngôn ngữ đơn giản.

---

## 1. Strategy (Chiến Lược)

### 💡 Định nghĩa dễ nhớ
> **Strategy** hoạt động giống như **Chọn phương thức di chuyển trên Google Maps**. Cùng một điểm đi và điểm đến, nhưng bạn có thể chọn các "Chiến lược" khác nhau: Đi bộ, Đi xe máy, Đi ô tô, hoặc Đi xe buýt. Hệ thống sẽ thay đổi thuật toán tính đường đi tương ứng.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Một class thực hiện một công việc nhưng có nhiều thuật toán khác nhau để chạy. Nếu viết tất cả vào 1 class sẽ tạo ra hàng tá câu lệnh `if-else` hoặc `switch-case` siêu dài và khó bảo trì.
*   **Giải pháp:** Tách các thuật toán này ra thành các class riêng biệt (các Strategy) có chung một interface, sau đó truyền Strategy mong muốn vào class sử dụng tại runtime.

### 💻 Code ví dụ bằng C#

```csharp
using System;

namespace DesignPatterns.Behavioral.Strategy;

// 1. Strategy Interface
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}

// 2. Concrete Strategies
public class CreditCardPayment : IPaymentStrategy
{
    public void Pay(decimal amount) => Console.WriteLine($"💳 Đã thanh toán {amount:C} qua Thẻ Tín Dụng.");
}

// C# 9+ cho phép viết class siêu ngắn (Record hoặc Expression-bodied members)
public class PaypalPayment : IPaymentStrategy
{
    public void Pay(decimal amount) => Console.WriteLine($"🅿️ Đã thanh toán {amount:C} qua PayPal.");
}

// 3. Context (Đối tượng sử dụng Strategy)
public class ShoppingCart
{
    private IPaymentStrategy _paymentStrategy;

    // Cho phép client tiêm (inject) chiến lược thanh toán lúc runtime
    public void SetPaymentStrategy(IPaymentStrategy strategy)
    {
        _paymentStrategy = strategy;
    }

    public void Checkout(decimal totalAmount)
    {
        if (_paymentStrategy == null)
        {
            throw new InvalidOperationException("Vui lòng chọn phương thức thanh toán trước!");
        }
        _paymentStrategy.Pay(totalAmount);
    }
}
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **LINQ `OrderBy` / `Sort`:** Hàm `List<T>.Sort(IComparer<T> comparer)` cho phép bạn truyền vào các chiến lược so sánh khác nhau thông qua interface `IComparer<T>`.

---

## 2. Observer (Người Quan Sát)

### 💡 Định nghĩa dễ nhớ
> **Observer** hoạt động giống như việc **Ấn nút Subscribe một kênh YouTube**. Khi kênh đó ra video mới, YouTube sẽ tự động bắn thông báo đến điện thoại của hàng triệu người đăng ký. Bạn không cần phải mở app F5 liên tục để kiểm tra.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Một đối tượng thay đổi trạng thái và cần thông báo cho nhiều đối tượng khác cập nhật theo, nhưng bạn không muốn các đối tượng này phụ thuộc cứng vào nhau (loose coupling).
*   **Giải pháp:** Tạo một danh sách "người đăng ký" (Observers) bên trong đối tượng chính (Subject). Khi Subject có biến động, nó sẽ duyệt danh sách này và gọi hàm cập nhật của từng Observer.

### 💻 Code ví dụ bằng C# (Sử dụng Event/Delegate chính thống của C#)

Trong C#, Observer được tích hợp trực tiếp vào ngôn ngữ thông qua từ khóa `event` và `delegate` cực kỳ mạnh mẽ mà không cần viết các interface `ISubject`/`IObserver` rườm rà:

```csharp
using System;

namespace DesignPatterns.Behavioral.Observer;

// Lớp chứa dữ liệu sự kiện
public class StockPriceChangedEventArgs : EventArgs
{
    public string Symbol { get; }
    public decimal Price { get; }

    public StockPriceChangedEventArgs(string symbol, decimal price)
    {
        Symbol = symbol;
        Price = price;
    }
}

// Subject (Đối tượng được quan sát)
public class StockTicker
{
    public string Symbol { get; }
    private decimal _price;

    public StockTicker(string symbol) => Symbol = symbol;

    // Định nghĩa Event sử dụng Action hoặc EventHandler
    public event EventHandler<StockPriceChangedEventArgs>? PriceChanged;

    public decimal Price
    {
        get => _price;
        set
        {
            if (_price != value)
            {
                _price = value;
                // Phát thông báo tới toàn bộ Observers đăng ký
                PriceChanged?.Invoke(this, new StockPriceChangedEventArgs(Symbol, _price));
            }
        }
    }
}

// Cách sử dụng (Client đăng ký sự kiện):
// var ticker = new StockTicker("MSFT");
// ticker.PriceChanged += (sender, args) => Console.WriteLine($"📈 Biến động giá: {args.Symbol} -> {args.Price:C}");
// ticker.Price = 250.50m; // Tự động in ra dòng chữ biến động giá
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **Rx.NET (Reactive Extensions):** Thư viện đỉnh cao áp dụng Observer Pattern để xử lý lập trình bất đồng bộ theo luồng dữ liệu (Data streams).
*   **`INotifyPropertyChanged` trong WPF / MAUI:** Sử dụng để Bind dữ liệu tự động giữa View và ViewModel.

---

## 3. Command (Mệnh Lệnh)

### 💡 Định nghĩa dễ nhớ
> **Command** giống như **Tờ Order món ăn tại nhà hàng**. Khách hàng ghi món vào tờ giấy (Command), bồi bàn chuyển tờ giấy đó cho đầu bếp (Receiver). Đầu bếp đọc tờ giấy và thực thi món ăn. Tờ giấy order có thể được lưu trữ, chuyển đi, hoặc xé bỏ (Undo) một cách độc lập.

### 🏛 Ví dụ Kinh điển trong Framework
*   **WPF `ICommand`:** Giao diện cơ bản để liên kết (Bind) hành động của các Button trong UI với các hàm xử lý logic trong ViewModel.

---

## 4. Iterator (Duyệt Phần Tử)

### 💡 Định nghĩa dễ nhớ
> **Iterator** giống như **Nút Next bài hát trên Spotify**. Bạn không cần quan tâm danh sách phát là một Array tĩnh, một Linked List, hay lấy từ Database trực tuyến. Bạn chỉ cần bấm "Next" để nghe bài tiếp theo.

### 🏛 Ví dụ Kinh điển trong Framework
*   **`IEnumerable<T>` và `IEnumerator<T>` trong C#:** Toàn bộ cú pháp vòng lặp `foreach` và các phương thức mở rộng LINQ (`Where`, `Select`) đều hoạt động dựa trên Iterator Pattern ngầm định.

---

## 5. State (Trạng Thái)

### 💡 Định nghĩa dễ nhớ
> **State** giống như **Nút Nguồn trên điện thoại**. Nếu điện thoại đang **Tắt màn hình**, bấm nút nguồn sẽ làm **Mở màn hình**. Nếu điện thoại đang **Mở màn hình**, bấm nút nguồn sẽ làm **Tắt màn hình**. Cùng 1 thao tác nhấn nút, nhưng hành vi thay đổi hoàn toàn tùy thuộc vào trạng thái hiện tại.

### 💻 Code ví dụ bằng C#

```csharp
using System;

namespace DesignPatterns.Behavioral.State;

// 1. State Interface
public interface IDocumentState
{
    void Review(DocumentContext context);
    void Publish(DocumentContext context);
}

// 2. Concrete States
public class DraftState : IDocumentState
{
    public void Review(DocumentContext context)
    {
        Console.WriteLine("🔍 Bản nháp đang được duyệt. Chuyển sang trạng thái Moderation.");
        context.SetState(new ModerationState());
    }

    public void Publish(DocumentContext context) => Console.WriteLine("❌ Lỗi: Không thể xuất bản trực tiếp bản nháp khi chưa duyệt!");
}

public class ModerationState : IDocumentState
{
    public void Review(DocumentContext context) => Console.WriteLine("⚠️ Tài liệu đã nằm trong hàng chờ duyệt rồi.");

    public void Publish(DocumentContext context)
    {
        Console.WriteLine("🎉 Tài liệu được duyệt thành công! Trạng thái: Published.");
        context.SetState(new PublishedState());
    }
}

public class PublishedState : IDocumentState
{
    public void Review(DocumentContext context) => Console.WriteLine("❌ Tài liệu đã xuất bản, không cần duyệt lại.");
    public void Publish(DocumentContext context) => Console.WriteLine("⚠️ Tài liệu đã được xuất bản công khai từ trước.");
}

// 3. Context (Đối tượng chứa Trạng thái)
public class DocumentContext
{
    private IDocumentState _state = new DraftState(); // Mặc định là bản nháp

    public void SetState(IDocumentState state) => _state = state;

    public void Review() => _state.Review(this);
    public void Publish() => _state.Publish(this);
}
```

---

## 6. Template Method (Khuôn Mẫu)

### 💡 Định nghĩa dễ nhớ
> **Template Method** giống như **Công thức làm mì gói**. Công thức gồm 3 bước cố định: 1. Đun sôi nước, 2. Bỏ mì và gia vị vào, 3. Chờ 3 phút. Bạn không thể đổi thứ tự này, nhưng bạn có thể tùy biến ở bước 2 bằng cách thêm trứng, thịt bò hoặc rau cải tùy ý.

### 🏛 Ví dụ Kinh điển trong Framework
*   **ASP.NET Core `BackgroundService`:** Class trừu tượng này cung cấp sẵn các cơ chế start/stop background job của .NET. Bạn chỉ cần kế thừa và override duy nhất một method chứa logic chạy ngầm của riêng bạn: `protected abstract Task ExecuteAsync(CancellationToken stoppingToken)`.

---

## 7. Mediator (Trung Gian)

### 💡 Định nghĩa dễ nhớ
> **Mediator** hoạt động y hệt **Tháp điều khiển không lưu tại sân bay**. Hàng chục máy bay (Colleagues) cất cánh và hạ cánh không liên lạc trực tiếp với nhau (vì sẽ gây hỗn loạn và tai nạn). Tất cả đều phải liên lạc qua Tháp điều khiển (Mediator), tháp này sẽ điều phối thứ tự hạ cánh an toàn.

### 🏛 Ví dụ Kinh điển trong Framework
*   **MediatR Library trong .NET:** Thư viện cực kỳ phổ biến trong các dự án Clean Architecture / CQRS giúp tách biệt hoàn toàn Request và Controller bằng cách gom toàn bộ giao tiếp qua một đối tượng trung gian là `IMediator`.

---

## 8. Chain of Responsibility (Chuỗi Trách Nhiệm)

### 💡 Định nghĩa dễ nhớ
> **Chain of Responsibility** giống như **Hệ thống tổng đài tự động**. Đầu tiên bạn gặp Máy trả lời tự động, nếu máy không giải quyết được sẽ chuyển máy tới Nhân viên tư vấn, nếu nhân viên không giải quyết được sẽ chuyển lên Trưởng bộ phận. Yêu cầu đi qua một chuỗi người xử lý cho đến khi có người giải quyết xong.

### 🏛 Ví dụ Kinh điển trong Framework
*   **ASP.NET Core Middleware Pipeline:** Khi một Http Request đi vào Web Server, nó sẽ đi qua một chuỗi các middleware (Authentication, Routing, Logging, Cors, Cache...) được cấu hình tuần tự. Mỗi middleware có quyền xử lý và quyết định có cho Request đi tiếp tới middleware tiếp theo (`next()`) hay ngắt chuỗi trả về lỗi luôn (short-circuit).

---

## 9. Memento (Vật Kỷ Niệm / Lưu Trạng Thái)

### 💡 Định nghĩa dễ nhớ
> **Memento** hoạt động giống như phím tắt **Ctrl + Z (Undo)** hoặc cơ chế **Save/Load game**. Nó chụp lại toàn bộ trạng thái của một đối tượng tại một thời điểm rồi lưu vào bộ nhớ, cho phép bạn khôi phục lại trạng thái cũ bất cứ lúc nào mà không vi phạm tính đóng gói.

---

## 10. Visitor (Khách Truy Cập)

### 💡 Định nghĩa dễ nhớ
> **Visitor** giống như **Bác sĩ đến khám bệnh tại nhà**. Cơ thể bạn (Elements) giữ nguyên, nhưng bác sĩ (Visitor) vào nhà để thực hiện các nghiệp vụ: Đo huyết áp, Xét nghiệm máu, Kê đơn thuốc mà không làm thay đổi cấu trúc cơ thể của bạn.

---

## 11. Interpreter (Thông Dịch)

### 💡 Định nghĩa dễ nhớ
> **Interpreter** giống như **Ứng dụng dịch ngôn ngữ trực tiếp**. Nó định nghĩa một bộ ngữ pháp cho một ngôn ngữ cụ thể và viết bộ thông dịch để phân tích, thực thi ngôn ngữ đó.

### 🏛 Ví dụ Kinh điển trong Framework
*   **LINQ Expressions (`Expression<Func<T, bool>>`):** Trình biên dịch C# sử dụng Interpreter Pattern để phân tích cú pháp biểu đồ Lambda Expression của bạn và dịch nó sang câu lệnh SQL tương ứng để chạy dưới database.
