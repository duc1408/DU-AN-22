> Nhà tài trợ: **[Recall.ai](https://www.recall.ai/product/meeting-transcription-api?utm_source=github&utm_medium=sponsorship&utm_campaign=jianchang512-pyvideotrans) - Meeting Transcription API**
>
> Nếu bạn đang tìm kiếm một API nhận dạng giọng nói cho các cuộc họp, hãy tham khảo **[Recall.ai](https://www.recall.ai/product/meeting-transcription-api?utm_source=github&utm_medium=sponsorship&utm_campaign=jianchang512-pyvideotrans)** , một dịch vụ API tích hợp tốt với Zoom, Google Meet, Microsoft Teams và nhiều nền tảng khác.


---

# pyVideoTrans

<div align="center">

**Một công cụ nguồn mở mạnh mẽ để Dịch video / Nhận dạng giọng nói / Lồng tiếng AI / Dịch thuật phụ đề**

[English](../README.md) | [**Tài liệu**](https://pyvideotrans.com) | [**Hỏi đáp trực tuyến**](https://bbs.pyvideotrans.com) | [**Hướng dẫn WebUI**](webui_vi.md)

[![License](https://img.shields.io/badge/License-GPL_v3-blue.svg)](../LICENSE) [![Python](https://img.shields.io/badge/Python-3.10%2B-green.svg)](https://www.python.org/) [![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)]()

</div>

**pyVideoTrans** được phát triển nhằm mục đích dịch video từ ngôn ngữ này sang ngôn ngữ khác một cách liền mạch, cung cấp quy trình xử lý toàn diện bao gồm: nhận dạng giọng nói, dịch thuật phụ đề, lồng tiếng đa vai trò bằng AI và đồng bộ hóa âm thanh - hình ảnh. Công cụ này hỗ trợ cả việc triển khai ngoại tuyến (offline) trên máy cục bộ và tích hợp nhiều dịch vụ API trực tuyến phổ biến hiện nay.

<img width="1566" height="912" alt="image" src="https://github.com/user-attachments/assets/2d5bd178-3dc0-45ee-bc1c-dbb5f6705cf4" />

---

## ✨ Các tính năng cốt lõi

> [Kiến trúc kỹ thuật và nguyên lý xử lý](architecture_vi.md)

- **🎥 Dịch video tự động hoàn toàn**: Quy trình xử lý chỉ với một cú nhấp chuột: Nhận dạng giọng nói (ASR) → Dịch phụ đề → Tổng hợp giọng nói (TTS) → Đồng bộ tổng hợp video mới.
- **🎙️ Nhận dạng giọng nói / Tạo phụ đề**: Hỗ trợ chuyển đổi hàng loạt tệp video/âm thanh thành phụ đề định dạng SRT, tích hợp tính năng **Phân tách người nói (Speaker Diarization)** để tự động nhận dạng các vai trò khác nhau.
- **🗣️ Lồng tiếng AI đa vai trò**: Hỗ trợ chỉ định các giọng đọc AI khác nhau cho từng người nói trong video.
- **🧬 Nhân bản giọng nói (Voice Cloning)**: Tích hợp các mô hình tiên tiến như **F5-TTS, CosyVoice, GPT-SoVITS** cho phép nhân bản giọng nói với dữ liệu mẫu tối thiểu (zero-shot).
- **🧠 Hỗ trợ mô hình đa dạng**:
  - **ASR**: Faster-Whisper (Cục bộ), OpenAI Whisper, Alibaba Qwen, ByteDance Volcano, Azure, Google, v.v.
  - **Dịch thuật bằng LLM**: DeepSeek, ChatGPT, Claude, Claude, Gemini, MiniMax, Ollama (Cục bộ), Alibaba Bailian, v.v.
  - **TTS**: Edge-TTS (Miễn phí), OpenAI, Azure, Minimaxi, ChatTTS, ChatterBox, v.v.
- **🖥️ Biên tập tương tác trực tiếp**: Cho phép tạm dừng ở từng giai đoạn (nhận dạng, dịch thuật, lồng tiếng) để người dùng hiệu đính thủ công, đảm bảo độ chính xác tối đa.
- **🛠️ Hộp công cụ tiện ích**: Bao gồm các công cụ bổ trợ như tách giọng nói/nhạc nền, ghép video/phụ đề, căn chỉnh âm hình, so khớp bản thảo văn bản.
- **💻 Chế độ dòng lệnh (CLI)**: Hỗ trợ chạy không giao diện (headless), cực kỳ tiện lợi cho việc triển khai trên máy chủ hoặc xử lý hàng loạt.
- **🌐 Giao diện Web (WebUI)**: Giao diện dựa trên nền tảng trình duyệt, thích hợp cho việc truy cập từ xa hoặc triển khai trong mạng nội bộ.

---

## 🚀 Khởi động nhanh (Dành cho người dùng Windows)

Chúng tôi cung cấp phiên bản đóng gói sẵn dạng `.exe` cho người dùng Windows 10/11, không cần phải cấu hình môi trường Python phức tạp.

1. **Tải xuống**: [Nhấp vào đây để tải xuống phiên bản đóng gói sẵn mới nhất](https://github.com/jianchang512/pyvideotrans/releases)
2. **Giải nén**: Giải nén tệp đã tải xuống vào một đường dẫn **không chứa ký tự tiếng Trung hoặc khoảng trắng** (ví dụ: `D:\pyVideoTrans`).
3. **Chạy ứng dụng**: Nhấp đúp chuột vào tệp `sp.exe` trong thư mục giải nén để khởi động phần mềm.

> **Lưu ý**:
> * Không chạy phần mềm trực tiếp từ bên trong ứng dụng giải nén (như WinRAR, 7-Zip).
> * Để sử dụng tính năng tăng tốc phần cứng bằng GPU, vui lòng đảm bảo bạn đã cài đặt **CUDA 12.8** và **cuDNN 9.11**.

---

## 🛠️ Triển khai từ mã nguồn (Dành cho nhà phát triển macOS / Linux / Windows)

Chúng tôi khuyên bạn nên sử dụng công cụ quản lý gói **[`uv`](https://docs.astral.sh/uv/)** để có tốc độ cài đặt nhanh nhất và khả năng cô lập môi trường tốt nhất.

### 1. Chuẩn bị trước

* **Python**: Khuyên dùng phiên bản 3.10.
* **FFmpeg**: Bắt buộc phải cài đặt và cấu hình đường dẫn trong biến môi trường (Environment Variables).
  * **macOS**: `brew install ffmpeg libsndfile git`
  * **Linux (Ubuntu/Debian)**: `sudo apt-get install ffmpeg libsndfile1-dev`
  * **Windows**: [Tải xuống FFmpeg](https://ffmpeg.org/download.html) và cấu hình Path, hoặc đặt trực tiếp các tệp `ffmpeg.exe` và `ffprobe.exe` vào thư mục gốc của dự án.

### 2. Cài đặt uv (nếu chưa cài đặt)

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### 3. Sao chép dự án và Cài đặt

```bash
git clone https://github.com/jianchang512/pyvideotrans.git
cd pyvideotrans
uv sync
```

> Mặc định, các kênh cục bộ như `qwen-tts`, `qwen-asr`, `moss-tts`, `chatterbox` sẽ không được cài đặt tự động. Nếu bạn cần cài đặt tất cả, vui lòng chạy lệnh `uv sync --all-extra`
> - Cài đặt riêng `qwen-tts`：`uv sync --extra qwentts`
> - Cài đặt riêng `qwen-asr`：`uv sync --extra qwenasr`
> - Cài đặt riêng `moss-tts`：`uv sync --extra mosstts`
> - Cài đặt riêng `chatterbox`：`uv sync --extra chatterbox`

### 4. Khởi chạy phần mềm

**Giao diện đồ họa (GUI)**:
```bash
uv run sp.py
```

**Sử dụng dòng lệnh (CLI)**:

```bash
# Ví dụ dịch video
uv run cli.py --task vtv --name "./video.mp4" --source_language_code zh-cn --target_language_code en --voice_role "en-US-GuyNeural"

# Ví dụ nhận dạng giọng nói sang phụ đề
uv run cli.py --task stt --name "./audio.wav" --model_name large-v3

# Ví dụ dịch thuật phụ đề
uv run cli.py --task sts --name "./subs.srt" --target_language_code en

# Ví dụ lồng tiếng cho phụ đề
uv run cli.py --task tts --name "./subs.srt" --voice_role "zh-CN-YunyangNeural"
```

> [Hướng dẫn chi tiết tham số CLI](cli_vi.md)

**Khởi chạy WebUI** (thích hợp cho truy cập từ xa hoặc mạng nội bộ):
```bash
uv sync --extra webui
uv run webui.py
```

> [Hướng dẫn sử dụng WebUI](webui_vi.md)

### 5. (Tùy chọn) Cấu hình tăng tốc GPU

1. Nếu máy tính của bạn được trang bị card đồ họa NVIDIA, vui lòng chạy các lệnh sau để cài đặt phiên bản PyTorch hỗ trợ tăng tốc CUDA:

```bash
# Gỡ cài đặt phiên bản CPU
uv remove torch torchaudio

# Cài đặt phiên bản hỗ trợ CUDA (ví dụ với CUDA 12.x)
uv add torch==2.7 torchaudio==2.7 --index-url https://download.pytorch.org/whl/cu128
uv add nvidia-cublas-cu12 nvidia-cudnn-cu12
```

2. [Nếu bạn sử dụng card đồ họa AMD, bạn có thể xem tài liệu này để cấu hình tăng tốc](whisper_net_setup_vi.md)

---

## 🧩 Các kênh và Mô hình được hỗ trợ (Một phần)

| Danh mục | Kênh / Mô hình | Mô tả |
| :--- | :--- | :--- |
| **Nhận dạng giọng nói (ASR)** | **Faster-Whisper** (Cục bộ) | Khuyên dùng, tốc độ xử lý nhanh, độ chính xác cao |
| | WhisperX / Parakeet | Hỗ trợ đối chiếu thời gian và phân tách người nói |
| | Ali Qwen3-ASR / ByteDance Volcano | Dịch vụ API trực tuyến, cực kỳ hiệu quả với tiếng Trung |
| **Dịch thuật (LLM/MT)** | **DeepSeek** / ChatGPT | Hỗ trợ hiểu ngữ cảnh, bản dịch tự nhiên hơn |
| | MiniMax AI | Mô hình lớn MiniMax M3, tương thích giao diện OpenAI |
| | Google / Microsoft | Dịch máy truyền thống, tốc độ phản hồi nhanh |
| | Ollama / M2M100 | Hỗ trợ dịch thuật ngoại tuyến hoàn toàn cục bộ |
| **Tổng hợp giọng nói (TTS)** | **Edge-TTS** | API miễn phí của Microsoft, giọng đọc tự nhiên |
| | **F5-TTS / CosyVoice** | Hỗ trợ **Nhân bản giọng nói**, yêu cầu cài đặt cục bộ |
| | GPT-SoVITS / ChatTTS | Dịch vụ tổng hợp giọng nói nguồn mở chất lượng cao |
| | 302.AI / OpenAI / Azure | Dịch vụ API thương mại chất lượng cao |

---

## 📚 Tài liệu & Hỗ trợ

* **Trang tài liệu chính thức**: [https://pyvideotrans.com](https://pyvideotrans.com) (bao gồm hướng dẫn chi tiết, cấu hình API, các câu hỏi thường gặp)
* **Cộng đồng hỏi đáp**: [https://bbs.pyvideotrans.com](https://bbs.pyvideotrans.com) (bạn có thể gửi tệp lỗi, AI sẽ tự động phân tích và phản hồi)
* **GitHub Wiki**: [Kiến trúc kỹ thuật](architecture_vi.md) | [Tài liệu CLI](cli_vi.md) | [Hướng dẫn WebUI](webui_vi.md) | [Nguyên lý đồng bộ âm hình](Synchronize_vi.md) | [Câu hỏi thường gặp](faq_vi.md)

## ⚠️ Miễn trừ trách nhiệm

Phần mềm này là một dự án mã nguồn mở, miễn phí và phi thương mại. Người dùng tự chịu trách nhiệm pháp lý hoàn toàn đối với mọi hậu quả phát sinh từ việc sử dụng phần mềm này (bao gồm nhưng không giới hạn ở việc gọi API của bên thứ ba, xử lý video có bản quyền). Vui lòng tuân thủ luật pháp, quy định của địa phương và điều khoản sử dụng của nhà cung cấp dịch vụ liên quan.

## 🙏 Lời cảm ơn

Dự án này sử dụng và dựa trên các dự án mã nguồn mở tuyệt vời sau (một phần):

* [FFmpeg](https://github.com/FFmpeg/FFmpeg)
* [PySide6](https://pypi.org/project/PySide6/)
* [faster-whisper](https://github.com/SYSTRAN/faster-whisper)
* [openai-whisper](https://github.com/openai/whisper)
* [edge-tts](https://github.com/rany2/edge-tts)
* [F5-TTS](https://github.com/SWivid/F5-TTS)
* [CosyVoice](https://github.com/FunAudioLLM/CosyVoice)
* [Gradio](https://www.gradio.app/) (WebUI)

---

*Tác giả: [jianchang512](https://github.com/jianchang512)*
