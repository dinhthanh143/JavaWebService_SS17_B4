Phần 1 - Phân tích logic chiến lược kiến trúc
Đối với một hệ thống mang đặc thù như Blockchain Explorer (vừa phải chịu tải hàng triệu request tra cứu dữ liệu thời gian thực, vừa phải phục vụ các dịch vụ phân tán như hệ thống Notification/Alerting và chạy đa nền tảng Web/Mobile), cơ chế xác thực đóng vai trò quyết định đến sự sống còn của hạ tầng.

Dưới đây là bảng đối chiếu chiến lược dựa trên các quy tắc nghiệp vụ cốt lõi:

1. Khả năng mở rộng (Scalability)
   Session-based: Server phải tạo và duy trì một bản ghi trạng thái (Stateful) trong bộ nhớ RAM hoặc Database trung tâm (Redis/Memcached) cho mỗi người dùng đăng nhập. Khi lượng người dùng đồng thời lên tới hàng triệu, chi phí đồng bộ hóa session giữa các cụm server (Session Clustering) trở thành một "cơn ác mộng" về chi phí phần cứng và làm nghẽn băng thông hệ thống.

JWT: Hoạt động theo cơ chế phi trạng thái (Stateless). Mọi thông tin định danh của người dùng được đóng gói và ký số trực tiếp vào Token rồi giao cho Client tự giữ. Server không tốn dù chỉ 1 byte RAM để lưu trạng thái, giúp hệ thống có khả năng mở rộng theo chiều ngang (Horizontal Scaling) vô hạn bằng cách thêm các node server thông thường phía sau Load Balancer.

2. Hoạt động trên môi trường phân tán (Microservices)
   Session-based: Khi một request đi qua Gateway và phân nhánh vào các service chuyên biệt (ví dụ: Alerting Service, Wallet Tracking Service), các service này buộc phải thực hiện một cuộc gọi ngược (RPC/REST) về Session Server trung tâm để xác minh xem Session ID này có hợp lệ không. Điều này vô hình trung tạo ra một điểm nghẽn duy nhất (Single Point of Failure - SPoF) và làm tăng độ trễ (Latency) của toàn hệ thống.

JWT: Mang tính chất tự xác thực (Self-contained). Mỗi Microservice chỉ cần nắm giữ chung một khóa bí mật (Secret Key) hoặc Khóa công khai (Public Key - RS256) là có thể tự giải mã và thẩm định token tại chỗ độc lập, không cần tốn chi phí giao tiếp mạng nội bộ (Network I/O).

3. Trải nghiệm đa nền tảng (Cross-device: Web & Mobile App)
   Session-based: Dựa dẫm hoàn toàn vào cơ chế tự động gửi Cookie của trình duyệt Web. Đối với Mobile App (iOS/Android), Cookie không hoạt động tự nhiên, đòi hỏi lập trình viên phải viết thêm các bộ mã nguồn tùy chỉnh phức tạp để quản lý Cookie Container, dễ phát sinh lỗi đồng bộ.

JWT: Token bản chất là một chuỗi ký tự (String) thuần túy. Mobile App lưu chuỗi này vào Secure Storage (Keychain/EncryptedSharedPreferences), Web lưu vào Local Storage hoặc HTTP-only Cookie. Việc đính kèm Token vào Header Authorization: Bearer là quy chuẩn đồng nhất trên mọi nền tảng, giúp trải nghiệm liền mạch tuyệt đối.

ĐỀ XUẤT KIẾN TRÚC: Lựa chọn JSON Web Token (JWT) làm phương pháp xác thực cốt lõi cho hệ thống Blockchain Explorer.