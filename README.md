# 🔒 File Upload Vulnerabilities (Lỗ hổng tải tệp)

😘Hey! mình là Hoàng, người đang tìm kiếm đam mê đây là blog đầu tay của mình public mong mọi thứ sẽ ổn.

## Giới thiệu:
Mình được anh mình chỉ [portswigger](https://portswigger.net) để luyện tập và đây cũng là bài đầu tiên của mình ở đó. Ở đây khá hay với bài tập đầu mục đích là làm nổi bật công cụ **burp** làm được gì. 
Khá trùng hợp là lỗi **log4j** cũng lổi thời gian gần đây, khi mình mới bắt đầu tìm hiểu phần `Lỗ hổng tải tệp` này. Theo mình tìm hiểu thì **log4j** là một phần trong phần `Lỗ hổng tải tệp`. Ta thấy được lỗ hổng tải tệp không có trong *OWASP2020* nhưng nó cũng rất nguy hiểm, Nó giúp hacker điều khiển được chương trình hay đọc được các thông tin bảo mật của trang web.

```sh
❓Vậy lỗ hổng tải tệp là gì?
Lỗ hổng tải tệp là khi máy chủ web cho phép người dùng tải tệp lên hệ thống tệp nhưng không thể xác định được tệp hay quyền hạn của các tệp.Điều này dẫn đến một chức năng cơ bản cũng có thể tải được các tệp tùy ý có khả năng nguy hiểm 
Điều này có thể tạo ra cơ hội cho những kẻ tấn công tiêm các tập tin độc hại vào máy của bạn. Nếu tin tặc có thể tìm ra cách thực thi các tập tệp đó, chúng làm tổn hại đến hệ thống của bạn.

```
## Yêu cầu:
Tải Burp và cài đặt trên trình duyệt web
Bạn có thể xem ở đây [Burp Suite Installation and Configuration for Windows 10](https://www.youtube.com/watch?v=fDPOMHaeICQ)

## Lab
### Lab1: Remote code execution via web shell upload

Ở bài lab này rất dễ bị tấn công nó không có thực hiện bất kỳ xác thực nào đối với các tệp người dùng tải lên trước khi lưu trữ chúng trên hệ thống tệp của máy chủ.
Yêu cầu của bài là ta cần tải 1 tệp php cơ bản và sử dụng nó để lọc nội dung của tệp *home/carlos/secret*.

Đầu tiên ta cần chuẩn bị một file exploit.php
Bên trong file chúng ta sẽ code:
<?php echo file_get_contents('/home/carlos/secret'); ?>
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d063a676-cde2-473a-b1e7-563914e885ef/Untitled.png)

Sau khi đăng nhập xong ta sẽ vào giao diện upload ảnh.
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6469997-1b19-4ee1-a6fe-7d677119ea45/Untitled.png)
*Đây là giao diện upload ảnh*

Tiếp theo ta sẽ chọn vào `Brose...` sẽ vào được thư mục tải tệp.
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1229225-b6ba-46d9-bf25-4f81e0d27186/Untitled.png)
*Đây là giao diện sau khi vào*

Nhấn chọn file `exploit.php` và Open là xong.

Tiếp theo:
Chúng ta chuyển tiếp sang ứng dụng `Burp`:
- Trong Burp, đi tới Proxy> Lịch sử HTTP 
- Ở đây ta có thể xem các requestđi qua `Burp Proxy`
- Chúng ta sẽ chú ý đến GET /files/avatar/exploit.php
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ce794db4-0c19-4a6e-84e6-4172aaa18424/Untitled.png)

- Tiếp theo ta sẽ nhấp chuột phải chọn `send to repeate` ở mục Repeater ta sẽ chọn `Send` Và xem kết quả ở phần bên phải 
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a0f50a88-cee5-4730-89d6-004a5fc65881/Untitled.png)

- Có kết quả rồi bây giờ ta chỉ việc submit thôi.

##### Tổng kết bài lab1:
- Do đây là bài cơ bản giúp chúng ta làm quen với `Burp` nên thực hiện khá nhanh chóng và dễ dàng.
- Lỗi tải tệp lên ở đây là lỗi sơ khai nhất không hề có bảo mật gì khi tải tệp. Giúp cho hacker dễ dàng lấy đi thông tin bảo mật của chủ trang web.

### Lab2: Web shell upload via Content-Type restriction bypass

 Ở bài lab này người ta sử dụng phương pháp xác thực loại tệp. Nói một cách dễ hiểu hơn ở mục tải ảnh như bài lab bên trên ta không thể tải trực tiếp file php nữa, bởi vì file tải nên phải thộc kiểu img/png.

 ```sh
 Trước khi vào bài ta cần làm quen với content-type(kiểu dứ liệu). Khi gửi biểu mẫu HTML, trình duyệt của bạn thường gửi dữ liệu được cung cấp trong một POST yêu cầu với loại nội dung `application/x-www-form-url-encoded`. Cái này chỉ phù hợp khi nó là nhập tên, địa chỉ... Nó không phù hợp với tài liệu hay chuỗi nhị phân. Trong trường hợp này `multipart/form-data` là cách tiếp cận ưu tiên.
 ```
 ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8308274f-5bbb-4ad4-9360-e95a703bdf8c/Untitled.png)
 
 - Ở ảnh này ta thấy nội dung được chia thành 2 phần riêng biệt. `Content-Disposition` là tiêu đề cho mỗi phần còn `content-type` xác định kiểu dữ liệu cho mỗi phần. Tiêu đề này cho biết kiểu MIME của dữ liệu đã được gửi bằng cách sử dụng đầu vào này.

 ```sh
 💡MIME là gì?
Theo wiki: Giao thức mở rộng thư điện tử Internet đa mục đích hay MIME (Multipurpose Internet Mail Extensions) là một tiêu chuẩn Internet về định dạng cho thư điện tử.
 ```

 - Một cách mà web có thể xác thực kiểm tra tệp tải lên là xem Content-type tiêu đề có khớp với MIME dự kiến không. VD: chỉ cho phép các loại tệp image/jpeg và image/png. Biện pháp bảo vệ này có thể dễ dàng bị bỏ qua bởi các công cụ `Burp` .

 Tương tự như các phần trên khi tải file `exploit.php` lên thì sẽ không được. 
 
 ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ca4ce69-1568-4e4a-8ba6-1cc439a76f52/Untitled.png)

 Đầu tiên bạn chọn `Back to My Account` để trở về
 Bây giờ bạn hãy chọn Proxy > Intercept > Intercept is off để bật nó lên rồi bạn thử tải lại file `exploit.php`.
 Bạn giờ hãy vào lại `Burp`.

 ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c3f0198f-027d-44e1-bc38-c870ba8f943f/Untitled.png)

 Bạn sẽ vào chỉnh sửa phần contnt type mà mình đã gạch chân thành `image/jpeg'`và chọn Forward ở bên góc trái.

 ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1ac83bbb-037a-433b-8640-6d7e54857779/Untitled.png)

 Vậy là ta đã thành công tải tệp lên. Các bước còn lại sẽ giống với bài lab1 he.

 ##### Tổng kết bài lab2

 - Cá nhân mình thấy bài này tuy thêm được phần xác định loại file nhưng vẫn rất dễ dàng bị dánh lừa.
