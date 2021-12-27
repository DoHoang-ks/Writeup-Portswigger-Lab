# 🔒 File Upload Vulnerabilities

😘Hey! mình là Hoàng, người đang tìm kiếm đam mê đây là blog đầu tay của mình public mong mọi thứ sẽ ổn.

## Giới thiệu:

Mình được anh mình chỉ [portswigger](https://portswigger.net) để luyện tập và đây cũng là bài đầu tiên của mình ở đó. Ở đây khá hay với bài tập đầu mục đích là làm nổi bật công cụ **burp** làm được gì.
Khá trùng hợp là lỗi **log4j** cũng lổi thời gian gần đây, khi mình mới bắt đầu tìm hiểu phần `Upload File` này. Theo mình tìm hiểu thì **log4j** là một phần trong phần `Upload File`. Ta thấy được `Upload File` không có trong _OWASP2020_ nhưng nó cũng rất nguy hiểm, Nó giúp hacker điều khiển được chương trình hay đọc được các thông tin bảo mật của trang web.

```sh
❓Vậy Upload File Vulnerabilities là gì?
Upload File Vulnerabilities là khi máy chủ web cho phép người dùng tải tệp lên hệ thống tệp nhưng không thể xác định được tệp hay quyền hạn của các tệp.Điều này dẫn đến một chức năng cơ bản cũng có thể tải được các tệp tùy ý có khả năng nguy hiểm.
Điều này có thể tạo ra cơ hội cho những kẻ tấn công tiêm các tập tin độc hại vào máy của bạn. Nếu tin tặc có thể tìm ra cách thực thi các tập tệp đó, chúng làm tổn hại đến hệ thống của bạn.

```

## Yêu cầu:

Tải Burp và cài đặt trên trình duyệt web
Bạn có thể xem ở đây [Burp Suite Installation and Configuration for Windows 10](https://www.youtube.com/watch?v=fDPOMHaeICQ)

## Lab

### Lab1: Remote code execution via web shell upload

Ở bài lab này rất dễ bị tấn công nó không có thực hiện bất kỳ xác thực nào đối với các tệp người dùng tải lên trước khi lưu trữ chúng trên hệ thống tệp của máy chủ.
Yêu cầu của bài là ta cần tải 1 tệp php cơ bản và sử dụng nó để lọc nội dung của tệp _home/carlos/secret_.

Đầu tiên ta cần chuẩn bị một file exploit.php
Bên trong file chúng ta sẽ code:

<?php echo file_get_contents('/home/carlos/secret'); ?>

<img src="image\Screenshot 2021-12-24 215734.png">

Sau khi đăng nhập xong ta sẽ vào giao diện upload ảnh.

<img src="image\Screenshot 2021-12-24 214543.png">_Đây là giao diện upload ảnh_

Tiếp theo ta sẽ chọn vào `Brose...` sẽ vào được thư mục tải tệp.

<img src="image\Screenshot 2021-12-24 215928.png">

Nhấn chọn file `exploit.php` và Open là xong.

Tiếp theo:
Chúng ta chuyển tiếp sang ứng dụng `Burp`:

- Trong Burp, đi tới Proxy> Lịch sử HTTP
- Ở đây ta có thể xem các requestđi qua `Burp Proxy`
- Chúng ta sẽ chú ý đến GET /files/avatar/exploit.php

  <img src="image\Untitled.png">

- Tiếp theo ta sẽ nhấp chuột phải chọn `send to repeate` ở mục Repeater ta sẽ chọn `Send` Và xem kết quả ở phần bên phải

  <img src="image\Untitled (1).png">

- Có kết quả rồi bây giờ ta chỉ việc submit thôi.

##### Tổng kết bài lab1:

- Do đây là bài cơ bản giúp chúng ta làm quen với `Burp` nên thực hiện khá nhanh chóng và dễ dàng.
- Lỗi tải tệp lên ở đây là lỗi sơ khai nhất không hề có bảo mật gì khi tải tệp. Giúp cho hacker dễ dàng lấy đi thông tin bảo mật của chủ trang web.

### Lab2: Web shell upload via Content-Type restriction bypass

Ở bài lab này người ta sử dụng phương pháp xác thực loại tệp. Nói một cách dễ hiểu hơn ở mục tải ảnh như bài lab bên trên ta không thể tải trực tiếp file php nữa, bởi vì file tải nên phải thộc kiểu img/png.

```sh
Trước khi vào bài ta cần làm quen với content-type(kiểu dứ liệu). Khi gửi biểu mẫu HTML, trình duyệt của bạn thường gửi dữ liệu được cung cấp trong một POST yêu cầu với loại nội dung `application/x-www-form-url-encoded`. Cái này chỉ phù hợp khi nó là nhập tên, địa chỉ... Nó không phù hợp với tài liệu hay chuỗi nhị phân. Trong trường hợp này `multipart/form-data` là cách tiếp cận ưu tiên.
```

<img src="image\Untitled (2).png">

- Ở ảnh này ta thấy nội dung được chia thành 2 phần riêng biệt. `Content-Disposition` là tiêu đề cho mỗi phần còn `content-type` xác định kiểu dữ liệu cho mỗi phần. Tiêu đề này cho biết kiểu MIME của dữ liệu đã được gửi bằng cách sử dụng đầu vào này.

```sh
💡MIME là gì?
Theo wiki: Giao thức mở rộng thư điện tử Internet đa mục đích hay MIME (Multipurpose Internet Mail Extensions) là một tiêu chuẩn Internet về định dạng cho thư điện tử.
```

- Một cách mà web có thể xác thực kiểm tra tệp tải lên là xem Content-type tiêu đề có khớp với MIME dự kiến không. VD: chỉ cho phép các loại tệp image/jpeg và image/png. Biện pháp bảo vệ này có thể dễ dàng bị bỏ qua bởi các công cụ `Burp` .

Tương tự như các phần trên khi tải file `exploit.php` lên thì sẽ không được.

<img src="image\Untitled (3).png">

Đầu tiên bạn chọn `Back to My Account` để trở về
Bây giờ bạn hãy chọn `Proxy` > `Intercept` > `Intercept is off` để bật nó lên.

<img src="image\Untitled (5).png">

Rồi bạn thử tải lại file `exploit.php`.
Bạn giờ hãy vào lại `Burp`.

<img src="image\Untitled (5).png">

Bạn sẽ vào chỉnh sửa phần contnt type mà mình đã gạch chân thành `image/jpeg'`và chọn Forward ở bên góc trái.

<img src="image\Untitled (6).png">

Vậy là ta đã thành công tải tệp lên. Các bước còn lại sẽ giống với bài lab1 he.

##### Tổng kết bài lab2

- Cá nhân mình thấy bài này tuy thêm được phần xác định loại file nhưng vẫn rất dễ dàng bị dánh lừa.

## Lab3: Web shell upload via path traversal

Ở bài lab này sẽ ở lever cao hơn bài trước, ta sẽ thấy được file php của ta vẫn được tải lên nhưng lại không thực thi được code. Lúc này máy chủ chỉ thực hiện các fille có kiểu MIME mà đã được định cấu hình để thực thi. Nếu không phải kiểu MIME chỉ định nó sẽ thông báo lỗi hoặc trong một số trường hợp chúng serve nội dung của tệp dưới một dạng văn bản thuần túy thay thế.

Để làm được bài này ta cần hiểu thêm về `Directory traversal`
Sau khi đã hiểu khái quát được giờ ta sẽ bắt tay vào làm bài lab này.

Như các bài lab trên bây giờ ta thử tải 1 file `exploit.php`lúc này ta sẽ nhận được thông báo tải file thành công.

<img src="image\lab3.png">

Nhưng ở trong `Repeater` ta thấy được file `exploit.php` truyền lên ở dạng text không được thực thi.

<img src="image\lab3-1.png">

Như đã nói ở trên.

```sh
 Nếu không phải kiểu MIME chỉ định nó sẽ thông báo lỗi hoặc trong một số trường hợp chúng serve nội dung của tệp dưới một dạng văn bản thuần túy thay thế.
```

Để giải quyết vấn đề này ta thử tải file `exploit.php` lên thư mục cha của nó. Muốn làm được cách này mình nghĩ bạn phải hiểu [Directory traversal](https://portswigger.net/web-security/file-path-traversal).

Bây giờ ta thử tải file `exploit.php` lên thư mục cha của nó bằng cách gửi file vào `..\exploit.php`
Ta sẽ vào `purb` và làm điều đó.

<img src="image\lab3-2.png">

Nhưng điều này có vẻ vẫn chưa được ở mục history của Proxy

<img src="image\lab3-3.png">

Ở dạng bài này máy chủ web đã loại bỏ bất cứ trình tự nào trước khi chuyển đầu vào của ứng dụng.
Bây giờ ta có thể thử bằng cách [URL endcoding](https://www.urlencoder.org/). Ta sẽ `endcode` ký tự ..\ thay bằng ..%2f

<img src="image\lab3-4.png">

Sau khi sử xong ta forward và về history chọn GET /files/avatar/..%2fexploit.php click chuột phải chọn [Directory traversal](https://portswigger.net/web-security/file-path-traversal).

<img src="image\lab3-5.png">

Ở kết quả `Repeater` ta sen và được kết quả.

<img src="image\lab3-6.png">

##### Tổng kết bài lab3

Ta thấy được ở bài này có hai vấn đề cần giải quyết:

- Làm sao để tải file `exploit.php` lên thư mục cha để tránh bị báo lỗi hoặc serve dưới dạng văn bản thuần túy.
- làm sao để `endcode` tránh cho máy chủ web đã loại bỏ bất cứ trình tự nào trước khi chuyển đầu vào của ứng dụng
