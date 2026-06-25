# Kiến trúc kỹ thuật và Nguyên lý triển khai pyVideoTrans

`pyvideotrans` là một công cụ dịch thuật và lồng tiếng video mã nguồn mở mạnh mẽ (phiên bản v4.03), có khả năng tự động dịch video và lồng tiếng bằng ngôn ngữ đích. Triết lý thiết kế cốt lõi của nó là mô-đun hóa, luồng xử lý đa luồng (multi-threaded pipeline) và điều khiển linh hoạt qua các tổ hợp cờ trạng thái (flag) để hỗ trợ nhiều chế độ làm việc khác nhau.

![](https://pvtr2.pyvideotrans.com/1760167240539_image.png)

---

## I. Luồng xử lý cốt lõi

![](https://pvtr2.pyvideotrans.com/1760165489380_image.png)

Phần mềm chia nhỏ quy trình dịch thuật và lồng tiếng video thành **9 giai đoạn độc lập**, tạo thành một dây chuyền xử lý tự động. Mỗi nhiệm vụ được điều khiển bởi 5 cờ bolean (`should_recogn`, `should_trans`, `should_dubbing`, `should_hebing`, `should_separate`) để kiểm soát giai đoạn nào được chạy hoặc bỏ qua, từ đó hỗ trợ các chế độ làm việc khác nhau.

### 1.1 Chín giai đoạn xử lý

| Giai đoạn | Phương thức | Nhiệm vụ |
|------|------|------|
| **① Tiền xử lý** | `prepare()` | Tách luồng video không tiếng và luồng âm thanh gốc từ video; tùy chọn tách giọng nói/nhạc nền (UVR/Spleeter); tùy chọn giảm nhiễu; tạo thư mục tạm và thư mục đầu ra. |
| **② Nhận dạng giọng nói** | `recogn()` | Gọi công cụ ASR (mặc định là Faster-Whisper, hỗ trợ 22 kênh) để chuyển âm thanh thành phụ đề SRT có mốc thời gian; tùy chọn phục hồi dấu câu, cấu trúc lại câu bằng LLM. |
| **③ Phân tách người nói** | `diariz()` | Gọi mô hình phân tách người nói (hỗ trợ 4 backend: built, ali_CAM, pyannote, reverb), phân loại và gắn nhãn phụ đề theo người nói. |
| **④ Dịch phụ đề** | `trans()` | Dịch phụ đề SRT gốc sang phụ đề ngôn ngữ đích thông qua các kênh dịch thuật (hỗ trợ 24 kênh); hỗ trợ xuất phụ đề song ngữ. |
| **⑤ Lồng tiếng** | `dubbing()` | Dựa trên nội dung và mốc thời gian phụ đề dịch, gọi công cụ TTS (hỗ trợ 34 kênh) để tạo từng file âm thanh lồng tiếng tương ứng; hỗ trợ nhân bản giọng nói (cắt từ âm thanh gốc). |
| **⑥ Căn chỉnh âm hình** | `align()` | Sử dụng lớp `SpeedRate` để xử lý: tăng tốc giọng đọc lồng tiếng, làm chậm phân đoạn video, loại bỏ khoảng lặng giữa các phụ đề, căn chỉnh bắt buộc âm thanh lồng tiếng; hỗ trợ điều chỉnh âm lượng. |
| **⑦ Nhận dạng lần 2** | `recogn2pass()` | Nhận dạng lại giọng nói đối với tệp âm thanh lồng tiếng thu được để tạo phụ đề có mốc thời gian khớp chính xác hơn (chỉ chạy khi bật lồng tiếng và không nhúng phụ đề song ngữ). |
| **⑧ Tổng hợp cuối cùng** | `assembling()` | Kết hợp video không tiếng, âm thanh lồng tiếng, nhạc nền và phụ đề ngôn ngữ đích thành file video hoàn chỉnh cuối cùng (sử dụng ffmpeg). |
| **⑨ Kết thúc** | `task_done()` | Di chuyển file đầu ra từ thư mục tạm thời sang thư mục lưu trữ được chỉ định, dọn dẹp các tệp tạm và gửi thông báo hoàn thành. |

### 1.2 Các cờ điều khiển luồng xử lý

Được định nghĩa tại `videotrans/task/_base.py:20-29`, năm cờ điều khiển này được tự động tính toán trong `TransCreate.__post_init__()` dựa trên cấu hình người dùng thiết lập:

```python
should_recogn: bool    # Có cần nhận dạng giọng nói không (Mặc định là True nếu không có phụ đề sẵn)
should_trans: bool     # Có cần dịch thuật không (Mặc định là True nếu ngôn ngữ gốc ≠ ngôn ngữ đích)
should_dubbing: bool   # Có cần lồng tiếng không (True nếu chọn nhân vật lồng tiếng khác 'No')
should_hebing: bool    # Có cần ghép/nhúng không (True nếu không ở chế độ chỉ trích xuất 'tiqu' và có lồng tiếng hoặc nhúng phụ đề)
should_separate: bool  # Có cần tách giọng nói và nhạc nền không
```

### 1.3 Ví dụ về chuyển đổi chế độ làm việc

Các tính năng khác nhau được thực hiện thông qua tổ hợp của các cờ điều khiển:

| Tính năng | should_recogn | should_trans | should_dubbing | should_hebing |
|------|:---:|:---:|:---:|:---:|
| Dịch video lồng tiếng (Chế độ tiêu chuẩn) | ✓ | ✓ | ✓ | ✓ |
| Trích xuất phụ đề từ video/âm thanh (tiqu) | ✓ | Tùy chọn | ✗ | ✗ |
| Lồng tiếng cho file phụ đề có sẵn | ✗ | ✗ | ✓ | ✓ |
| Chỉ dịch file phụ đề SRT | ✗ | ✓ | ✗ | ✗ |

### 1.4 Hệ thống lớp con nhiệm vụ (Task Subclasses)

Lớp `BaseTask` có 4 lớp con cụ thể, mỗi lớp đáp ứng một kịch bản sử dụng khác nhau:

| Lớp con | Tệp nguồn | Cấu hình kế thừa | Kịch bản sử dụng |
|------|------|----------------|---------|
| `TransCreate` | `task/trans_create.py` | `TaskCfgVTT` | Quy trình dịch video lồng tiếng đầy đủ (Tiêu chuẩn hoặc trích xuất tiqu) |
| `SpeechToText` | `task/speech2text.py` | `TaskCfgSTT` | Nhận dạng giọng nói hàng loạt sang phụ đề |
| `DubbingSrt` | `task/dubbing.py` | `TaskCfgTTS` | Lồng tiếng hàng loạt cho phụ đề |
| `TranslateSrt` | `task/translate_srt.py` | `TaskCfgSTS` | Dịch thuật hàng loạt phụ đề SRT |

---

## II. Hệ thống lớp cấu hình nhiệm vụ (Task Configuration Dataclasses)

Phiên bản v4.03 đã tái cấu trúc cấu hình nhiệm vụ thành một hệ thống `@dataclass` kế thừa phân tầng (`videotrans/task/taskcfg.py`, 261 dòng):

```
@dataclass TaskCfgBase              ← Thuộc tính chung (đường dẫn, mã ngôn ngữ, thư mục tạm, v.v.)
    ├── @dataclass TaskCfgSTT       ← Các thuộc tính nhận dạng (recogn_type, model_name, rephrase, v.v.)
    ├── @dataclass TaskCfgTTS       ← Các thuộc tính lồng tiếng (tts_type, voice_role, voice_autorate, v.v.)
    ├── @dataclass TaskCfgSTS       ← Các thuộc tính dịch thuật (translate_type)
    └─ @dataclass TaskCfgVTT       ← Cấu hình đầy đủ cho dịch video (kế thừa STT + TTS + STS, bổ sung trường cho video)
```

Các lớp dữ liệu bổ trợ:

| Lớp dữ liệu | Tệp nguồn | Công dụng |
|--------|------|------|
| `InputFile` | `task/taskcfg.py` | Thông tin siêu dữ liệu của tệp đầu vào (name, dirname, noextname, basename, ext, uuid, target_dir), hỗ trợ truy cập dạng dict. |
| `SignMsg` | `task/taskcfg.py` | Đối tượng tin nhắn tín hiệu (type, uuid, text), cung cấp các phương thức `is_stop()` và `is_error()`, dùng để truyền tin giữa luồng Worker và luồng chính UI. |
| `SrtItem` | `task/taskcfg.py` | Đối tượng của một dòng phụ đề đơn lẻ (text, start_time, end_time, startraw, endraw, line, time, spk, filename). |

`SrtItem` hỗ trợ truy cập đồng thời dưới dạng thuộc tính (`item.text`) và dạng từ điển (`item['text']`), đồng thời hỗ trợ lặp thông qua phương thức `items()`.

---

## III. Kiến trúc xử lý tác vụ bất đồng bộ đa luồng

Phần mềm sử dụng kiến trúc đa luồng với hàng đợi (multi-threaded multi-queue) dựa trên **mẫu thiết kế "Người sản xuất - Người tiêu dùng" (Producer-Consumer)**. Luồng `MultVideo` đóng vai trò là người sản xuất, đẩy đối tượng nhiệm vụ vào hàng đợi đầu tiên của dây chuyền; 9 lớp con cụ thể của `BaseWorker` đóng vai trò người tiêu dùng, lắng nghe các hàng đợi chuyên biệt tương ứng.

### 3.1 Dây chuyền hàng đợi xử lý

```
                     MultVideo (Người sản xuất)
                           │
                    app_cfg.prepare_queue
                           ▼
                    WorkerPrepare (×N)
                    ┌────────┼────────┐
                    │ should_recogn ?  │
                    ▼        ▼        ▼
            regcon_queue  trans_queue  dubb_queue / assemb_queue / taskdone_queue
                 │
                 ▼
           WorkerRegcon (×N)
                 │
          diariz_queue
                 │
                 ▼
           WorkerDiariz (×N)
            ┌────┼────┐
            ▼    ▼    ▼
       trans_queue  dubb_queue  assemb_queue / taskdone_queue
            │
            ▼
      WorkerTrans (×1)
       ┌────┼────┐
       ▼    ▼    ▼
  dubb_queue  assemb_queue  taskdone_queue
       │
       ▼
 WorkerDubb (×1)
       │
  align_queue
       │
       ▼
 WorkerAlign (×1)
  ┌────┼────┐
  ▼    ▼    ▼
regcon2_queue  assemb_queue  taskdone_queue
  │
  ▼
WorkerRegcon2Pass (×1)
  ┌────┼────┐
  ▼    ▼
assemb_queue  taskdone_queue
  │
  ▼
WorkerAssemb (×N)
  │
taskdone_queue
  │
  ▼
WorkerTaskDone (×1)
  │
(Kết thúc)
```

### 3.2 Thiết kế lớp cơ sở Worker

Tất cả các luồng xử lý đều kế thừa từ lớp `BaseWorker(QThread)` (`videotrans/task/job.py:13-66`):

```python
class BaseWorker(QThread):
    def __init__(self, name, queue):
        self.name = name
        self.queue = queue

    def run(self):
        while True:
            if app_cfg.exit_soft:          # Cờ thoát mềm toàn cục
                return
            try:
                trk = self.queue.get(timeout=1)  # Chờ 1 giây để lấy nhiệm vụ từ hàng đợi
            except Empty:
                continue
            if trk.uuid in app_cfg.stoped_uuid_set:  # Nhiệm vụ đã bị dừng bởi người dùng
                continue
            try:
                self.process_task(trk)       # Lớp con triển khai logic xử lý cụ thể
            except Exception as e:
                self.handle_error(e, trk)    # Xử lý lỗi tập trung
```

Mỗi lớp con ghi đè (override) các phương thức sau:

| Phương thức | Giải thích |
|------|------|
| `process_task(trk)` | **Bắt buộc** — Thực hiện logic xử lý của giai đoạn đó và chuyển tiếp trk sang hàng đợi tiếp theo. |
| `get_error_prefix(trk)` | Tùy chọn — Trả về chuỗi tiền tố lỗi (ví dụ: `"Lỗi nhận dạng giọng nói [Faster-Whisper]"`). |
| `cleanup_on_error(trk)` | Tùy chọn — Logic dọn dẹp khi xảy ra lỗi. |

Phương thức `handle_error()` thống nhất gọi hàm `get_msg_from_except()` để chuyển đổi ngoại lệ hệ thống thành thông báo lỗi thân thiện dễ đọc cho người dùng, sau đó gửi đi thông qua `trk.signal()`.

### 3.3 Logic định tuyến của Worker

Sau khi thực hiện xong `process_task(trk)`, mỗi Worker sẽ dựa trên các cờ trạng thái của `trk` để quyết định hàng đợi tiếp theo cần chuyển tới:

```
WorkerPrepare     →  regcon_queue | trans_queue | dubb_queue | assemb_queue | taskdone_queue
WorkerRegcon      →  diariz_queue (Vô điều kiện)
WorkerDiariz      →  trans_queue | dubb_queue | assemb_queue | taskdone_queue (Lỗi phân tách speaker không chặn luồng)
WorkerTrans       →  dubb_queue | assemb_queue | taskdone_queue
WorkerDubb        →  align_queue (Vô điều kiện)
WorkerAlign       →  regcon2_queue | assemb_queue | taskdone_queue (Chỉ chuyển tới regcon2 khi có thuộc tính recogn2pass)
WorkerRegcon2Pass →  assemb_queue | taskdone_queue
WorkerAssemb      →  taskdone_queue (Vô điều kiện)
WorkerTaskDone    →  (Kết thúc xử lý)
```

### 3.4 Tính toán động số lượng luồng (Worker Instances)

Hàm `start_thread()` (`videotrans/task/job.py:206-245`) tự động tính toán số lượng phiên bản của từng loại Worker dựa trên cấu hình GPU/CPU hệ thống:

| Worker | Số luồng chạy | Lý do thiết lập |
|--------|--------|------|
| `WorkerPrepare` | 1 ~ 4 | Tác vụ nặng (mã hóa/giải mã video) |
| `WorkerRegcon` | 1 ~ 4 | Tác vụ nặng (ASR suy luận mô hình) |
| `WorkerDiariz` | 1 ~ 4 | Tác vụ nặng (phân tách người nói) |
| `WorkerTrans` | **Cố định 1** | Tác vụ gọi API mạng, giới hạn để tránh bị chặn tần suất yêu cầu (rate limit) |
| `WorkerDubb` | **Cố định 1** | Gọi API TTS trực tuyến, tránh bị chặn tần suất yêu cầu |
| `WorkerRegcon2Pass` | **Cố định 1** | Giai đoạn phụ trợ |
| `WorkerAlign` | **Cố định 1** | Công việc đồng bộ âm hình chạy đơn luồng |
| `WorkerAssemb` | 1 ~ 4 | Tác vụ nặng (ffmpeg tổng hợp/mã hóa video) |
| `WorkerTaskDone` | **Cố định 1** | Thao tác di chuyển/xóa file |

Logic tính toán `task_nums`: Ưu tiên sử dụng giá trị thiết lập thủ công từ `settings.process_max_gpu`; nếu không có, tự động dò tìm dựa trên `multi_gpus` + `NVIDIA_GPU_NUMS` (1 GPU = 1, 2-3 GPU = 2, ≥4 GPU = 4, không có GPU = 1).

### 3.5 Gửi nhiệm vụ hàng loạt: MultVideo

Lớp `MultVideo(QThread)` (`videotrans/task/mult_video.py`, 54 dòng) chịu trách nhiệm duyệt qua danh sách nhiều video do người dùng lựa chọn, tạo các đối tượng `TransCreate` tương ứng và đẩy chúng vào hàng đợi `prepare_queue`. Luồng hỗ trợ quản lý qua tham số `batch_nums`:

- `batch_nums == 0`: Đẩy toàn bộ các nhiệm vụ vào hàng đợi cùng lúc (xử lý song song tối đa).
- `batch_nums == 1`: Đẩy tuần tự, chỉ đẩy nhiệm vụ tiếp theo sau khi nhiệm vụ trước đó đã hoàn thành 100%.
- `batch_nums > 1`: Xử lý theo từng nhóm (batch), đẩy N nhiệm vụ cùng lúc và đợi cả nhóm hoàn thành mới đẩy nhóm kế tiếp.

### 3.6 Cơ chế thoát mềm (Soft Exit)

Khi cờ toàn cục `app_cfg.exit_soft` được đặt thành `True`, tất cả các Worker sẽ nhận biết ở chu kỳ kiểm tra tiếp theo và thoát khỏi vòng lặp một cách an toàn. Tập hợp `app_cfg.stoped_uuid_set` được dùng để lưu các UUID của nhiệm vụ bị người dùng hủy bỏ, giúp các Worker tự động bỏ qua chúng khi lấy ra khỏi hàng đợi.

---

## IV. Thiết kế lớp cốt lõi và Quan hệ kế thừa

### 4.1 Sơ đồ kế thừa các lớp chính

```
@dataclass BaseCon                    ← videotrans/configure/base.py
    │                                  Định nghĩa các thuộc tính cơ bản và phương thức tiện ích
    ├── @dataclass BaseTask           ← videotrans/task/_base.py
    │       │                          Định nghĩa 8 phương thức giai đoạn trống và 5 cờ điều khiển
    │       ├── @dataclass TransCreate ← videotrans/task/trans_create.py (Lớp xử lý dịch video lõi, ~1678 dòng)
    │       ├── @dataclass SpeechToText ← videotrans/task/speech2text.py (Dịch âm thanh sang phụ đề hàng loạt)
    │       ├── @dataclass DubbingSrt  ← videotrans/task/dubbing.py (Lồng tiếng cho phụ đề hàng loạt)
    │       └── @dataclass TranslateSrt ← videotrans/task/translate_srt.py (Dịch phụ đề SRT hàng loạt)
    │
    ├── @dataclass BaseRecogn         ← videotrans/recognition/_base.py
    │       │                          Cắt âm thanh bằng VAD, ghép phụ đề, xử lý CJK
    │       └── 22 lớp con (nạp động)  Triển khai cụ thể cho từng kênh nhận dạng giọng nói ASR
    │
    ├── @dataclass BaseTrans          ← videotrans/translator/_base.py
    │       │                          Quản lý bộ nhớ đệm MD5, lập lịch dịch từng dòng hoặc dịch toàn bộ
    │       └── 24 lớp con (nạp động)  Triển khai cụ thể cho từng kênh dịch thuật
    │
    └── @dataclass BaseTTS            ← videotrans/tts/_base.py
            │                          Lập lịch điều phối đa luồng/bất đồng bộ song song
            └── 34 lớp con (nạp động)  Triển khai cụ thể cho từng kênh tổng hợp giọng nói TTS
```

Tất cả các lớp kênh (channel) đều là `@dataclass`, thực hiện việc khởi tạo thông qua phương thức `__post_init__` thay vì hàm khởi dựng truyền thống `__init__`.

### 4.2 BaseCon —— Lớp cơ sở cấp cao nhất

Tệp nguồn `videotrans/configure/base.py` (296 dòng) định nghĩa các chức năng cốt lõi dùng chung cho tất cả các lớp kế thừa:

| Phương thức | Nhiệm vụ |
|------|------|
| `_exit()` | Kiểm tra xem nhiệm vụ hiện tại có cần dừng lại hay không (dựa trên cờ `exit_soft` hoặc UUID nằm trong `stoped_uuid_set`). |
| `signal(**kwargs)` | Gửi thông tin thông báo lên giao diện UI (qua `push_queue()` → `SignalHub`. Ở chế độ dòng lệnh CLI, thông tin được in trực tiếp ra màn hình terminal). |
| `_set_proxy(type)` | Thiết lập hoặc xóa bỏ proxy mạng HTTP (tác động trực tiếp lên biến `app_cfg.proxy` và các biến môi trường hệ thống). |
| `_new_process(callback, title, is_cuda, kwargs)` | **Thực thi các tác vụ tốn tài nguyên hoặc dễ treo trong một tiến trình con độc lập** (trả về kết quả dưới dạng tuple `(data, error)`). |
| `_signal_of_process(logs_file)` | Dò tìm trạng thái và tiến độ của tiến trình con thông qua việc kiểm tra thuộc tính mtime của file nhật ký JSON. |
| `convert_to_wav()` | Chuẩn hóa toàn bộ âm thanh đầu vào về dạng file WAV âm thanh nổi kép (stereo) tần số mẫu 48kHz (tùy chọn loại bỏ khoảng lặng). |
| `_base64_to_audio()` / `_audio_to_base64()` | Mã hóa và giải mã âm thanh dưới dạng chuỗi Base64. |
| `_process_callback(data)` | Hàm gọi ngược (callback) cập nhật tiến độ tải xuống (chuyển tiếp tới `signal()`). |

Hàm `BaseCon.__post_init__()` sẽ tự động gọi phương thức `_set_proxy(type='set')` để cập nhật cấu hình proxy mạng ngay khi đối tượng được khởi tạo.

### 4.3 BaseTask —— Lớp cơ sở của nhiệm vụ

Tệp nguồn `videotrans/task/_base.py:10-167` định nghĩa các phương thức đại diện cho từng giai đoạn và các tiện ích dùng chung cho các lớp con:

**Các phương thức giai đoạn** (Mặc định không thực hiện gì, được ghi đè bởi các lớp con):
`prepare()`, `recogn()`, `diariz()`, `trans()`, `dubbing()`, `align()`, `assembling()`, `task_done()`

> Lưu ý: Phương thức `recogn2pass()` được định nghĩa riêng trong lớp `TransCreate`, không nằm trong lớp cơ sở `BaseTask`.

**Các phương thức dùng chung**:

| Phương thức | Nhiệm vụ |
|------|------|
| `_unlink_size0(file)` | Tự động phát hiện và xóa các tệp lỗi có dung lượng bằng 0. |
| `_save_srt_target(srtstr, file)` | Định dạng danh sách đối tượng `SrtItem` thành chuỗi chuẩn SRT, ghi ra file và phát tín hiệu `replace_subtitle`. |
| `check_target_sub(source, target)` | Kiểm tra tính nhất quán về số lượng dòng phụ đề trước và sau khi dịch; nếu lệch dòng, tự động điều chỉnh so khớp dựa trên mốc thời gian. |
| `set_end(succeed=False)` | Đánh dấu tác vụ kết thúc, gửi thông báo thành công và xóa bỏ thư mục lưu trữ file tạm. |
| `_edgetts_single(target_audio, kwargs)` | Gọi bất đồng bộ dịch vụ Edge-TTS để lồng tiếng cho một dòng thoại đơn lẻ (tích hợp khả năng tự động thử lại khi proxy lỗi). |

### 4.4 TransCreate —— Triển khai cốt lõi dịch video

Tệp nguồn `videotrans/task/trans_create.py` (khoảng 1678 dòng) chứa toàn bộ các logic chi tiết của quy trình xử lý 9 giai đoạn. Một số phương thức nội bộ quan trọng:

| Phương thức | Nhiệm vụ |
|------|------|
| `__post_init__()` | Khởi tạo toàn bộ đường dẫn tệp tin, tính toán các cờ điều khiển luồng và khởi chạy luồng đo đếm thời gian tiến độ. |
| `_split_novoice_byraw()` | Tách luồng video không tiếng từ video gốc (ưu tiên tăng tốc phần cứng giải mã h264_cuvid, tự động chuyển về libx264 nếu lỗi). |
| `_split_audio_byraw()` | Trích xuất âm thanh PCM đơn kênh 16kHz + tùy chọn tách giọng nói/nhạc nền từ video gốc. |
| `_tts()` | Xây dựng danh sách `queue_tts` (gồm các tệp âm thanh tham chiếu phục vụ nhân bản giọng), gọi phương thức `tts.run()` để thực hiện lồng tiếng. |
| `_create_ref_from_vocal()` | Sử dụng đa luồng (ThreadPoolExecutor) để cắt các đoạn âm thanh tương ứng từ luồng giọng nói gốc làm mẫu tham chiếu cho nhân bản giọng. |
| `_recogn_succeed()` | Xử lý sau khi kết thúc giai đoạn nhận dạng giọng nói (ở chế độ trích xuất tiqu, sao chép file phụ đề ra thư mục kết quả). |
| `_back_music()` | Trộn (mix) tệp nhạc nền do người dùng cung cấp với luồng âm thanh lồng tiếng mới. |
| `_separate()` | Trộn lại phần âm thanh nền tách được trước đó vào luồng âm thanh lồng tiếng mới. |
| `_process_subtitles()` | Thực hiện việc ghi/nhúng phụ đề vào khung hình video (hỗ trợ phụ đề cứng/mềm, đơn ngữ/song ngữ, định dạng kiểu chữ). |

### 4.5 Kênh tiến trình con độc lập (Subprocess Channel)

Để ngăn việc mô hình nhận dạng giọng nói `faster-whisper` bị sập (crash) kéo theo toàn bộ ứng dụng chính bị tắt theo, các công cụ nhận dạng nặng như `Faster-Whisper`, `Faster-Whisper-XXL`, `Whisper.cpp` (và một số công cụ TTS như `QWEN3LOCAL_TTS`) đều được ủy quyền chạy thông qua tiến trình con độc lập bằng phương thức `BaseCon._new_process()`, được quản lý bởi `GlobalProcessManager`.

Các tiến trình con báo cáo tiến độ bằng cách ghi vào file log JSON. Phương thức `BaseCon._signal_of_process()` chạy trên một luồng giám sát riêng để liên tục kiểm tra thời gian thay đổi (mtime) của file log này. Khi phát hiện mtime thay đổi, nó sẽ phân tích nội dung JSON và gọi `signal()` để cập nhật lên giao diện.

---

## V. Cấu trúc hệ thống cấu hình

Hệ thống lưu trữ cấu hình phần mềm được phân chia thành 3 lớp rõ rệt (`videotrans/configure/config.py`, 902 dòng), tất cả đều được xây dựng qua định dạng `@dataclass`:

| Lớp cấu hình | Khả năng lưu trữ | Công dụng | Ví dụ thuộc tính |
|--------|--------|------|---------|
| `AppCfg` | Chỉ lưu trên RAM | Quản lý các luồng hàng đợi, trạng thái chạy, điều khiển luồng và ngữ cảnh thời gian thực. | `prepare_queue`, `exit_soft`, `stoped_uuid_set`, `current_status`, `line_roles`, `exec_mode`, `video_codec`, `onlyone_source_sub`, `onlyone_target_sub`, `proxy`, `SUPPORT_LANG` |
| `AppSettings` | Tệp `videotrans/cfg.json` | Cài đặt toàn cục của phần mềm, danh sách các mô hình khả dụng. | `homedir`, `model_list`, `vad_type`, `cuda_com_type` |
| `AppParams` | Tệp `videotrans/params.json` | Các tham số tùy chọn yêu thích của người dùng, khóa API (API Key). | `source_language`, `recogn_type`, `chatgpt_key`, `voice_role`, `app_mode` |

Các biến đơn lệ (singleton) quan trọng được tự động khởi tạo khi ứng dụng tải mô-đun:

```python
app_cfg: AppCfg = AppCfg()        # Trạng thái chạy thời gian thực (chứa 9 thực thể Queue)
settings: AppSettings = AppSettings()  # Tải thông tin từ file cfg.json
params: AppParams = AppParams()    # Tải thông tin từ file params.json
```

### 5.1 Các đặc tính của AppSettings

- Hỗ trợ truy cập nhanh dạng từ điển `settings['key']` hoặc phương thức lấy dữ liệu có giá trị dự phòng `settings.get('key', default)`.
- Phương thức `get()` tự động kiểm tra và chuyển đổi kiểu dữ liệu phù hợp cho các trường số (dựa trên danh sách cho phép kiểu int và float).
- Phương thức `_get_defaults()` định nghĩa các giá trị mặc định cho khoảng 100 mục cấu hình của phần mềm.
- Hỗ trợ tự động ánh xạ các tên thuộc tính có chứa dấu gạch ngang (ví dụ: chuyển từ `"initial_prompt_zh-cn"` sang `"initial_prompt_zh_cn"`).

### 5.2 Các đặc tính của AppParams

- Phương thức `_get_defaults()` định nghĩa giá trị mặc định cho khoảng 100 tham số người dùng tùy chọn.
- Phương thức `getset_params(update_data)` hỗ trợ cập nhật hàng loạt các tham số (ví dụ: dùng để thu thập toàn bộ dữ liệu từ các ô điều khiển trên giao diện chính khi người dùng nhấn nút bắt đầu).
- Toàn bộ các trường lưu trữ API Key đều được quản lý tại đây để phục vụ cho hàm xác thực `is_input_api()`.

### 5.3 Trạng thái chạy AppCfg

- Khởi tạo 9 hàng đợi `Queue(maxsize=0)` (dung lượng vô hạn) từ luồng chuẩn bị `prepare_queue` đến luồng hoàn thành `taskdone_queue`.
- `queue_novice: Dict` — Dùng để theo dõi tiến độ tách video không tiếng của từng tệp (khóa key = uuid, giá trị value = 'ing' hoặc 'end').
- `line_roles: Dict` — Lưu trữ thông tin về nhân vật giọng đọc được phân bổ cho từng dòng thoại ở chế độ dịch đơn video.
- `child_forms: Dict` — Lưu trữ bộ nhớ đệm (cache) của các cửa sổ phụ đã mở để tránh việc khởi tạo lại trùng lặp gây tốn RAM.
- `exec_mode` — Chế độ thực thi chương trình (lưu giá trị giao diện đồ họa 'gui' hoặc dòng lệnh 'cli').
- `video_codec` / `codec_cache` — Bộ nhớ đệm của bộ giải mã/mã hóa video.
- `onlyone_source_sub` / `onlyone_target_sub` / `onlyone_trans` — Quản lý trạng thái xử lý phụ đề trong chế độ dịch đơn video.
- `SUPPORT_LANG` — Danh sách các ngôn ngữ được phần mềm hỗ trợ xử lý.

### 5.4 Khởi tạo biến môi trường hệ thống

Hàm khởi tạo môi trường `_set_env()` được tự động thực thi khi nạp thư viện cấu hình (`videotrans/configure/config.py:44-72`), thiết lập các biến sau:
- Hướng thư mục lưu trữ cache của các mô hình AI (`MODELSCOPE_CACHE`, `HF_HOME`, `HF_HUB_CACHE`) về thư mục con `models/` nằm trong thư mục gốc dự án.
- Thiết lập biến giao diện `QT_API = 'pyside6'`.
- Nối thêm đường dẫn của thư mục phần mềm ffmpeg/sox đi kèm vào biến môi trường hệ thống `PATH`.
- Thiết lập biến `OMP_NUM_THREADS = 1` để tối ưu tài nguyên tính toán.
- Tăng thời gian chờ tải mô hình từ HuggingFace lên `HF_HUB_DOWNLOAD_TIMEOUT = 3600` giây.

---

## VI. GlobalProcessManager —— Quản lý bể tiến trình con

Tệp nguồn `videotrans/process/signelobj.py` (167 dòng) triển khai một đối tượng quản lý tiến trình con đơn lệ ở cấp độ lớp (Class-level Singleton) mang tên `GlobalProcessManager`:

```
GlobalProcessManager (Đơn lệ cấp lớp)
    ├── _executor_cpu: multiprocessing.Pool
    │       số lượng tiến trình con = max(min(RAM_khả_dụng/4GB, 8, số_nhân_CPU), 1)  ← Tính toán dựa trên dung lượng RAM trống thực tế
    │       maxtasksperchild = 1  ← Mỗi tiến trình con chỉ thực hiện đúng 1 tác vụ rồi tự giải phóng khởi động lại để chống rò rỉ RAM
    │
    └── _executor_gpu: multiprocessing.Pool
            số lượng tiến trình con = số lượng GPU (Ưu tiên thiết lập thủ công tại settings.process_max_gpu)
            maxtasksperchild = 1
```

### 6.1 Quy mô bể tiến trình CPU

Để tránh làm đứng máy người dùng, số lượng tiến trình con CPU không sử dụng giá trị cố định, mà được tính toán động thông qua hàm `psutil.virtual_memory().available` để đo dung lượng RAM trống hiện tại của hệ thống. Cứ mỗi 4GB RAM trống sẽ cho phép mở thêm 1 tiến trình con, giới hạn trong khoảng **1 đến 8** tiến trình và không vượt quá số lượng luồng vật lý của CPU (`os.cpu_count()`). Giá trị này cũng có thể bị ghi đè nếu người dùng thiết lập thủ công mục `settings.process_max`.

### 6.2 Quy mô bể tiến trình GPU

Ưu tiên sử dụng giá trị thiết lập thủ công từ cấu hình `settings.process_max_gpu`; nếu không có cấu hình này, hệ thống tự động phát hiện dựa trên `multi_gpus` và `NVIDIA_GPU_NUMS` (không có card = 1, có card nhưng không chạy chế độ đa GPU = 1, chạy đa GPU = giá trị nhỏ nhất trong các số: số GPU thực tế, 8, số nhân CPU).

### 6.3 Giao diện gửi nhiệm vụ

```python
GlobalProcessManager.submit_task_cpu(func, **kwargs)   → AsyncResultFutureWrapper
GlobalProcessManager.submit_task_gpu(func, **kwargs)   → AsyncResultFutureWrapper
```

Lớp `AsyncResultFutureWrapper` sẽ bao bọc kết quả trả về bất đồng bộ `AsyncResult` từ hàm `Pool.apply_async` thành giao diện tương thích với lớp `Future` chuẩn của Python (hỗ trợ gọi các phương thức `.result()` và `.done()`).

### 6.4 Kịch bản áp dụng

Được gọi tập trung qua phương thức `BaseCon._new_process()`, dùng để chạy các tác vụ nặng: suy luận mô hình nhận dạng giọng nói ASR, tổng hợp giọng đọc TTS, loại bỏ tiếng ồn nền, tách giọng hát/nhạc nền, phân tách người nói, phục hồi dấu câu tự động. Toàn bộ các tác vụ này chạy trong tiến trình riêng, nếu sập sẽ không làm ảnh hưởng đến tiến trình GUI chính.

---

## VII. SignalHub —— Trung tâm tin nhắn xuyên luồng

Tệp nguồn `videotrans/configure/signal_hub.py` (33 dòng) thực hiện việc chuyển phát tin nhắn xuyên luồng dựa trên hệ thống Tín hiệu (Signal) đơn lệ của thư viện Qt:

```python
class SignalHub(QObject):
    _instance = None
    new_message = Signal(str, object)  # Cấu trúc gồm (uuid, SignMsg)

    @classmethod
    def instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

    @Slot(str, object)
    def post(self, uuid=None, data=None):
        self.new_message.emit(uuid, data)  # Tự động chọn kết nối kiểu QueuedConnection khi gửi xuyên luồng
```

### Luồng đi của tin nhắn tín hiệu

```
BaseCon.signal(**kwargs)
    → push_queue(uuid, SignMsg(**kwargs))     [Cập nhật tại configure/config.py]
        → SignalHub.instance().post(uuid, data)
            → Tín hiệu new_message (Kết nối kiểu QueuedConnection)
                → Giao diện nhận và xử lý WinAction.update_data(uuid, data)   [mainwin/_actions.py]
                    → Phân phối theo loại tin nhắn (type):
                        'logs'|'error'|'succeed'|'set_precent' → Cập nhật thanh tiến trình set_process_btn_text()
                        'edit_subtitle_source' → Mở hộp thoại sửa phụ đề gốc EditRecognResultDialog
                        'edit_subtitle_target' → Mở hộp thoại chọn giọng đọc cho từng dòng SpeakerAssignmentDialog
                        'edit_dubbing' → Mở hộp thoại sửa file âm thanh lồng tiếng EditDubbingResultDialog
                        'replace_subtitle' → Cập nhật khu vực hiển thị phụ đề
                        'end' → Chuyển trạng thái kết thúc nhiệm vụ update_status('end')
```

### Danh mục các loại tin nhắn chính

| Loại tin nhắn | Ý nghĩa | Logic xử lý tương ứng |
|------|------|---------|
| `logs` | Nhật ký xử lý thông thường | Cập nhật dòng chữ hiển thị tiến độ trên thanh tiến trình |
| `error` | Báo lỗi | Đổi màu thanh tiến trình sang màu đỏ, đưa nhiệm vụ vào hàng đợi để thử lại |
| `succeed` | Thành công | Đổi màu thanh tiến trình sang màu xanh lá, đánh dấu nhiệm vụ hoàn thành |
| `set_precent` | Phần trăm tiến độ | Định dạng hiển thị chuỗi thời gian chạy và phần trăm hoàn thành |
| `edit_subtitle_source` | Yêu cầu sửa phụ đề gốc | Điểm tạm dừng số ① trong chế độ dịch đơn video |
| `edit_subtitle_target` | Yêu cầu sửa phụ đề dịch | Điểm tạm dừng số ② trong chế độ dịch đơn video |
| `edit_dubbing` | Yêu cầu sửa kết quả lồng tiếng | Điểm tạm dừng số ③ trong chế độ dịch đơn video |
| `replace_subtitle` | Thay thế nội dung phụ đề | Cập nhật lại khung chữ hiển thị phụ đề chính |
| `subtitle` | Đẩy thêm dòng phụ đề mới | Đưa thêm một dòng phụ đề mới hiển thị lên màn hình |
| `end` | Hoàn thành toàn bộ | Thực hiện gọi hàm update_status('end') |
| `disabled_edit` | Khóa sửa phụ đề | Khóa không cho người dùng sửa chữ trong chế độ chạy hàng loạt |

---

## VIII. Nạp động các kênh dịch vụ (Dynamic Channel Loading)

Tệp nguồn `videotrans/__init__.py` (35 dòng) cung cấp một cơ chế nạp động thư viện theo dạng tải chậm (lazy loading) để tiết kiệm RAM:

```python
@dataclass
class ChannelProvider:
    name: str           # Tên hiển thị trên giao diện người dùng
    imp: str            # Hậu tố nạp thư viện (ví dụ: "._whisper" → nạp "videotrans.recognition._whisper")
    key_name: str|None  # Tên khóa API Key tương ứng lưu trong file params.json
    win: str|None       # Tên cửa sổ cấu hình tương ứng trong winform
```

Hàm nạp lớp động `get_class()`:

```python
def get_class(channel_id=0, provider_type=None, _ID_NAME_DICT=None):
    _key = f'{provider_type}-{channel_id}'
    if _key in _loaded_modules:
        return _loaded_modules[_key]
    module = importlib.import_module(f'videotrans.{provider_type}{_module_map.imp}')
    for _, obj in inspect.getmembers(module, inspect.isclass):
        if obj.__module__ == module.__name__:
            _loaded_modules[_key] = obj
            return obj
```

Hệ thống từ điển kênh `_ID_NAME_DICT` của 3 mô-đun chính:

| Mô-đun | Số lượng kênh | Vị trí định nghĩa |
|------|--------|---------|
| Nhận dạng giọng nói (recognition) | 22 | `videotrans/recognition/__init__.py:48-79` |
| Dịch thuật phụ đề (translator) | 24 | `videotrans/translator/__init__.py:60-90` |
| Tổng hợp lồng tiếng (tts) | **34** | `videotrans/tts/__init__.py:75-116` |

### 8.1 Hàm thực thi tập trung

Mỗi mô-đun đều xuất ra một hàm `run()` làm cổng thực thi chính, bên trong hàm sẽ gọi `get_class()` để lấy lớp kênh tương ứng, khởi tạo thực thể đối tượng và chạy:

```python
# Gọi nhận dạng giọng nói trong recognition/__init__.py
def run(*, recogn_type, detect_language, audio_file, ...) -> List[SrtItem]:
    _cls = get_class(recogn_type, "recognition", _ID_NAME_DICT)
    return _cls(**kwargs).run()

# Gọi dịch thuật trong translator/__init__.py
def run(*, translate_type, text_list, source_code, target_code, ...) -> List[SrtItem]:
    _cls = get_class(translate_type, "translator", _ID_NAME_DICT)
    return _cls(**kwargs).run()

# Gọi lồng tiếng trong tts/__init__.py
def run(*, queue_tts, language, tts_type, ...) -> None:
    _cls = get_class(tts_type, "tts", _ID_NAME_DICT)
    return _cls(**kwargs).run()
```

### 8.2 Kiểm tra API Key

Mỗi mô-đun cung cấp hàm kiểm tra API Key `is_input_api(channel_id)` để dò xem khóa cấu hình tương ứng trong `params.json` đã được điền hay chưa. Nếu chưa điền, phần mềm sẽ tự động bật cửa sổ cấu hình của kênh đó lên để người dùng điền thông tin trước khi chạy.

### 8.3 Bộ nhớ đệm dịch thuật

Lớp cơ sở `BaseTrans` (`videotrans/translator/_base.py`) quản lý bộ nhớ đệm dịch thuật thông qua mã băm MD5 để tránh dịch trùng lặp tốn chi phí:
- Khóa băm cache key = `md5(tên_kênh + địa_chỉ_api + tên_mô_hình + ngôn_ngữ_gốc + ngôn_ngữ_đích + chuỗi_văn_bản)`
- File cache được lưu trữ trong thư mục `{TEMP_ROOT}/translate_cache/`
- Đọc cache qua phương thức `_get_cache()`, lưu cache mới qua `_set_cache()`.

### 8.4 Xử lý đặc biệt cho nhóm ngôn ngữ CJK (Trung, Nhật, Hàn, Việt...)

Tệp nguồn `videotrans/recognition/_base.py:58-80` trong hàm `__post_init__` của lớp `BaseRecogn` sẽ kiểm tra ngôn ngữ và thực hiện tối ưu riêng cho nhóm ngôn ngữ Đông Á CJK (zh, ja, ko, yu, th, km, yue):
- Cờ `join_word_flag`: Nhóm ngôn ngữ CJK sẽ ghép nối các từ sát nhau mà không chèn dấu cách khoảng trắng (các ngôn ngữ hệ Latinh khác sẽ tự động chèn khoảng trắng).
- Thuộc tính `maxlen`: Giới hạn độ dài tối đa của chữ trên một dòng phụ đề là `settings.cjk_len` (mặc định 15 chữ) đối với ngôn ngữ CJK, và `settings.other_len` (mặc định 40 chữ) đối với các ngôn ngữ khác.
- Thuộc tính `jianfan`: Tự động chuyển đổi chữ Trung Quốc giản thể/phồn thể nếu cờ cấu hình `settings.zh_hant_s` được bật.

---

## IX. Chế độ tương tác dịch đơn video (Interactive Single Video Mode)

Khi người dùng chọn xử lý duy nhất **1 video** ở chế độ chạy **Tiêu chuẩn (biaozhun)**, chương trình sẽ áp dụng một mô hình xử lý tương tác trực quan hoàn toàn khác so với luồng chạy hàng loạt hàng đợi.

### 9.1 Luồng thực thi trong duy nhất 1 luồng QThread

Tiến trình xử lý được thực hiện tuần tự trong luồng `Worker(QThread)` của file `videotrans/task/only_one.py` (148 dòng), gửi tín hiệu trao đổi thông tin với luồng giao diện chính thông qua cờ `uito = Signal(str, SignMsg)`:

```
Worker.run()
    ├── Khởi tạo trk = TransCreate(cfg=TaskCfgVTT(**self.cfg))
    ├── Chạy trk.prepare()
    ├── Chạy trk.recogn()
    ├── Chạy trk.diariz()
    ├── [Điểm tạm dừng ①] → Phát tín hiệu _post(type='edit_subtitle_source')
    │    Chờ người dùng hiệu đính lại chữ/mốc thời gian phụ đề gốc → Nhấn "Xác nhận" trên bảng sửa
    ├── Chạy trk.trans() (Nếu cờ should_trans được bật)
    ├── [Điểm tạm dừng ②] → Phát tín hiệu _post(type='edit_subtitle_target')
    │    Chờ người dùng hiệu đính phụ đề dịch + gán giọng lồng tiếng cho từng dòng thoại → Nhấn "Xác nhận"
    ├── Chạy trk.dubbing() (Nếu cờ should_dubbing được bật)
    ├── [Điểm tạm dừng ③] → Phát tín hiệu _post(type='edit_dubbing')
    │    Chờ người dùng nghe thử và sửa lại file ghi âm lồng tiếng tương ứng → Nhấn "Xác nhận"
    ├── Chạy trk.align() (Căn chỉnh âm hình)
    ├── Chạy trk.recogn2pass() (Nhận dạng giọng nói lần 2)
    ├── Chạy trk.assembling() (Ghép nối tổng hợp video)
    └── Chạy trk.task_done() (Hoàn thành)
```

### 9.2 Sự khác biệt cốt lõi so với chế độ hàng loạt (Batch Mode)

| Đặc tính | Chế độ dịch đơn video | Chế độ dịch hàng loạt |
|------|-----------|---------|
| Cơ chế luồng chạy | Chạy tuần tự trực tiếp trong luồng `Worker(QThread)`, không dùng hàng đợi | Đẩy qua dây chuyền hàng đợi từ luồng này sang luồng khác của 9 Worker |
| Trao đổi tín hiệu | Gửi trực tiếp qua tín hiệu `uito` kết nối với `WinAction.update_data()` | Gửi qua `BaseCon.signal()` → `push_queue()` → trung tâm `SignalHub` |
| Hỗ trợ tạm dừng hiệu đính | Hỗ trợ 3 điểm dừng để người dùng biên tập sửa đổi chữ và giọng đọc | Không hỗ trợ dừng biên tập giữa chừng, chạy một mạch đến hết |
| Hiển thị tiến độ | Cập nhật chữ và thời gian chạy trực tiếp lên khung biên tập phụ đề | Hiển thị qua thanh tiến trình (progress bar) và nút bấm trạng thái |

---

## X. Động cơ đồng bộ âm hình (SpeedRate)

Tệp nguồn `videotrans/task/_rate.py` (877 dòng) chứa logic cốt lõi của 2 động cơ đồng bộ âm hình: `SpeedRate` và `TtsSpeedRate`. Chi tiết nguyên lý hoạt động vui lòng xem thêm tại file [Hướng dẫn nguyên lý căn chỉnh đồng bộ thời gian âm thanh - hình ảnh](Synchronize_vi.md).

---

## XI. Khởi chạy ứng dụng và Triển khai giao diện đồ họa GUI

### 11.1 Luồng khởi chạy ứng dụng

Tệp `sp.py` (221 dòng) là điểm đầu vào duy nhất để khởi động phần mềm, quy trình diễn ra như sau:

```
Tệp sp.py (Thực thi khi chạy trực tiếp __main__)
  │
  ├── 1. Gọi multiprocessing.freeze_support() và đặt phương thức khởi chạy set_start_method('spawn')
  ├── 2. Thiết lập hàm qInstallMessageHandler() để ẩn bớt các cảnh báo dòng lệnh không cần thiết của Qt
  ├── 3. Đăng ký hàm atexit.register(cleanup) để đảm bảo dọn dẹp file tạm khi tắt phần mềm
  ├── 4. Bật cấu hình hỗ trợ hiển thị DPI cao QApplication.setHighDpiScaleFactorRoundingPolicy()
  ├── 5. Khởi tạo đối tượng ứng dụng chính QApplication
  ├── 6. Kiểm tra xem phần mềm có đang được chạy từ một gói nén ảo không (dạng file đóng gói PyInstaller)
  ├── 7. Hiển thị cửa sổ chào mừng StartWindow (màn hình Splash screen không viền, mờ nhẹ)
  │       └── Kích hoạt bộ hẹn giờ QTimer.singleShot(100ms) → chuyển tới hàm initialize_full_app()
  │           ├── Hướng toàn bộ đầu ra dòng lệnh sys.stdout và sys.stderr vào file nhật ký log hàng ngày
  │           ├── Thiết lập hàm bắt lỗi toàn cục show_global_error_dialog để hiện bảng báo lỗi khi crash
  │           ├── Phân tích các tham số truyền vào từ dòng lệnh (ví dụ: mã ngôn ngữ --lang)
  │           ├── Tải tài nguyên giao diện được biên dịch sẵn darkstyle_rc.py
  │           ├── Nạp bảng phong cách hiển thị QSS (từ file videotrans/styles/style.qss)
  │           ├── Đọc lại kích thước cửa sổ lưu từ phiên chạy trước (thông qua lớp QSettings)
  │           └── Khởi tạo đối tượng MainWindow chính → kết nối uito với splash.update_lable để hiện tiến trình nạp
  │               └── Bên trong MainWindow.__init__()
  │                   ├── Gọi setupUi() → điền dữ liệu vào các ô lựa chọn (kênh dịch, nhận dạng, TTS, ngôn ngữ...)
  │                   ├── Khởi chạy luồng check card đồ họa AiLoaderThread → sau khi có kết quả GPU, gọi _start_workers()
  │                   ├── Gọi _start_workers() → kích hoạt hàm start_thread() để mở 9 luồng Worker nền
  │                   ├── Gọi _set_default() → khôi phục lại các tùy chọn gần nhất của người dùng
  │                   ├── Gọi _bind_signal() → liên kết sự kiện hành động cho khoảng 60 nút bấm/ô chọn trên giao diện
  │                   ├── Kết nối tín hiệu tin nhắn mới SignalHub.new_message với hàm thu nhận win_action.update_data
  │                   └── Phát tín hiệu uito.emit('end') → đóng màn hình chào mừng Splash screen, hiện giao diện chính
  └── 8. Kích hoạt vòng lặp sự kiện Qt app.exec()
```

---

## XII. Thư mục mã nguồn và Phân bổ chức năng

```
Thư mục gốc /
├── sp.py                       # ★ Điểm đầu vào chính khởi chạy ứng dụng GUI (221 dòng)
├── cli.py                      # ★ Điểm đầu vào chính khởi chạy ứng dụng dòng lệnh CLI
├── models/                     # Thư mục lưu trữ các mô hình AI ngoại tuyến tải về (định dạng ONNX, bin, v.v.)
├── logs/                       # Lưu trữ file nhật ký hoạt động hàng ngày (tên file dạng YYYYMMDD.log)
├── ffmpeg/                     # Thư mục chứa các file nhị phân ffmpeg và sox đi kèm
├── f5-tts/                     # Thư mục lưu các file âm thanh mẫu phục vụ nhân bản giọng nói F5-TTS
├── docs/                       # Thư mục chứa tài liệu hướng dẫn kỹ thuật
├── tmp/                        # Thư mục gốc chứa các tệp tin tạm thời
│   ├── _temp/                  # Thư mục tạm của các tiến trình xử lý video/âm thanh đang chạy
│   └── translate_cache/        # Thư mục lưu các file cache kết quả dịch thuật dạng MD5
│
└── videotrans/                 # Toàn bộ mã nguồn cốt lõi của phần mềm
    │   __init__.py             # ★ Khai báo phiên bản VERSION, định nghĩa đối tượng nạp lớp động get_class()
    │   cfg.json                # Lưu cài đặt cấu hình phần mềm AppSettings
    │   params.json             # Lưu các tùy chọn tham số của người dùng AppParams
    │   codec.json              # File cache thông tin về bộ mã hóa video khả dụng của hệ thống
    │
    ├── configure/              # Quản lý cấu hình toàn cục, hàng đợi và lớp cơ sở
    │   ├── config.py           # ★ Định nghĩa AppCfg / AppSettings / AppParams / cơ chế ghi log / dịch thuật tr() (902 dòng)
    │   ├── base.py             # ★ Định nghĩa lớp cơ sở BaseCon (gồm các hàm chạy tiến trình con, gửi tín hiệu, proxy) (296 dòng)
    │   ├── contants.py         # ★ Khai báo các hằng số toàn cục (danh sách mô hình, danh sách ngôn ngữ, ký tự dấu câu)
    │   ├── excepts.py          # ★ Khai báo hệ thống lỗi ngoại lệ và hàm lọc lỗi get_msg_from_except() (376 dòng)
    │   ├── signal_hub.py       # ★ Khai báo lớp phát tín hiệu xuyên luồng đơn lệ SignalHub (33 dòng)
    │   └── whispernet_config.py # Cấu hình cho động cơ nhận dạng Whisper.NET
    │
    ├── task/                   # Logic xử lý tác vụ và quản lý các luồng Worker nền
    │   ├── _base.py            # ★ Định nghĩa lớp cơ sở BaseTask (chứa 8 phương thức giai đoạn và cờ trạng thái) (167 dòng)
    │   ├── taskcfg.py          # ★ Các lớp cấu hình Dataclasses của tác vụ (TaskCfgBase, VTT, InputFile, SignMsg) (261 dòng)
    │   ├── trans_create.py     # ★ Logic cốt lõi xử lý dịch video 9 giai đoạn TransCreate (~1678 dòng)
    │   ├── speech2text.py      # Lớp con SpeechToText phục vụ chuyển âm thanh sang phụ đề hàng loạt
    │   ├── dubbing.py          # Lớp con DubbingSrt phục vụ lồng tiếng cho phụ đề hàng loạt
    │   ├── translate_srt.py    # Lớp con TranslateSrt phục vụ dịch thuật phụ đề hàng loạt
    │   ├── job.py              # ★ Triển khai chi tiết 9 lớp con Worker và hàm khởi chạy start_thread() (245 dòng)
    │   ├── only_one.py         # ★ Triển khai luồng Worker đơn tương tác dịch 1 video (148 dòng)
    │   ├── mult_video.py       # Triển khai luồng gửi hàng loạt video MultVideo (54 dòng)
    │   ├── _rate.py            # Động cơ đồng bộ thời gian âm hình SpeedRate và TtsSpeedRate (877 dòng)
    │   └── ...
    │
    ├── recognition/            # Thư mục chứa 22 kênh nhận dạng giọng nói ASR
    │   ├── __init__.py         # Quản lý ID kênh nhận dạng, nạp động lớp nhận dạng qua run()
    │   ├── _base.py            # Lớp cơ sở nhận dạng BaseRecogn (chứa chia câu VAD, gộp dòng CJK, xử lý độ dài) (400 dòng)
    │   └── _*.py               # Triển khai riêng của từng kênh (whisper, whisperx, whispernet, funasr...)
    │
    ├── translator/             # Thư mục chứa 24 kênh dịch thuật phụ đề
    │   ├── __init__.py         # Quản lý ID kênh dịch, mã ngôn ngữ, nạp động lớp qua run() (860 dòng)
    │   ├── _base.py            # Lớp cơ sở dịch thuật BaseTrans (chứa cache bhash MD5, lập lịch đa luồng) (176 dòng)
    │   └── _*.py               # Triển khai riêng của từng kênh (google, deepl, chatgpt, deepseek...)
    │
    ├── tts/                    # Thư mục chứa 34 kênh lồng tiếng tổng hợp giọng đọc TTS
    │   ├── __init__.py         # Quản lý ID kênh TTS, cờ nhân bản giọng nói, nạp động lớp qua run() (192 dòng)
    │   ├── _base.py            # Lớp cơ sở lồng tiếng BaseTTS (điều phối hàng đợi và luồng chạy bất đồng bộ) (304 dòng)
    │   └── _*.py               # Triển khai riêng của từng kênh (edgetts, openaitts, azuretts, cosyvoice...)
    │
    ├── process/                # Các tiến trình con độc lập chạy nền
    │   ├── signelobj.py        # ★ Bể tiến trình CPU và GPU toàn cục GlobalProcessManager (167 dòng)
    │   ├── prepare_audio.py    # Tiến trình con tách nhạc nền, khử ồn, nhận dạng speaker, khôi phục dấu câu
    │   └── stt_fun.py          # Điểm đầu vào của các tiến trình con nhận dạng giọng nói ngoại tuyến
    │
    ├── mainwin/                # Khởi tạo giao diện chính và phân bổ xử lý sự kiện
    │   ├── main_win.py         # ★ MainWindow khởi tạo giao diện và nạp các thành phần GUI (528 dòng)
    │   ├── _actions.py         # ★ Bộ điều khiển WinAction quản lý thu thập dữ liệu và chuyển đổi trạng thái (798 dòng)
    │   └── _actions_base.py    # ★ Lớp bổ trợ WinActionBase quản lý nút bấm, card đồ họa, proxy, file (590 dòng)
    │
    ├── component/              # Các thành phần điều khiển giao diện UI dùng chung
    │   ├── progressbar.py      # Thanh tiến trình cập nhật trạng thái động bằng màu sắc
    │   ├── onlyone_set_recogn.py # Bảng sửa phụ đề gốc ở chế độ tương tác dịch đơn video
    │   ├── onlyone_set_role.py   # Bảng sửa phụ đề dịch và chọn giọng lồng tiếng cho từng câu thoại
    │   └── onlyone_set_editdubb.py # Bảng nghe thử và thu âm lại file lồng tiếng từng câu thoại bị lệch
    │
    ├── winform/                # Khởi tạo lười (lazy-load) khoảng 65 cửa sổ phụ cấu hình kênh hoặc tính năng riêng biệt
    │   ├── __init__.py         # Trình quản lý nạp động cửa sổ phụ qua phương thức get_win() (91 dòng)
    │   └── *.py                # File cấu hình giao diện riêng của từng kênh dịch, ASR, TTS hoặc tính năng phụ
    │
    ├── styles/                 # Chứa file giao diện QSS, logo và tài nguyên đa phương tiện
    ├── prompts/                # Thư mục chứa 31 file mẫu câu lệnh (prompt) dịch thuật bằng AI
    └── voicejson/              # Thư mục chứa 14 file cấu hình danh sách giọng đọc của các kênh TTS
```

---

## XIII. Hướng dẫn phát triển mở rộng kênh dịch vụ

### 13.1 Hướng dẫn thêm một kênh dịch thuật mới

Giả sử bạn muốn thêm một kênh dịch thuật mới mang tên `MyTranslator`:

#### Bước 1: Tạo tệp triển khai kênh mới

Trong thư mục `videotrans/translator/`, tạo tệp mới đặt tên là `_mytranslator.py`:

```python
from dataclasses import dataclass
from videotrans.translator._base import BaseTrans

@dataclass
class MyTranslator(BaseTrans):
    def __post_init__(self):
        super().__post_init__()
        self.api_url = 'https://api.example.com/translate'

    def _item_task(self, data: dict) -> str:
        # Nhận vào thông tin dòng chữ, mã ngôn ngữ gốc và mã ngôn ngữ đích
        text = data['text']
        source = data['source_code']
        target = data['target_code']
        # Gọi hàm gửi yêu cầu đến dịch vụ dịch thuật của bạn để lấy kết quả
        result = call_my_api(text, source, target)
        return result
```

#### Bước 2: Phân bổ ID kênh dịch thuật và đăng ký vào hệ thống

Mở file `videotrans/translator/__init__.py`:

```python
MYTRANSLATOR_INDEX = 24   # Chỉ định một số nguyên ID duy nhất không trùng lặp

# Khai báo đăng ký kênh mới vào cuối từ điển _ID_NAME_DICT:
_ID_NAME_DICT[MYTRANSLATOR_INDEX] = ChannelProvider(
    "My Translator",          # Tên hiển thị trên giao diện chính
    imp="._mytranslator",     # Tên import tệp triển khai ở Bước 1
    key_name="mytranslator_key", # Tên thuộc tính lưu khóa API trong params.json
    win="mytranslator"        # Tên tệp cửa sổ cấu hình trong winform
)
```

#### Bước 3: Cấu hình thêm thuộc tính lưu trữ cho người dùng

Mở file `videotrans/configure/config.py`, tìm hàm `AppParams._get_defaults()` để thêm các trường lưu cấu hình mặc định cho kênh mới:

```python
"mytranslator_key": "",
"mytranslator_model": "model-v1",
```

#### Bước 4: Tạo cửa sổ cấu hình trên giao diện

Trong thư mục `videotrans/winform/`, tạo tệp cấu hình giao diện `mytranslator.py` chứa phương thức hiển thị `openwin()`. Sau đó đăng ký cửa sổ phụ này vào từ điển `_module_map` của file `videotrans/winform/__init__.py`:

```python
"mytranslator": ".mytranslator",
```

---

### 13.2 Hướng dẫn thêm một kênh lồng tiếng TTS mới

Quy trình thực hiện hoàn toàn tương tự như việc thêm kênh dịch thuật:

1. Tạo file triển khai `videotrans/tts/_mytts.py` kế thừa từ lớp cơ sở `BaseTTS`.
2. Mở file `videotrans/tts/__init__.py` để phân bổ ID và đăng ký thông tin vào từ điển `_ID_NAME_DICT`.
3. Nếu kênh TTS này hỗ trợ tính năng nhân bản giọng nói, hãy điền ID của kênh đó vào danh sách `SUPPORT_CLONE`.
4. Nếu giọng lồng tiếng của kênh này thay đổi động dựa trên việc chọn ngôn ngữ đích, hãy điền ID vào danh sách `CHANGE_BY_LANGUAGE`.
5. Khai báo các trường cấu hình mặc định (như API Key, URL kết nối) trong hàm `AppParams._get_defaults()`.
6. Thiết lập và đăng ký cửa sổ cấu hình GUI trong thư mục `videotrans/winform/` và file `_module_map`.

---

### 13.3 Hướng dẫn thêm một kênh nhận dạng giọng nói ASR mới

Quy trình thực hiện tương tự: tạo file triển khai kế thừa từ lớp cơ sở `BaseRecogn`, đăng ký ID kênh trong `videotrans/recognition/__init__.py` và bắt buộc phải viết hàm `.run()` trả về dữ liệu danh sách đối tượng chứa phụ đề `List[SrtItem]`.
