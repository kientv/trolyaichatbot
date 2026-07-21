# API Tokens

Quản lý token xác thực để tích hợp API.

#### Truy cập

Từ menu trái, chọn **API Tokens** trong mục **API**.

#### Tạo token mới

1. Nhấn **Tạo token mới**
2. Nhập mô tả cho token
3. Chọn quyền hạn (Read, Write, Admin)
4. Nhấn **Tạo**
5. **Lưu token** - chỉ hiển thị một lần!

#### Sử dụng token

Thêm token vào header của mọi request API:

```
Authorization: Bearer YOUR_API_KEY
```

#### Quản lý token

- **Danh sách**: Xem tất cả token đã tạo
- **Vô hiệu hóa**: Tạm dừng token khi không sử dụng
- **Xóa**: Xóa vĩnh viễn token không cần thiết

> Không chia sẻ token với người khác. Nếu token bị rò rỉ, hãy vô hiệu hóa ngay lập tức.
