# Facebook Messenger

## Kết nối Facebook Messenger

1. Đăng nhập vào [Dashboard](https://app.aichatbot.com.vn/dashboard)
2. Chọn AI Chatbot cần cấu hình
3. Chuyển sang tab **Publish** > click sub-tab **Integrations**
4. Bấm **Connect with Facebook**
5. Trình duyệt sẽ mở cửa sổ Facebook OAuth
6. Chọn **Fanpage** bạn muốn tích hợp và bấm **Tiếp tục**
7. Quay lại giao diện AI Chatbot, bạn sẽ thấy Fanpage đã hiển thị trong **Connected Channels**
8. (Tuỳ chọn) Tích vào **Automatic conversation transfer** nếu muốn bot tự động dừng khi có nhân viên trả lời. (Xem [Chuyển tiếp từ AI Chatbot](#chuyen-tiep-tu-ai-chatbot) bên dưới)

> **Lưu ý:**
> - Phải là Admin của Fanpage
> - Nếu vừa đổi mật khẩu Facebook, bạn cần re-connect lại

## Chuyển tiếp từ AI Chatbot

Để AI Chatbot có thể tự động chuyển tiếp (tự động dừng khi có nhân viên trả lời, hoặc tự động quay lại sau 10 phút nhân viên rời đi), bạn cần thiết lập AI Chatbot là ứng dụng chính.

1. Vào fanpage của bạn
2. Truy cập vào fanpage đang tích hợp với AI Chatbot: [https://www.facebook.com/settings/?tab=pages](https://www.facebook.com/settings/?tab=pages)
3. Vào **Settings & privacy**, chọn **Page setup** và vào **Advanced Messaging**

Phần **Connected App**, bấm vào **Edit** của app AI Chatbot. Hãy bật **Access standby channel** và **Take control of conversations** rồi lưu lại.

4. **Messenger Messaging**: Chọn **Default routing app** là AI Chatbot

Vậy là bạn đã hoàn tất thiết lập chuyển tiếp Facebook fanpage với AI Chatbot.

- Khi AI đang chat, có nhân viên trả lời thì AI sẽ dừng 10 phút kể từ tin nhắn cuối
- Sau 10 phút nếu khách hàng nhắn tin, AI sẽ tiếp tục trả lời khách hàng

## Gỡ Facebook Messenger

Việc này cần thiết khi bạn:
- Tạm thời gỡ để nhân viên hỗ trợ khách hàng trên fanpage
- Không còn sử dụng app.aichatbot.com.vn
- Gặp lỗi khi tích hợp Fanpage

### Bước 1: Truy cập Business Integrations

1. Mở trình duyệt và truy cập: [https://www.facebook.com/settings/?tab=business_tools](https://www.facebook.com/settings/?tab=business_tools)
2. Trong menu bên trái, chọn **Business integrations** > **Active**

### Bước 2: Xác định ứng dụng cần gỡ

Bạn sẽ thấy danh sách các app đang có quyền kết nối với tài khoản Facebook của bạn, bao gồm `app.aichatbot.com.vn`.

### Bước 3: Gỡ quyền ứng dụng

Tìm dòng `app.aichatbot.com.vn` và nhấn nút **Remove**.

### Bước 4: Xác nhận gỡ bỏ

- Không cần tick ô xóa bài viết nếu bạn chỉ muốn ngắt kết nối kỹ thuật
- Nhấn **Remove** để hoàn tất

### Lưu ý sau khi gỡ

- AI Chatbot sẽ không còn quyền truy cập Fanpage & tin nhắn
- Nếu muốn kết nối lại, cần làm lại quy trình tích hợp từ đầu trong AI Chatbot
- Đảm bảo đăng nhập đúng Facebook Admin của Fanpage khi tích hợp lại
