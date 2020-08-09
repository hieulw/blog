---
title: buffer overflow - CMU-CS 428 SAIS
date: 2020-08-08T15:45:00.163Z
draft: false
categories:
  - CTF
tags:
  - pwn
  - gdb
---
Mở file `run1` bằng `gdb`. Disassemble và Decompiler hàm main:

![](/img/2020-08-09_08-03.png)

![](/img/2020-08-09_08-07.png)

Ta thấy chương trình dùng fgets để lấy chuỗi `char *s` nhập từ bàn phím không quá `0x15`(21) kí tự và thực hiện so sánh biến `int32_t iStack20` với `0x41424344` nếu đúng sẽ in ra `diem.txt`. Ta đặt breakpoint ở hàm compare để xem hàm sẽ dùng gì để so sánh.

Chạy chương trình và nhập vào quá 21 kí tự.

![](/img/2020-08-09_08-20.png)

![](/img/2020-08-09_08-22.png)

Ta thấy trong stack chỉ lưu đến ...EEEE vậy là hàm chỉ lấy 20 kí tự đầu tiên.

![](/img/2020-08-09_08-32.png)

Ta để ý hàm so sánh cmp lấy `[ebp - 0xc]` để so sánh với `0x41424344`. Ta thử in `$ebp-0xc` để xem trong đó có gì.

![](/img/2020-08-09_08-34.png)

`0x45454545` chính là 4 kí tự EEEE cuối cùng. Vậy bây giờ chạy lại chương trình thay 4 kí tự cuối cùng bằng `0x41424344` thôi. Vì stack theo cơ chế Last in, First out nên ta phải đảo ngược `0x41424344` thành `0x44434241` để khi lưu vào và lấy ra sẽ đúng theo thứ tự cần so sánh. 4 kí tự cuối sẽ là `DCBA`.

![](/img/2020-08-09_08-39.png)

![](/img/2020-08-09_08-40.png)

![](/img/2020-08-09_08-42.png)

Ta thấy chương trình đã tạo process mới `/usr/bin/bash` vậy là hàm `system()` đã chạy.

![](/img/2020-08-09_08-50.png)

Bài 2 tương tự bài 1 nhưng lần này so sánh với `0x1020304`

![](/img/2020-08-09_08-46.png)

`0x01020304` là những kí tự không in ra được màn hình vì vậy không thể copy như câu 1. Lúc này ta dùng lệnh `echo` hoặc `printf` để nhập vào chương trình thay vì nhập tay.

![](/img/2020-08-09_08-50_1.png)

Lệnh `printf` trên sẽ in ra `0x04030201` và các kí tự còn lại sẽ lấp bởi khoảng cách theo format.