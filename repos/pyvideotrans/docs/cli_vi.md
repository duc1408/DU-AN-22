# Hướng dẫn sử dụng Giao diện dòng lệnh (CLI) pyVideoTrans

pyVideoTrans hỗ trợ vận hành qua dòng lệnh không giao diện đồ họa, thích hợp cho việc triển khai trên máy chủ, xử lý hàng loạt, tích hợp vào đường dẫn tự động hóa, v.v.

---

## Mục lục

- [Yêu cầu hệ thống](#yêu-cầu-hệ-thống)
- [Cách dùng cơ bản](#cách-dùng-cơ-bản)
- [Tùy chọn toàn cục](#tùy-chọn-toàn-cục)
- [Tổng quan loại nhiệm vụ](#tổng-quan-loại-nhiệm-vụ)
- [STT — Nhận dạng giọng nói](#stt--nhận-dạng-giọng-nói)
- [TTS — Lồng tiếng văn bản](#tts--lồng-tiếng-văn-bản)
- [STS — Dịch thuật phụ đề](#sts--dịch-thuật-phụ-đề)
- [VTV — Dịch thuật video](#vtv--dịch-thuật-video)
- [Công cụ truy vấn](#công-cụ-truy-vấn)
- [Ví dụ đầy đủ](#ví-dụ-đầy-đủ)
- [Câu hỏi thường gặp](#câu-hỏi-thường-gặp)

---

## Yêu cầu hệ thống

| Danh mục | Yêu cầu |
|------|------|
| Python | 3.10 |
| Trình quản lý gói | [uv](https://docs.astral.sh/uv/) |
| FFmpeg | Bắt buộc phải cài đặt và cấu hình biến môi trường (Bản đóng gói Windows đã tích hợp sẵn) |
| Tăng tốc GPU (Tùy chọn) | Card đồ họa NVIDIA + CUDA 12.8 + cuDNN 9.11 |

### Cách khởi chạy

```bash
# Triển khai từ mã nguồn
uv run cli.py [Tham số...]

# Phiên bản đóng gói Windows
cli.exe [Tham số...]
```

> **Lưu ý**: Phiên bản đóng gói Windows (`cli.exe`) không yêu cầu cài đặt Python, bạn có thể chạy trực tiếp.

---

## Cách dùng cơ bản

```bash
uv run cli.py --task <Loại nhiệm vụ> --name "<Đường dẫn tệp>" [Tham số khác]
```

**Bốn loại nhiệm vụ:**

| Nhiệm vụ | Giải thích | Quy trình xử lý |
|------|------|--------|
| `stt` | Nhận dạng giọng nói — Chuyển đổi giọng nói từ video/âm thanh thành phụ đề SRT | Tiền xử lý → Nhận dạng giọng nói → Phân tách người nói → Xuất phụ đề |
| `tts` | Lồng tiếng văn bản — Chuyển đổi phụ đề SRT hoặc văn bản thuần túy thành tệp âm thanh | Tiền xử lý → Tạo giọng lồng tiếng → Căn chỉnh âm hình → Xuất âm thanh |
| `sts` | Dịch phụ đề — Dịch phụ đề SRT sang ngôn ngữ đích | Tiền xử lý → Dịch thuật → Xuất phụ đề |
| `vtv` | Dịch video — Quy trình đầy đủ: Nhận dạng → Dịch → Lồng tiếng → Tổng hợp video | Tiền xử lý → Nhận dạng → Phân tách người nói → Dịch → Lồng tiếng → Căn chỉnh → Nhận dạng lần 2 → Tổng hợp video |

---

## Tùy chọn toàn cục

| Tùy chọn | Giải thích | Giá trị mặc định |
|------|------|--------|
| `--task {stt,tts,sts,vtv}` | **Bắt buộc** — Loại nhiệm vụ | — |
| `--name FILE` | **Bắt buộc** — Đường dẫn tuyệt đối đến tệp đầu vào | — |
| `--output-dir DIR` | Thư mục đầu ra | `<Thư mục phần mềm>/output/<Tên tệp>/` |
| `--list {providers,languages,models}` | Truy vấn danh sách các kênh/ngôn ngữ/mô hình khả dụng | — |
| `--log-level {DEBUG,INFO,WARNING,ERROR}` | Mức độ ghi log | `WARNING` |
| `-v, --verbose` | Đầu ra chi tiết (tương đương `--log-level INFO`) | Không |
| `-q, --quiet` | Chế độ im lặng, chỉ xuất thông báo lỗi | Không |
| `--version` | Hiển thị phiên bản | — |
| `-h, --help` | Hiển thị thông tin trợ giúp | — |

---

## Tổng quan loại nhiệm vụ

### Các tham số bắt buộc cho từng nhiệm vụ

| Nhiệm vụ | `--name` | `--voice_role` | `--source_language_code` | `--target_language_code` |
|------|:---:|:---:|:---:|:---:|
| `stt` | ✅ | — | — | — |
| `tts` | ✅ | ✅ | — | — |
| `sts` | ✅ | — | Tùy chọn (mặc định auto) | ✅ |
| `vtv` | ✅ | Tùy chọn (mặc định No) | ✅ | ✅ |

### Phạm vi tham số được hỗ trợ cho từng nhiệm vụ

| Tham số | stt | tts | sts | vtv |
|------|:---:|:---:|:---:|:---:|
| `--recogn_type` | ✅ | — | — | ✅ |
| `--detect_language` | ✅ | — | — | ✅ |
| `--model_name` | ✅ | — | — | ✅ |
| `--cuda` | ✅ | — | — | ✅ |
| `--remove_noise` | ✅ | — | — | ✅ |
| `--enable_diariz` | ✅ | — | — | ✅ |
| `--nums_diariz` | ✅ | — | — | ✅ |
| `--rephrase` | ✅ | — | — | ✅ |
| `--fix_punc` | ✅ | — | — | ✅ |
| `--tts_type` | — | ✅ | — | ✅ |
| `--voice_role` | — | ✅ | — | ✅ |
| `--voice_rate` | — | ✅ | — | ✅ |
| `--volume` | — | ✅ | — | ✅ |
| `--pitch` | — | ✅ | — | ✅ |
| `--voice_autorate` | — | ✅ | — | ✅ |
| `--align_sub_audio` | — | ✅ | — | ✅ |
| `--translate_type` | — | — | ✅ | ✅ |
| `--source_language_code` | — | — | ✅ | ✅ |
| `--target_language_code` | — | — | ✅ | ✅ |
| `--video_autorate` | — | — | — | ✅ |
| `--is_separate` | — | — | — | ✅ |
| `--recogn2pass` | — | — | — | ✅ |
| `--subtitle_type` | — | — | — | ✅ |
| `--clear_cache` | — | — | — | ✅ |

---

## STT — Nhận dạng giọng nói

Chuyển đổi âm thanh hoặc giọng nói từ video thành file phụ đề SRT có mốc thời gian.

### Giải thích tham số

| Tham số | Kiểu dữ liệu | Mặc định | Giải thích |
|------|------|--------|------|
| `--recogn_type` | int | `0` | Mã số kênh nhận dạng giọng nói (0=faster-whisper, 1=openai-whisper, ...) |
| `--detect_language` | str | `auto` | Ngôn ngữ phát âm (auto=tự động phát hiện, zh-cn, en, ja, vi, ...) |
| `--model_name` | str | `tiny` | Tên mô hình (chỉ có hiệu lực với faster-whisper/openai-whisper) |
| `--cuda` | flag | Không | Kích hoạt tăng tốc phần cứng bằng GPU qua CUDA |
| `--remove_noise` | flag | Không | Kích hoạt bộ giảm nhiễu |
| `--enable_diariz` | flag | Không | Kích hoạt tính năng phân tách người nói |
| `--nums_diariz` | int | `-1` | Số lượng người nói (-1=tự động phát hiện) |
| `--rephrase` | int | `0` | Cấu trúc lại câu (0=mặc định, 1=sử dụng LLM) |
| `--fix_punc` | flag | Không | Khôi phục dấu câu tự động |

### Ví dụ

**Sử dụng cơ bản nhất — Sử dụng mô hình faster-whisper mặc định để nhận dạng video tiếng Trung:**

```bash
uv run cli.py --task stt --name "60.mp4"
```

> Mặc định sử dụng faster-whisper + mô hình tiny, xuất file phụ đề SRT vào thư mục `output/60-mp4/`.

**Chỉ định mô hình large-v3 + tăng tốc GPU CUDA:**

```bash
uv run cli.py --task stt --name "60.mp4" --recogn_type 0 --model_name large-v3 --cuda
```

**Chỉ định ngôn ngữ nói là tiếng Trung + giảm nhiễu:**

```bash
uv run cli.py --task stt --name "60.mp4" --detect_language zh-cn --remove_noise --cuda
```

**Sử dụng kênh nhận dạng openai-whisper:**

```bash
uv run cli.py --task stt --name "60.mp4" --recogn_type 1 --model_name large-v3 --cuda
```

**Bật phân tách người nói (chỉ định có 2 người nói):**

```bash
uv run cli.py --task stt --name "60.mp4" --enable_diariz --nums_diariz 2 --cuda
```

**Bật cấu trúc lại câu bằng LLM + tự động điền dấu câu:**

```bash
uv run cli.py --task stt --name "60.mp4" --rephrase 1 --fix_punc --cuda
```

**Tùy chỉnh thư mục đầu ra:**

```bash
uv run cli.py --task stt --name "60.mp4" --output-dir "D:/my_output" --cuda
```

---

## TTS — Lồng tiếng văn bản

Chuyển đổi tệp phụ đề SRT hoặc tệp văn bản thuần túy thành tệp âm thanh lồng tiếng.

### Giải thích tham số

| Tham số | Kiểu dữ liệu | Mặc định | Giải thích |
|------|------|--------|------|
| `--tts_type` | int | `0` | Mã số kênh lồng tiếng (0=Edge-TTS, ...) |
| `--voice_role` | str | **Bắt buộc** | Tên giọng đọc (Voice Role) |
| `--voice_rate` | str | `+0%` | Tốc độ nói (ví dụ: `+20%` để nói nhanh hơn, `-10%` để nói chậm đi) |
| `--volume` | str | `+0%` | Âm lượng (ví dụ: `+50%` để to hơn, `-30%` để nhỏ đi) |
| `--pitch` | str | `+0Hz` | Cao độ (ví dụ: `+10Hz` giọng cao hơn, `-5Hz` giọng trầm hơn) |
| `--voice_autorate` | flag | Không | Tự động tăng tốc độ lồng tiếng để khớp với mốc thời gian phụ đề gốc |
| `--align_sub_audio` | flag | Không | Bắt buộc thay đổi mốc thời gian phụ đề để khớp với thời gian file lồng tiếng |
| `--target_language_code` | str | `None` | Mã ngôn ngữ đích |

### Ví dụ

**Cách dùng cơ bản — Sử dụng Edge-TTS lồng tiếng cho phụ đề tiếng Trung:**

```bash
uv run cli.py --task tts --name "zw.srt" --voice_role "zh-CN-YunyangNeural"
```

> Sử dụng giọng nam Yunyang (miễn phí của Edge-TTS) để tạo âm thanh lồng tiếng cho phụ đề.

**Lồng tiếng bằng tiếng Anh (tự động dịch từ tiếng Trung sang tiếng Anh trước khi lồng tiếng):**

```bash
uv run cli.py --task tts --name "zw.srt" --voice_role "en-US-GuyNeural" --target_language_code en
```

**Điều chỉnh tốc độ nói và âm lượng:**

```bash
uv run cli.py --task tts --name "zw.srt" --voice_role "zh-CN-YunyangNeural" --voice_rate=+20% --volume=+10%
```

**Điều chỉnh cao độ giọng trầm hơn:**

```bash
uv run cli.py --task tts --name "zw.srt" --voice_role "zh-CN-YunyangNeural" --pitch=-5Hz
```

**Bật tự động tăng tốc giọng nói lồng tiếng cho khớp thời gian:**

```bash
uv run cli.py --task tts --name "zw.srt" --voice_role "zh-CN-YunyangNeural" --voice_autorate
```

**Sử dụng kênh lồng tiếng khác (ví dụ: OpenAI TTS, kiểm tra mã số kênh qua lệnh `--list providers`):**

```bash
uv run cli.py --task tts --name "zw.srt" --tts_type <mã_kênh> --voice_role "alloy"
```

### Các giọng đọc Edge-TTS phổ biến

| Tên giọng đọc | Giới tính | Ngôn ngữ | Mô tả |
|----------|------|------|------|
| `zh-CN-YunyangNeural` | Nam | Tiếng Trung | Vân Dương — Phong cách đọc tin tức |
| `zh-CN-XiaoxiaoNeural` | Nữ | Tiếng Trung | Tiêu Tiêu — Giao tiếp tự nhiên |
| `zh-CN-YunxiNeural` | Nam | Tiếng Trung | Vân Hy — Trẻ trung năng động |
| `en-US-GuyNeural` | Nam | Tiếng Anh | Guy — Giọng nam tự nhiên |
| `en-US-JennyNeural` | Nữ | Tiếng Anh | Jenny — Giọng nữ tự nhiên |
| `en-US-AriaNeural` | Nữ | Tiếng Anh | Aria — Giọng nữ chuyên nghiệp |
| `en-US-EmmaNeural` | Nữ | Tiếng Anh | Emma — Giọng nữ ấm áp |
| `en-US-BrianNeural` | Nam | Tiếng Anh | Brian — Giọng nam trầm ấm |
| `vi-VN-HoaiAnNeural` | Nữ | Tiếng Việt | Hoài An — Giọng nữ tự nhiên |
| `vi-VN-NamMinhNeural` | Nam | Tiếng Việt | Nam Minh — Giọng nam tự nhiên |

> Xem danh sách đầy đủ tất cả các giọng đọc khả dụng bằng cách chạy lệnh `uv run cli.py --list providers` hoặc xem trong bảng lựa chọn TTS trên giao diện GUI.

---

## STS — Dịch thuật phụ đề

Dịch file phụ đề SRT từ ngôn ngữ này sang ngôn ngữ khác.

### Giải thích tham số

| Tham số | Kiểu dữ liệu | Mặc định | Giải thích |
|------|------|--------|------|
| `--translate_type` | int | `0` | Mã số kênh dịch thuật (0=Google Translate, ...) |
| `--source_language_code` | str | `auto` | Mã ngôn ngữ gốc (auto=tự động phát hiện) |
| `--target_language_code` | str | **Bắt buộc** | Mã ngôn ngữ đích |

### Ví dụ

**Sử dụng cơ bản — Dịch phụ đề tiếng Trung sang tiếng Anh:**

```bash
uv run cli.py --task sts --name "zw.srt" --target_language_code en
```

> Mặc định sử dụng Google Translate, ngôn ngữ gốc tự động phát hiện.

**Chỉ định rõ ngôn ngữ gốc là tiếng Trung:**

```bash
uv run cli.py --task sts --name "zw.srt" --source_language_code zh-cn --target_language_code en
```

**Sử dụng kênh dịch thuật khác (ví dụ: DeepSeek, kiểm tra mã số kênh qua lệnh `--list providers`):**

```bash
uv run cli.py --task sts --name "zw.srt" --translate_type <mã_kênh> --target_language_code en
```

**Dịch sang tiếng Nhật:**

```bash
uv run cli.py --task sts --name "zw.srt" --target_language_code ja
```

**Dịch sang tiếng Việt:**

```bash
uv run cli.py --task sts --name "zw.srt" --target_language_code vi
```

---

## VTV — Dịch thuật video

Quy trình dịch video đầy đủ bao gồm: Nhận dạng giọng nói → Dịch phụ đề → Lồng tiếng → Hợp nhất âm hình. Đây là loại tác vụ phổ biến nhất nhưng cũng có thiết lập phức tạp nhất.

### Giải thích tham số

Chế độ VTV hỗ trợ tất cả các tham số của các tác vụ STT + TTS + STS kết hợp với các tham số bổ sung dưới đây:

| Tham số | Kiểu dữ liệu | Mặc định | Giải thích |
|------|------|--------|------|
| `--source_language_code` | str | **Bắt buộc** | Mã ngôn ngữ gốc (không được sử dụng auto) |
| `--target_language_code` | str | **Bắt buộc** | Mã ngôn ngữ đích |
| `--voice_role` | str | `No` | Giọng đọc lồng tiếng (`No`=chỉ dịch phụ đề, không lồng tiếng) |
| `--video_autorate` | flag | Không | Tự động làm chậm video tại phân đoạn tương ứng nếu file lồng tiếng quá dài |
| `--is_separate` | flag | Không | Tách giọng nói khỏi nhạc nền gốc |
| `--recogn2pass` | flag | Không | Nhận dạng giọng nói lần 2 để tạo mốc thời gian phụ đề chuẩn xác hơn |
| `--subtitle_type` | int | `1` | Loại phụ đề nhúng (0=Không nhúng, 1=Nhúng phụ đề cứng, 2=Nhúng phụ đề mềm, 3=Nhúng phụ đề cứng song ngữ, 4=Nhúng phụ đề mềm song ngữ) |
| `--clear_cache` | flag | Có | Tự động xóa các file tạm sau khi hoàn thành |
| `--no-clear-cache` | flag | — | Không xóa tệp tạm thời |

### Ví dụ

**Cách dùng đơn giản — Dịch video tiếng Trung sang tiếng Anh (không lồng tiếng, chỉ nhúng phụ đề):**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en
```

> Mặc định sử dụng nhận dạng faster-whisper + dịch thuật Google Translate + không lồng tiếng, nhúng phụ đề cứng tiếng Anh vào video đầu ra.

**Quy trình đầy đủ — Dịch video tiếng Trung sang tiếng Anh kèm lồng tiếng:**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural"
```

> Sử dụng giọng đọc nam Guy của Edge-TTS để lồng tiếng tiếng Anh cho video.

**Tăng tốc GPU CUDA + Mô hình nhận dạng chất lượng cao:**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --cuda --recogn_type 0 --model_name large-v3
```

**Tách giọng nói và nhạc nền (tăng chất lượng nhận dạng và lồng tiếng):**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --is_separate --cuda
```

**Nhúng phụ đề cứng song ngữ + Nhận dạng giọng nói lần 2:**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --subtitle_type 3 --recogn2pass --cuda
```

**Nhúng phụ đề mềm (có thể bật/tắt trên trình phát) + Tự động tăng tốc độ giọng đọc lồng tiếng:**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --subtitle_type 2 --voice_autorate --cuda
```

**Làm chậm video (làm chậm hình ảnh video tại phân đoạn mà giọng lồng tiếng dài hơn hình):**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --video_autorate --cuda
```

**Tùy chỉnh thư mục đầu ra + Giữ lại file tạm:**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --output-dir "D:/translated" --no-clear-cache
```

**Dịch sang tiếng Nhật và lồng tiếng:**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code ja --voice_role "ja-JP-KeitaNeural" --cuda
```

**Dịch sang tiếng Việt và lồng tiếng:**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code vi --voice_role "vi-VN-NamMinhNeural" --cuda
```

**Chạy chế độ im lặng (chỉ hiển thị lỗi ra màn hình):**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" -q
```

**Bật ghi log chi tiết (phục vụ mục đích gỡ lỗi debug):**

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" -v
```

---

## Công cụ truy vấn

### Liệt kê tất cả các kênh dịch vụ khả dụng

```bash
uv run cli.py --list providers
```

Ví dụ kết quả hiển thị:

```
=== Available Providers ===
...
```

### Liệt kê tất cả các mã ngôn ngữ được hỗ trợ

```bash
uv run cli.py --list languages
```

### Liệt kê các mô hình khả dụng của faster-whisper

```bash
uv run cli.py --list models
```

---

## Ví dụ đầy đủ

Giả định:
- File video tiếng Trung gốc: `60.mp4`
- File phụ đề tiếng Trung gốc: `zw.srt`
- Ngôn ngữ cần dịch sang: Tiếng Anh
- Giọng đọc lồng tiếng: `en-US-GuyNeural` của Edge-TTS

### Kịch bản 1: Chỉ nhận dạng giọng nói (Tạo phụ đề tiếng Trung)

```bash
uv run cli.py --task stt --name "60.mp4" --detect_language zh-cn --cuda
```

### Kịch bản 2: Chỉ dịch thuật phụ đề (Chuyển phụ đề Trung -> Anh)

```bash
uv run cli.py --task sts --name "zw.srt" --source_language_code zh-cn --target_language_code en
```

### Kịch bản 3: Chỉ lồng tiếng phụ đề (Tạo giọng lồng tiếng Anh từ phụ đề Trung)

```bash
uv run cli.py --task tts --name "zw.srt" --voice_role "en-US-GuyNeural" --target_language_code en
```

### Kịch bản 4: Dịch video đầy đủ (Trung -> Anh, có lồng tiếng)

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --cuda
```

### Kịch bản 5: Dịch video chất lượng cao (Tách nhạc nền + GPU + Nhận dạng lần 2)

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --cuda --is_separate --recogn2pass --model_name large-v3
```

### Kịch bản 6: Nhúng phụ đề song ngữ vào video

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --subtitle_type 3 --cuda
```

### Kịch bản 7: Dịch video sang tiếng Nhật

```bash
uv run cli.py --task vtv --name "60.mp4" --source_language_code zh-cn --target_language_code ja --voice_role "ja-JP-KeitaNeural" --cuda
```

### Kịch bản 8: Xử lý hàng loạt tệp bằng lệnh lặp (Shell Loop)

```bash
# Bash / Git Bash (trên Linux/macOS hoặc Git Bash Windows)
for f in *.mp4; do
  uv run cli.py --task vtv --name "$f" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --cuda
done
```

```powershell
# PowerShell (trên Windows)
Get-ChildItem *.mp4 | ForEach-Object {
  uv run cli.py --task vtv --name $_.FullName --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural" --cuda
}
```

---

## Câu hỏi thường gặp

### Q: Làm thế nào để xem tất cả các kênh lồng tiếng và giọng đọc khả dụng?

```bash
uv run cli.py --list providers
```

Hoặc bạn có thể mở giao diện GUI của phần mềm để kiểm tra trực tiếp danh sách cập nhật trong danh mục thiết lập TTS.

### Q: Làm cách nào để xử lý các đường dẫn tệp chứa khoảng trắng?

Vui lòng đặt đường dẫn tệp trong dấu ngoặc kép:

```bash
uv run cli.py --task vtv --name "D:/my videos/60.mp4" --source_language_code zh-cn --target_language_code en
```

### Q: Làm cách nào để kích hoạt GPU cho tăng tốc xử lý?

Thêm tham số `--cuda` vào câu lệnh của bạn. Lưu ý máy bạn cần trang bị card đồ họa NVIDIA và cấu hình các thư viện CUDA/cuDNN tương thích.

### Q: Làm sao để dịch bằng mô hình ngôn ngữ lớn (LLM) cục bộ?

Cần chạy dịch vụ LLM cục bộ (ví dụ: qua Ollama) trước, sau đó chỉ định mã số kênh dịch tương ứng qua tham số `--translate_type` (đảm bảo thông tin API của kênh đó đã được cấu hình trước trên bản GUI của phần mềm).

---

## Mã kết thúc (Exit Codes)

| Mã kết thúc | Ý nghĩa |
|--------|------|
| `0` | Tác vụ hoàn thành thành công |
| `1` | Xảy ra lỗi trong quá trình thực hiện |
| `130` | Bị gián đoạn bởi người dùng (Ctrl+C) |
| `2` | Lỗi tham số truyền vào (do argparse tự động kết thúc) |

---

## Tài liệu liên quan

- [Hướng dẫn bắt đầu](https://pyvideotrans.com/getstart)
- [Tài liệu chế độ CLI](https://pyvideotrans.com/cli)
- [Hướng dẫn kênh nhận dạng giọng nói](https://pyvideotrans.com/yuyinshibiequdao)
- [Hướng dẫn kênh dịch thuật](https://pyvideotrans.com/fanyiqudao)
- [Hướng dẫn kênh lồng tiếng](https://pyvideotrans.com/peiyinqudao)
- [Câu hỏi thường gặp FAQ](https://pyvideotrans.com/faq)
- [Kiến trúc kỹ thuật và nguyên lý](architecture_vi.md)
