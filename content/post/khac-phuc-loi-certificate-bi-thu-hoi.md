---
title: "Khắc phục lỗi certificate bị thu hồi (Caddy server)"
author: "hoad"
date: 2021-06-13T15:44:17+07:00
---

Trong số những dự án mà mình thực hiện, có một vài dự án sử dụng Caddy làm load balencer. Ưu điểm của Caddy là cấu hình dễ nhớ, xử lý khá tối ưu và yêu cầu cấu hình hạ tầng không quá cao. Nhờ thế mà tiết kiệm được chi phí vận hành hệ thống.

Tuy nhiên! Vào một ngày đẹp trời, mình truy cập vào hệ thống thì nhận được thông báo certificate bị thu hồi.

![image-20210613093327282](/_img/image-20210613093327282.png)

Kiểm tra ngày hết hạn thì thấy rằng vẫn còn hơn hai tháng nữa mới phải làm mới (renewal). Kiểm tra trạng thái trên trang cung cấp thông tin chứng chỉ thì trạng thái hiển thị "Revoked" (Chính xác là đã bị thu hồi).

![image-20210613093631490](/_img/image-20210613093631490.png)

Tìm các nguyên nhân gây ra vấn đề trên thì chỉ nhận được một số thông tin rất chung chung. Trích nguyên văn một đoạn trong bài viết [What is a Certificate Revocation List (CRL) vs OCSP?](https://blog.keyfactor.com/certificate-revocation-list-crl-ocsp):

* The CA discovers it has improperly and wrongfully issued a certificate
* A certificate is believed or is discovered to be fraudulent
* A certificate's private key has been compromised
* The issuing CA has been compromised
* The web site owner ceases doing business and no longer owns the domain name or the server defined in the certificate
* During the web site authentication and validation the requester misrepresents some information used in the process, or the web site owner has violated the terms of its agreement with the CA

Ok. Biết tạm vậy đã. Giờ khắc phục thế nào?

Về lý thuyết, Caddy sẽ tự động làm mới chứng chỉ khi hết hạn. Trong trường hợp này, chứng chỉ vẫn còn hiệu lực nên có vẻ như nó không xem xét tiếp tình huống bị thu hồi khi chưa hết hạn. Vì vậy, giải pháp nhanh và dễ thực hiện nhất trong tình huống này là đăng ký một chứng chỉ mới và cấu hình lại thông tin chứng chỉ mới cho Caddy (thay vì để Caddy tự quản lý).

## Các bước thực hiện

### Bước 1 - Cài đặt Certbot

Tham khảo các bước cài đặt Certbot tại đây: https://certbot.eff.org/instructions

### Bước 2 - Đăng ký chứng chỉ

Certbot chưa hỗ trợ Caddy trong việc cập nhật cấu hình tự động cho web server, nên việc này cần làm thủ công.

Bước 2.1 - Cấu hình Caddy chạy theo chế độ phục vụ file với directive `file_server` và chọn một thư mục webroot nào đó.

Bước 2.2 - Thực thi lệnh dưới đây để lấy private key và certificate.

```
sudo certbot certonly --webroot
```

Sau khi thực hiện bước này thành công, chúng ta sẽ có hai tập tin `.pem` lưu mã hash của chứng chỉ và khóa riêng tư (private key). Hai thông tin này được lưu tại thư mục `/etc/letsencrypt/live/<tên-domain>`

### Bước 3 - Cập nhật cấu hình

Cập nhật cấu hình với module `tls` và trỏ đường dẫn đến hai tập tin nhận được ở trên. (Có thể sử dụng giải pháp cấu hình Caddyfile hoặc admin API).

Khởi động lại service và tiếp tục thưởng thức những ngày tháng vui vẻ với Caddy.

## Một vài lưu ý

### Lỗi "Permission denied to /etc/letsencrypt/live"

Nếu quá trình khởi động lại service gặp lỗi `Permission denied to /etc/letsencrypt/live` thì có thể giải quyết bằng việc thực hiện thao tác mở rộng quyền truy cập vào các file `.pem` được cấp.

```
sudo chmod 755 /etc/letsencrypt/live/
```

Nếu thực thi lệnh trên mà lỗi vẫn còn tiếp diễn thì hãy xem tập tin trên có phải là một `soft linked file` không (Bằng cách sử dụng lệnh `ls -la`). Nếu đúng là `soft linked file` thì mở rộng quyền truy cập các tập tin gốc sẽ giải quyết được vấn đề.

### Thông báo chứng chỉ bị thu hồi sẽ không hiển thị trên tất cả trình duyệt (phía client)

Nếu chứng chỉ trên server bị thu hồi, có thể xảy ra hiện tượng sau với những người dùng truy cập vào hệ thống: "Một số thì truy cập được bình thường (không có dấu hiệu lỗi hay bị ảnh hưởng) và một số thì nhận thông báo chứng chỉ không an toàn". Hiện tượng này sẽ diễn ra trong khoảng 24 đến 48 giờ (chưa có số liệu chính xác).

Nếu người dùng (hoặc khách hàng) phản ánh về lỗi trên, chúng ta có thể kiểm tra lại qua các dịch vụ kiểm tra SSL/TLS. Một số dịch vụ mà mình sử dụng cung cấp các thông tin chi tiết và chính xác là:

* https://www.ssllabs.com/
* https://ssltools.digicert.com/
* https://crt.sh/

![image-20210613151356387](/_img/image-20210613151356387.png)

