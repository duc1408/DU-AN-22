# Hướng dẫn cài đặt Whisper.NET

## Đây là gì?

Whisper.NET là một công cụ nhận dạng giọng nói, cho phép card đồ họa AMD của bạn sử dụng công nghệ tăng tốc Vulkan để nhận dạng giọng nói (thay vì sử dụng công nghệ CUDA của NVIDIA).

## Trường hợp sử dụng phù hợp

Thích hợp cho người dùng chạy hệ điều hành **Windows và sử dụng card đồ họa AMD**.

**Bối cảnh**: Trong quá trình sử dụng Whisper.cpp, tôi nhận thấy phiên bản mới nhất không còn hỗ trợ GPU cho card đồ họa AMD trên môi trường Windows nữa. Điều này dẫn đến việc người dùng Windows sử dụng card AMD buộc phải dùng CPU để nhận dạng giọng nói, khiến tốc độ xử lý rất chậm. Sau khi tham khảo ý kiến và thảo luận với AI, tôi đã chọn giải pháp kỹ thuật Whisper.NET và triển khai thành công nhờ khả năng lập trình của AI.

**Môi trường thử nghiệm**: Hiện tại mới chỉ thử nghiệm thành công trên cấu hình **Windows 11 23H2 + RX 6650 XT**. Các môi trường khác người dùng có thể tự kiểm tra, rất mong nhận được phản hồi kết quả từ các bạn.

---

## Bước 1: Tải xuống các tệp DLL

### Danh sách các tệp cần tải xuống

**DLL quản lý (Managed DLLs)** (Tải xuống và đặt vào thư mục `deps/`):

