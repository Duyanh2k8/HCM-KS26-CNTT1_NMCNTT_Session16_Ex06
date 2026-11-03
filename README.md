# Bài 6 : Kiểm tra và tối ưu hiệu năng Portal
## Phần 1: Phân tích
### 1. Quy trình kiểm tra bằng Network
1. Mở website: [http://portal.greenlog.vn](https://portal.greenlog.vn/web/database/selector).
2. Nhấn F12 để mở Chrome DevTools.
3. Chọn tab Network.
4. Tích chọn Disable cache để vô hiệu hóa bộ nhớ đệm trong lúc DevTools đang mở.
5. Nhấn Ctrl + R để tải lại trang.
6. Quan sát danh sách Request và biểu đồ Waterfall.
7. Chọn từng Request để xem các thông tin:
- Status Code: mã trạng thái HTTP.
- Type: loại tài nguyên.
- Size: dung lượng tài nguyên.
- Time: thời gian tải.
- Waterfall: tiến trình tải tài nguyên theo thời gian.
### 2. Xác định nút thắt cổ chai (Bottleneck)
Trong tình huống đề bài, trang chủ mất khoảng 20 giây để hiển thị đầy đủ. Những nguyên nhân cần kiểm tra gồm:
|Vấn đề|Biểu hiện trên Network|Ảnh hưởng|
|-|-|-|
|File ảnh bản đồ quá lớn|Size lớn, Waterfall kéo dài|Làm chậm hiển thị bản đồ|
|Request lỗi 404|Status màu đỏ, mã 404|Tài nguyên không được tải|
|CSS/JS tải chậm|Thời gian tải cao|Có thể trì hoãn hiển thị giao diện|
|Tài nguyên chưa được Cache|Tải lại vẫn truyền nhiều dữ liệu|Tốn băng thông và thời gian|

**Kết luận phân tích:**

Nút thắt cổ chai cần ưu tiên kiểm tra là các file hình ảnh bản đồ có dung lượng lớn và những Request bị lỗi 404. Hình ảnh lớn có thể chiếm nhiều thời gian truyền tải, trong khi các Request 404 khiến tài nguyên không thể được tải thành công.
## Phần 2: Triển khai
Đề xuất quy trình tối ưu gồm 3 bước:
### Bước 1. Sửa lỗi đứt link (404)
- Mở tab Network trong Chrome DevTools.
- Tìm các Request có Status Code 404.
- Kiểm tra URL của những tài nguyên bị lỗi.
- Xác định đường dẫn sai hoặc file bị thiếu.
- Sửa lại đường dẫn trong HTML, CSS hoặc JavaScript.
- Tải lại trang và kiểm tra kết quả.

**Kết quả mong đợi:** Các tài nguyên cần thiết được tải thành công với mã HTTP 200 OK, không còn lỗi 404.
### Bước 2. Nén và tối ưu hình ảnh
- Sử dụng bộ lọc Img trong tab Network để tìm các file hình ảnh.
- Kiểm tra dung lượng và thời gian tải của từng file.
- Chuyển đổi hình ảnh sang định dạng WebP hoặc AVIF khi phù hợp.
- Resize hình ảnh theo kích thước hiển thị thực tế.
- Nén ảnh để giảm dung lượng nhưng vẫn đảm bảo chất lượng.
- Sử dụng Lazy Loading cho hình ảnh chưa cần hiển thị ngay.

**Kết quả mong đợi:** Giảm dung lượng hình ảnh, rút ngắn thời gian tải và giúp bản đồ lộ trình hiển thị nhanh hơn.
### Bước 3. Bật cơ chế Browser Cache
Cấu hình máy chủ cho phép trình duyệt lưu lại các tài nguyên tĩnh như CSS, JavaScript, hình ảnh và font chữ.

Ví dụ cấu hình HTTP Header:
```HTTP
Cache-Control: public, max-age=31536000, immutable
```
Cấu hình trên phù hợp với các tài nguyên có tên file chứa mã phiên bản hoặc mã hash. Khi nội dung thay đổi, cần đổi tên file để trình duyệt tải phiên bản mới.

**Kết quả mong đợi:**
- Giảm số lần tải lại tài nguyên.
- Tiết kiệm băng thông.
- Giảm tải cho máy chủ.
- Tăng tốc độ truy cập website ở những lần tiếp theo.
## Phần 3: Kiểm thử
### 1. Kiểm tra mã 304 Not Modified

Sau khi cấu hình Browser Cache, thực hiện:
1. Mở Chrome DevTools và chọn tab Network.
2. Bỏ chọn Disable cache.
3. Nhấn Ctrl + R để tải lại trang.
4. Quan sát cột Status của các Request.
5. Kiểm tra những tài nguyên trả về mã 304 Not Modified.
### 2. Giải thích mã 304 Not Modified
HTTP 304 Not Modified có nghĩa là tài nguyên trên máy chủ không thay đổi so với phiên bản trình duyệt đang lưu trong bộ nhớ đệm.

Thay vì tải lại toàn bộ nội dung, trình duyệt có thể sử dụng phiên bản đã lưu trước đó.

**Đây là tín hiệu tốt vì:**- 
- Giảm lượng dữ liệu truyền qua mạng.
- Tiết kiệm băng thông hệ thống.
- Giảm số lượng dữ liệu máy chủ phải gửi.
- Giúp website tải nhanh hơn khi người dùng truy cập lại.
- Cải thiện trải nghiệm sử dụng của shipper.

**Lưu ý:** Mã 304 vẫn có trao đổi Request và Response qua mạng, nhưng không cần truyền lại toàn bộ nội dung tài nguyên.
### 3. Bảng đánh giá kết quả
|Tiêu chí|Kết quả mong đợi|
|-|-|
|CSS quan trọng|HTTP 200 OK khi tải mới|
|JavaScript quan trọng|HTTP 200 OK khi tải mới|
|Request hình ảnh|Không còn lỗi 404|
|Hình ảnh bản đồ|Đã được tối ưu dung lượng|
|Browser Cache|Hoạt động bình thường|
|Reload trang|Có thể xuất hiện 304 Not Modified|
|Tốc độ tải trang|Nhanh hơn so với ban đầu|
