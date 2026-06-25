---
title: Các lỗi thường gặp và cách khắc phục
date: 2024-01-22 14:33:00
description: Mục Trợ giúp/Giới thiệu trên thanh menu có chứa nhiều liên kết hữu ích như tải mô hình, cấu hình CUDA, v.v. Bạn có thể mở ra để tham khảo khi gặp sự cố.
---

# Các câu hỏi thường gặp và Giải pháp cho pyVideoTrans

Để giúp bạn sử dụng `pyVideoTrans` một cách tốt nhất, chúng tôi đã tổng hợp các câu hỏi thường gặp và giải pháp khắc phục dưới đây.

Trong mục `Trợ giúp / Giới thiệu` trên thanh menu của ứng dụng chứa rất nhiều liên kết hữu ích như đường dẫn tải mô hình, hướng dẫn cấu hình CUDA, v.v. Khi gặp sự cố, bạn có thể click mở ra để tham khảo.

![image.png](https://pvtr2.pyvideotrans.com/images/c02de778f70546ba9a1eb2772ee9f18b~tplv-73owjymdk6-jj-mark_0_0_0_0_q75.webp)

> **Cách xem nhật ký lỗi (log)**: Thư mục `logs/` tại thư mục gốc của phần mềm chứa các tệp nhật ký `.log` được đặt tên theo ngày. Khi xảy ra lỗi, bạn có thể sao chép khoảng 30 dòng cuối cùng trong file log tương ứng để tìm kiếm sự trợ giúp.
>
> **Cách khôi phục cài đặt gốc**: Xóa 4 tệp tin `cfg.json`, `params.json`, `codec.json`, `ass.json` trong thư mục `videotrans/`, sau đó khởi động lại phần mềm là xong.

---

## Phần 1: Các vấn đề khi cài đặt và khởi động

### 1. Nhấp đúp chuột vào `sp.exe` nhưng phần mềm không mở hoặc không phản hồi trong thời gian dài?

Đây là hiện tượng bình thường khi khởi chạy lần đầu, xin đừng lo lắng.

*   **Nguyên nhân**: Phần mềm được xây dựng trên thư viện giao diện `PySide6`, giao diện chính chứa rất nhiều thành phần cần khởi tạo nên lần đầu chạy sẽ tốn thời gian nạp. Tùy thuộc vào cấu hình máy tính của bạn, thời gian khởi động có thể dao động từ **5 giây đến 2 phút**.
*   **Giải pháp**:
    1.  **Kiên nhẫn chờ đợi**: Vui lòng đợi một lát sau khi nhấp đúp chuột.
    2.  **Kiểm tra phần mềm diệt virus**: Một số trình diệt virus hoặc tường lửa có thể chặn chương trình khởi chạy. Hãy thử tạm thời tắt chúng hoặc thêm phần mềm này vào danh sách loại trừ/đáng tin cậy.
    3.  **Kiểm tra đường dẫn thư mục**: Đảm bảo đường dẫn thư mục chứa phần mềm **chỉ bao gồm chữ cái tiếng Anh và số**, không được có chữ tiếng Việt có dấu, ký tự đặc biệt hoặc khoảng trắng. Ví dụ: Đường dẫn tốt là `D:\pyVideoTrans`, đường dẫn dễ gây lỗi là `D:\Phần mềm dịch thuật\video trans`.
    4.  **Lỗi sau khi ghi đè bản cập nhật**: Nếu bạn gặp lỗi không mở được sau khi sao chép đè bản cập nhật mới, có thể bạn đã thao tác sai. Hãy tải lại gói phần mềm đầy đủ, giải nén và ghi đè lại bản cập nhật mới.

### 2. Gặp thông báo lỗi thiếu tệp `python310.dll` khi khởi động?

Lỗi này xảy ra khi bạn mới chỉ tải về gói cập nhật (patch) chứ chưa tải gói phần mềm đầy đủ (full package).

*   **Giải pháp**:
    1.  Đi tới trang chủ/trang phát hành để tải **gói phần mềm đầy đủ**.
    2.  Giải nén gói đầy đủ vào một thư mục.
    3.  Tải gói cập nhật mới nhất và giải nén chép đè toàn bộ tệp vào thư mục gói đầy đủ vừa giải nén.

### 3. Phần mềm có cần cài đặt phức tạp không?

Đây là phần mềm dạng di động (portable) **không cần cài đặt**. Bạn chỉ cần tải gói đầy đủ, giải nén và nhấp đúp vào `sp.exe` để chạy trực tiếp.

### 4. Tại sao phần mềm diệt virus báo chứa mã độc hoặc ngăn chặn ứng dụng?

*   **Nguyên nhân**: Phần mềm được đóng gói bằng công cụ `PyInstaller` và không có chứng chỉ số thương mại (digital signature). Một số phần mềm diệt virus sẽ cảnh báo bảo mật dựa trên điều này, đây là hiện tượng **báo nhầm phổ biến (false positive)**.
*   **Giải pháp**:
    1.  **Thêm vào danh sách tin cậy**: Thêm thư mục ứng dụng vào vùng đáng tin cậy của trình diệt virus trên máy của bạn.
    2.  **Chạy từ mã nguồn**: Nếu bạn là nhà phát triển, bạn có thể chọn triển khai trực tiếp từ mã nguồn để tránh hoàn toàn cảnh báo này.

### 5. Phần mềm có hỗ trợ hệ điều hành Windows 7 không?

**Không hỗ trợ**. Rất nhiều thư viện lõi của phần mềm (như PyTorch, PySide6) đã ngừng hỗ trợ Windows 7. Bạn bắt buộc phải sử dụng Windows 10 hoặc Windows 11.

### 6. Cách cài đặt chạy từ nguồn trên macOS / Linux?

*   **Yêu cầu hệ thống**:
    *   Python 3.10
    *   FFmpeg (cài qua `brew install ffmpeg` hoặc `apt install ffmpeg`)
    *   Trình quản lý gói `uv`
    *   Thư viện `libsndfile`
*   **Các bước triển khai**:
    ```bash
    git clone https://github.com/jianchang512/pyvideotrans
    cd pyvideotrans
    uv sync
    uv run sp.py
    ```
*   **Cài các kênh tùy chọn**: Chạy `uv sync --all-extra` để cài đặt tất cả các gói mở rộng (qwen-tts, qwen-asr, moss-tts, chatterbox).

### 7. Gặp lỗi sau khi cài đặt chạy từ mã nguồn?

Một số lỗi phổ biến và cách khắc phục:
*   **Chưa cài FFmpeg**: Hãy chắc chắn FFmpeg đã được cài đặt và cấu hình trong biến môi trường hệ thống.
*   **Thiếu thư viện phụ thuộc**: Chạy lại lệnh `uv sync` để cập nhật cài đặt.
*   **Phiên bản Python không đúng**: Bắt buộc phải sử dụng Python 3.10 (phiên bản đã được chỉ định trong tệp `.python-version`).

---

## Phần 2: Tính năng cốt lõi và Thiết lập cấu hình

### 8. Làm cách nào để nâng cao độ chính xác khi nhận dạng giọng nói?

Độ chính xác nhận dạng chủ yếu phụ thuộc vào kích thước mô hình bạn chọn và các cấu hình liên quan.

*   **Lựa chọn mô hình (Model)**: Trong chế độ nhận dạng "faster" hoặc "openai", mô hình càng lớn thì độ chính xác càng cao, tuy nhiên tốc độ xử lý sẽ chậm hơn và ngốn tài nguyên phần cứng hơn.
    *   `tiny`: Kích thước nhỏ nhất, tốc độ nhanh nhất nhưng tỷ lệ lỗi cao.
    *   `base` / `small` / `medium`: Cân bằng tốt giữa tài nguyên sử dụng và chất lượng, là các mô hình được dùng nhiều nhất.
    *   `large-v3`: Kích thước lớn nhất, chất lượng tốt nhất, yêu cầu phần cứng cao nhất (yêu cầu bộ nhớ VRAM card đồ họa từ 8GB trở lên).
*   **Tối ưu hóa cấu hình**: Nhấp chuột vào `Menu -> Công cụ -> Tùy chọn nâng cao` (Advanced Options)

Tìm đến phần cấu hình `Điều chỉnh nhận dạng giọng nói faster/openai` và chỉnh sửa các tham số sau:

- **Ngưỡng giọng nói (Speech Threshold)** đặt thành `0.5`
- **Thời lượng tối thiểu/mili-giây (Min speech duration)** đặt thành `3000`
- **Thời lượng giọng nói tối đa/giây (Max speech duration)** đặt thành `6`
- **Thời gian lặng phân tách/mili-giây (Silence duration)** đặt thành `140`
- **Từ nóng (Hotwords)**: Nếu video chứa nhiều thuật ngữ chuyên ngành, bạn hãy điền chúng vào đây, phân tách nhau bằng dấu phẩy.

*   **Xử lý khử tạp âm**: Nếu video có nhạc nền lớn hoặc nhiều tiếng ồn, hãy chọn mục `Tách giọng nói & nhạc nền` (Separate vocal) trong phần thiết lập tham số nâng cao để cải thiện đáng kể kết quả nhận dạng.

### 9. Tại sao chất lượng hình ảnh/độ phân giải của video đầu ra bị giảm sút?

Mọi thao tác liên quan đến **nén lại/mã hóa lại video (re-encode)** đều gây suy hao chất lượng hình ảnh ở một mức độ nào đó. Để giữ nguyên chất lượng video gốc cao nhất, hãy đảm bảo đáp ứng các điều kiện sau:

1.  **Định dạng video gốc**: Sử dụng video định dạng **MP4 mã hóa H.264 (libx264)** có độ tương thích tốt nhất.
2.  **Tắt tính năng làm chậm video**: Không chọn mục "Làm chậm video" trong phần cài đặt.
3.  **Không nhúng phụ đề cứng**: Bạn có thể chọn không nhúng phụ đề hoặc chọn nhúng **phụ đề mềm**. Việc nhúng phụ đề cứng sẽ ép buộc FFmpeg phải mã hóa lại toàn bộ các khung hình video.
4.  **Cấu hình chất lượng đầu ra (CRF)**: Trong tùy chọn nâng cao, chỉ số kiểm soát chất lượng video mặc định là 23. Bạn có thể giảm xuống 18 hoặc thấp hơn (thấp nhất là 0). Giá trị càng thấp chất lượng video đầu ra càng cao, đi kèm dung lượng file sẽ lớn hơn.
5.  **Tốc độ nén video**: Mặc định là `fast`, bạn có thể đổi thành `slow` hoặc `slower` để chất lượng mã hóa tốt hơn, đánh đổi bằng việc thời gian xuất video lâu hơn.
6.  **Chuẩn mã hóa**: Mặc định là `264`, có thể chọn `265` để có chất lượng nén hình ảnh tốt hơn ở cùng mức dung lượng.

### 10. Tại sao kích thước tệp video đầu ra lại quá lớn?

1. Điều chỉnh chỉ số **Kiểm soát chất lượng video đầu ra (CRF)** trong phần tùy chọn nâng cao lên khoảng 25-51. Giá trị này càng lớn dung lượng file xuất ra càng nhỏ, tuy nhiên chất lượng hình ảnh sẽ giảm.
2. Chọn chuẩn mã hóa là `265` thay vì `264` để tối ưu hóa nén dung lượng mà vẫn giữ nguyên chất lượng hình ảnh.

### 11. Làm thế nào để cấu hình proxy mạng?

Một số dịch vụ dịch thuật hoặc lồng tiếng trực tuyến (như Google, OpenAI, Gemini) có thể bị chặn truy cập ở một số khu vực hoặc quốc gia, yêu cầu cần cấu hình proxy mạng.

*   **Cách cấu hình**: Điền địa chỉ proxy của bạn vào ô "Địa chỉ proxy mạng" trên giao diện chính của phần mềm.
*   **Định dạng yêu cầu**: Thường có định dạng dạng `http://127.0.0.1:10808` (cổng port cụ thể tùy thuộc vào phần mềm VPN/Proxy bạn đang sử dụng).
*   **Lưu ý quan trọng**: Nếu bạn không sử dụng proxy hoặc không có dịch vụ proxy khả dụng, **vui lòng để trống ô này**. Việc cấu hình sai địa chỉ sẽ dẫn đến lỗi kết nối phần mềm.
*   **Các API trong nước không cần proxy**: Các dịch vụ như Baidu, Tencent, Alibaba, DeepSeek, Zhipu AI, Volcano... mặc định chạy trực tiếp không đi qua proxy.
*   **Các dịch vụ cục bộ không cần proxy**: GPT-SoVITS, ChatTTS, F5-TTS... chạy ngoại tuyến nên sẽ tự động bỏ qua cấu hình proxy.

### 12. Làm thế nào để tùy chỉnh phông chữ, màu sắc và kiểu dáng phụ đề?

Trên giao diện chính -> chọn Cài đặt tham số nâng cao -> chọn mục **Chỉnh sửa kiểu hiển thị phụ đề cứng** (Hard Subtitle Style Edit).

---

## Phần 3: Các vấn đề về nhận dạng giọng nói (ASR)

### 13. Kết quả nhận dạng bị trống hoặc bị lỗi ký tự lạ (loạn mã)

*   **Nguyên nhân**: Chọn sai ngôn ngữ gốc, video không có tiếng người nói rõ ràng, hoặc card đồ họa bị tràn bộ nhớ VRAM.
*   **Giải pháp**:
    1.  Kiểm tra xem "Ngôn ngữ gốc" đã chọn chính xác hay chưa (hạn chế dùng Auto nếu ngôn ngữ quá đặc thù).
    2.  Kiểm tra xem video có bị tiếng nhạc nền lấn át giọng nói hay không (hãy thử bật tính năng tách giọng hát).
    3.  Lỗi tràn VRAM: Hãy hạ kích thước mô hình nhận dạng xuống (ví dụ dùng `small` thay vì `large`), giảm `beam_size`, hoặc chọn định dạng lượng tử hóa `int8` trong tùy chọn nâng cao.
    4.  Thử đổi sang kênh nhận dạng giọng nói khác (ví dụ chuyển từ faster-whisper sang openai-whisper).

### 14. Tốc độ nhận dạng giọng nói rất chậm

*   **Nguyên nhân**: Chọn mô hình kích thước lớn nhưng chưa bật tính năng tăng tốc bằng card đồ họa GPU.
*   **Giải pháp**:
    1.  **Bật tăng tốc CUDA**: Đảm bảo máy tính đã được cài đặt driver tương thích, CUDA Toolkit 12.8+ và cuDNN 9.x, sau đó tích chọn mục `Tăng tốc CUDA` trên phần mềm.
    2.  **Sử dụng mô hình nhỏ hơn**: Đổi từ mô hình `large-v3` sang `medium` hoặc `small`.
    3.  **Tối ưu hóa chạy trên CPU**: Trong cài đặt nâng cao, đổi `Kiểu dữ liệu tính toán` sang `int8`.

### 15. Lỗi báo tràn bộ nhớ card đồ họa / RAM (`Unable to allocate`, `CUDA out of memory`)

*   **Nguyên nhân**: Mô hình nhận dạng quá nặng hoặc bộ nhớ card đồ họa đang bị các chương trình khác chiếm dụng.
*   **Giải pháp**:
    1.  **Hạ cấp mô hình**: Đổi mô hình nhận dạng từ `large-v3` sang `medium`, `small` hoặc `base`. Mô hình `large-v3` yêu cầu tối thiểu 8GB VRAM card đồ họa để hoạt động bình thường.
    2.  **Điều chỉnh cài đặt nâng cao**: Vào `Menu -> Công cụ -> Tùy chọn nâng cao`, thay đổi các cấu hình sau:
        *   `Kiểu dữ liệu CUDA`: Đổi từ `float32` sang `float16` hoặc `int8`.
        *   `beam_size`: Giảm từ `5` xuống `1`.
        *   `best_of`: Giảm từ `5` xuống `1`.
        *   `Bối cảnh (Condition on previous text)`: Chuyển từ `true` sang `false`.
    3.  **Cấu hình đa card đồ họa**: Nếu máy tính có nhiều card đồ họa, kiểm tra xem card mặc định số 0 có dung lượng VRAM quá thấp không. Hãy nâng cấp ứng dụng lên bản v3.98-317 trở lên để phần mềm tự động ưu tiên lựa chọn card đồ họa có bộ nhớ lớn nhất.

### 16. Phân tách người nói (Diarization) không chính xác

*   **Nguyên nhân**: Mô hình phân tách gặp khó khăn trong các môi trường tạp âm lớn hoặc có nhiều người nói chen lấn nhau.
*   **Giải pháp**:
    1.  Bật tính năng `Nhận dạng người nói` trong phần thiết lập tham số nâng cao và chỉ định rõ số lượng người nói thực tế.
    2.  Trong cài đặt nâng cao, hãy thử thay đổi mô hình backend phân tách người nói (mặc định, Ali CAM++, pyannote).
    3.  Lưu ý khi chọn mô hình `pyannote`, bạn cần đăng ký tài khoản trên HuggingFace để lấy mã Token và đồng ý với điều khoản cấp quyền của mô hình.

### 17. Kết quả câu thoại tệ hơn sau khi sử dụng LLM để cấu trúc lại câu

*   **Nguyên nhân**: Mô hình ngôn ngữ cục bộ kích thước nhỏ (ví dụ dòng 7B) chưa đủ độ thông minh, hoặc cấu trúc câu lệnh prompt quá phức tạp.
*   **Giải pháp**:
    1.  Chuyển sang sử dụng các mô hình trực tuyến mạnh mẽ hơn như DeepSeek-V3, GPT-4o...
    2.  Tối ưu rút gọn câu lệnh prompt (chỉnh sửa file mẫu prompt tại thư mục `videotrans/prompts/recharge/recharge-llm.txt`).
    3.  **Không khuyên dùng** tính năng cấu trúc lại câu bằng LLM nếu bạn đang sử dụng chế độ nhân bản giọng nói `clone`.

### 18. Phụ đề và giọng đọc lồng tiếng bị lệch mốc thời gian sau khi dịch

Đây là hiện tượng phổ biến xảy ra do sự khác biệt giữa các ngôn ngữ.

*   **Nguyên nhân**: Độ dài câu thoại và số lượng âm tiết thay đổi sau khi dịch từ ngôn ngữ này sang ngôn ngữ khác, khiến thời lượng phát âm thanh lồng tiếng thực tế bị ngắn đi hoặc dài ra so với mốc thời gian phụ đề gốc. Ví dụ một câu tiếng Trung nói trong 2 giây khi dịch sang tiếng Anh có thể mất tới 3-4 giây để đọc hết.
*   **Giải pháp**:
    1.  **Bật tăng tốc âm thanh**: Chọn mục `Tăng tốc âm thanh` để tự động đẩy nhanh tốc độ đọc của các câu thoại lồng tiếng bị tràn mốc thời gian phụ đề.
    2.  **Bật làm chậm video**: Chọn mục `Làm chậm video` để làm chậm khung hình hình ảnh video khớp với âm thoại.
    3.  **Chọn kết hợp cả hai**: Khi tỷ lệ lệch vượt quá 1.2x, hệ thống sẽ tự động phối hợp tăng tốc âm thanh một nửa và làm chậm video một nửa để đạt chất lượng tự nhiên nhất.
    4.  **Căn chỉnh tốc độ nói chung**: Thiết lập tham số `Tốc độ lồng tiếng` (ví dụ nhập `+10%`) để đẩy nhanh tốc độ nói của toàn bộ file lồng tiếng.
    5.  **Sử dụng nhận dạng lần 2**: Chọn mục `Nhận dạng lần 2` để sinh lại mốc thời gian phụ đề khớp hoàn toàn với file lồng tiếng mới sau khi ghép.

> Xem chi tiết nguyên lý tại file [Hướng dẫn nguyên lý căn chỉnh đồng bộ thời gian âm thanh - hình ảnh](Synchronize_vi.md).

### 19. Nhận dạng lần 2 là gì? Khi nào nên sử dụng?

Nhận dạng lần 2 (2-pass ASR) là thao tác chạy nhận dạng giọng nói lại một lần nữa trên file âm thanh lồng tiếng mới thu được, từ đó sinh ra mốc thời gian phụ đề chính xác nhất và câu thoại gọn gàng nhất theo tiếng nói mới.

*   **Trường hợp áp dụng**: Thích hợp khi bạn chọn chế độ nhúng một dòng phụ đề và yêu cầu chữ phụ đề khớp khít tuyệt đối với giọng lồng tiếng mới.
*   **Cách cài đặt**: Chọn mục `Nhận dạng lần 2` trên giao diện. Bạn có thể tinh chỉnh thời lượng tối đa/tối thiểu của phân đoạn nhận dạng lần 2 trong phần cấu hình nâng cao.
*   **Lưu ý**: Bật tính năng này sẽ khiến quá trình xuất video tổng hợp mất thêm thời gian xử lý.

---

## Phần 4: Các vấn đề về dịch thuật

### 20. Bản dịch có dòng trống hoặc chứa cả câu lệnh prompt của AI

*   **Nguyên nhân**: Mô hình ngôn ngữ cục bộ kích thước nhỏ chưa đủ thông minh để hiểu đúng định dạng yêu cầu, hoặc AI tự động gộp các dòng phụ đề lại với nhau.
*   **Giải pháp**:
    1.  Thay đổi sang sử dụng các mô hình trực tuyến thông minh hơn như DeepSeek, GPT-4...
    2.  Tắt tùy chọn "Gửi toàn bộ phụ đề" trong cài đặt nâng cao, chuyển sang chế độ dịch tuần tự theo từng dòng.
    3.  Thiết lập tham số luồng dịch `trans_thread=1` để giảm xử lý đồng thời.
    4.  [Xem chi tiết nguyên nhân và cách xử lý tại đây](/faq17).

### 21. Bản dịch bị lọc bỏ do vi phạm chính sách nội dung của AI

*   **Thông báo lỗi**: `Nội dung bị lọc bỏ do vi phạm chính sách kiểm soát nội dung AI`.
*   **Nguyên nhân**: Nội dung trong video của bạn bị hệ thống kiểm duyệt bảo mật của nhà cung cấp dịch vụ AI chặn lại.
*   **Giải pháp**:
    1.  Biên tập sửa lại file phụ đề gốc để xóa bỏ hoặc thay thế các từ nhạy cảm.
    2.  Thay đổi kênh dịch thuật khác (ví dụ chuyển từ OpenAI sang DeepSeek hoặc Google Translate).

### 22. Bản dịch bị lệch dòng so với phụ đề gốc

*   **Nguyên nhân**: AI tự động gộp các dòng phụ đề trong quá trình dịch dẫn đến tổng số dòng bị lệch so với phụ đề gốc.
*   **Giải pháp**:
    1.  Tắt tùy chọn "Gửi toàn bộ phụ đề" trong phần cấu hình nâng cao.
    2.  Thiết lập số luồng dịch song song bằng 1.
    3.  Sử dụng các mô hình trực tuyến có khả năng xử lý ngữ cảnh tốt hơn.

### 23. Bộ nhớ đệm dịch thuật (Translate Cache) gây lỗi kết quả

*   **Nguyên nhân**: Phần mềm có cơ chế lưu đệm kết quả dịch. Khi bạn sửa đổi câu lệnh prompt hoặc thay đổi kênh dịch nhưng file đầu vào giữ nguyên, hệ thống vẫn lấy kết quả cũ từ cache.
*   **Giải pháp**:
    1.  Tích chọn mục `Dọn dẹp tệp đã tạo` (Clear generated files) trên giao diện chính của phần mềm.
    2.  Hoặc truy cập thư mục `tmp/translate_cache/` của dự án và xóa bỏ toàn bộ các file cache chứa tại đây.

---

## Phần 5: Các vấn đề về lồng tiếng (TTS)

### 24. Dịch vụ Edge-TTS báo lỗi 403 hoặc tạo ra file âm thanh lặng không có tiếng

*   **Nguyên nhân**: Bạn bị giới hạn lượt truy cập (rate limit) từ dịch vụ miễn phí của Microsoft do gửi quá nhiều yêu cầu lồng tiếng trong thời gian ngắn.
*   **Giải pháp**:
    1.  Vào phần cài đặt nâng cao, chỉnh tham số "Số luồng lồng tiếng đồng thời" (TTS thread count) xuống bằng 1.
    2.  Chỉnh tham số "Thời gian tạm dừng sau khi lồng tiếng" lên khoảng 5-10 giây để giãn cách các yêu cầu gửi đi.
    3.  Nếu đang cấu hình proxy mạng, có thể Edge-TTS gặp lỗi không chạy được qua proxy của bạn. Hãy tạo một file trống tên `edgetts-noproxy.txt` trong thư mục gốc của phần mềm để bắt buộc Edge-TTS chạy trực tiếp bỏ qua cấu hình proxy.

### 25. Các dịch vụ F5-TTS / CosyVoice / GPT-SoVITS báo lỗi không kết nối được

*   **Nguyên nhân**: Bạn chưa khởi chạy dịch vụ TTS ngoại tuyến cục bộ tương ứng hoặc điền sai địa chỉ API kết nối.
*   **Giải pháp**:
    1.  Đảm bảo cửa sổ dòng lệnh terminal của dịch vụ TTS ngoại tuyến của bạn vẫn đang mở và chạy bình thường.
    2.  Kiểm tra lại xem địa chỉ cổng kết nối API đã chính xác hay chưa.
    3.  Đối với dịch vụ GPT-SoVITS, bạn bắt buộc phải khởi chạy qua file dịch vụ `api.py` hoặc `api_v2.py`, không được kết nối đến cổng giao diện web 7860.
    4.  Nếu đang cấu hình địa chỉ là `0.0.0.0`, hãy đổi lại thành `127.0.0.1`.

### 26. Dịch vụ GPT-SoVITS báo lỗi `{"detail":"Not Found"}`

*   **Nguyên nhân**: Sai phiên bản API hoặc cấu hình sai cổng kết nối.
*   **Giải pháp**:
    1.  Kiểm tra xem dịch vụ đang khởi chạy bằng file `api.py` hay `api_v2.py`, sau đó chọn đúng cờ cấu hình `api_v2?` tương ứng trên phần mềm.
    2.  Đảm bảo địa chỉ cổng kết nối trỏ đến API (mặc định là cổng 9880), không trỏ đến cổng giao diện WebUI (mặc định là 7860).

### 27. Dịch vụ Index-TTS báo lỗi `Value: 'Same as the voice reference' is not in the list`

*   **Nguyên nhân**: Lỗi dịch thuật bất đồng bộ đa ngôn ngữ bên trong gói phần mềm Index-TTS.
*   **Giải pháp**: Mở tệp `webui.py` trong thư mục gốc của dự án Index-TTS của bạn, tìm và thay thế chuỗi chữ Trung Quốc `i18n("与音色参考音频相同")` thành chuỗi tiếng Anh `Same as the voice reference`.

### 28. Dịch vụ Azure-TTS báo lỗi thiếu thư viện `Microsoft.CognitiveServices.Speech.core.dll`

*   **Nguyên nhân**: Máy tính của bạn thiếu bộ thư viện runtime VC++ của Microsoft.
*   **Giải pháp**:
    1.  Nếu bạn đang sử dụng gói cập nhật patch, hãy tải lại gói phần mềm đầy đủ.
    2.  Tải và cài đặt gói [Microsoft Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe) mới nhất, sau đó khởi động lại máy tính.

### 29. Giọng đọc lồng tiếng có âm thanh kim loại hoặc bị rè

*   **Nguyên nhân**: Tỷ lệ tăng tốc độ âm thoại quá cao (lớn hơn 3x) hoặc file âm thanh mẫu dùng để clone giọng nói có chất lượng kém.
*   **Giải pháp**:
    1.  Bật thêm tính năng làm chậm video để chia sẻ bớt thời lượng chênh lệch với âm thanh.
    2.  Nâng cao chất lượng file âm thanh mẫu: sử dụng file định dạng WAV thời lượng 5-10 giây có tiếng nói đơn rõ ràng, không lẫn tiếng ồn hay nhạc nền.
    3.  Bật tính năng tách giọng nói gốc để lọc sạch tạp âm nền.

---

## Phần 6: Các vấn đề về nhân bản giọng nói (Voice Cloning)

### 30. Lồng tiếng bằng nhân vật nhân bản (clone) bị lỗi hoặc chất lượng giọng rất kém

*   **Nguyên nhân**: File âm thanh tham chiếu cắt từ video không nằm trong khoảng độ dài vàng từ 3-10 giây, hoặc mốc thời gian phụ đề bị xáo trộn do sử dụng tính năng cấu trúc lại câu bằng LLM.
*   **Giải pháp**:
    1.  **Tuyệt đối không bật tính năng cấu trúc lại câu bằng LLM**: LLM tự động ngắt lại câu sẽ làm thay đổi mốc thời gian, khiến phần mềm trích xuất sai phân đoạn âm thanh tham chiếu từ video gốc.
    2.  **Kiểm soát thời lượng phụ đề**: Trong phần cấu hình nâng cao nhận dạng giọng nói, đặt `Thời lượng tối đa` từ 6-10 giây và `Thời lượng tối thiểu` từ 3000-4000 mili-giây.
    3.  Tích chọn mục `Gộp phụ đề quá ngắn` và `Tự động phân tách phân đoạn Whisper`.
    4.  Hãy thử chuyển sang kênh lồng tiếng `OmniVoice-TTS` để có độ tương thích tốt hơn với các file tham chiếu ngắn.
    5.  Bật tính năng tách giọng hát để lấy được file âm thanh tham chiếu sạch nhất.

### 31. Làm thế nào để sử dụng tệp âm thanh tham chiếu tự chuẩn bị ngoài?

1.  Chuẩn bị một tệp âm thanh định dạng WAV độ dài 5-10 giây (giọng nói rõ, không nhạc nền).
2.  Sao chép tệp này vào thư mục con `f5-tts/` nằm trong thư mục gốc của phần mềm.
3.  Vào `Menu -> Thiết lập TTS -> Cài đặt âm thanh tham chiếu`, điền theo định dạng: `tên_file.wav#nội_dung_chữ_trong_âm_thanh`.
4.  Tại giao diện chính, ở ô chọn nhân vật lồng tiếng, bạn sẽ tìm thấy tên file âm thanh tương ứng để chọn.

> **Lưu ý**: Đối với dịch vụ GPT-SoVITS, file âm thanh tham chiếu của bạn cần được đặt trong thư mục gốc của phần mềm GPT-SoVITS, không đặt trong thư mục `f5-tts/` của pyVideoTrans.

---

## Phần 7: Đồng bộ và xuất video thành phẩm

### 32. Gặp lỗi `ffprobe exec error` hoặc lỗi liên quan đến `ffmpeg` trong quá trình chạy

*   **Nguyên nhân**: Đường dẫn chứa file video đầu vào quá dài hoặc chứa ký tự đặc biệt kén hệ thống.
*   **Giải pháp**:
    1.  Di chuyển file video của bạn vào các thư mục nông hơn (ví dụ đưa ra `D:\videos`).
    2.  Đổi tên file thành ký tự tiếng Anh viết liền hoặc chữ số ngắn gọn.
    3.  Xóa bỏ các ký tự lạ hoặc biểu tượng cảm xúc emoji trong tên file.

### 33. Phần mềm báo lỗi video "Không chứa luồng âm thanh" (No audio track)

*   **Nguyên nhân 1**: Video thực tế không có tiếng (một số nền tảng chia tách luồng hình và luồng tiếng riêng biệt khi tải về).
*   **Nguyên nhân 2**: Định dạng codec âm thanh trong video không được hỗ trợ (ví dụ mã AV1).
*   **Nguyên nhân 3**: Tiếng ồn nền quá lớn lấn át hoàn toàn giọng nói con người.
*   **Giải pháp**:
    1.  Mở video trên máy tính để kiểm tra xem có phát ra tiếng bình thường hay không.
    2.  Thực hiện chuyển đổi video của bạn về định dạng mã hóa chuẩn H.264/MP4 bằng các phần mềm chuyển đổi trước khi đưa vào hệ thống.
    3.  Bật tính năng giảm nhiễu hoặc tách giọng nói.

### 34. Làm thế nào để xuất video chất lượng gốc không nén lại hình ảnh (lossless)?

Video thành phẩm sẽ được ghép nối trực tiếp không qua mã hóa lại nếu đáp ứng đủ các điều kiện sau:
1.  Video đầu vào sử dụng chuẩn mã hóa tương thích tốt `mp4/h.264/yuv420p`.
2.  Trong cài đặt nâng cao, mục `Mã hóa 264/265` chọn giá trị `264`.
3.  Không bật tính năng `Làm chậm video`.
4.  Không bật tính năng nhúng `Phụ đề cứng` (nhúng phụ đề mềm sẽ không ảnh hưởng).

> Lưu ý: Nếu âm thanh lồng tiếng sau khi xử lý dài hơn thời lượng video gốc, phần âm thanh vượt quá thời lượng video sẽ bị cắt bỏ ở cuối để đảm bảo khớp thời lượng hình ảnh gốc.

### 35. Chất lượng âm thoại lồng tiếng, phụ đề và hình ảnh không khớp đồng bộ sau khi chạy xong

Đây là hiện tượng chênh lệch thời gian tự nhiên giữa các ngôn ngữ khi dịch.

*   **Nguyên nhân**: Cùng một ý nghĩa nhưng độ dài câu thoại và thời gian đọc thay đổi tùy theo ngôn ngữ.
*   **Giải pháp**:
    1.  Kích hoạt tính năng `Tăng tốc âm thanh` và/hoặc `Làm chậm video`.
    2.  Điều chỉnh thanh `Tốc độ lồng tiếng` (ví dụ lên `+10%`) để tăng tốc độ đọc chung.
    3.  Kích hoạt tính năng `Nhận dạng lần 2` để vẽ lại mốc thời gian phụ đề khớp với tiếng nói mới.
    4.  Xem thêm chi tiết tại [Hướng dẫn nguyên lý căn chỉnh đồng bộ thời gian âm thanh - hình ảnh](Synchronize_vi.md).

### 36. Ứng dụng liên tục báo lỗi tràn RAM/VRAM card đồ họa

*   **Giải pháp xử lý (thực hiện theo thứ tự ưu tiên)**:
    1.  **Hạ cấp mô hình nhận dạng**: Chuyển từ mô hình `large-v3` sang các mô hình nhỏ hơn như `medium`, `small` hoặc `base`.
    2.  **Tối ưu hóa cài đặt nâng cao**:
        *   `Kiểu dữ liệu CUDA`: Chuyển sang `float16` hoặc `int8`.
        *   `beam_size`: Chuyển về mức `1`.
        *   `best_of`: Chuyển về mức `1`.
        *   `Bối cảnh (Condition on previous text)`: Chọn `false`.

### 37. Máy tính có cài CUDA nhưng phần mềm vẫn báo không nhận GPU?

Vui lòng kiểm tra các nguyên nhân sau:

*   **Phiên bản CUDA không tương thích**: Phần mềm yêu cầu cài đặt CUDA Toolkit phiên bản từ 12.8 trở lên.
*   **Driver card đồ họa quá cũ**: Vui lòng truy cập trang chủ NVIDIA để tải và cập nhật driver card màn hình mới nhất.
*   **Thiếu thư viện cuDNN**: Hãy chắc chắn cuDNN 9.x đã được cài đặt và cấu hình đường dẫn trong biến môi trường hệ thống.
*   **Hạn chế thiết bị**: Tính năng tăng tốc CUDA chỉ hỗ trợ card đồ họa hãng NVIDIA (N card). Các card AMD hoặc Intel GPU không hỗ trợ giao thức này.
*   **Thiếu biến môi trường**: Kiểm tra xem biến môi trường của hệ thống đã trỏ đến thư mục `bin` và `lib` của CUDA hay chưa.

### 38. Mức độ sử dụng GPU rất thấp khi chạy, có bình thường không?

**Hoàn toàn bình thường**. Luồng xử lý của ứng dụng diễn ra theo thứ tự: `Nhận dạng giọng nói (ASR) -> Dịch phụ đề -> Lồng tiếng (TTS) -> Tổng hợp ghép nối video`.

Chỉ duy nhất ở bước đầu tiên **"Nhận dạng giọng nói (ASR)"**, hệ thống mới tận dụng tối đa tài nguyên của card đồ họa GPU để chạy mô hình suy luận. Toàn bộ các bước sau đó chủ yếu sử dụng tài nguyên của CPU. Vì vậy GPU ở trạng thái rảnh rỗi trong phần lớn thời gian chạy là hoàn toàn bình thường.

### 39. Chạy xong vài video thấy dung lượng ổ cứng bị đầy?

Hiện tượng này thường xảy ra khi bạn bật tính năng "Làm chậm video" để đồng bộ âm hình.

*   **Nguyên nhân**: Tính năng này sẽ thực hiện cắt video gốc thành rất nhiều file nhỏ tương ứng với từng câu phụ đề để xử lý tốc độ hình ảnh cho từng đoạn, từ đó phát sinh ra lượng file tạm cực kỳ lớn trong ổ đĩa của bạn.
*   **Giải pháp**:
    1.  **Dọn dẹp thủ công**: Sau khi hoàn thành xuất video, bạn có thể truy cập thư mục **`tmp/`** trong thư mục gốc của phần mềm và xóa sạch các file tạm nằm trong đó.
    2.  **Dọn dẹp tự động**: Khi bạn đóng phần mềm một cách bình thường, chương trình sẽ tự động dọn dẹp các tệp tạm này cho bạn.

### 40. Thử chạy lại một video nhưng kết quả nhận dạng và phụ đề không thay đổi?

*   **Nguyên nhân**: Phần mềm có cơ chế lưu trữ kết quả nhận dạng trước đó để tiết kiệm thời gian chạy. Nếu phát hiện tệp video đã có kết quả cũ, nó sẽ nạp trực tiếp kết quả cũ lên mà không chạy lại.
*   **Giải pháp**: Tích chọn vào mục **`Dọn dẹp tệp đã tạo`** (Clear generated files) ở góc trên bên trái của giao diện chính trước khi chạy.

![](https://pvtr2.pyvideotrans.com/1760281358634_image.png)

---

## Phần 8: Các vấn đề khi xử lý hàng loạt

### 41. Xử lý hàng loạt nhiều video bị dừng/đóng băng giữa chừng

Mặc định khi chạy hàng loạt, chương trình sẽ xử lý song song chéo nhiều phân đoạn của các video để tối ưu hiệu năng, điều này có thể gây quá tải và làm đứng tài nguyên máy của bạn.

*   **Giải pháp**: Tích chọn mục **Xử lý tuần tự khi dịch hàng loạt** (Force serial processing in batch mode) trong phần tùy chọn nâng cao để bắt buộc phần mềm xử lý xong video này mới chuyển sang video tiếp theo.

### 42. Làm thế nào để kiểm soát số lượng tác vụ chạy đồng thời khi dịch hàng loạt?

Trong mục `Tùy chọn nâng cao -> Thiết lập chung`:
*   `Số tác vụ CPU đồng thời`: Đặt số tác vụ chạy song song tối đa bằng CPU, khuyên dùng không vượt quá số nhân vật lý của máy bạn.
*   `Số tác vụ GPU đồng thời`: Số lượng tác vụ chạy song song trên GPU, trừ khi máy bạn lắp nhiều card hoặc VRAM card màn hình > 24G, khuyến nghị nên đặt giá trị là 1.
*   `Số lượng mỗi lô khi dịch hàng loạt`: Đặt bằng 1 để xử lý tuần tự từng file một, đặt bằng 0 để hệ thống tự động chạy đồng thời toàn bộ file đầu vào.

---

## Phần 9: Giải thích chi tiết các tùy chọn nâng cao

### 43. Sự khác biệt giữa tăng tốc âm thanh và làm chậm video?

| Tùy chọn | Hiệu quả xử lý | Trường hợp áp dụng |
|------|------|---------|
| **Tăng tốc âm thanh** | Đọc nhanh file lồng tiếng để nhét vừa mốc thời gian phụ đề gốc. Giọng nói có thể bị dồn dập nhẹ. | Âm thoại lồng tiếng dài hơn phụ đề gốc từ 1 đến 2 lần. |
| **Làm chậm video** | Kéo dài thời gian hiển thị hình ảnh video để chờ đọc hết âm thoại. Hình ảnh có thể bị gián đoạn nhẹ. | Âm thoại lồng tiếng dài hơn phụ đề gốc từ 2 lần trở lên. |
| **Kết hợp cả hai** | Mỗi bên chia đôi khoảng chênh lệch thời gian để cùng gánh vác, đem lại cảm giác tự nhiên nhất. | Âm thoại lồng tiếng dài hơn rất nhiều so với phụ đề gốc. |

### 44. Vai trò của tùy chọn `Gửi toàn bộ phụ đề` khi dịch thuật?

Khi bật tùy chọn này, phần mềm sẽ gửi kèm cả thông tin mốc thời gian và số thứ tự dòng phụ đề cho AI để AI hiểu rõ ngữ cảnh câu nói và dịch mượt mà hơn. Tuy nhiên điều này có thể khiến AI tự động gộp các dòng phụ đề lại với nhau. Khuyến nghị:
*   Bật tùy chọn này khi sử dụng các mô hình trực tuyến thông minh (như DeepSeek, GPT-4o).
*   Tắt tùy chọn này khi sử dụng các mô hình ngoại tuyến cục bộ có kích thước nhỏ.

### 45. Sự khác biệt giữa `Nhận dạng lần 2` và `Cấu trúc lại câu bằng LLM`?

| Tùy chọn | Thời điểm chạy | Công dụng |
|------|------|------|
| **Cấu trúc lại câu bằng LLM** | Ngay sau khi nhận dạng giọng nói xong | AI giúp sửa các lỗi chính tả, ngắt lại các câu quá dài cho tự nhiên hơn |
| **Nhận dạng lần 2** | Sau khi đã tạo xong file lồng tiếng thành phẩm | Chạy nhận dạng lại file ghi âm lồng tiếng mới để tạo mốc thời gian phụ đề khớp khít tuyệt đối với tiếng mới |

> **Không khuyên dùng** tính năng cấu trúc lại câu bằng LLM nếu bạn đang chọn giọng đọc lồng tiếng dạng nhân bản `clone`.

### 46. Lựa chọn loại phụ đề nhúng như thế nào cho phù hợp?

| Loại nhúng | Giải thích | Trường hợp áp dụng |
|------|------|---------|
| Không nhúng phụ đề | Chỉ thay thế luồng tiếng mới cho video đầu ra, hoàn toàn không xuất hiện chữ phụ đề trên màn hình. | Khi chỉ có nhu cầu nghe tiếng lồng. |
| Nhúng phụ đề cứng | Vẽ chết chữ phụ đề vào khung hình video thành phẩm, không tắt đi được. | Đảm bảo hiển thị phụ đề trên mọi loại trình phát video khác nhau. |
| Nhúng phụ đề mềm | Phụ đề được tích hợp thành một track riêng trong video, người dùng có thể tùy ý bật hoặc tắt trên trình phát. | Khi cần sự linh hoạt, dễ dàng bật tắt hoặc trích xuất phụ đề sau này. |
| Nhúng phụ đề cứng (Song ngữ) | Vẽ chết đồng thời cả dòng phụ đề dịch và phụ đề gốc trên khung hình. | Phục vụ nhu cầu xem đối chiếu song ngữ. |
| Nhúng phụ đề mềm (Song ngữ) | Tích hợp track phụ đề song ngữ vào video, hỗ trợ bật tắt linh hoạt. | Xem đối chiếu song ngữ và có thể ẩn đi khi cần. |

---

## Phần 10: Tệp tin và Đường dẫn lưu trữ

### 47. Yêu cầu đối với đường dẫn file đầu vào?

1.  **Độ dài đường dẫn**: Windows giới hạn độ dài đường dẫn dòng lệnh tối đa 260 ký tự, vì vậy bạn nên đặt tệp ở các thư mục ngắn.
2.  **Ký tự đặc biệt**: Tên file không được chứa các ký tự lạ kén hệ thống như `?*` hoặc biểu tượng cảm xúc emoji.
3.  **Ký tự tiếng Việt có dấu**: Mặc dù phần mềm có hỗ trợ, nhưng khuyến nghị nên đặt tên không dấu để hạn chế tối đa các lỗi tương thích thư viện ngoài.
4.  **Khoảng trắng**: Đường dẫn có thể chứa khoảng trắng, nhưng tốt nhất vẫn nên hạn chế sử dụng.

### 48. Video thành phẩm được lưu ở đâu?

*   **Vị trí mặc định**: Nằm trong thư mục con `_video_out/` ngay tại thư mục chứa file video gốc của bạn.
*   **Vị trí xuất các tính năng đơn lẻ**: Các file kết quả của tính năng chạy hàng loạt như trích phụ đề, lồng tiếng, dịch phụ đề... được lưu trong thư mục `output/` của dự án.
*   **Tùy chỉnh thư mục lưu**: Bạn có thể chọn đường dẫn lưu tệp mong muốn tại ô chọn thư mục đầu ra trên giao diện chính.

### 49. Làm cách nào để nạp file phụ đề SRT có sẵn vào hệ thống để dịch nhanh?

1.  Tại thư mục chứa video gốc, tạo một thư mục con tên là `_video_out/`.
2.  Trong thư mục đó, tạo tiếp thư mục con trùng tên với file video (ví dụ video tên là `myvideo.mp4` thì tạo thư mục con tên `myvideo-mp4`, bắt buộc viết liền và giữ nguyên phần mở rộng định dạng).
3.  Sao chép các file phụ đề của bạn vào thư mục con này, đặt tên tương ứng là `zh-cn.srt` (phụ đề ngôn ngữ gốc) và `en.srt` (phụ đề ngôn ngữ đích).
4.  Đưa video vào phần mềm chạy dịch thuật, hệ thống phát hiện có phụ đề sẵn sẽ tự động bỏ qua giai đoạn nhận dạng giọng nói ASR và giai đoạn dịch thuật để tiến hành lồng tiếng luôn.

---

## Phần 11: Các câu hỏi về giao diện dòng lệnh CLI

### 50. Cách sử dụng cơ bản của giao diện CLI?

```bash
uv run cli.py --task <Loại_nhiệm_vụ> --name "<Đường_dẫn_tệp>" [Tham số khác]
```

Các loại nhiệm vụ hỗ trợ: `stt` (nhận dạng giọng nói), `tts` (lồng tiếng văn bản), `sts` (dịch phụ đề), `vtv` (dịch video đầy đủ).

### 51. Cách kiểm tra danh sách các kênh và mã ngôn ngữ hỗ trợ qua dòng lệnh?

```bash
uv run cli.py --list providers    # Xem toàn bộ kênh khả dụng
uv run cli.py --list languages    # Xem toàn bộ mã ngôn ngữ hỗ trợ
uv run cli.py --list models       # Xem các mô hình khả dụng cho faster-whisper
```

### 52. Một số lỗi phổ biến khi chạy dòng lệnh CLI

*   **`--name is required`**: Chưa truyền đường dẫn file đầu vào.
*   **`File not found`**: Sai đường dẫn hoặc tệp tin không tồn tại.
*   **`--voice_role is required`**: Bắt buộc phải chỉ định giọng đọc lồng tiếng khi chạy chế độ TTS.
*   **`--target_language_code is required`**: Bắt buộc phải chọn ngôn ngữ đích khi chạy chế độ dịch STS/VTV.

---

## Phần 12: Thông tin chung khác

### 53. Phần mềm có hỗ trợ triển khai bằng Docker hay không?

Hiện tại **chưa hỗ trợ**.

### 54. Phần mềm có thể nhận dạng được chữ phụ đề vẽ trên hình ảnh video không (tính năng OCR)?

**Không thể**. Phần mềm này hoạt động dựa trên nguyên lý phân tích **luồng âm thanh** trong video để nhận dạng giọng nói của con người và chuyển đổi thành chữ viết. Nó hoàn toàn không có khả năng đọc hay quét chữ hiển thị trên hình ảnh video.
[Nếu có nhu cầu trích xuất phụ đề từ hình ảnh, vui lòng tham khảo một dự án khác của tác giả tại đây](https://pyvideotrans.com/ocrsp).

### 55. Tôi có thể thêm một ngôn ngữ mới vào phần mềm được không?

**Có thể tự thêm ngôn ngữ mới vào hệ thống, hướng dẫn chi tiết xem tại đây: [Thêm ngôn ngữ mới](https://pyvideotrans.com/newlanguage)**.

### 56. Phần mềm có mất phí sử dụng không? Có thể dùng cho thương mại không?

*   **Chi phí**: Đây là dự án **hoàn toàn miễn phí và mã nguồn mở**, bạn có thể sử dụng tất cả các tính năng miễn phí. Tuy nhiên, nếu bạn cấu hình sử dụng các cổng dịch vụ dịch thuật/TTS/ASR trả phí của bên thứ ba, bạn sẽ phải tự thanh toán chi phí cho nhà cung cấp dịch vụ đó, điều này hoàn toàn độc lập với phần mềm này.
*   **Thương mại**: Cá nhân và doanh nghiệp được phép **tự do sử dụng** phần mềm này. Tuy nhiên, nếu bạn muốn tích hợp mã nguồn của dự án này vào sản phẩm thương mại của riêng mình, bạn bắt buộc phải tuân thủ điều khoản của giấy phép **GPL-v3**. Ngoài ra, một số mô hình hoặc API dịch vụ trực tuyến của các bên thứ ba có thể có điều khoản thương mại riêng, vui lòng tham khảo quy định của nền tảng đó.

### 57. Phần mềm có đội ngũ hỗ trợ kỹ thuật trực tiếp không?

Không có. Đây là một dự án mã nguồn mở phi lợi nhuận do cá nhân phát triển tự nguyện khi rảnh rỗi, do đó không có đội ngũ hỗ trợ khách hàng riêng. Vui lòng đọc kỹ tài liệu hướng dẫn và bảng FAQ này khi gặp sự cố.
Hoặc bạn cũng có thể chọn quét mã QR WeChat ở góc dưới bên phải phần mềm để donate ủng hộ và ghi lại thông tin liên hệ để nhận được hỗ trợ kỹ thuật có tính phí.

### 58. Link tải phần mềm và các mô hình ngoại tuyến ở đâu?

*   **Trang tải gói phần mềm**: [pyvideotrans.com/downpackage](https://pyvideotrans.com/downpackage)
*   **Kho lưu trữ mã nguồn**: [github.com/jianchang512/pyvideotrans](https://github.com/jianchang512/pyvideotrans)

### 59. Báo cáo lỗi và gửi Log

*   **Vị trí file log**: Thư mục `logs` trong thư mục gốc của chương trình, các file log được đặt tên theo định dạng năm-tháng-ngày.
*   **Cách phản hồi**: Khi gặp bảng báo lỗi, bạn có thể click nút "Báo lỗi" để tự động gửi thông tin lên diễn đàn hỗ trợ; hoặc sao chép 30 dòng log cuối cùng để gửi lên các diễn đàn cộng đồng.

### 60. Tại sao phiên bản mới không còn tùy chọn "Tự động phát hiện" ở ô lựa chọn Ngôn ngữ gốc?

Trên bảng giao diện "Nhận dạng giọng nói sang phụ đề hàng loạt", bạn vẫn có thể chọn "Tự động phát hiện" (Auto). Nhưng trên giao diện "Dịch thuật video", tùy chọn này đã bị loại bỏ. Lý do là vì quy trình dịch video phức tạp bao gồm dịch thuật và lồng tiếng (liên quan đến file âm thanh mẫu tham chiếu) yêu cầu bắt buộc phải xác định rõ mã ngôn ngữ gốc để tránh phát sinh lỗi trong quá trình xử lý của các API bên thứ ba. Nếu bạn chỉ muốn chuyển âm thanh sang phụ đề, hãy sử dụng tính năng chuyên biệt "Nhận dạng giọng nói sang phụ đề hàng loạt" ở bảng bên trái.

---

## Bảng tra cứu nhanh lỗi và cách khắc phục

| Lỗi gặp phải | Nguyên nhân phổ biến | Giải pháp khắc phục nhanh |
|------|---------|---------|
| Không mở được phần mềm | Trình diệt virus chặn / Sai đường dẫn chứa file | Thêm vào danh sách loại trừ / Di chuyển thư mục phần mềm sang đường dẫn tiếng Anh không khoảng trắng |
| Thiếu file python310.dll | Bạn mới chỉ tải gói cập nhật | Tải gói đầy đủ và giải nén lại |
| Nhận dạng ra kết quả trống | Chọn sai ngôn ngữ gốc / Lẫn nhiều tạp âm | Cấu hình đúng ngôn ngữ / Bật tính năng tách giọng nói |
| Báo lỗi tràn bộ nhớ card màn hình | Chọn mô hình nhận dạng quá nặng | Chọn mô hình nhỏ hơn / Hạ tỷ lệ CUDA sang int8 / Giảm beam_size |
| Không nhận GPU tăng tốc | Chưa cài CUDA / Driver quá cũ | Cài đặt CUDA 12.8+ / Cập nhật driver card đồ họa mới nhất |
| Bản dịch xuất hiện dòng trống | AI tự động gộp các dòng phụ đề | Tắt tùy chọn "Gửi toàn bộ phụ đề" / Đổi sang mô hình AI trực tuyến mạnh hơn |
| Edge-TTS báo lỗi 403 | Bị Microsoft giới hạn tần suất yêu cầu | Giảm số luồng lồng tiếng đồng thời / Thêm thời gian tạm dừng |
| Tiếng và hình chạy lệch nhau | Khác biệt thời lượng tự nhiên giữa các ngôn ngữ | Bật tính năng tăng tốc âm thanh / Làm chậm video |
| Lỗi ffprobe/ffmpeg | Đường dẫn file quá dài hoặc tên file chứa ký tự lạ | Đổi tên file ngắn gọn không dấu / Di chuyển file ra thư mục ngoài nông hơn |
| Ổ cứng bị đầy dung lượng | Phát sinh quá nhiều file tạm khi làm chậm video | Thực hiện dọn dẹp thư mục `tmp/` của dự án |
| Chất lượng clone giọng rất kém | File âm thanh mẫu có thời lượng không phù hợp | Chọn mẫu dài từ 5-10 giây / Không bật tính năng LLM ngắt lại câu |
| Dịch vụ GPT-SoVITS báo lỗi 404 | Khởi chạy sai phiên bản API | Kiểm tra xem đang chạy api.py hay api_v2.py để chọn cấu hình tương ứng |
