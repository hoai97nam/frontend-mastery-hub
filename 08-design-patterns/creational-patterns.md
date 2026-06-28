# 🏗 Creational Patterns (Mẫu Khởi Tạo)

> **Mẫu Khởi Tạo (Creational Patterns)** tập trung vào cơ chế khởi tạo đối tượng. Chúng giúp hệ thống độc lập với cách các đối tượng của nó được tạo ra, sắp đặt và đại diện.

---

## 📂 Các Pattern Trong Nhóm

1.  **[Singleton](#1-singleton)** — Duy nhất một instance.
2.  **[Factory Method](#2-factory-method)** — Để lớp con quyết định loại đối tượng được tạo.
3.  **[Abstract Factory](#3-abstract-factory)** — Tạo ra một gia đình các đối tượng liên quan.
4.  **[Builder](#4-builder)** — Kiến tạo đối tượng phức tạp từng bước một.
5.  **[Prototype](#5-prototype)** — Nhân bản đối tượng từ một mẫu có sẵn.

---

## 1. Singleton (Độc Bản)

### 💡 Định nghĩa dễ nhớ
> **Singleton** giống như **Chính phủ của một quốc gia**. Chỉ có duy nhất một chính phủ nắm quyền tại một thời điểm, và mọi người dân đều truy cập vào cùng một chính phủ để yêu cầu xử lý công việc.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Có những tài nguyên cực kỳ đắt đỏ hoặc cần sự đồng bộ duy nhất trên toàn hệ thống (ví dụ: bộ quản lý cấu hình, kết nối cơ sở dữ liệu, logging service). Nếu tạo quá nhiều instance sẽ gây lãng phí RAM và xung đột dữ liệu.
*   **Giải pháp:** Đảm bảo class chỉ có duy nhất một instance toàn cục và cung cấp một điểm truy cập duy nhất đến instance đó.

### 💻 Code ví dụ bằng C# (Modern C#)

Cách triển khai tối ưu và an toàn nhất trong môi trường đa luồng (Thread-safe) sử dụng `Lazy<T>` của C#:

```csharp
using System;

namespace DesignPatterns.Creational.Singleton;

public sealed class DatabaseConnection
{
    // Sử dụng Lazy<T> giúp trì hoãn việc khởi tạo (Lazy Initialization) 
    // và tự động đảm bảo Thread-Safety tuyệt đối từ .NET runtime.
    private static readonly Lazy<DatabaseConnection> _instance = 
        new Lazy<DatabaseConnection>(() => new DatabaseConnection());

    // 1. Private Constructor ngăn cản việc khởi tạo từ bên ngoài bằng từ khóa 'new'
    private DatabaseConnection()
    {
        Console.WriteLine("🔌 Kết nối Database được khởi tạo (Chỉ chạy 1 lần duy nhất)!");
    }

    // 2. Điểm truy cập toàn cục duy nhất
    public static DatabaseConnection Instance => _instance.Value;

    public void Query(string sql)
    {
        Console.WriteLine($"⚡ Đang thực thi truy vấn: {sql}");
    }
}

// Cách sử dụng:
// var db1 = DatabaseConnection.Instance;
// var db2 = DatabaseConnection.Instance;
// Console.WriteLine(ReferenceEquals(db1, db2)); // True (Cả hai biến trỏ tới cùng 1 vùng nhớ)
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **ASP.NET Core Dependency Injection:** Khi bạn đăng ký một service dưới dạng `builder.Services.AddSingleton<IMyService, MyService>()`. ASP.NET Core DI Container sẽ quản lý vòng đời của service này và trả về đúng một instance duy nhất cho mọi request từ client.
*   **`System.Console` hoặc `System.GC`** trong .NET Class Library.

### 👁 Kỹ thuật ngầm ít người nhận ra
Nhiều lập trình viên lạm dụng Singleton và biến nó thành một **Anti-pattern** (mẫu phản tác dụng) vì nó tạo ra **Global State** (trạng thái toàn cục) ẩn. Điều này làm cho việc viết Unit Test trở nên cực kỳ khó khăn do các test case bị ảnh hưởng chéo bởi trạng thái lưu trong Singleton.
*   **Giải pháp ẩn:** Đừng viết Singleton chay kiểu cổ điển (`Classic Singleton`). Hãy sử dụng **Dependency Injection (DI)** để đăng ký class bình thường dưới dạng **Singleton Lifetime**. Class vẫn là class thường (dễ viết Mock/Stub để test), nhưng việc đảm bảo duy nhất 1 instance được giao lại cho DI Container quản lý.

---

## 2. Factory Method (Phương thức Nhà máy)

### 💡 Định nghĩa dễ nhớ
> **Factory Method** giống như **Nút đặt món ăn trên App Delivery**. Bạn (Client) chỉ cần ấn nút "Đặt Pizza" hoặc "Đặt Burger", hệ thống sẽ tự động gọi cửa hàng làm ra món đó mà bạn không cần biết công thức nhào bột hay nhiệt độ nướng.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Lớp cha cần xử lý nghiệp vụ chung nhưng không thể biết trước lớp con nào sẽ được khởi tạo tại runtime. Nếu dùng từ khóa `new` trực tiếp, code sẽ bị phụ thuộc cứng (tightly coupled) vào các class cụ thể đó.
*   **Giải pháp:** Định nghĩa một interface/abstract class để tạo đối tượng, nhưng để các lớp con tự quyết định class nào sẽ được khởi tạo.

### 💻 Code ví dụ bằng C#

```csharp
using System;

namespace DesignPatterns.Creational.FactoryMethod;

// 1. Product Interface
public interface IGovermentDocument
{
    void PrintDescription();
}

// 2. Concrete Products
public class Passport : IGovermentDocument
{
    public void PrintDescription() => Console.WriteLine("📖 Hộ chiếu quốc tế - Cho phép xuất nhập cảnh.");
}

public class DrivingLicense : IGovermentDocument
{
    public void PrintDescription() => Console.WriteLine("🪪 Bằng lái xe - Cho phép điều khiển phương tiện.");
}

// 3. Creator (Abstract Factory)
public abstract class DocumentOffice
{
    // Factory Method
    public abstract IGovermentDocument CreateDocument();

    // Lớp cha vẫn có thể thực hiện business logic dựa trên Product được tạo ra
    public void IssueDocument()
    {
        var doc = CreateDocument();
        Console.Write("🎟 Đang cấp phát giấy tờ: ");
        doc.PrintDescription();
    }
}

// 4. Concrete Creators
public class PassportOffice : DocumentOffice
{
    public override IGovermentDocument CreateDocument() => new Passport();
}

public class DrivingLicenseOffice : DocumentOffice
{
    public override IGovermentDocument CreateDocument() => new DrivingLicense();
}
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **`IHttpClientFactory` (.NET Core):** Thay vì tự khởi tạo `new HttpClient()`, bạn inject `IHttpClientFactory` và gọi `CreateClient()`. Nó giúp quản lý vòng đời của socket ngầm bên dưới, tránh rò rỉ socket (Socket Exhaustion).
*   **`DbProviderFactory` (ADO.NET):** Cung cấp các phương thức để tạo ra các đối tượng kết nối tùy thuộc vào loại CSDL cấu hình (như `SqlConnection`, `MySqlConnection`).

### 👁 Kỹ thuật ngầm ít người nhận ra
Trong thực tế phát triển phần mềm, mô hình **Simple Factory** (tạo một class Factory chứa hàm static có câu lệnh `switch-case` để new đối tượng) được dùng nhiều gấp 10 lần **Factory Method** chuẩn của GoF. Mặc dù Simple Factory vi phạm nguyên lý OCP (vì phải sửa code hàm static khi có class mới), nhưng nó lại cực kỳ đơn giản và đủ dùng cho 90% dự án thông thường.

---

## 3. Abstract Factory (Nhà máy Trừu tượng)

### 💡 Định nghĩa dễ nhớ
> **Abstract Factory** giống như **Cửa hàng nội thất IKEA**. Họ bán các gói combo nội thất theo chủ đề: chủ đề **Cổ điển** (Ghế gỗ, Bàn gỗ) hoặc chủ đề **Hiện đại** (Ghế nhựa plastic, Bàn kính). Cửa hàng đảm bảo bạn không bao giờ mua nhầm một cái Ghế nhựa hiện đại để lắp với cái Bàn gỗ cổ điển.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Hệ thống cần tạo ra các bộ (gia đình) sản phẩm liên quan chặt chẽ với nhau mà không muốn phụ thuộc vào các class cụ thể của từng sản phẩm đó.
*   **Giải pháp:** Định nghĩa một interface đại diện cho "Nhà máy tổng" để tạo ra các sản phẩm liên quan mà không cần chỉ định class cụ thể của chúng.

### 💻 Code ví dụ bằng C#

```csharp
using System;

namespace DesignPatterns.Creational.AbstractFactory;

// --- Định nghĩa các loại sản phẩm (Products) ---
public interface IButton { void Render(); }
public interface ICheckbox { void Render(); }

// Bộ sản phẩm giao diện Windows
public class WindowsButton : IButton { public void Render() => Console.WriteLine("Render nút kiểu Windows 🪟"); }
public class WindowsCheckbox : ICheckbox { public void Render() => Console.WriteLine("Render ô check kiểu Windows ☑️"); }

// Bộ sản phẩm giao diện macOS
public class MacButton : IButton { public void Render() => Console.WriteLine("Render nút kiểu macOS 🍎"); }
public class MacCheckbox : ICheckbox { public void Render() => Console.WriteLine("Render ô check kiểu macOS 🔘"); }


// --- Định nghĩa Nhà máy Trừu tượng (Abstract Factory) ---
public interface IUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

// Các Nhà máy cụ thể
public class WindowsFactory : IUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **`IServiceProvider` (Dependency Injection Container của .NET):** Nó đóng vai trò như một Abstract Factory khổng lồ. Bạn yêu cầu bất kỳ dịch vụ nào (`GetService(typeof(T))`), nó sẽ tự động phân giải cấu trúc cây dependency để khởi tạo và trả về gia đình đối tượng tương thích.

---

## 4. Builder (Xây dựng)

### 💡 Định nghĩa dễ nhớ
> **Builder** giống như **Tự chọn bánh mì SubWay / Pizza**. Bạn sẽ chọn từng thành phần một: bánh mì thường hay đen? Nhân thịt bò hay gà? Thêm phô mai không? Sốt gì? Cuối cùng, nhân viên mới "Build" ra chiếc bánh hoàn chỉnh cho bạn.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Một đối tượng có quá nhiều thuộc tính tùy chọn, dẫn đến constructor bị phình to (Telescoping Constructor) hoặc phải viết quá nhiều overload constructor.
*   **Giải pháp:** Tách biệt quá trình xây dựng một đối tượng phức tạp ra khỏi đại diện của nó, cho phép cùng một quá trình xây dựng tạo ra nhiều đại diện khác nhau.

### 💻 Code ví dụ bằng C# (Fluent API Builder)

```csharp
using System;

namespace DesignPatterns.Creational.Builder;

public class Email
{
    public string To { get; internal set; } = string.Empty;
    public string From { get; internal set; } = string.Empty;
    public string Subject { get; internal set; } = string.Empty;
    public string Body { get; internal set; } = string.Empty;
    public bool IsHtml { get; internal set; }

    public void Send() => Console.WriteLine($"📨 Gửi email từ [{From}] tới [{To}] với tiêu đề '{Subject}'");
}

// EmailBuilder thiết kế theo dạng Fluent API (Method Chaining)
public class EmailBuilder
{
    private readonly Email _email = new();

    public EmailBuilder To(string toAddress)
    {
        _email.To = toAddress;
        return this; // Trả về chính nó để nối chuỗi phương thức
    }

    public EmailBuilder From(string fromAddress)
    {
        _email.From = fromAddress;
        return this;
    }

    public EmailBuilder WithSubject(string subject)
    {
        _email.Subject = subject;
        return this;
    }

    public EmailBuilder WithBody(string body, bool isHtml = false)
    {
        _email.Body = body;
        _email.IsHtml = isHtml;
        return this;
    }

    // Bước cuối cùng để lấy ra kết quả hoàn chỉnh
    public Email Build()
    {
        if (string.IsNullOrEmpty(_email.To) || string.IsNullOrEmpty(_email.From))
        {
            throw new InvalidOperationException("Email phải có người gửi và người nhận!");
        }
        return _email;
    }
}

// Cách sử dụng:
// Email email = new EmailBuilder()
//                  .From("sender@gmail.com")
//                  .To("receiver@gmail.com")
//                  .WithSubject("Hello C#")
//                  .WithBody("Nội dung email", isHtml: true)
//                  .Build();
// email.Send();
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **`StringBuilder` (System.Text):** Giúp cộng chuỗi hiệu năng cao mà không sinh thêm rác vùng nhớ heap.
*   **`WebApplicationBuilder` (ASP.NET Core):** Cú pháp kinh điển khi khởi tạo ứng dụng: `var builder = WebApplication.CreateBuilder(args); builder.Services.AddControllers(); var app = builder.Build();`.

---

## 5. Prototype (Mẫu thử / Nhân bản)

### 💡 Định nghĩa dễ nhớ
> **Prototype** giống như **Nút "Duplicate" slide trong PowerPoint** hoặc phím tắt **Ctrl+C / Ctrl+V**. Thay vì tạo một slide mới tinh rồi định dạng lại từ đầu, bạn nhân bản trực tiếp slide cũ rồi sửa một vài chữ trên đó.

### 🛠 Vấn đề & Giải pháp
*   **Vấn đề:** Khởi tạo một đối tượng mới bằng từ khóa `new` tốn quá nhiều tài nguyên (đọc file cấu hình từ ổ đĩa, truy vấn database lấy dữ liệu thô).
*   **Giải pháp:** Sao chép (clone) trực tiếp từ một instance đang tồn tại mà không cần phụ thuộc vào class cụ thể của nó.

### 💻 Code ví dụ bằng C#

C# hỗ trợ sẵn giao diện `ICloneable`, tuy nhiên trong thực tế người ta thường tự viết phương thức Clone có kiểu trả về tường minh (Type-safe) để tránh phải cast kiểu dữ liệu:

```csharp
using System;

namespace DesignPatterns.Creational.Prototype;

public interface IPrototype<T>
{
    T Clone();
}

public class GameCharacter : IPrototype<GameCharacter>
{
    public string Name { get; set; }
    public int Level { get; set; }
    public Weapon EquippedWeapon { get; set; } // Tham chiếu đối tượng phức tạp

    public GameCharacter(string name, int level, Weapon weapon)
    {
        Name = name;
        Level = level;
        EquippedWeapon = weapon;
    }

    // Sao chép nông (Shallow Clone) vs Sao chép sâu (Deep Clone)
    public GameCharacter Clone()
    {
        // 1. Sao chép nông: Sử dụng phương thức MemberwiseClone có sẵn của .NET.
        // Chỉ copy kiểu dữ liệu nguyên thủy (Value type) và tham chiếu (Reference type) chứ không nhân bản đối tượng mà tham chiếu trỏ tới.
        var cloned = (GameCharacter)this.MemberwiseClone();
        
        // 2. Sao chép sâu: Tạo hẳn một instance mới cho các tham chiếu phức tạp
        cloned.EquippedWeapon = new Weapon(this.EquippedWeapon.Name, this.EquippedWeapon.Damage);
        
        return cloned;
    }
}

public class Weapon
{
    public string Name { get; set; }
    public int Damage { get; set; }

    public Weapon(string name, int damage)
    {
        Name = name;
        Damage = damage;
    }
}
```

### 🏛 Ví dụ Kinh điển trong Framework
*   **`ICloneable` Interface** trong BCL (Base Class Library) của .NET.
*   **`MemberwiseClone()` method:** Phương thức `protected` của lớp gốc `System.Object` giúp sao chép vùng nhớ thô cực kỳ nhanh.
