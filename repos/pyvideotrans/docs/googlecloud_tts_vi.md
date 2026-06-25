# Tích hợp Google Cloud Text-to-Speech

## Tổng quan
Tích hợp này bổ sung Google Cloud Text-to-Speech làm nhà cung cấp TTS mới trong VideoTrans. Nó cung cấp khả năng tổng hợp giọng nói chất lượng cao hỗ trợ nhiều ngôn ngữ và giọng đọc khác nhau.

## Tính năng
- Hỗ trợ hơn 16 ngôn ngữ bao gồm:
  - Tiếng Bồ Đào Nha (Brazil)
  - Tiếng Anh (Mỹ/Anh)
  - Tiếng Tây Ban Nha
  - Tiếng Pháp
  - Tiếng Đức
  - Tiếng Ý
  - Tiếng Nhật
  - Tiếng Hàn
  - Tiếng Trung
  - Tiếng Nga
  - Tiếng Hindi
  - Tiếng Ả Rập
  - Tiếng Thổ Nhĩ Kỳ
  - Tiếng Thái
  - Tiếng Việt
  - Tiếng Indonesia
- Nhiều tùy chọn giọng đọc cho mỗi ngôn ngữ
- Có thể điều chỉnh tốc độ nói và cao độ
- Hỗ trợ nhiều định dạng âm thanh (MP3, LINEAR16, OGG_OPUS)
- Giao diện cấu hình thân thiện với người dùng

## Yêu cầu
1. Gói Python:
   ```bash
   pip install google-cloud-texttospeech>=2.14.0
   ```

2. Dự án Google Cloud:
   - Tạo một dự án trong [Google Cloud Console](https://console.cloud.google.com)
   - Kích hoạt Cloud Text-to-Speech API
   - Tạo tài khoản dịch vụ (service account) và tải xuống file JSON chứa khóa thông tin xác thực

## Cấu hình
1. Trong VideoTrans, đi tới Cài đặt > Google Cloud TTS
2. Cấu hình các thiết lập sau:
   - **Credential JSON**: Đường dẫn đến file thông tin xác thực JSON của tài khoản dịch vụ Google Cloud của bạn
   - **Language**: Chọn ngôn ngữ đích (ví dụ: "vi-VN" cho tiếng Việt hoặc "pt-BR" cho tiếng Bồ Đào Nha Brazil)
   - **Voice**: Chọn từ các giọng đọc có sẵn cho ngôn ngữ đã chọn
   - **Audio Encoding**: Chọn định dạng đầu ra (MP3, LINEAR16, hoặc OGG_OPUS)

## Hướng dẫn sử dụng
1. Chọn "Google Cloud TTS" làm nhà cung cấp dịch vụ TTS của bạn
2. Chọn ngôn ngữ đích của bạn
3. Chọn một giọng đọc từ các tùy chọn có sẵn
4. Điều chỉnh tốc độ nói và cao độ nếu cần thiết
5. Tiến hành dịch video như bình thường

## Khắc phục sự cố
Các sự cố thường gặp và giải pháp:

1. **"Credentials not found" (Không tìm thấy thông tin xác thực)**
   - Xác minh lại đường dẫn đến file thông tin xác thực JSON của bạn
   - Đảm bảo file có quyền đọc phù hợp

2. **"No voices available" (Không có giọng đọc khả dụng)**
   - Kiểm tra xem thông tin xác thực của bạn có quyền truy cập vào Text-to-Speech API hay chưa
   - Xác minh xem ngôn ngữ được chọn có được hỗ trợ hay không
   - Kiểm tra nhật ký log để biết thông tin lỗi chi tiết

3. **"Invalid speaking rate" (Tốc độ nói không hợp lệ)**
   - Tốc độ nói phải ở dạng phần trăm (ví dụ: "+10%", "-5%")
   - Mặc định là "+0%"

4. **"Invalid pitch" (Cao độ không hợp lệ)**
   - Cao độ phải ở đơn vị Hz (ví dụ: "+2Hz", "-1Hz")
   - Mặc định là "+0Hz"

## Đóng góp
Bạn có thể tự do:
- Báo cáo lỗi
- Đề xuất cải tiến
- Thêm hỗ trợ cho nhiều ngôn ngữ hơn
- Cải thiện giao diện cấu hình

## Giấy phép
Tích hợp này tuân theo cùng một giấy phép với dự án VideoTrans chính.

## Lời cảm ơn
- Google Cloud Text-to-Speech API
- Đội ngũ phát triển VideoTrans vì dự án nền tảng
- Các nhà đóng góp đã hỗ trợ tích hợp này
