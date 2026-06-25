# Hướng dẫn nguyên lý căn chỉnh đồng bộ thời gian âm thanh - hình ảnh

Tài liệu này giải thích chi tiết về nguyên lý triển khai của mô đun "Lồng tiếng, Phụ đề và Căn chỉnh Video" (`videotrans/task/_rate.py`) trong pyVideoTrans. Mô đun này có nhiệm vụ căn chỉnh chính xác âm thanh lồng tiếng sau dịch thuật với video không tiếng gốc trên mốc thời gian, sau đó kết hợp lại để tạo thành một video mới chạy mượt mà.

---

## Mục lục

- [I. Bối cảnh bài toán](#i-bối-cảnh-bài-toán)
- [II. Các thách thức cốt lõi](#ii-các-thách-thức-cốt-lõi)
- [III. Tổng quan chiến lược căn chỉnh](#iii-tổng-quan-chiến-lược-căn-chỉnh)
- [IV. Tiền xử lý dữ liệu: Mở rộng mốc thời gian](#iv-tiền-xử-lý-dữ-liệu-mở-rộng-mốc-thời-gian)
- [V. Chế độ 1: Chỉ tăng tốc âm thanh](#v-chế-độ-1-chỉ-tăng-tốc-âm-thanh)
- [VI. Chế độ 2: Chỉ làm chậm video](#vi-chế-độ-2-chỉ-làm-chậm-video)
- [VII. Chế độ 3: Kết hợp âm thanh + video](#vii-chế-độ-3-kết-hợp-âm-thanh--video)
- [VIII. Chế độ 4: Ghép nối không thay đổi tốc độ](#viii-chế-độ-4-ghép-nối-không-thay-đổi-tốc-độ)
- [IX. Chi tiết triển khai thay đổi tốc độ âm thanh](#ix-chi-tiết-triển-khai-thay-đổi-tốc-độ-âm-thanh)
- [X. Chi tiết triển khai thay đổi tốc độ video](#x-chi-tiết-triển-khai-thay-đổi-tốc-độ-video)
- [XI. Ghép nối và căn chỉnh âm thanh cuối cùng](#xi-ghép-nối-và-căn-chỉnh-âm-thanh-cuối-cùng)
- [XII. Ghép nối các phân đoạn video](#xii-ghép-nối-các-phân-đoạn-video)
- [XIII. TtsSpeedRate: Trường hợp chỉ lồng tiếng](#xiii-ttsspeedrate-trường-hợp-chỉ-lồng-tiếng)
- [XIV. Khả năng tương thích đa nền tảng](#xiv-khả-năng-tương-thích-đa-nền-tảng)
- [XV. Các giới hạn đã biết và lưu ý](#xv-các-giới-hạn-đã-biết-và-lưu-ý)

---

## I. Bối cảnh bài toán

Quy trình đầy đủ của pyVideoTrans để dịch video từ ngôn ngữ A sang ngôn ngữ B:

```text
Video gốc (Ngôn ngữ A)
    │
    ├─→ Tách luồng video không tiếng (novoice.mp4)
    ├─→ Trích xuất âm thanh → Nhận dạng giọng nói (ASR) → Phụ đề ngôn ngữ A
    ├─→ Dịch thuật → Phụ đề ngôn ngữ B
    ├─→ Lồng tiếng (TTS) → Các tệp âm thanh lồng tiếng ngôn ngữ B riêng lẻ (wav)
    │
    └─→ 【Mô đun này】Kết hợp lồng tiếng B + Phụ đề B + Video không tiếng → Căn chỉnh ghép nối → Video mới
```

**Mâu thuẫn cốt lõi**: Khi diễn đạt cùng một ý nghĩa bằng các ngôn ngữ khác nhau, số lượng âm tiết và cấu trúc ngữ pháp sẽ khác nhau, dẫn đến thời lượng âm thanh lồng tiếng không khớp với thời lượng phụ đề gốc ban đầu.

**Ví dụ**:
- Phân đoạn phụ đề gốc tiếng Trung: `0:03.000 ~ 0:06.000` (Thời lượng 3 giây)
- Âm thanh lồng tiếng tiếng Anh sau dịch: Thực tế mất tới 4.2 giây để phát xong
- Chênh lệch: Tràn thời gian `4.2 - 3.0 = 1.2` giây

Nếu không xử lý, nó sẽ dẫn đến:
1. Âm thanh lồng tiếng lệch khớp với hình ảnh video (miệng nhân vật đã dừng nhưng tiếng vẫn nói tiếp).
2. Phụ đề không đồng bộ với giọng nói.
3. Độ lệch của các mốc thời gian phụ đề tích lũy tăng dần theo thời gian video.

---

## II. Các thách thức cốt lõi

### 2.1 Giới hạn độ chính xác của FFmpeg

FFmpeg không thể xử lý chính xác tuyệt đối video ở mức mili-giây. Khi thay đổi tốc độ bằng cách sử dụng PTS (Presentation Time Stamp), video đầu ra cuối cùng có thể ngắn hơn hoặc dài hơn một chút so với thời lượng mong muốn. Sai số này rất nhỏ ở một phân đoạn đơn lẻ (vài mili-giây), nhưng khi ghép hàng trăm phân đoạn lại với nhau, sai số sẽ tích lũy thành một khoảng đáng kể.

### 2.2 Tốc độ khung hình không cố định (Variable Frame Rate)

Tốc độ khung hình của video có thể là 25fps, 29.97fps, 30fps, v.v. Thời lượng của một số phân đoạn có thể nhỏ hơn thời lượng của 1 khung hình. Việc thay đổi tốc độ đối với các phân đoạn cực ngắn này bằng FFmpeg rất dễ gặp lỗi thất bại.

### 2.3 Tính không thể dự đoán của sự khác biệt ngôn ngữ

Sự thay đổi thời lượng lồng tiếng phụ thuộc vào:
- Sự khác biệt về mật độ âm tiết giữa ngôn ngữ gốc và ngôn ngữ đích.
- Đặc tính tốc độ nói của từng công cụ TTS.
- Sự khác biệt về cấu trúc ngữ pháp của các câu.
- Có sử dụng công nghệ nhân bản giọng nói hay không (trong chế độ nhân bản giọng nói, sự thay đổi thời lượng càng khó kiểm soát hơn).

---

## III. Tổng quan chiến lược căn chỉnh

pyVideoTrans cung cấp bốn chế độ căn chỉnh, được kiểm soát bởi hai cờ bolean:

| Chế độ | `should_audiorate` | `should_videorate` | Giải thích |
|------|:---:|:---:|------|
| **Chỉ tăng tốc âm thanh** | ✅ | ✗ | Tăng tốc độ âm thanh lồng tiếng để khớp với thời lượng phụ đề |
| **Chỉ làm chậm video** | ✗ | ✅ | Làm chậm video để khớp với thời lượng âm thanh lồng tiếng |
| **Kết hợp âm thanh + video** | ✅ | ✅ | Cả hai cùng gánh một nửa thời gian chênh lệch |
| **Ghép nối không thay đổi tốc độ** | ✗ | ✗ | Ghép trực tiếp, chèn khoảng lặng vào khoảng trống |

```text
                    ┌───────────────────────────────────┐
                    │  Thời lượng tiếng > Phụ đề gốc?   │
                    └─────────────────┬─────────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
                   Không                                Có
                    │                                   │
            ┌───────┴───────┐                 ┌─────────┴─────────┐
            │ Không xử lý    │                 │ Tính tỷ lệ tăng   │
            │ Ghép trực tiếp │                 │ tốc ratio=tiếng/gốc│
            └───────────────┘                 └─────────┬─────────┘
                                                        │
                                            ┌───────────┴───────────┐
                                            │                       │
                                    ratio ≤ 1.2                 ratio > 1.2
                                            │                       │
                                   ┌────────┴────────┐     ┌────────┴────────┐
                                   │ Chỉ tăng tốc    │     │ Mỗi bên gánh    │
                                   │ âm thanh        │     │ một nửa thời    │
                                   │ Không chậm video│     │ gian chênh lệch │
                                   └─────────────────┘     └─────────────────┘
```

---

## IV. Tiền xử lý dữ liệu: Mở rộng mốc thời gian

### 4.1 Vấn đề: Khoảng lặng giữa các phụ đề

Các mốc thời gian phụ đề gốc thường chứa các khoảng trống lặng không có thoại:

```text
Phụ đề 1: 0:00.000 ~ 0:03.000  (Thời lượng 3s)
          ─────── Khoảng lặng 0.5s ───────
Phụ đề 2: 0:03.500 ~ 0:07.000  (Thời lượng 3.5s)
```

Nếu trực tiếp tăng tốc âm thanh lồng tiếng của phụ đề 1 để khớp với thời lượng 3s, trong khi thực tế không gian trống khả dụng kéo dài tới 3.5s (cho đến khi phụ đề tiếp theo bắt đầu), chúng ta sẽ lãng phí mất 0.5s không gian đệm này, dẫn đến việc tăng tốc âm thanh không cần thiết hoặc bị ép quá nhanh.

### 4.2 Giải pháp: Mở rộng thời gian kết thúc của mỗi phụ đề

Trong giai đoạn tiền xử lý, mốc `end_time` của mỗi dòng phụ đề được sửa đổi thành mốc `start_time` của dòng phụ đề tiếp theo, từ đó gộp khoảng lặng vào phạm vi thời gian khả dụng của dòng phụ đề hiện tại:

```text
Trước khi xử lý:
Phụ đề 1: start=0ms,    end=3000ms   (3s)
Phụ đề 2: start=3500ms, end=7000ms   (3.5s)

Sau khi xử lý:
Phụ đề 1: start=0ms,    end=3500ms   (3.5s) ← Mở rộng đến mốc bắt đầu kế tiếp
Phụ đề 2: start=3500ms, end=7000ms   (3.5s) ← Dòng cuối mở rộng đến hết video
```

### 4.3 Mã nguồn chính

```python
def _prepare_data(self):
    """Làm sạch và tiền xử lý dữ liệu"""
    for i in range(len(self.queue_tts)):
        current = self.queue_tts[i]

        # Lưu lại thời gian bắt đầu gốc
        current['start_time_source'] = current['start_time']

        # Nếu làm chậm video và phụ đề đầu bắt đầu trước 100ms, đặt lại bắt đầu từ 0
        if self.should_videorate and i == 0 and current['start_time'] < 100:
            current['start_time_source'] = 0

        # Điểm mấu chốt: Mở rộng thời gian kết thúc đến thời gian bắt đầu của phụ đề tiếp theo
        if i < len(self.queue_tts) - 1:
            next_sub = self.queue_tts[i + 1]
            current['end_time_source'] = next_sub['start_time']
            current['end_time'] = next_sub['start_time']
        else:
            # Dòng cuối cùng: Mở rộng đến hết thời lượng video gốc
            current['end_time_source'] = self.raw_total_time
            current['end_time'] = self.raw_total_time

        # Tính toán thời lượng khả dụng sau khi mở rộng
        current['source_duration'] = current['end_time_source'] - current['start_time_source']
```

### 4.4 So sánh hiệu quả

```text
Giả định dữ liệu gốc:
Phụ đề 1: start=1000ms, end=3000ms (2s), Lồng tiếng thực tế = 3.5s
Phụ đề 2: start=3500ms, end=6000ms (2.5s), Lồng tiếng thực tế = 2.0s

Sau khi xử lý mở rộng:
Phụ đề 1: source_duration = 3500 - 1000 = 2500ms (Mở rộng thêm 500ms khoảng lặng)
Phụ đề 2: source_duration = 6000 - 3500 = 2500ms

Tỷ lệ tăng tốc mới:
Phụ đề 1: 3.5 / 2.5 = 1.4x (Ban đầu cần 3.5/2.0 = 1.75x)
Phụ đề 2: Không cần tăng tốc (2.0 < 2.5)
```

**Kết luận**: Việc mở rộng mốc thời gian giúp giảm tỷ lệ tăng tốc âm thanh của Phụ đề 1 từ 1.75x xuống 1.4x, hạn chế đáng kể độ biến dạng của âm thanh và cải thiện chất lượng giọng nói.

---

## V. Chế độ 1: Chỉ tăng tốc âm thanh

### 5.1 Chiến lược

Khi thời lượng âm thanh lồng tiếng > thời lượng phụ đề khả dụng, hệ thống sẽ thực hiện tăng tốc âm thanh lồng tiếng để khớp với thời lượng phụ đề. Tỷ lệ tăng tốc tối đa không được vượt quá giá trị cấu hình `max_audio_speed_rate` (mặc định là 100).

```text
Âm thanh lồng tiếng: ═══════════════════════  (3500ms)
Phụ đề khả dụng:     ══════════════           (2500ms)
                               ↓ Tăng tốc 1.4x
Kết quả:             ══════════════           (2500ms)
```

### 5.2 Mã nguồn chính

```python
# Chỉ tăng tốc âm thanh
if self.should_audiorate and not self.should_videorate:
    if dubb_dur > source_dur:
        ratio = dubb_dur / source_dur
        if ratio > self.max_audio_speed_rate:
            # Vượt quá giới hạn tăng tốc tối đa, giới hạn tốc độ ở mức trần
            audio_target = int(dubb_dur / self.max_audio_speed_rate)
        else:
            # Tăng tốc để khớp hoàn toàn với thời lượng phụ đề
            audio_target = source_dur
```

### 5.3 Đăng ký nhiệm vụ tăng tốc âm thanh

```python
if self.should_audiorate and audio_target < dubb_dur:
    self.audio_data.append({
        "filename": it['filename'],       # Đường dẫn file âm thanh lồng tiếng
        "dubb_time": dubb_dur,            # Thời lượng lồng tiếng gốc
        "target_time": audio_target        # Thời lượng mục tiêu cần đạt sau tăng tốc
    })
```

---

## VI. Chế độ 2: Chỉ làm chậm video

### 6.1 Chiến lược

Khi thời lượng lồng tiếng thực tế > thời lượng phụ đề gốc khả dụng, hệ thống sẽ làm chậm phân đoạn video tương ứng để kéo dài thời lượng hình ảnh khớp với âm thanh. Hệ số nhân PTS không được vượt quá `max_video_pts_rate` (mặc định là 10).

```text
Phân đoạn video gốc: ══════════════       (2500ms)
Âm thanh lồng tiếng: ═══════════════════  (3500ms)
                             ↓ Làm chậm PTS=1.4
Kết quả video mới:   ═══════════════════  (3500ms)
```

### 6.2 Nguyên lý PTS

Mốc thời gian hiển thị PTS (Presentation Time Stamp) điều khiển thời điểm hiển thị của từng khung hình video. Bộ lọc `setpts` của FFmpeg có thể sửa đổi PTS này:

```text
setpts=1.0*PTS  → Tốc độ bình thường
setpts=2.0*PTS  → Chậm 2 lần (thời gian hiển thị của mỗi khung hình tăng gấp đôi)
setpts=0.5*PTS  → Nhanh 2 lần (thời gian hiển thị của mỗi khung hình giảm một nửa)
```

### 6.3 Mã nguồn chính

```python
# Chỉ làm chậm video
elif not self.should_audiorate and self.should_videorate:
    if dubb_dur > source_dur:
        video_target = dubb_dur  # Thời lượng video mục tiêu = Thời lượng lồng tiếng
        pts = video_target / source_dur
        if pts > self.max_video_pts_rate:
            # Vượt quá giới hạn làm chậm tối đa, giới hạn tỷ lệ ở mức trần
            video_target = int(source_dur * self.max_video_pts_rate)
```

### 6.4 Đăng ký phân đoạn video

```python
if self.should_videorate:
    pts = video_target / source_dur if source_dur > 0 else 1.0
    self.video_for_clips.append({
        "start": it['start_time_source'],   # Điểm bắt đầu cắt video
        "end": it['end_time_source'],        # Điểm kết thúc cắt video
        "target_time": video_target,         # Thời lượng đầu ra mục tiêu
        "pts": pts,                          # Hệ số nhân PTS
        "tts_index": i,                      # Chỉ số phụ đề tương ứng
        "line": it['line']                   # Dòng phụ đề số mấy
    })
```

---

## VII. Chế độ 3: Kết hợp âm thanh + video

### 7.1 Chiến lược

Khi cả tính năng tăng tốc âm thanh và làm chậm video cùng được bật, hệ thống sẽ tự động lựa chọn giải pháp phối hợp dựa trên tỷ lệ chênh lệch:

| Tỷ lệ chênh lệch (ratio) | Chiến lược phối hợp | Giải thích |
|:---:|------|------|
| ≤ 1.2 | Chỉ tăng tốc âm thanh | Do chênh lệch nhỏ, tăng tốc âm thanh ít ảnh hưởng tới chất lượng tai nghe, tránh việc làm chậm video không cần thiết |
| > 1.2 | Chia đôi gánh nặng | Mỗi bên (âm thanh và video) tự gánh chịu một nửa khoảng thời gian chênh lệch |

```text
Ví dụ: Thời lượng phụ đề gốc là 2500ms, lồng tiếng thực tế là 6000ms, ratio = 2.4

Phương án A (Chỉ tăng tốc âm thanh khi ratio ≤ 1.2):
  Không áp dụng do ratio > 1.2 (nếu áp dụng âm thanh phải tăng tốc tới 2.4x -> méo tiếng nặng)

Phương án B (Chia đôi gánh nặng khi ratio > 1.2, thực tế áp dụng):
  Khoảng chênh lệch: diff = 6000 - 2500 = 3500ms
  Thời lượng mốc giao nhau: joint_target = 2500 + 3500/2 = 4250ms
  Âm thanh cần tăng tốc để phát xong trong 4250ms (Tỷ lệ 1.41x) -> Giọng nói vẫn khá tự nhiên
  Video làm chậm để kéo dài thời gian thành 4250ms (PTS=1.7) -> Hình ảnh chậm nhẹ, chấp nhận được
```

### 7.2 Mã nguồn chính

```python
elif self.should_audiorate and self.should_videorate:
    if dubb_dur > source_dur:
        ratio = dubb_dur / source_dur
        if ratio <= self.BOTH_MODE_AUDIO_ONLY_THRESHOLD:  # Ngưỡng 1.2
            # Chênh lệch thấp, chỉ tăng tốc âm thanh, giữ nguyên tốc độ video
            audio_target = source_dur
            video_target = source_dur
        else:
            # Chênh lệch lớn, chia đôi khoảng chênh lệch cho cả hai gánh vác
            diff = dubb_dur - source_dur
            joint_target = int(source_dur + (diff / 2))
            audio_target = joint_target
            video_target = joint_target
```

### 7.3 Tại sao chọn 1.2 làm ngưỡng phân định?

- **Khi tăng tốc âm thanh ≤ 1.2x**: Tai người bình thường hầu như không nhận thấy sự thay đổi về cao độ hoặc biến dạng của giọng nói.
- **Vượt quá 1.2x**: Bắt đầu có cảm giác giọng nói bị dồn dập, méo âm, do đó cần có sự trợ giúp kéo dài thời gian từ phía làm chậm video.

---

## VIII. Chế độ 4: Ghép nối không thay đổi tốc độ

### 8.1 Chiến lược

Khi cả hai tính năng tăng tốc âm thanh và làm chậm video đều tắt, hệ thống sẽ ghép nối trực tiếp các tệp âm thanh lồng tiếng theo đúng mốc thời gian phụ đề. Các khoảng trống giữa các thoại sẽ được lấp đầy bằng các file khoảng lặng (silence), trừ khi tùy chọn xóa bỏ khoảng lặng giữa các thoại được bật.

Nếu bật tính năng bắt buộc căn chỉnh mốc thời gian phụ đề theo âm thanh (`align_sub_audio`), thời gian hiển thị phụ đề sẽ tự động thay đổi dựa trên thời lượng thực tế của tệp âm thanh lồng tiếng, đảm bảo chữ xuất hiện khi bắt đầu phát tiếng và biến mất khi âm thanh kết thúc.

### 8.2 Quy tắc ghép nối

```text
Mốc thời gian phụ đề gốc:
├── 0ms ──── 1000ms ──── 3500ms ──── 6000ms ──── 8000ms
│   Lặng     Thoại 1      Thoại 2      Thoại 3
│  (1000ms)  (2500ms)     (2500ms)     (2000ms)

Kết quả ghép nối:
├── [Khoảng lặng 1000ms] + [Lồng tiếng 1] + [Lồng tiếng 2] + [Lồng tiếng 3] + [Lặng kết thúc]
```

### 8.3 Mã nguồn chính

```python
def _run_no_rate_change_mode(self):
    audio_concat_list = []
    total_audio_duration = 0

    for i, it in enumerate(self.queue_tts):
        prev_end = 0 if i == 0 else self.queue_tts[i-1].get('end_pos_for_concat', 0)
        start_time = it['start_time']

        # Tính khoảng trống tĩnh so với câu trước
        gap = start_time - prev_end

        # Nếu không loại bỏ khoảng lặng, chèn file lặng vào khoảng trống
        if not self.remove_silent_mid and gap > 0:
            audio_concat_list.append(self._create_silen_file(f"gap_{i}", gap))
            total_audio_duration += gap

        # Ghép tệp âm thanh lồng tiếng thực tế
        if it.get('filename') and Path(it['filename']).exists():
            audio_concat_list.append(it['filename'])
            dubb_len = len(AudioSegment.from_file(it['filename']))

        total_audio_duration += dubb_len
        it['end_pos_for_concat'] = total_audio_duration

        # Nếu chọn căn chỉnh phụ đề theo giọng đọc, điều chỉnh mốc thời gian
        if self.align_sub_audio:
            it['start_time'] = total_audio_duration - dubb_len
            it['end_time'] = total_audio_duration

    # Chèn khoảng lặng ở cuối video nếu tổng thời lượng âm thanh ngắn hơn thời lượng video gốc
    if self.raw_total_time > total_audio_duration:
        audio_concat_list.append(
            self._create_silen_file("tail_end", self.raw_total_time - total_audio_duration)
        )
```

---

## IX. Chi tiết triển khai thay đổi tốc độ âm thanh

### 9.1 Hai động cơ thay đổi tốc độ

pyVideoTrans hỗ trợ hai cơ chế thay đổi tốc độ âm thanh khác nhau, tự động lựa chọn theo mức độ ưu tiên:

| Cơ chế | Độ ưu tiên | Phụ thuộc | Đặc điểm |
|------|:---:|------|------|
| **Rubber Band** | Cao | `pyrubberband` + CLI `rubberband` | Chất lượng âm thanh tốt nhất, giữ nguyên cao độ của giọng nói |
| **FFmpeg atempo** | Thấp | FFmpeg (tích hợp sẵn) | Không yêu cầu cài thêm thư viện phụ thuộc ngoài, chất lượng âm thanh trung bình |

### 9.2 Thay đổi tốc độ bằng Rubber Band

```python
def _change_speed_rubberband(input_path, target_duration):
    # Đọc tệp âm thanh
    y, sr = sf.read(input_path)
    current_duration = round((len(y) / sr) * 1000)

    # Tính tỷ lệ kéo giãn thời gian
    time_stretch_rate = current_duration / target_duration
    time_stretch_rate = max(0.2, min(time_stretch_rate, 50.0))

    # Thực hiện kéo giãn (giữ nguyên cao độ)
    y_stretched = pyrb.time_stretch(y, sr, time_stretch_rate)

    # Chuyển kênh âm thanh đơn (mono) sang kênh đôi (stereo) nếu cần
    if y_stretched.ndim == 1:
        y_stretched = np.column_stack((y_stretched, y_stretched))

    # Lưu lại vào tệp
    sf.write(input_path, y_stretched, sr)
```

**Ưu điểm của Rubber Band**:
- Sử dụng thuật toán Phase Vocoder giúp giữ nguyên cao độ giọng nói khi thay đổi tốc độ.
- Hỗ trợ biên độ thay đổi lớn (lên tới 50x) mà không gây biến dạng giọng nghiêm trọng.
- Tốc độ xử lý nhanh và hỗ trợ đa luồng.

### 9.3 Thay đổi tốc độ bằng FFmpeg atempo (Phương án dự phòng)

```python
def _precise_speed_up_audio(input_path, target_duration):
    current_duration_ms = len(AudioSegment.from_file(input_path, format='wav'))

    # Giới hạn của bộ lọc atempo trong FFmpeg: tham số chỉ được nằm trong khoảng [0.5, 2.0]
    # Nếu tốc độ cần thay đổi vượt quá giới hạn, phải liên kết chuỗi nhiều bộ lọc atempo
    atempo_list = []
    speed_factor = current_duration_ms / target_duration

    while speed_factor > 2.0:
        atempo_list.append("atempo=2.0")
        speed_factor /= 2.0

    atempo_list.append(f"atempo={speed_factor}")
    filter_str = ",".join(atempo_list)

    # Ví dụ: Tăng tốc 8x → liên kết chuỗi thành "atempo=2.0,atempo=2.0,atempo=2.0"
    cmd = [
        '-y', '-i', input_path,
        '-filter:a', filter_str,
        '-t', f"{target_duration/1000.0}",  # Bắt buộc cắt phần thừa ở mốc thời lượng mục tiêu
        '-ar', "48000", '-ac', "2",
        '-c:a', 'pcm_s16le',
        f'{input_path}-after.wav'
    ]
    tools.runffmpeg(cmd)
    shutil.copy2(f'{input_path}-after.wav', input_path)
```

**Nguyên lý liên kết chuỗi atempo**:

```text
Phạm vi hoạt động của 1 bộ lọc atempo: [0.5, 2.0]

Trường hợp cần tăng tốc 8.0 lần:
  8.0 = 2.0 × 2.0 × 2.0
  → Biểu thức lọc: "atempo=2.0,atempo=2.0,atempo=2.0"

Trường hợp cần tăng tốc 3.0 lần:
  3.0 = 2.0 × 1.5
  → Biểu thức lọc: "atempo=2.0,atempo=1.5"

Trường hợp cần tăng tốc 1.3 lần:
  1.3 nằm trong khoảng [0.5, 2.0], không cần chia nhỏ
  → Biểu thức lọc: "atempo=1.3"
```

### 9.4 Tăng tốc đa tiến trình song song

Nhiệm vụ thay đổi tốc độ của nhiều file âm thanh được xử lý song song thông qua `ProcessPoolExecutor` để tối ưu hóa thời gian chạy:

```python
def _execute_audio_speedup_rubberband(self):
    _wok = min(12, len(self.audio_data), max(os.cpu_count() - 1, 1))

    with ProcessPoolExecutor(max_workers=int(_wok)) as pool:
        for i, d in enumerate(self.audio_data):
            pool.submit(
                _change_speed_rubberband if HAS_RUBBERBAND else _precise_speed_up_audio,
                d['filename'],
                d['target_time']
            )
```

---

## X. Chi tiết triển khai thay đổi tốc độ video

### 10.1 Nguyên lý sửa đổi PTS trong video

FFmpeg sử dụng bộ lọc `setpts` để điều chỉnh thời điểm hiển thị khung hình, qua đó thay đổi tốc độ chạy video:

```text
Chuỗi khung hình gốc:
  Khung 1(0ms) → Khung 2(33ms) → Khung 3(66ms) → Khung 4(100ms)  [Cho video 30fps]

Cấu hình setpts=2.0*PTS (Làm chậm 2x):
  Khung 1(0ms) → Khung 2(66ms) → Khung 3(132ms) → Khung 4(200ms)

Cấu hình setpts=0.5*PTS (Tăng tốc 2x):
  Khung 1(0ms) → Khung 2(16ms) → Khung 3(33ms) → Khung 4(50ms)
```

### 10.2 Cấu trúc lệnh FFmpeg

```python
def _cut_video_get_duration(i, task, novoice_mp4_original, preset, crf, fps_mode):
    # Cấu hình các tham số cắt
    ss_time = tools.ms_to_time_string(ms=task['start'], sepflag='.')
    source_duration_s = (task['end'] - task['start']) / 1000.0
    target_duration_s = task.get('target_time', source_duration_ms) / 1000.0
    pts_factor = task.get('pts', 1.0)

    cmd = [
        '-y',
        '-ss', ss_time,                    # Điểm bắt đầu cắt phân đoạn
        '-t', f'{source_duration_s:.6f}',  # Thời lượng cắt
        '-i', input_video_path,
        '-an',                             # Loại bỏ luồng âm thanh gốc
        '-c:v', 'libx264',                # Đặt lại codec mã hóa video
        '-g', '1',                         # Đặt GOP=1 để đảm bảo cắt chính xác từng khung hình đầu
        '-preset', preset,                 # Tốc độ mã hóa
        '-crf', crf,                       # Mức độ nén chất lượng
        '-pix_fmt', 'yuv420p'              # Định dạng điểm ảnh
    ]

    # Cấu hình bộ lọc thay đổi tốc độ PTS
    if abs(pts_factor - 1.0) >= 0.001:
        cmd.extend(['-vf', f'setpts={pts_factor}*PTS'])
    else:
        cmd.extend(['-vf', 'setpts=PTS'])

    cmd.extend(fps_mode)  # Chế độ đồng bộ khung hình VFR hoặc CFR
    cmd.extend(['-t', f'{target_duration_s:.6f}'])  # Giới hạn thời lượng đầu ra bắt buộc
    cmd.append(os.path.basename(task['filename']))
```

### 10.3 Lựa chọn chế độ đồng bộ khung hình (FPS Mode)

```python
self.fps_mode = ["-fps_mode", "vfr"]  # Mặc định sử dụng tốc độ khung hình biến đổi (VFR)

if settings.get('fps_mode') == 'cfr':
    video_fps = tools.get_video_info(novoice_mp4, video_fps=True)
    self.fps_mode = ["-r", f"{video_fps}", "-fps_mode", "cfr"]
```

| Chế độ | Giải thích | Trường hợp áp dụng |
|------|------|---------|
| **VFR** (Variable Frame Rate) | Cho phép tốc độ khung hình thay đổi động, cho hiệu ứng mượt hơn | Khuyên dùng mặc định |
| **CFR** (Constant Frame Rate) | Ép buộc tốc độ khung hình cố định suốt video để tăng tính tương thích | Dùng khi gặp lỗi không phát được trên một số trình phát kén định dạng |

### 10.4 Cơ chế dự phòng sự cố (Fallback)

Nếu phân đoạn video sau khi điều chỉnh tốc độ bị lỗi (kích thước file đầu ra < 1024 byte), hệ thống sẽ tự động chuyển sang chế độ cắt gốc không đổi tốc độ để đảm bảo quy trình không bị ngắt quãng:

```python
if not file_path.exists() or file_path.stat().st_size < 1024:
    # Dự phòng sự cố: Thực hiện cắt không thay đổi tốc độ
    cmd_backup = [
        '-y', '-ss', ss_time,
        '-t', f'{source_duration_s:.6f}',
        '-i', input_video_path,
        '-an', '-c:v', 'libx264',
        '-g', '1', '-preset', preset, '-crf', crf,
        '-pix_fmt', 'yuv420p',
        '-vf', 'setpts=PTS',  # Giữ nguyên mốc thời gian hiển thị gốc
    ] + fps_mode
    cmd_backup.append(os.path.basename(task['filename']))
    tools.runffmpeg(cmd_backup, force_cpu=True, cmd_dir=work_dir)
```

### 10.5 Xử lý đa tiến trình song song

```python
def _video_speeddown(self):
    _wok = min(12, len(data), max(os.cpu_count() - 1, 1))

    with ProcessPoolExecutor(max_workers=int(_wok)) as pool:
        for i, d in enumerate(data):
            pool.submit(_cut_video_get_duration, i, d,
                       self.novoice_mp4_original,
                       self.preset, self.crf, self.fps_mode)
```

---

## XI. Ghép nối và căn chỉnh âm thanh cuối cùng

### 11.1 Nguyên tắc căn chỉnh

Bất kể áp dụng chế độ thay đổi tốc độ nào, việc ghép nối tệp âm thanh cuối cùng đều tuân theo nguyên tắc:

1. **Mỗi câu thoại lồng tiếng sẽ có một "khoang chứa" riêng**, thời lượng khoang chứa được xác định từ các bước tính toán tốc độ trước đó.
2. **Nếu file lồng tiếng ngắn hơn khoang chứa**: Chèn thêm âm thanh lặng ở cuối phân đoạn.
3. **Nếu file lồng tiếng dài hơn khoang chứa**: Cắt bỏ phần đuôi thoại thừa để ép vừa vặn vào khoang chứa.
4. **Nếu file lồng tiếng bằng thời lượng khoang chứa**: Đưa thẳng vào ghép nối.

```text
Mốc thời gian khoang chứa:
├── [Khoang 1: 3500ms] ├── [Khoang 2: 2500ms] ├── [Khoang 3: 2000ms] ──→

Cách xử lý nội dung bên trong:
Khoang 1:
├── [File thoại 1: 3200ms] + [Khoảng lặng chèn bù: 300ms]

Khoang 2:
├── [File thoại 2: 2500ms] (Vừa vặn tuyệt đối)

Khoang 3:
├── [File thoại 3: 2800ms] → Cắt bỏ phần đuôi 800ms thừa, chỉ lấy 2000ms đầu
```

### 11.2 Mã nguồn chính

```python
def _concat_audio_aligned(self):
    audio_list = []
    current_timeline = self.queue_tts[0]['start_time']

    # Chèn khoảng lặng ở đầu video (nếu mốc bắt đầu thoại 1 lớn hơn 0)
    if current_timeline > 0:
        audio_list.append(self._create_silen_file("head_0", current_timeline))

    for i, it in enumerate(self.queue_tts):
        # Tính thời lượng khoang chứa: lấy thời lượng thực tế của video làm chậm (nếu có), hoặc thời lượng phụ đề mở rộng
        slot_duration = it.get('final_duration', it['source_duration'])

        if slot_duration <= 0:
            slot_duration = max(1, it['source_duration'])

        # Đọc tệp âm thanh lồng tiếng
        seg = AudioSegment.from_file(audio_file)
        current_slot_audio_len = len(seg)

        # Xử lý 3 tình huống
        if current_slot_audio_len > slot_duration:
            # Bị tràn: thực hiện cắt bớt phần thừa
            cut_seg = seg[:slot_duration]
            cut_seg.export(final_slot_path, format='wav')
            audio_list.append(final_slot_path)

        elif current_slot_audio_len < slot_duration:
            # Thiếu hụt: chèn thêm khoảng lặng ở đuôi phân đoạn
            diff = slot_duration - current_slot_audio_len
            audio_list.append(audio_file)
            audio_list.append(self._create_silen_file(f"tail_{i}", diff))

        else:
            # Khớp thời gian
            audio_list.append(audio_file)

        # Cập nhật lại mốc thời gian hiển thị phụ đề sau căn chỉnh
        it['start_time'] = current_timeline
        it['end_time'] = current_timeline + slot_duration
        current_timeline += slot_duration

    self._exec_concat_audio(audio_list)
```

### 11.3 Khởi tạo file khoảng lặng

```python
def _create_silen_file(self, name, duration_ms):
    path = Path(self.cache_folder, f"silence_{name}.wav").as_posix()
    duration_ms = max(1, int(duration_ms))
    AudioSegment.silent(duration=duration_ms, frame_rate=48000) \
                .set_channels(2) \
                .export(path, format="wav")
    return path
```

### 11.4 Lệnh ghép nối bằng FFmpeg

```python
def _exec_concat_audio(self, file_list):
    # Khởi tạo file chứa danh sách file cần ghép nối
    concat_txt = Path(self.cache_folder, 'final_audio_concat.txt').as_posix()
    tools.create_concat_txt(file_list, concat_txt=concat_txt)

    # Sử dụng bộ lọc ghép nối không giải nén (copy trực tiếp luồng để tối ưu hiệu năng)
    cmd = [
        '-y', '-f', 'concat', '-safe', '0',
        '-i', concat_txt,
        '-c:a', 'copy',
        temp_wav
    ]
    tools.runffmpeg(cmd, force_cpu=True, cmd_dir=self.cache_folder)
```

---

## XII. Ghép nối các phân đoạn video

### 12.1 Sơ đồ quy trình

```text
Video không tiếng gốc (novoice.mp4)
    │
    ├─→ Cắt và điều chỉnh phân đoạn 1 (clip_0_1.400.mp4)  ← làm chậm PTS=1.4
    ├─→ Cắt và điều chỉnh phân đoạn 2 (clip_1_1.000.mp4)  ← giữ nguyên PTS=1.0
    ├─→ Cắt và điều chỉnh phân đoạn 3 (clip_2_1.700.mp4)  ← làm chậm PTS=1.7
    │
    └─→ Dùng FFmpeg ghép lại (no-reencode) → Video không tiếng đã căn chỉnh (novoice.mp4 mới)
```

### 12.2 Mã nguồn thực hiện ghép nối video

```python
def _concat_video(self, processed_clips):
    txt_content = []
    for clip in processed_clips:
        if clip.get('actual_duration', 0) > 0 and Path(clip['filename']).exists():
            txt_content.append(f"file '{clip['filename']}'")

    # Sử dụng tính năng concat của FFmpeg (copy trực tiếp luồng không mã hóa lại)
    cmd = [
        '-y', '-f', 'concat', '-safe', '0',
        '-i', concat_list,
        '-c', 'copy',
        output_path
    ]
    tools.runffmpeg(cmd, force_cpu=True, cmd_dir=self.cache_folder)

    # Ghi đè file video không tiếng đã căn chỉnh mới lên file cũ
    shutil.move(output_path, self.novoice_mp4)
```

---

## XIII. TtsSpeedRate: Trường hợp chỉ lồng tiếng

### 13.1 So sánh sự khác biệt

Lớp `TtsSpeedRate` kế thừa từ `SpeedRate`, chuyên dùng để căn chỉnh mốc thời gian trong trường hợp "Lồng tiếng văn bản từ file phụ đề độc lập":

| Đặc tính | Lớp cha `SpeedRate` | Lớp con `TtsSpeedRate` |
|------|-----------|-------------|
| Điều chỉnh làm chậm video | Có hỗ trợ | **Bị tắt bỏ** (`should_videorate=False`) |
| Giới hạn tăng tốc tối đa | Có cấu hình động | Cố định ở ngưỡng 100 |
| Mức độ mở rộng thời gian | Đầy đủ (lưu mốc `start_time_source`) | Đơn giản hóa (chỉ mở rộng `end_time`) |
| Kết quả đầu ra | Gồm cả video và âm thanh | Chỉ tạo tệp âm thanh lồng tiếng ghép nối |

### 13.2 Tiền xử lý đơn giản hóa

```python
class TtsSpeedRate(SpeedRate):
    def _prepare_data(self):
        _len = len(self.queue_tts)
        for i in range(_len):
            current = self.queue_tts[i]
            if i < _len - 1:
                # Chỉ di chuyển thời gian kết thúc mà không cần ghi lại thời gian bắt đầu gốc
                current['end_time'] = self.queue_tts[i + 1]['start_time']

            current['source_duration'] = current['end_time'] - current['start_time']
```

### 13.3 Tính toán đơn giản hóa

```python
def _calculate_adjustments(self):
    for i, it in enumerate(self.queue_tts):
        source_dur = it['source_duration']
        dubb_dur = it['dubb_time']

        if dubb_dur > source_dur:
            # Ép buộc tăng tốc độ âm thanh lồng tiếng để nhét vừa mốc thời gian phụ đề
            self.audio_data.append({
                "filename": it['filename'],
                "dubb_time": dubb_dur,
                "target_time": source_dur
            })
```

---

## XIV. Khả năng tương thích đa nền tảng

### 14.1 Chuẩn hóa đường dẫn hệ thống

Toàn bộ đường dẫn tệp tin đều được định dạng bằng phương thức `Path.as_posix()`, tự động đổi các ký tự phân tách trên Windows (dấu gạch chéo ngược `\`) thành dấu gạch chéo xuôi `/`, giúp FFmpeg hoạt động ổn định trên cả Windows, Linux và macOS.

```python
input_video_path = Path(novoice_mp4_original).resolve().as_posix()
work_dir = Path(task['filename']).parent.as_posix()
```

### 14.2 Thực thi FFmpeg thống nhất

Các tiến trình đều thông qua giao thức `tools.runffmpeg()` để kiểm soát lệnh gọi, hệ thống sẽ tự động xử lý:
- Tự động bao bọc đường dẫn bằng dấu nháy nếu phát hiện đường dẫn chứa ký tự khoảng trắng.
- Tìm kiếm tệp thực thi FFmpeg (từ biến môi trường PATH hệ thống hoặc sử dụng file nhị phân đính kèm trong thư mục `ffmpeg/`).
- Hỗ trợ lắp ráp các mảng tham số dòng lệnh một cách chính xác.

### 14.3 Kiểm soát luồng chạy

Thay vì dùng `multiprocessing.Pool`, hệ thống dùng `ProcessPoolExecutor` của Python để hỗ trợ quản lý tài nguyên tốt hơn, tránh lỗi rò rỉ bộ nhớ hoặc lỗi đóng tiến trình con bất thường trên hệ điều hành Windows.

### 14.4 Tương tác tệp tin sạch

Toàn bộ quá trình quét và xóa file được đảm bảo thực thi qua các thư viện hiện đại của Python như `Path.glob()` và `Path.unlink()`, giúp mã nguồn sạch và giảm thiểu sai sót đường dẫn hệ thống.

---

## XV. Các giới hạn đã biết và lưu ý

### 15.1 Giới hạn độ chính xác của FFmpeg

- Do giới hạn cơ bản của cấu trúc luồng video, FFmpeg không thể định vị chính xác ở mức mili-giây tuyệt đối. Các video sau khi thay đổi tốc độ PTS có thể dài hoặc ngắn hơn vài chục mili-giây.
- Để tránh việc sai lệch tích lũy dẫn đến âm thanh chạy lệch hình về cuối video, hệ thống luôn lấy mốc thời gian của âm thanh ghép nối làm mốc chuẩn chính xác nhất, sau đó bù lặng hoặc cắt đuôi để tổng thời lượng âm thanh và video ghép cuối cùng luôn trùng khớp với nhau.

### 15.2 Đối phó với phân đoạn siêu ngắn

- Các phân đoạn video có thời lượng nhỏ hơn thời lượng của 1 khung hình (ví dụ: nhỏ hơn 33 mili-giây đối với video 30fps) sẽ khiến thuật toán thay đổi tốc độ của FFmpeg bị lỗi.
- **Biện pháp giảm thiểu**: Giai đoạn tiền xử lý mở rộng mốc thời gian phụ đề sẽ gộp các khoảng lặng đệm vào phân đoạn trước, giúp đảm bảo mỗi phân đoạn video được cắt ra luôn có thời lượng tối thiểu vài trăm mili-giây.

### 15.3 Suy giảm chất lượng âm thanh khi tăng tốc quá mức

- **Rubber Band**: Đạt chất lượng gần như không suy giảm nếu tỷ lệ tăng tốc dưới 3x. Khi vượt quá 5x, giọng nói sẽ xuất hiện tiếng robot cơ học.
- **FFmpeg atempo**: Giọng nói có thể bị thay đổi cao độ nhẹ nếu tăng tốc vượt quá 2x.
- **Lời khuyên**: Nếu phát hiện phân đoạn cần tăng tốc lớn (> 3x), người dùng nên bật cả chế độ làm chậm video để hai bên phối hợp gánh vác, giảm áp lực tăng tốc cho tệp âm thanh.

### 15.4 Hiện tượng giật khung hình khi làm chậm video

- Bộ lọc PTS chỉ kéo dài thời gian hiển thị của mỗi khung hình mà không tạo thêm khung hình trung gian mới.
- Đối với video gốc có FPS thấp (như 24fps), việc làm chậm trên 2x sẽ khiến thời gian hiển thị mỗi khung hình kéo dài quá 83ms, người xem sẽ có cảm giác hình ảnh bị giật nhẹ.
- **Lời khuyên**: Cố gắng khống chế tỷ lệ làm chậm video dưới mức 2x.

### 15.5 Tự động loại bỏ tệp rác lỗi

Các tệp video phân đoạn được cắt ra nếu có kích thước nhỏ hơn 1024 byte (chỉ chứa header metadata mà không có dữ liệu hình ảnh thực tế) sẽ được coi là tệp lỗi và tự động bị bỏ qua trong danh sách ghép nối:

```python
if clip.get('actual_duration', 0) > 0 and Path(clip['filename']).exists():
    # Phân đoạn hợp lệ, đưa vào danh sách ghép nối
    txt_content.append(f"file '{path}'")
else:
    logger.warning(f"[Video-Concat] Bỏ qua phân đoạn lỗi: {clip.get('filename')}")
```

---

## Phụ lục: Sơ đồ luồng xử lý đầy đủ

```text
                    ┌────────────────────────────────────────┐
                    │       Đầu vào: Danh sách queue_tts     │
                    │       (Mỗi phụ đề + File lồng tiếng)   │
                    └───────────────────┬────────────────────┘
                                        │
                    ┌───────────────────┴────────────────────┐
                    │      Cờ should_audiorate hoặc          │
                    │      should_videorate được kích hoạt?  │
                    └───────────────────┬────────────────────┘
                                        │
                  ┌─────────────────────┴─────────────────────┐
                  │                                           │
                  Có                                        Không
                  │                                           │
         ┌────────┴────────┐                         ┌────────┴────────┐
         │ _prepare_data() │                         │  _run_no_rate_  │
         │ Mở rộng mốc     │                         │  change_mode()  │
         │ thời gian phụ đề│                         │  Ghép trực tiếp │
         └────────┬────────┘                         │  chèn khoảng lặng│
                  │                                  └────────┬────────┘
         ┌────────┴────────┐                                  │
         │   _calculate_   │                                  │
         │  adjustments()  │                                  │
         │ Tính tỷ lệ căn  │                                  │
         │ chỉnh tốc độ    │                                  │
         └────────┬────────┘                                  │
                  │                                           │
    ┌─────────────┴─────────────┐                             │
    │                           │                             │
Điều chỉnh âm thanh        Điều chỉnh video                   │
    │                           │                             │
┌───┴───┐                   ┌───┴───┐                         │
│RB/    │                   │_cut_  │                         │
│atempo │                   │video_ │                         │
│Tăng   │                   │get_   │                         │
│tốc    │                   │durati-│                         │
└───┬───┘                   │on()   │                         │
    │                       │Sửa PTS│                         │
    │                       └───┬───┘                         │
    │                           │                             │
    │                       ┌───┴───┐                         │
    │                       │_con-  │                         │
    │                       │cat_   │                         │
    │                       │video()│                         │
    │                       │Ghép   │                         │
    │                       │video  │                         │
    │                       └───┬───┘                         │
    │                           │                             │
    └─────────────┬─────────────┘                             │
                  │                                           │
         ┌────────┴────────┐                                  │
         │ _concat_audio_  │◄─────────────────────────────────┘
         │ aligned()       │
         │ Căn chỉnh và    │
         │ ghép nối âm thoại│
         └────────┬────────┘
                  │
         ┌────────┴────────┐
         │ _exec_concat_   │
         │ audio()         │
         │ FFmpeg xuất file│
         └────────┬────────┘
                  │
         ┌────────┴────────┐
         │ Đầu ra: File âm │
         │ thanh lồng tiếng│
         │ + Phụ đề căn lại│
         └─────────────────┘
```
