# Hướng dẫn sử dụng pyVideoTrans WebUI

![](https://pvtr2.pyvideotrans.com/1782189968417_image.png)


## ⚠️ Lưu ý quan trọng

> **Phiên bản WebUI chỉ hỗ trợ một phần tính năng**, chủ yếu được sử dụng trong các trường hợp sau:
> - Triển khai trên máy chủ đám mây (Cloud Server) để truy cập dịch vụ dịch thuật từ xa.
> - Triển khai trong mạng nội bộ LAN (tách biệt máy chủ xử lý và máy khách sử dụng).
>
> **Để sử dụng đầy đủ các tính năng**, vui lòng sử dụng phần mềm máy tính (Desktop Client - `sp.exe`) hoặc chạy trực tiếp từ mã nguồn (`sp.py`).
>
> Phiên bản Desktop hỗ trợ nhiều kênh cấu hình API hơn, cho phép biên tập chỉnh sửa phụ đề trực quan theo thời gian thực, xử lý hàng loạt và nhiều tính năng nâng cao khác.

---


## I. Chuẩn bị môi trường

### 1.1 Cài đặt các thư viện phụ thuộc

Giao diện WebUI phụ thuộc vào thư viện Gradio, mặc định chưa được cài đặt. Bạn cần cài đặt thêm bằng lệnh sau:

```bash
# Thực hiện tại thư mục gốc của dự án
uv sync --extra webui
```

### 1.2 Khởi chạy dịch vụ

```bash
# Khởi chạy cơ bản (lắng nghe tất cả các card mạng, cổng mặc định 7860)
uv run webui.py

# Chỉ định cổng cụ thể
uv run webui.py --port 8080

# Chỉ định địa chỉ lắng nghe (chỉ cho phép truy cập từ máy nội bộ)
uv run webui.py --host 127.0.0.1

# Truy cập qua mạng công cộng (tạo liên kết công khai Gradio)
uv run webui.py --share
```

### 1.3 Truy cập giao diện

Sau khi khởi chạy thành công, trình duyệt sẽ tự động mở trang web. Hoặc bạn có thể truy cập thủ công:

- Máy cục bộ: `http://127.0.0.1:7860`
- Mạng nội bộ LAN: `http://<IP_MÁY_CHỦ>:7860`
- Mạng công cộng (chế độ --share): Màn hình console sẽ in ra một liên kết dạng `*.gradio.live`

---

## II. Hướng dẫn các tính năng giao diện

### 2.1 Chọn tệp đầu vào

Nhấn vào vùng "Chọn tệp video/âm thanh" (Select video/audio file) để tải lên video hoặc file âm thanh bạn muốn dịch.

Các định dạng hỗ trợ:
- Video: mp4, mkv, avi, mov, webm, mpeg, ogg, mts, ts, wmv, flv
- Âm thanh: wav, mp3, m4a, flac, aac, wma, ogg

### 2.2 Nhận dạng giọng nói (ASR)

| Tham số | Giải thích | Giá trị mặc định |
|------|------|--------|
| Kênh nhận dạng (ASR Provider) | Chọn công cụ nhận dạng giọng nói | faster-whisper (tích hợp sẵn) |
| Mô hình (Model) | Chọn kích thước mô hình nhận dạng | large-v3-turbo |

**Các kênh khả dụng** (miễn phí/tích hợp sẵn ngoại tuyến):
- `faster-whisper (tích hợp sẵn)` — Khuyên dùng, tốc độ nhanh, chất lượng cao.
- `openai-whisper (tích hợp sẵn)` — Độ chính xác nhỉnh hơn một chút, tốc độ chậm hơn.
- `Qwen-ASR (tích hợp sẵn)` — Hiệu quả tốt với tiếng Trung.
- `FunASR-Chinese (tích hợp sẵn)` — Tối ưu hóa cho tiếng Trung.
- `Huggingface_ASR (tích hợp sẵn)` — Hỗ trợ nhiều mô hình đa ngôn ngữ.

**Giải thích về mô hình** (cho faster-whisper / openai-whisper):
- `tiny` — Nhanh nhất, độ chính xác thấp.
- `base` / `small` — Cân bằng giữa tốc độ và độ chính xác.
- `medium` — Độ chính xác khá tốt.
- `large-v3` — Tốt nhất, yêu cầu VRAM card đồ họa từ 8GB trở lên.
- `large-v3-turbo` — Khuyên dùng, cân đối hoàn hảo giữa tốc độ và chất lượng.

### 2.3 Dịch thuật phụ đề

| Tham số | Giải thích | Giá trị mặc định |
|------|------|--------|
| Kênh dịch thuật (Translation Provider) | Chọn công cụ dịch thuật | Google (miễn phí) |
| Ngôn ngữ gốc | Ngôn ngữ đang nói trong video | Tiếng Anh |
| Ngôn ngữ đích | Ngôn ngữ muốn dịch sang | Tiếng Trung giản thể |

**Các kênh dịch thuật khả dụng** (miễn phí):
- `Google (miễn phí)` — Chất lượng dịch tốt, cần thiết lập proxy mạng ở một số khu vực.
- `Microsoft (miễn phí)` — Không cần proxy, nhưng có thể bị giới hạn tần suất yêu cầu (rate limit).
- `M2M100 (ngoại tuyến)` — Sử dụng mô hình dịch thuật cục bộ trên máy.

### 2.4 Lồng tiếng phụ đề (TTS)

| Tham số | Giải thích | Giá trị mặc định |
|------|------|--------|
| Kênh lồng tiếng (TTS Provider) | Chọn công cụ tổng hợp giọng nói TTS | Edge-TTS (miễn phí) |
| Giọng đọc (Voice Role) | Chọn nhân vật phát âm | Thay đổi theo kênh |

**Các kênh lồng tiếng khả dụng**:
- `Edge-TTS (miễn phí)` — Dịch vụ miễn phí của Microsoft, giọng đọc rất tự nhiên.
- `Qwen3-TTS (tích hợp sẵn)` — Mô hình TTS ngoại tuyến của Alibaba.
- `MOSS-TTS-Nano (tích hợp sẵn)` — Hỗ trợ đa ngôn ngữ ngoại tuyến.
- `Piper (tích hợp sẵn)` — Mô hình TTS ngoại tuyến nhẹ và nhanh.
- `VITS (tích hợp sẵn)` — Lồng tiếng song ngữ Trung - Anh.
- `Supertonic3 (tích hợp sẵn)` — Hỗ trợ đa ngôn ngữ ngoại tuyến.
- `ChatterBox (tích hợp sẵn)` — Hỗ trợ đa ngôn ngữ ngoại tuyến, chất lượng tốt.
- `gTTS (miễn phí)` — Google TTS, chất lượng cơ bản.

**Giọng đọc**: Danh sách giọng đọc sẽ tự động cập nhật khi bạn thay đổi kênh lồng tiếng hoặc ngôn ngữ đích.

### 2.5 Căn chỉnh âm hình & Phụ đề

| Tham số | Giải thích | Giá trị mặc định |
|------|------|--------|
| Tăng tốc độ giọng đọc | Tự động tăng tốc độ file lồng tiếng nếu dài hơn thời gian phụ đề gốc | ✅ Chọn |
| Làm chậm video | Làm chậm video tại phân đoạn tương ứng nếu file lồng tiếng quá dài | ☐ Không chọn |
| Tốc độ lồng tiếng | Điều chỉnh tốc độ giọng đọc chung (-50% ~ +50%) | 0% |
| Âm lượng lồng tiếng | Điều chỉnh âm lượng giọng đọc (-95% ~ +100%) | 0% |
| Cao độ | Điều chỉnh cao độ giọng đọc (-100Hz ~ +100Hz) | 0Hz |
| Loại phụ đề nhúng | Chọn phương thức nhúng phụ đề vào video | Nhúng phụ đề cứng (hard subtitles) |

**Giải thích về các loại phụ đề nhúng**:
- Không nhúng phụ đề — Chỉ thay thế/lồng âm thanh mới.
- Nhúng phụ đề cứng — Phụ đề được ghi đè (vẽ cứng) vĩnh viễn vào khung hình video.
- Nhúng phụ đề mềm — Phụ đề được thêm dưới dạng một track độc lập, người xem có thể bật/tắt trên trình phát video.
- Nhúng phụ đề cứng (song ngữ) — Nhúng cứng cả phụ đề gốc và phụ đề dịch đồng thời.
- Nhúng phụ đề mềm (song ngữ) — Nhúng track phụ đề mềm song ngữ.

### 2.6 Các thiết lập nâng cao

| Tham số | Giải thích | Giá trị mặc định |
|------|------|--------|
| Giảm nhiễu | Loại bỏ tiếng ồn nền của âm thanh gốc | ☐ Không chọn |
| Xử lý dấu câu | Giữ dấu câu mặc định / Phục hồi dấu câu / Xóa dấu câu | Giữ dấu câu mặc định |
| Tách giọng nói & nhạc nền | Tách riêng giọng nói và âm thanh/nhạc nền gốc | ☐ Không chọn |
| Nhúng lại nhạc nền | Trộn lại nhạc nền gốc sau khi đã lồng giọng nói mới | ✅ Chọn |
| Cách xử lý nhạc nền | Cắt ngắn nhạc nền theo video / Lặp lại nhạc nền | Cắt ngắn nhạc nền |
| Âm lượng nhạc nền | Điều chỉnh âm lượng nhạc nền gốc (0.0 ~ 2.0) | 0.8 |
| Bật tăng tốc CUDA | Sử dụng GPU NVIDIA để tăng tốc (cần cài đặt CUDA tương ứng) | ☐ Không chọn |

### 2.7 Chỉnh sửa kiểu hiển thị phụ đề cứng

Nhấp vào "🎨 Chỉnh sửa kiểu hiển thị phụ đề cứng" (Hard Subtitle Style Edit) để mở rộng bảng tùy chỉnh giao diện phụ đề:

**Kiểu phụ đề chính (Primary Subtitle Style)**:
- Tên phông chữ, cỡ chữ.
- Màu chữ chính, màu viền chữ, màu nền (nếu có).
- Chữ đậm, chữ nghiêng, gạch chân, gạch ngang.

**Kiểu phụ đề phụ (Secondary Subtitle Style)** (dùng khi hiển thị phụ đề song ngữ):
- Tên phông chữ, cỡ chữ.
- Màu chữ chính, màu viền chữ, màu nền.
- Chữ đậm, chữ nghiêng.

**Cấu hình kiểu dáng chung**:
- Kiểu viền (Chỉ viền / Hộp nền mờ đục).
- Độ dày viền, đổ bóng chữ.
- Tỷ lệ co giãn ngang/dọc, khoảng cách chữ, góc xoay.
- Lề trái/phải/dọc.
- Vị trí căn lề (chọn 1 trong 9 vị trí trên khung hình).

Sau khi thay đổi, nhấn "💾 Lưu cấu hình" (Save Style). Cấu hình này sẽ được áp dụng cho tất cả các tác vụ nhúng phụ đề cứng tiếp theo.

---

## III. Thực hiện dịch thuật

### 3.1 Các bước thực hiện

1. Tải lên tệp video hoặc âm thanh cần xử lý.
2. Cấu hình tham số Nhận dạng giọng nói (ASR).
3. Cấu hình tham số Dịch thuật (chọn ngôn ngữ gốc và ngôn ngữ đích).
4. Cấu hình tham số Lồng tiếng (chọn nhà cung cấp TTS và giọng đọc phù hợp).
5. Điều chỉnh các thông số đồng bộ, phụ đề và thiết lập nâng cao.
6. Nhấp vào "🚀 Bắt đầu thực hiện" (Start Process).

### 3.2 Quá trình xử lý

Sau khi nhấn bắt đầu:
- Nút bấm sẽ đổi thành "⏳ Đang thực hiện..." và bị vô hiệu hóa để tránh việc nhấn đúp.
- Vùng log hiển thị bên phải sẽ cập nhật tiến trình thực hiện của từng giai đoạn theo thời gian thực.
- Khi hoàn thành, nút bấm sẽ chuyển lại thành "🚀 Bắt đầu thực hiện".

### 3.3 Các giai đoạn xử lý

```
Giai đoạn 1/8: Tiền xử lý (tách âm thanh và video)
Giai đoạn 2/8: Nhận dạng giọng nói (ASR)
Giai đoạn 3/8: Phân biệt người nói (Speaker Diarization)
Giai đoạn 4/8: Dịch thuật phụ đề
Giai đoạn 5/8: Tạo giọng lồng tiếng (TTS)
Giai đoạn 6/8: Căn chỉnh âm thanh & hình ảnh (Align)
Giai đoạn 7/8: Nhận dạng lần 2 (Recognize 2-pass)
Giai đoạn 8/8: Tổng hợp video/âm thanh cuối cùng
```

### 3.4 Kết quả đầu ra

Sau khi hoàn thành:
- **Khu vực xem trước video (Video Preview)**: Hiển thị file video kết quả đầu tiên, bạn có thể xem trực tuyến ngay.
- **Khu vực tải tệp (File Download)**: Liệt kê danh sách tất cả các tệp đầu ra (SRT, WAV, TXT...), bạn chỉ cần click vào để tải về máy.

Tất cả các tệp đầu ra sẽ được lưu trữ cục bộ trong thư mục `output/<tên_tệp>/`.

---

## IV. Các câu hỏi thường gặp (FAQ)

### Q: Khởi chạy gặp lỗi `No module named gradio`

Bạn cần cài đặt thêm thư viện gradio:

```bash
uv sync --extra webui
```

### Q: Xuất hiện cảnh báo khi chọn các kênh dịch vụ không khả dụng

Đây là phản ứng bình thường. WebUI hiện tại chỉ mở các kênh miễn phí và tích hợp ngoại tuyến cục bộ. Các kênh yêu cầu khóa API trả phí tạm thời không cấu hình được trên giao diện này. Nếu chọn, hệ thống sẽ tự động quay về lựa chọn hợp lệ gần nhất.

### Q: Tốc độ xử lý quá chậm

- Quá trình nhận dạng và lồng tiếng ngoại tuyến cần tải mô hình về máy, lần chạy đầu tiên sẽ khá chậm.
- Bật tính năng tăng tốc CUDA sẽ giúp cải thiện đáng kể tốc độ (yêu cầu máy có card đồ họa NVIDIA tương thích).
- Chọn mô hình nhận dạng nhỏ hơn (ví dụ: `base` thay vì `large`) để tăng tốc độ xử lý.

### Q: Làm thế nào để sử dụng các kênh dịch vụ yêu cầu khóa API?

Giao diện WebUI hiện tại chưa hỗ trợ điền trực tiếp khóa API. Vui lòng cấu hình các khóa này trên phiên bản phần mềm máy tính (`sp.exe` hoặc chạy `sp.py`). Sau khi lưu trên bản Desktop, phiên bản WebUI có thể tự động đọc lại các thiết lập này từ cấu hình hệ thống.

### Q: Làm thế nào để triển khai trên máy chủ Cloud?

```bash
# 1. Cài đặt các thư viện phụ thuộc
uv sync --extra webui

# 2. Khởi chạy dịch vụ (lắng nghe tất cả các card mạng)
uv run webui.py --host 0.0.0.0 --port 7860

# 3. Sử dụng trình duyệt trên máy khách truy cập http://<IP_MÁY_CHỦ>:7860
```

### Q: Cách tạo liên kết truy cập công cộng qua mạng Internet?

```bash
uv run webui.py --share
```

Hệ thống sẽ tạo ra một đường dẫn ngẫu nhiên dạng `*.gradio.live` giúp bạn truy cập được từ mọi nơi qua Internet (đây là liên kết tạm thời, sẽ mất hiệu lực khi bạn tắt chương trình).

---

## V. So sánh tính năng giữa phiên bản WebUI và Desktop

| Tính năng | WebUI | Bản Desktop (sp.exe / sp.py) |
|------|:-----:|:----------------------:|
| Dịch video (Quy trình đầy đủ) | ✅ | ✅ |
| Nhận dạng giọng nói (Kênh miễn phí/Cục bộ) | ✅ | ✅ |
| Nhận dạng giọng nói (Kênh API trả phí) | ❌ | ✅ |
| Dịch phụ đề (Kênh miễn phí) | ✅ | ✅ |
| Dịch phụ đề (Kênh API trả phí) | ❌ | ✅ |
| Lồng tiếng (Kênh miễn phí/Cục bộ) | ✅ | ✅ |
| Lồng tiếng (Kênh API trả phí) | ❌ | ✅ |
| Cấu hình khóa API | ❌ | ✅ |
| Chỉnh sửa phụ đề thời gian thực (Interactive Edit) | ❌ | ✅ |
| Xử lý hàng loạt nhiều tệp | ❌ | ✅ |
| Thiết lập Tách giọng nói & nhạc nền | ✅ | ✅ |
| Chỉnh sửa kiểu dáng phụ đề cứng | ✅ | ✅ |
| Xem trước/Phát video trực tiếp | ✅ | ❌ |
| Truy cập từ xa (Remote Access) | ✅ | ❌ |
| Cấu hình Proxy mạng | ❌ | ✅ |
| Đầy đủ tùy chọn nâng cao | ❌ | ✅ |
