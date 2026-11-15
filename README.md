# Smart Home Mini

Bài thực hành 10 – Phân tích thiết kế hệ thống (Factory, Singleton, Facade).

- Mã sinh viên: 202418901
- Họ tên: Nguyễn Việt Hoàng

Ứng dụng console C# (.NET 8) điều khiển ngôi nhà thông minh với ba thiết bị: `SmartLight`, `SmartFan`, `SmartDoor`.

## Cấu trúc

```
SmartHomeMini/
├── SmartHomeMini.csproj
├── src/
│   ├── IDevice.cs            abstraction chung: TurnOn / TurnOff
│   ├── ILockable.cs          khả năng khóa: Lock / Unlock (chỉ cửa)
│   ├── SmartLight.cs         đèn
│   ├── SmartFan.cs           quạt
│   ├── SmartDoor.cs          cửa (IDevice + ILockable)
│   ├── DeviceFactory.cs      Factory
│   ├── HomeConfiguration.cs  Singleton
│   ├── SmartHomeFacade.cs    Facade
│   ├── TestRunner.cs         kiểm thử T1-T5
│   └── Program.cs            client / điểm vào
└── docs/
    ├── class-diagram.mmd     nguồn sơ đồ lớp UML (Mermaid)
    └── class-diagram.png     sơ đồ lớp UML
```

## Cách chạy

Cần .NET SDK 8.0 trở lên.

```bash
dotnet run                    # demo ba pattern, sau đó chạy kiểm thử T1-T5
dotnet run -- test            # chỉ chạy kiểm thử T1-T5
dotnet run -- interactive     # tự chọn loại thiết bị để Factory tạo
```

## Vị trí sử dụng và lợi ích của từng pattern

| Pattern | Vị trí sử dụng | Lợi ích đạt được |
|---|---|---|
| **Factory** | `DeviceFactory.Create(type, name)`, được `Program` gọi để tạo thiết bị | Client chỉ biết `IDevice`, không gọi constructor của lớp cụ thể; chuỗi loại không phân biệt hoa/thường; loại sai gây `ArgumentException` thay vì trả `null`; thêm loại mới chỉ sửa một chỗ |
| **Singleton** | `HomeConfiguration.Instance`, được `SmartHomeFacade.GoToSleep()` đọc `NightLightLevel` | Toàn ứng dụng dùng một cấu hình nhất quán; `Lazy<T>` tạo khi cần và an toàn đa luồng; `sealed` + constructor `private` chặn tạo thêm thể hiện |
| **Facade** | `SmartHomeFacade` với `GoToWork()` và `GoToSleep()` | Client kích hoạt chế độ Đi làm / Đi ngủ bằng một lời gọi, không cần biết thứ tự thao tác trên từng thiết bị; facade làm việc qua `IDevice` và `ILockable` nên không phụ thuộc lớp cụ thể |

## Một hạn chế của thiết kế

`SmartHomeFacade.GoToSleep()` gọi trực tiếp `HomeConfiguration.Instance`, tức là phụ thuộc ẩn vào một đối tượng toàn cục. Vì vậy không thể thay cấu hình giả khi kiểm thử, và kết quả của T5 phụ thuộc vào giá trị toàn cục hiện tại. Cách cải tiến: tạo `IHomeConfiguration`, truyền vào constructor của facade (Dependency Injection, đăng ký `AddSingleton`), khi đó không cần tự cài Singleton pattern.

## Kết quả kiểm thử

| Mã | Tình huống kiểm tra | Kết quả mong đợi | Kết quả |
|---|---|---|---|
| T1 | Tạo `light`, `fan`, `door` | Factory trả đúng kiểu đối tượng | Đạt |
| T2 | Tạo loại `camera` chưa hỗ trợ | Phát sinh `ArgumentException` | Đạt |
| T3 | Lấy `HomeConfiguration.Instance` hai lần | Hai biến cùng tham chiếu một đối tượng | Đạt |
| T4 | Gọi `GoToWork()` | Đèn tắt, quạt tắt, cửa khóa | Đạt |
| T5 | Gọi `GoToSleep()` | Quạt tắt, cửa khóa, đèn bật, hiện mức sáng | Đạt |

## Trả lời câu hỏi (Phần G)

**1. Factory đang che giấu quyết định thiết kế nào?**
Quyết định dùng lớp cụ thể nào và tạo ra nó như thế nào: ánh xạ từ chuỗi loại sang `SmartLight`, `SmartFan` hay `SmartDoor`, cùng việc chuẩn hóa chuỗi và xử lý loại không hợp lệ. Client chỉ nhận `IDevice`.

**2. Singleton bảo đảm điều gì, nhưng tạo ra rủi ro nào?**
Bảo đảm chỉ có một thể hiện duy nhất và một điểm truy cập toàn cục (an toàn đa luồng nhờ `Lazy<T>`). Rủi ro: trạng thái toàn cục và phụ thuộc ẩn, các lớp dùng nó bị ghép chặt, khó viết unit test độc lập (trạng thái còn lại giữa các test, khó thay bằng đối tượng giả), và dễ bị lạm dụng.

**3. Facade khác Factory ở mục đích chính như thế nào?**
Factory thuộc nhóm tạo đối tượng: trả lời "tạo đối tượng nào". Facade thuộc nhóm cấu trúc: trả lời "làm sao gọi gọn một nhóm thao tác trên nhiều đối tượng", cung cấp giao diện đơn giản và không tạo ra thiết bị.

**4. Nếu bỏ `SmartHomeFacade`, client sẽ phải biết thêm những chi tiết nào?**
Phải biết mỗi chế độ gồm những thiết bị nào, thứ tự gọi (ví dụ khóa cửa trước khi bật đèn ngủ), cửa là `ILockable` nên phải dùng `Lock()` chứ không phải `TurnOff()`, và tự đọc `NightLightLevel` từ `HomeConfiguration` để thông báo. Các chi tiết này bị lặp lại ở mọi nơi dùng, đổi chế độ thì phải sửa nhiều chỗ.