| Tên tệp | Phiên bản | Liên kết tải xuống |
|--------|------|----------|
| Whisper.net.dll | 1.9.0 | [Tải xuống](https://www.nuget.org/packages/Whisper.net/1.9.0) |
| Microsoft.Extensions.AI.Abstractions.dll | 10.0.0 | [Tải xuống](https://www.nuget.org/packages/Microsoft.Extensions.AI.Abstractions/10.0.0) |
| Microsoft.Bcl.AsyncInterfaces.dll | 10.0.0 | [Tải xuống](https://www.nuget.org/packages/Microsoft.Bcl.AsyncInterfaces/10.0.0) |
| System.Memory.dll | 4.6.3 | [Tải xuống](https://www.nuget.org/packages/System.Memory/4.6.3) |
| System.Buffers.dll | 4.6.1 | [Tải xuống](https://www.nuget.org/packages/System.Buffers/4.6.1) |
| System.Runtime.CompilerServices.Unsafe.dll | 6.1.2 | [Tải xuống](https://www.nuget.org/packages/System.Runtime.CompilerServices.Unsafe/6.1.2) |
| System.Numerics.Vectors.dll | 4.6.1 | [Tải xuống](https://www.nuget.org/packages/System.Numerics.Vectors/4.6.1) |

**DLL gốc (Native DLLs)** (Tải xuống và đặt vào thư mục `deps/native/`):

Tải xuống từ trang [Whisper.net.Runtime.Vulkan 1.9.0](https://www.nuget.org/packages/Whisper.net.Runtime.Vulkan/1.9.0), giải nén và sao chép **tất cả các tệp DLL** nằm trong thư mục `build/win-x64/` vào thư mục `deps/native/`.

Gói NuGet chứa các tệp sau (tất cả đều bắt buộc):

| Tên tệp | Dung lượng | Công dụng |
|--------|------|------|
| whisper.dll | 473KB | Nhân xử lý nhận dạng giọng nói |
| libwhisper.dll | 473KB | Tên thay thế của whisper.dll (bắt buộc) |
| ggml-whisper.dll | 66KB | Thư viện tính toán |
| libggml-whisper.dll | 66KB | Tên thay thế của ggml-whisper.dll |
| ggml-base-whisper.dll | 528KB | Thư viện cơ sở (phụ thuộc bắt buộc) |
| libggml-base-whisper.dll | 528KB | Tên thay thế của ggml-base-whisper.dll |
| ggml-cpu-whisper.dll | 590KB | Bộ xử lý dự phòng cho CPU |
| libggml-cpu-whisper.dll | 590KB | Tên thay thế của ggml-cpu-whisper.dll |
| ggml-vulkan-whisper.dll | 45MB | Tăng tốc GPU (Vulkan) |
| libggml-vulkan-whisper.dll | 45MB | Tên thay thế của ggml-vulkan-whisper.dll |

### Cách tải xuống từ NuGet?

1. Nhấp vào các liên kết ở trên để mở trang NuGet tương ứng.
2. Nhấn nút **"Download package"** để tải về tệp `.nupkg`.
3. Đổi đuôi tệp từ `.nupkg` thành `.zip`, sau đó mở tệp bằng phần mềm giải nén.
4. Tìm các tệp DLL bên trong:
   - Các DLL quản lý nằm trong thư mục `lib/netstandard2.0/`.
   - Các DLL gốc nằm trong thư mục `build/win-x64/`.

---

## Bước 2: Tải xuống mô hình ngôn ngữ (Model)

Truy cập trang [ggerganov/whisper.cpp models](https://github.com/ggerganov/whisper.cpp/tree/master/models) để tải xuống các tệp mô hình định dạng `.bin`, sau đó đặt chúng vào thư mục `models/` của dự án.

Khuyên dùng: `ggml-large-v3-turbo.bin` (cho kết quả tốt và tốc độ xử lý nhanh).

---

## Bước 3: Kiểm tra cấu trúc thư mục

Hãy chắc chắn rằng cấu trúc thư mục của bạn chính xác như sau:

```
pyvideotrans/
├─ models/
│  └─ ggml-large-v3-turbo.bin    ← Mô hình giọng nói
└─ deps/
   ├─ Whisper.net.dll            ← 7 tệp này là DLL quản lý
   ├─ Microsoft.Extensions.AI.Abstractions.dll
   ├─ Microsoft.Bcl.AsyncInterfaces.dll
   ├─ System.Memory.dll
   ├─ System.Buffers.dll
   ├─ System.Runtime.CompilerServices.Unsafe.dll
   ├─ System.Numerics.Vectors.dll
   └─ native/                     ← Sao chép toàn bộ DLL từ build/win-x64/ của NuGet vào đây
      ├─ whisper.dll
      ├─ libwhisper.dll
      ├─ ggml-whisper.dll
      ├─ libggml-whisper.dll
      ├─ ggml-base-whisper.dll
      ├─ libggml-base-whisper.dll
      ├─ ggml-cpu-whisper.dll
      ├─ libggml-cpu-whisper.dll
      ├─ ggml-vulkan-whisper.dll
      └─ libggml-vulkan-whisper.dll
```

---

## Bước 4: Hướng dẫn sử dụng

0. Triển khai dự án từ nguồn, chạy lệnh `uv sync --all-extras`. Nếu đã cài đặt trước đó, hãy chạy riêng lệnh `uv sync --extra dotnet` để cài đặt mô hình `pythonnet`.
1. Chạy lệnh `uv run sp.py` để mở phần mềm.
2. Tại hộp thoại tùy chọn "Nhận dạng giọng nói" (Speech Recognition), chọn **"Whisper.NET"**.
3. Chọn tệp mô hình mà bạn vừa tải xuống.
4. Nhấn nút Bắt đầu để sử dụng.

---

## Giải quyết sự cố thường gặp

### Lỗi hiển thị "Native Library not found" hoặc mã lỗi `0x8007007E`

- Kiểm tra xem thư mục `deps/native/` đã có đủ 10 tệp DLL hay chưa.
- Kiểm tra xem tên các tệp đã chính xác chưa.

### Tính năng tăng tốc GPU không hoạt động

- Cập nhật driver card đồ họa lên phiên bản mới nhất.
- Card đồ họa AMD cần hỗ trợ Vulkan (các dòng RX 400 trở lên).
- Card đồ họa NVIDIA cần dòng GTX 600 trở lên.

### Lỗi khởi tạo pythonnet thất bại

- Tải và cài đặt [.NET Runtime](https://dotnet.microsoft.com/download/dotnet) (chọn phiên bản mới nhất như .NET 8 hoặc .NET 9).

### Muốn kiểm tra xem card đồ họa có hỗ trợ Vulkan hay không

Mở cửa sổ dòng lệnh (cmd/terminal) và nhập:
```bash
vulkaninfo
```
Nếu màn hình hiển thị thông tin chi tiết về card đồ họa của bạn thì nghĩa là thiết bị có hỗ trợ Vulkan.
