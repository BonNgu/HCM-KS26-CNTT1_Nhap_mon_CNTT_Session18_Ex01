**1. Bối cảnh**

Nam là thực tập sinh tại GreenLogistics. Nam nhận nhiệm vụ truy cập trang quản trị tại địa chỉ [**phong-dieu-phoi.greenlog.vn**](http://phong-dieu-phoi.greenlog.vn) để cập nhật lộ trình xe tải. Tuy nhiên, trình duyệt liên tục báo lỗi **"DNS_PROBE_FINISHED_NXDOMAIN"**. Nam khẳng định do máy chủ công ty bị sập nguồn.

![](https://rikkeiedu-storage.s3.ap-southeast-2.amazonaws.com/uploads/editor-images/1787706870739_94f0a658-aa8f-4010-bf3a-69f680a15239.webp)**2. Yêu cầu**

- **Chức năng 1 (Phân tích mạng):** Giải thích chi tiết tiến trình DNS Lookup từ khi Nam gõ tên miền đến khi trình duyệt tìm thấy địa chỉ IP máy chủ.

- **Chức năng 2 (Chẩn đoán lỗi):** Phân biệt bản chất giữa lỗi DNS và lỗi 500 Internal Server Error để xác định lỗi này nằm ở phía Client hay Server.

**3. Ràng buộc**

Nam đang gõ nhầm địa chỉ thành [**phong-dieu-phoi.grenlog.vn**](http://phong-dieu-phoi.grenlog.vn) (thiếu một chữ 'e'). Hãy chỉ ra hậu quả của việc sai lệch tên miền trong "cuốn danh bạ Internet".

**4. Yêu cầu đầu ra**
Phần 1. Phân tích
1. Input và Output
- Input: Tên miền phong-dieu-phoi.grenlog.vn.
- Output: Địa chỉ IP (IP Address) của máy chủ.
- DNS (Domain Name System): Là hệ thống phân giải tên miền thành địa chỉ IP, giống như cuốn danh bạ Internet.
2. Quy trình DNS Lookup

Khi Nam nhập tên miền vào trình duyệt, quá trình diễn ra như sau:
1. Browser Cache: Kiểm tra IP đã được lưu trong trình duyệt chưa.
2. OS Cache: Hệ điều hành kiểm tra DNS Cache.
3. DNS Resolver: Gửi truy vấn đến máy chủ DNS của nhà mạng.
4. Root DNS Server: Hướng truy vấn đến máy chủ quản lý đuôi .vn.
5. TLD DNS Server: Xác định máy chủ DNS quản lý tên miền.
6. Authoritative DNS Server: Trả về IP nếu tên miền tồn tại.
7. Browser: Nhận IP và kết nối đến Web Server.
Sơ đồ luồng dữ liệu:
Nhập tên miền (Input)
        |
        v
Browser Cache → OS Cache
        |
        v
DNS Resolver
        |
        v
Root DNS Server
        |
        v
TLD DNS Server (.vn)
        |
        v
Authoritative DNS Server
        |
        v
Trả về địa chỉ IP (Output)
        |
        v
Trình duyệt kết nối Web Server


3. Hậu quả khi nhập sai tên miền

Nam nhập grenlog.vn thay vì greenlog.vn (thiếu chữ e). DNS xem đây là hai tên miền khác nhau. Nếu grenlog.vn không tồn tại hoặc không có bản ghi DNS phù hợp, hệ thống trả về lỗi DNS_PROBE_FINISHED_NXDOMAIN.

Do đó, trình duyệt không tìm được địa chỉ IP và chưa thể kết nối đến máy chủ. Lỗi này không chứng minh máy chủ công ty bị sập nguồn.
Phần 2. Triển khai
1. Phân biệt lỗi DNS và lỗi HTTP 500
Tiêu chí	DNS NXDOMAIN	HTTP 500
Bản chất	Không tìm thấy tên miền	Máy chủ gặp lỗi nội bộ
Vị trí lỗi	Phân giải DNS	Phía Server
HTTP Request	Chưa gửi được đến máy chủ đích	Đã đến máy chủ
Nguyên nhân	Sai tên miền, không có bản ghi DNS	Lỗi code, database hoặc cấu hình

2. Quy trình kiểm tra HTTP Status Code

Nam thực hiện các bước sau:
1. Kiểm tra lại tên miền đã nhập chính xác chưa.
2. Mở CMD và chạy lệnh nslookup phong-dieu-phoi.greenlog.vn để kiểm tra khả năng phân giải DNS.
3. Nếu DNS trả về IP, chạy lệnh curl -I https://phong-dieu-phoi.greenlog.vn để kiểm tra phản hồi HTTP.
4. Đọc mã trạng thái để xác định tình trạng hệ thống.
Một số mã HTTP phổ biến:
- 200 OK: Yêu cầu thành công.
- 301/302: Chuyển hướng.
- 404 Not Found: Không tìm thấy tài nguyên.
- 500 Internal Server Error: Máy chủ gặp lỗi nội bộ.
- 503 Service Unavailable: Máy chủ tạm thời không thể phục vụ.
Phần 3. Kiểm thử
Giả sử Nam sửa đúng tên miền nhưng vẫn nhận mã 404 Not Found, điều này có nghĩa là DNS đã phân giải thành công và trình duyệt đã kết nối được đến máy chủ.

Tuy nhiên, máy chủ không tìm thấy trang hoặc tài nguyên tại đường dẫn được yêu cầu.

Nguyên nhân có thể:
- Đường dẫn trang quản trị bị nhập sai.
- Trang quản trị đã bị xóa hoặc di chuyển.
- Route của ứng dụng chưa được cấu hình.
- Tài nguyên không tồn tại trên máy chủ.
Cách khắc phục: Kiểm tra lại URL, đường dẫn trang quản trị và cấu hình routing. Nếu vẫn xảy ra lỗi, cần báo bộ phận IT kiểm tra.

Kết luận: NXDOMAIN là lỗi không tìm thấy tên miền, 404 là không tìm thấy tài nguyên trên máy chủ, còn 500 là lỗi xử lý nội bộ phía máy chủ.
