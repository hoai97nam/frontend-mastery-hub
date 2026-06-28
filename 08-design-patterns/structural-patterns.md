# 🧱 Structural Patterns (Mẫu Cấu Trúc)

> **Mẫu Cấu Trúc (Structural Patterns)** tập trung vào cách liên kết các đối tượng và lớp để tạo nên những cấu trúc lớn hơn, linh hoạt hơn nhưng vẫn đảm bảo hiệu năng và tính tái sử dụng cao.

---

## 📂 Các Pattern Trong Nhóm

1.  **[Adapter](#1-adapter)** — Bộ chuyển đổi giao diện không tương thích.
2.  **[Decorator](#2-decorator)** — Thêm tính năng động mà không kế thừa.
3.  **[Facade](#3-facade)** — Giao diện đơn giản hóa cho hệ thống phức tạp.
4.  **[Proxy](#4-proxy)** — Đại diện ủy quyền kiểm soát truy cập đối tượng.
5.  **[Composite](#5-composite)** — Tổ chức đối tượng theo cấu trúc hình cây.
6.  **[Bridge](#6-bridge)** — Tách biệt phần trừu tượng khỏi phần hiện thực.
7.  **[Flyweight](#7-flyweight)** — Chia sẻ bộ nhớ tối ưu hóa tài nguyên.

---

## 1. Adapter (Bộ Chuyển Đổi)

### 💡 Định nghĩa dễ nhớ
> **Adapter** hoạt động giống hệt **Củ sạc chuyển đổi 2 chấu thành 3 chấu** khi đi du lịch nước ngoài hoặc **Đầu chuyển đổi cổng Lightning sang Type-C**. Nó giúp hai thiết bị hoàn toàn khác chuẩn kết nối được với nhau.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Bạn có một class cũ đang chạy rất tốt (hoặc thư viện bên thứ ba tải về) nhưng interface của nó không khớp với interface mà hệ thống hiện tại yêu cầu.
*   **Giải pháp:** Tạo ra một class trung gian (Adapter) kế thừa interface hiện tại và bọc (wrap) class cũ bên trong để chuyển đổi lời gọi hàm.

### 💻 Code ví dụ bằng C#

```csharp
using System;

namespace DesignPatterns.Structural.Adapter;

// 1. Target Interface (Giao diện mà hệ thống hiện tại mong muốn)
public interface ITargetJsonParser
{
    void ParseJson(string jsonContent);
}

// 2. Adaptee (Class cũ hoặc thư viện bên thứ 3 chỉ hiểu XML)
public class OldXmlParser
{
    public void ParseXml(string xmlContent)
    {
        Console.WriteLine($"📄 Đang phân tích dữ liệu định dạng XML: {xmlContent}");
    }
}

// 3. Adapter (Bộ chuyển đổi)
public class JsonToXmlAdapter : ITargetJsonParser
{
    private readonly OldXmlParser _xmlParser;

    public JsonToXmlAdapter(OldXmlParser xmlParser)
    {
        _xmlParser = xmlParser;
    }

    public void ParseJson(string jsonContent)
    {
        // Chuyển đổi Json sang Xml (giả định logic chuyển đổi)
        string xmlEquivalent = $"<xml>{jsonContent.Replace("{", "").Replace("}", "")}</xml>";
        
        // Gọi hàm của class cũ thông qua tham chiếu bọc bên trong
        _xmlParser.ParseXml(xmlEquivalent);
    }
}

// Cách sử dụng:
// OldXmlParser legacyParser = new();
// ITargetJsonParser parser = new JsonToXmlAdapter(legacyParser);
// parser.ParseJson("{'name': 'C#'}");
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **ADO.NET DataAdapter:** `SqlDataAdapter` đóng vai trò là cầu nối/adapter giữa cơ sở dữ liệu vật lý và đối tượng lưu dữ liệu trong bộ nhớ `DataSet`.

---

## 2. Decorator (Bộ Trang Trí)

### 💡 Định nghĩa dễ nhớ
> **Decorator** giống như việc **Đeo ốp lưng, dán kính cường lực** cho điện thoại. Điện thoại của bạn vẫn nghe gọi bình thường, nhưng giờ có thêm tính năng chống va đập, chống xước mà cấu trúc linh kiện bên trong điện thoại hoàn toàn không thay đổi.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Cần thêm tính năng mới cho đối tượng (ví dụ: Log tham số, Cache kết quả, Mã hóa dữ liệu) nhưng không muốn sửa đổi code class gốc (vi phạm OCP) hoặc không thể dùng Kế thừa (vì gây bùng nổ số lượng class con).
*   **Giải pháp:** Bọc đối tượng gốc bằng một đối tượng "Decorator" có cùng interface, cho phép thực thi hành vi bổ sung trước hoặc sau khi gọi đối tượng gốc.

### 💻 Code ví dụ bằng C# (Caching Decorator)

```csharp
using System;
using System.Collections.Generic;

namespace DesignPatterns.Structural.Decorator;

// 1. Component Interface
public interface IWeatherService
{
    string GetTemperature(string city);
}

// 2. Concrete Component (Class nghiệp vụ gốc)
public class WeatherService : IWeatherService
{
    public string GetTemperature(string city)
    {
        Console.WriteLine($"🌐 Đang cào dữ liệu từ Internet cho thành phố: {city}...");
        return "28°C"; // Giả lập gọi API tốn tài nguyên
    }
}

// 3. Decorator (Trang trí thêm tính năng Caching mà không sửa Class gốc)
public class CachedWeatherService : IWeatherService
{
    private readonly IWeatherService _innerService;
    private readonly Dictionary<string, string> _cache = new();

    public CachedWeatherService(IWeatherService innerService)
    {
        _innerService = innerService;
    }

    public string GetTemperature(string city)
    {
        if (_cache.TryGetValue(city, out var temp))
        {
            Console.WriteLine($"💾 [CACHE HIT] Lấy từ bộ nhớ cache cho: {city}");
            return temp;
        }

        var realTemp = _innerService.GetTemperature(city);
        _cache[city] = realTemp;
        return realTemp;
    }
}

// Cách sử dụng:
// IWeatherService realService = new WeatherService();
// IWeatherService cachedService = new CachedWeatherService(realService);
// cachedService.GetTemperature("Hanoi"); // Lần 1: Gọi API internet
// cachedService.GetTemperature("Hanoi"); // Lần 2: Lấy ngay từ Cache
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **`System.IO.Stream` trong .NET:** `BufferedStream`, `GZipStream`, `CryptoStream` đều kế thừa từ `Stream` và nhận vào một `Stream` khác ở Constructor. Chúng xếp chồng các tính năng (đệm, nén, mã hóa) lên luồng đọc ghi dữ liệu gốc một cách hoàn hảo.

---

## 3. Facade (Bề Ngoài)

### 💡 Định nghĩa dễ nhớ
> **Facade** giống như **Nút "Mua Ngay" trên trang Shopee/Lazada**. Chỉ bằng 1 cú click chuột, hệ thống phía sau sẽ tự động thực hiện 5-6 bước phức tạp: Kiểm tra kho, Trừ tiền tài khoản, Tạo vận đơn, Gửi SMS thông báo, và Báo bên shipper. Bạn không cần làm từng bước thủ công.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Hệ thống con (subsystem) của bạn quá phức tạp, gồm rất nhiều class phụ thuộc chéo lẫn nhau. Người dùng hoặc các lớp client bên ngoài rất khó tiếp cận nếu phải tự gọi từng class.
*   **Giải pháp:** Tạo ra một class đơn giản duy nhất (Facade) cung cấp các phương thức dễ dùng, gom toàn bộ các tương tác phức tạp bên trong lại.

### 💻 Code ví dụ bằng C#

```csharp
using System;

namespace DesignPatterns.Structural.Facade;

// Các thành phần phức tạp của hệ thống con (Subsystem Classes)
public class Inventory { public bool CheckStock(string item) => true; }
public class PaymentGateway { public bool Charge(decimal amount) => true; }
public class ShippingService { public void Ship(string item) => Console.WriteLine($"🚚 Đang vận chuyển {item}..."); }

// Facade (Giao diện đơn giản hóa)
public class OrderFacade
{
    private readonly Inventory _inventory = new();
    private readonly PaymentGateway _payment = new();
    private readonly ShippingService _shipping = new();

    public bool PlaceOrder(string item, decimal price)
    {
        Console.WriteLine("🛒 Bắt đầu xử lý đơn hàng...");
        if (!_inventory.CheckStock(item)) return false;
        if (!_payment.Charge(price)) return false;
        
        _shipping.Ship(item);
        Console.WriteLine("🎉 Đơn hàng hoàn tất!");
        return true;
    }
}
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **`HttpClient` (.NET):** Nó che giấu toàn bộ cấu trúc mạng siêu phức tạp (TCP sockets, DNS resolution, SSL handshakes, HTTP headers, Keep-alive connections) đằng sau các hàm cực kỳ ngắn gọn như `GetStringAsync(url)`.

---

## 4. Proxy (Ủy Quyền)

### 💡 Định nghĩa dễ nhớ
> **Proxy** giống như **Thư ký riêng của Giám đốc**. Bất kỳ ai muốn gặp hay gửi tài liệu cho Giám đốc (Real Subject) đều phải đi qua Thư ký (Proxy). Thư ký sẽ lọc xem ai đủ quyền hạn, kiểm tra lịch hẹn, hoặc trả lời trước những câu hỏi cơ bản để tránh làm phiền Giám đốc.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Đối tượng thực thi rất nặng hoặc nhạy cảm, bạn muốn kiểm soát quyền truy cập, ghi log hoạt động, hoặc trì hoãn việc khởi tạo nó cho tới khi thực sự cần thiết (Lazy Loading).
*   **Giải pháp:** Tạo một đối tượng trung gian (Proxy) có cùng interface với đối tượng gốc. Client sẽ tương tác với Proxy, Proxy kiểm tra điều kiện rồi mới chuyển tiếp yêu cầu đến đối tượng gốc.

### 💻 Code ví dụ bằng C# (Lazy Loading & Access Control Proxy)

```csharp
using System;

namespace DesignPatterns.Structural.Proxy;

public interface IReport
{
    void Display();
}

// Đối tượng thật cực kỳ nặng (Real Subject)
public class GiantReport : IReport
{
    public GiantReport()
    {
        Console.WriteLine("💾 Đang tải dữ liệu báo cáo 5GB từ ổ đĩa cứng (Rất chậm)...");
    }

    public void Display() => Console.WriteLine("📊 Hiển thị biểu đồ báo cáo tài chính năm.");
}

// Proxy đại diện
public class ReportProxy : IReport
{
    private GiantReport? _realReport;
    private readonly string _userRole;

    public ReportProxy(string userRole)
    {
        _userRole = userRole;
    }

    public void Display()
    {
        // 1. Kiểm soát quyền truy cập (Access Control)
        if (_userRole != "Admin")
        {
            Console.WriteLine("❌ Lỗi: Bạn không có quyền xem báo cáo này!");
            return;
        }

        // 2. Khởi tạo trễ (Lazy Loading): Chỉ new khi thực sự cần gọi
        _realReport ??= new GiantReport();
        _realReport.Display();
    }
}

// Cách sử dụng:
// IReport report = new ReportProxy("Staff");
// report.Display(); // Bị từ chối quyền
// IReport reportAdmin = new ReportProxy("Admin");
// reportAdmin.Display(); // Tải file và hiển thị thành công
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **Entity Framework Core Lazy Loading Proxies:** Khi bạn dùng virtual navigation properties, EF Core tự động tạo các Proxy Class lúc runtime kế thừa từ Class của bạn để tự động truy vấn thêm CSDL khi bạn gọi thuộc tính đó.

---

## 5. Composite (Hỗn Hợp / Cây)

### 💡 Định nghĩa dễ nhớ
> **Composite** giống như cấu trúc **Thư mục trên máy tính**. Một Thư mục chứa các File đơn lẻ, và cũng có thể chứa các Thư mục con khác. Nhưng khi bạn xem thuộc tính (Properties), hệ thống sẽ tính tổng dung lượng của toàn bộ cấu trúc cây y hệt nhau dù đó là File hay Thư mục.

### 🏛 Ví dụ Kinh điển trong Framework
*   **WPF / Windows Forms Controls:** Mọi control (như `Button`, `TextBox`) đều kế thừa từ `UIElement`. Class `Panel` hoặc `Grid` cũng kế thừa từ `UIElement` nhưng chứa một danh sách `Children` (các `UIElement` khác). Khi gọi hàm `Render()` của `Panel`, nó sẽ duyệt và vẽ toàn bộ các con của nó.

---

## 6. Bridge (Cầu Nối)

### 💡 Định nghĩa dễ nhớ
> **Bridge** giống như **Tay cầm điều khiển game độc lập (Xbox Controller)** và **Thiết bị chơi game (PC, PlayStation, Nintendo)**. Thay vì thiết kế một chiếc PC gắn cứng tay cầm PC, một máy PS gắn cứng tay cầm PS, ta tách biệt chúng ra bằng kết nối Bluetooth để tay cầm nào cũng chơi được trên máy nào.

### 🏛 Ví dụ Kinh điển trong Framework
*   **`Microsoft.Extensions.Logging`:** Hệ thống log tách biệt phần API ghi log (`ILogger`) và các bộ phát log ra ngoài (`ConsoleLoggerProvider`, `FileLoggerProvider`, `ApplicationInsightsLoggerProvider`).

---

## 7. Flyweight (Chia Sẻ Bộ Nhớ)

### 💡 Định nghĩa dễ nhớ
> **Flyweight** giống như việc **Thuê truyện/sách ở thư viện**. Thay vì mỗi người đọc tự bỏ tiền mua một cuốn sách giống hệt nhau về nhà xếp xó, thư viện chỉ mua 2-3 cuốn và cho hàng trăm người luân phiên mượn đọc để tiết kiệm tối đa diện tích và chi phí.

### 🏛 Ví dụ Kinh điển trong Framework
*   **String Interning trong .NET Runtime:** Khi bạn viết `string s1 = "hello"; string s2 = "hello";` trong C#, CLR không tạo ra hai vùng nhớ khác nhau cho cùng một chữ "hello". Nó lưu chuỗi này vào một bảng nội trú (Intern Pool) và cho cả hai biến `s1`, `s2` trỏ chung vào một tham chiếu bộ nhớ.
