---
title: forensic - CMU-CS 428 SAIS
date: 2020-08-01T17:08:19.699Z
categories:
  - CTF
tags:
  - forensic
  - ssl
---
Lê Quang Hiếu

2320114961

Mở file `10diem.pcapng` bằng wireshark:

![](img/forensic-cmu-0.png)

Chúng ta thử filter ra các gói tin `http` bằng cách nhập `http` vào khung filter trên cùng và được kết quả.

![](img/forensic-cmu-1.png)

Hầu như toàn là request khi khởi động trình duyệt, đây không phải là kết quả chúng ta mong muốn. Vậy có thể đoán được trang web sử dụng `https`.

Ta thử filter một vài protocol có thể hữu dụng như `ftp` , `ftp-data` 

Và kết quả là người dùng có thực hiện một session `ftp`. Bấm tổ hợp phím  `Ctrl+Alt+Shift+T` hoặc chuột phải vào packets chọn `Follow > TCP Stream` để hiện nội dung session dễ đọc hơn.

Phần màu đỏ là request của client và phần màu xanh là response từ server.

![](img/forensic-cmu-2.png)

Qua nội dung đoạn session trên thì người dùng đang upload file `sslkey.log` lên server.

Wireshark có thể capture được nội dung mà người dùng đã up, ta dùng filter `ftp-data` và tiếp tục tổ hợp phím `Ctrl+Alt+Shift+T` để lấy nội dung file `sslkey.log` 

![](img/forensic-cmu-3.png)

Nhìn vào nội dung có thể đoán đây là file chứa [pre-master secret](https://wiki.wireshark.org/TLS#Using_the_.28Pre.29-Master-Secret). Chọn *Save data as ASCII* và *Save as* để lưu vào máy.

Sau khi lưu, nhấn tổ hợp phím `Ctrl+Shift+P` trong wireshark để mở cửa sổ Preferences. Sau đó bên khung menu trái, chọn `Protocols > TLS`

![](img/forensic-cmu-4.png)

Tại mục `(Pre)-Master-Secret log filename` chọn tới file mà bạn lưu vừa nãy và bấm OK.

![](img/forensic-cmu-5.png)

Wireshark sẽ decrypt các gói tin đã mã hóa qua SSL/TLS cho bạn. Sau khi đã decrypt thành công, tiến hành filter `http2.body.reassembled.data` có nghĩa là lọc ra các packets `http2` có data được reassembled (hiểu nôm na là giải mã từ các gói tcp).

![](img/forensic-cmu-6.png)

Lần này chúng ta dùng tổ hợp phím `Ctrl+Alt+Shift+S` để mở ra session https của user.

![](img/forensic-cmu-7.png)

Chọn Show as UTF-8 để có thể đọc tiếng Việt. Sau đó tìm thử `10 điểm` là thông báo khi login thành công. Và khi nhìn khung input có `name="txtu"` đây chính là mã số sinh viên login.

Bấm `Find Next` để tìm tiếp cho đến khi đúng mã sinh viên cần tìm thì scroll lên phần chữ màu đỏ ở trên, đây chính là request trước đó của client.

![](img/forensic-cmu-8.png)

Ta thấy `txtp` chính là password cần tìm.

![](img/forensic-cmu-9.png)

Lưu ý lúc submit:

Password ở dạng hash md5 vì được mã hóa tại client trước khi submit lên server. Nên khi ta dùng password tìm được để submit thì password lại được mã hóa lần nữa.

Cách dễ nhất là tắt `Javascript` phía client rồi submit.

![](img/forensic-cmu-10.png)

Hoặc có thể dùng code để post thẳng lên server thông qua console của Chrome Dev Tools.

```jsx
let data = new URLSearchParams();
[...document.forms[0].elements].forEach((elm) => {
  data.append(elm.name, elm.value);
});

data.set("txtu", "2320114961");
data.set("txtp", "7162223401070fd4786b152e92d1b37c");

await fetch("/", {
  method: "POST",
  body: data,
});
```