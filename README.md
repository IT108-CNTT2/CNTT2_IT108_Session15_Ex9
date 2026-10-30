# CampusPay – Thiết kế phân quyền Google Drive

## Phần 1. Phân tích bài toán (I/O)

### 1. Đầu vào và đầu ra

- **Đầu vào:** 4 vai trò: Kỹ thuật (3), Thiết kế (2), Kinh doanh (2), Nhà đầu tư (1); tài liệu gồm mã nguồn, thiết kế UI, Business Plan, Slide Pitch và dữ liệu khách hàng.
- **Đầu ra:** Cây thư mục 2 cấp, bảng quyền theo vai trò, chia sẻ an toàn, lịch sử phiên bản và quyền nhà đầu tư hết hạn sau 14 ngày.
- **Nguyên tắc:** Ai cần xem thì cấp **Viewer**, cần góp ý thì **Commenter**, cần sửa mới cấp **Editor**.

### 2. Cấu trúc thư mục Google Drive (2 cấp)

```text
CampusPay/                    # Cấp 1: dự án (không chia sẻ rộng thư mục gốc)
├── 01_KyThuat/               # Cấp 2: code, tài liệu API, test
├── 02_ThietKe/               # UI, hình ảnh, tài nguyên đồ họa
├── 03_KinhDoanh/             # Business Plan nội bộ, đối tác
├── 04_DungChung/             # Slide Pitch, Business Summary
├── 05_BaoMat/                # Dữ liệu khách hàng: chỉ người được chỉ định
└── 06_NhaDauTu/             # Bản Business Plan đã duyệt + Slide Pitch PDF
```

**Lưu ý:** Không lưu API keys thật trong Drive; dùng Secret Manager hoặc biến môi trường an toàn. Không cấp quyền rộng ở `CampusPay/` vì quyền thư mục cha có thể được kế thừa xuống thư mục con.

### 3. Các bước thiết lập

1. Tạo thư mục `CampusPay` và 6 thư mục con như sơ đồ.
2. Đặt **General access = Restricted** cho toàn bộ tài liệu; chỉ mời email cụ thể.
3. Cấp quyền theo bảng dưới; ưu tiên cấp ở đúng thư mục/file cần dùng, không cấp quyền toàn dự án.
4. Với file mật: hạn chế tải/sao chép khi phù hợp; tắt **Editors can change permissions and share**.
5. Lưu tài liệu trên Cloud; kiểm tra **Version History** và sao lưu định kỳ.
6. Thử đăng nhập bằng tài khoản từng vai trò để kiểm tra quyền xem/sửa/tải.
7. Khi thay đổi nhân sự: thu hồi quyền trực tiếp, quyền nhóm và quyền từ thư mục cha; kiểm tra lại.

## Phần 2. Triển khai và chống lỗi

### 4. Bảng phân quyền (4 vai trò × 5 loại tài liệu)

| Loại tài liệu | Ban KT | Ban TK | Ban KD | Nhà đầu tư | Ghi chú bảo mật |
|---|---|---|---|---|---|
| Mã nguồn / tài liệu API | Editor | Không | Không | Không | Dùng Git để quản lý code; không đưa API keys lên Drive/Git. |
| Thiết kế UI / Assets | Viewer | Editor | Commenter | Không | KD chỉ góp ý; không sửa thiết kế. |
| Business Plan | Viewer | Không | Editor | Viewer* | *Nhà đầu tư chỉ xem **bản đã duyệt**, không truy cập bản nội bộ. |
| Slide Pitch | Commenter | Editor | Editor | Viewer* | *Chỉ cấp bản **PDF**, thời hạn 14 ngày. |
| Dữ liệu khách hàng | Editor* | Không | Viewer* | Không | *Chỉ người KT/KD được chỉ định; Restricted, không cấp cho cả ban. |

**Quy ước:** “Không” = không có quyền; **Viewer** = xem; **Commenter** = xem và bình luận; **Editor** = chỉnh sửa. Quyền trong bảng là quyền tối đa cho người có nhiệm vụ liên quan, không mặc định cấp cho mọi thành viên.

### 5. Giải quyết 3 bẫy dữ liệu

**Bẫy 1 – Ban KT cần đọc Business Plan:**
- Cấp **Viewer** cho tài liệu cần đọc, **Restricted** theo email.
- Ban KT xem được bối cảnh dự án nhưng không sửa nội dung. Không cần Editor.

**Bẫy 2 – Nhà đầu tư muốn tải Slide Pitch offline:**
- Xuất Slide Pitch thành **PDF**, cấp **Viewer** và **cho phép tải xuống**.
- **Không tắt** “Viewers can download”, vì nhà đầu tư cần tải PDF.
- Không chia sẻ bản Google Slides/PowerPoint gốc. Lưu ý: PDF tải về vẫn có thể bị chỉnh sửa bằng công cụ khác; Viewer không bảo vệ được bản đã tải.

**Bẫy 3 – Một thành viên nghỉ việc:**
- Vào **Share → Manage access → Remove access** ở các thư mục/file liên quan.
- Xóa khỏi **Google Groups/Shared drives** nếu có; kiểm tra cả quyền được kế thừa và link chia sẻ.
- Nếu dùng Google Workspace, quản trị viên khóa tài khoản và thu hồi phiên đăng nhập khi cần. Quyền đã thu hồi không xóa được bản sao từng tải về.

### 6. Version History và quyền hết hạn

- Dùng Google Docs/Sheets/Slides để xem **File → Version history → See version history**; đặt tên mốc quan trọng.
- Với file tải lên như PDF/ZIP, các phiên bản cũ có thể bị xóa sau **30 ngày hoặc khi có 100 phiên bản mới hơn**. Chọn **Keep forever** cho mốc quan trọng và duy trì bản sao lưu để đáp ứng yêu cầu lưu tối thiểu 30 ngày.
- Chia sẻ thư mục `06_NhaDauTu` bằng **Viewer**, **Restricted** theo email, chọn **Add expiration → 14 ngày**. Tính năng tự hết hạn cần tài khoản Google Workspace cơ quan/trường học đủ điều kiện. Nếu không có, cần giải pháp quản trị khác để tự động thu hồi, không được coi nhắc lịch thủ công là tự động.
- Sau 14 ngày, nhà đầu tư không mở được file trên Drive, nhưng **bản PDF đã tải xuống vẫn tồn tại**.

## Kết luận

**Least Privilege = đúng người, đúng tài liệu, đúng quyền, đúng thời hạn.** CampusPay nên dùng Restricted, tách tài liệu mật, giữ lịch sử phiên bản và thu hồi quyền ngay khi hết việc.

### Tham khảo

- [Google Drive – Chia sẻ và quản lý quyền](https://support.google.com/drive/answer/7166529)
- [Google Drive – Lịch sử phiên bản](https://support.google.com/drive/answer/2409045)
- [Google Drive – Phân quyền và quyền kế thừa](https://support.google.com/drive/answer/2494822)
