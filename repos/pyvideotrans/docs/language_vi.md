# Thêm gói ngôn ngữ mới (Language Pack)

1. Đầu tiên, hãy thực hiện đoạn mã dưới đây trong cửa sổ dòng lệnh (console) để kiểm tra mã ngôn ngữ hiện tại của hệ thống của bạn:

```python
import locale
locale.getdefaultlocale()[0]
```

Chuyển 2 ký tự đầu tiên của kết quả trả về thành dạng chữ thường, sau đó nối thêm đuôi `.json` để làm tên file JSON cần tạo. Ví dụ, nếu kết quả trả về là `en_US`, bạn hãy tạo file `en.json` trong thư mục `videotrans/language`. File `en.json` này chính là file ngôn ngữ.


> **Nguyên lý hoạt động:**
> Khi khởi chạy phần mềm, hệ thống sẽ lấy 2 ký tự đầu tiên dạng chữ thường của kết quả từ lệnh `locale.getdefaultlocale()[0]`, nối thêm đuôi `.json` để tạo thành tên file ngôn ngữ. Tiếp theo, hệ thống sẽ tìm kiếm file này trong thư mục `videotrans/language`. Nếu tìm thấy, ngôn ngữ đó sẽ được sử dụng; ngược lại, giao diện tiếng Anh mặc định sẽ được hiển thị.
> Nếu trong file cấu hình `videotrans/set.ini` có thiết lập giá trị cho khóa `lang=`, hệ thống sẽ sử dụng giá trị đó làm mã ngôn ngữ mặc định thay vì lấy kết quả từ `locale.getdefaultlocale()`.

Hiện tại dự án đã có sẵn 2 file ngôn ngữ là `en.json` và `zh.json`. Bạn có thể sao chép trực tiếp một trong hai file này, đổi tên và chỉnh sửa nội dung bên trong để tạo ra gói ngôn ngữ mới.

Mỗi file ngôn ngữ là một đối tượng JSON. Lớp ngoài cùng chứa 4 trường thông tin chính như sau:

```json
{
  "translate_language": {},
  "ui_lang": {},
  "toolbox_lang": {}, 
  "language_code_list": {}
}
```

Trong đó:
- `translate_language`: Chứa các văn bản hiển thị tiến trình, thông báo lỗi và các trạng thái tương tác khác.
- `ui_lang`: Tên hiển thị của các thành phần giao diện trên phần mềm chính.
- `toolbox_lang`: Tên hiển thị của các thành phần trên giao diện Hộp công cụ video (Video Toolbox).
- `language_code_list`: Tên hiển thị của các ngôn ngữ được hỗ trợ.

## Chỉnh sửa `translate_language`

```json
"translate_language": {
    "qianyiwenjian": "The video path or name contains non ASCII spaces. To avoid errors, it has been migrated to ",
    "mansuchucuo": "Video automatic slow error, please try to cancel the 'Video auto down' option"
}
```

Như trên, `translate_language` là một đối tượng JSON chứa các cặp `khóa: giá trị`. **Hãy giữ nguyên các khóa (field name)** và chỉ cần thay đổi phần giá trị (field value) sang ngôn ngữ tương ứng của bạn.

## Chỉnh sửa `ui_lang`

```json
"ui_lang": {
    "SP-video Translate Dubbing": "SP-video Translate Dubbing",
    "Multiple MP4 videos can be selected and automatically queued for processing": "Multiple MP4 videos can be selected and automatically queued for processing",
    "Select video..": "Select video.."
}
```

Tương tự như việc chỉnh sửa `translate_language`, **giữ nguyên phần khóa** và thay đổi phần giá trị sang ngôn ngữ tương ứng.

## Chỉnh sửa `toolbox_lang`

```json
"toolbox_lang": {
    "No voice video": "Silent video",
    "Open dir": "Open directory",
    "Audio Wav": "Audio file"
}
```

Tương tự, **giữ nguyên phần khóa** và thay đổi phần giá trị tương ứng.

## Chỉnh sửa `language_code_list`

```json
"language_code_list": {
    "zh-cn": "Simplified Chinese",
    "zh-tw": "Traditional Chinese",
    "en": "English",
    "fr": "French",
    "de": "German",
    "ja": "Japanese",
    "ko": "Korean",
    "ru": "Russian",
    "es": "Spanish",
    "th": "Thai",
    "it": "Italian",
    "pt": "Portuguese",
    "vi": "Vietnamese",
    "ar": "Arabic",
    "tr": "Turkish",
    "hi": "Hindi"
}
```

Cũng giống như các phần khác, **không thay đổi phần khóa**, hãy đổi giá trị sang tên hiển thị mong muốn của ngôn ngữ đó.

**Lưu ý sau khi hoàn thành:** 
Đảm bảo file được định dạng đúng chuẩn JSON, sau đó đặt file vào thư mục `videotrans/language`. Khi khởi động lại phần mềm, ngôn ngữ mới sẽ tự động được áp dụng. Nếu gói ngôn ngữ bạn tạo khác với ngôn ngữ mặc định của hệ thống, bạn có thể thiết lập dòng `lang=mã_ngôn_ngữ` trong file `set.ini` để bắt buộc ứng dụng sử dụng gói đó (ví dụ: `lang=zh` sẽ bắt buộc hiển thị nội dung tiếng Trung từ file `zh.json`).
