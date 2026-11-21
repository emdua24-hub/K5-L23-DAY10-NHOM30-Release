# Bảo mật khi làm việc

> Trả lời gắn với lô bạn vừa bàn giao, không chép slide. Mỗi mục vài câu.

## 1. Ca USB: hết giờ, muốn chép lô về nhà làm tiếp
Tuyệt đối không sao chép dữ liệu của dự án ra thiết bị lưu trữ cá nhân (như USB, ổ cứng ngoài) hay gửi file qua email cá nhân, vì đây là hành vi vi phạm nghiêm trọng chính sách bảo mật thông tin (ISMS/ISO 27001) và thỏa thuận bảo mật dữ liệu (NDA) với khách hàng, tiềm ẩn rủi ro lộ lọt dữ liệu dự án hoặc lây nhiễm mã độc. Việc nhờ người ngoài làm hộ cũng vi phạm nguyên tắc truy vết và phân định trách nhiệm cá nhân của annotator/reviewer trong hệ thống kiểm định. Nếu chưa hoàn thành công việc đúng hạn, kỹ thuật viên cần báo cáo Quản lý dự án để xin gia hạn hoặc xin cấp quyền truy cập từ xa an toàn qua mạng riêng ảo VPN nội bộ hoặc môi trường máy ảo VDI do công ty kiểm soát.

## 2. Năm việc: được hay không
1. Mở job được giao trên máy công ty: ĐƯỢC PHÉP, vì thiết bị của công ty đã được cài đặt các phần mềm bảo mật, kiểm soát truy cập và đáp ứng đầy đủ tiêu chuẩn an toàn thông tin của dự án.
2. Chụp màn hình gửi người quen: KHÔNG ĐƯỢC PHÉP, vì hành vi phát tán hình ảnh dự án ra bên ngoài vi phạm nghiêm trọng điều khoản bảo mật dữ liệu và quyền riêng tư của khách hàng.
3. Chép lô ra USB: KHÔNG ĐƯỢC PHÉP, do thiết bị lưu trữ ngoài không có mã hóa và kiểm soát truy cập, tiềm ẩn nguy cơ cao làm thất thoát dữ liệu và lây lan mã độc vào hệ thống.
4. Dán nhãn thật lên chatbot ngoài: KHÔNG ĐƯỢC PHÉP, vì việc đưa dữ liệu thật của dự án lên các dịch vụ AI công cộng của bên thứ ba sẽ làm rò rỉ dữ liệu và vi phạm sở hữu trí tuệ.
5. Đưa tài khoản CVAT cho đồng nghiệp: KHÔNG ĐƯỢC PHÉP, vì mỗi tài khoản được gắn định danh cá nhân để ghi nhận trách nhiệm pháp lý và audit log; việc chia sẻ tài khoản sẽ làm sai lệch dữ liệu truy vết người gán và người kiểm.

## 3. Ca khuôn mặt: thấy mặt người rõ trong lô, guideline yêu cầu làm mờ
Khi phát hiện hình ảnh có khuôn mặt người nhận diện rõ trong lô dữ liệu (ví dụ ảnh 000000000139 có chứa người thật và khuôn mặt rõ nét), kỹ thuật viên QC phải lập tức chuyển trạng thái lô dữ liệu sang Hold và gửi báo cáo khẩn cấp cho Quản lý dự án (PM) và Cán bộ phụ trách bảo vệ dữ liệu (DPO). Kỹ thuật viên tuyệt đối không được tự ý xóa ảnh hay dùng phần mềm cá nhân để tự làm mờ, vì hành động can thiệp thủ công này sẽ làm hỏng tính toàn vẹn của dataset (làm sai lệch manifest.json, sai mã băm sha256 của ảnh, phá vỡ cấu trúc scene_id và phân bổ job). Việc ẩn danh hóa khuôn mặt bắt buộc phải được xử lý lại từ đầu theo quy trình chuẩn của pipeline tiền xử lý dữ liệu đã được phê duyệt.

