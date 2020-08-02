---
date: "2020-05-13T22:26:17+07:00"
title: "Phục hồi Cisco IOS Image"
categories:
  - NET
tags:
  - cisco
  - ios
---

Nếu bạn vô tình xóa hệ điều hành của switch hoặc router. Đừng hoảng hốt, sẽ có cách để phục hồi lại image.

Truy cập vào [tfr.org](http://tfr.org/cisco-ios/) để tìm image của bạn. Bạn có thể tải phiên bản nào cũng được nhưng phải có dạng \*.bin

Sau đó sử dụng minicom hoặc phần mềm tương tự có hỗ trợ transfer file thông qua xmodem trên linux.

Vào Serial port setup cấu hình **Serial Device: /dev/ttyUSB0** (ls -A /dev | grep ttyUSB) và **Bps/Par/Bits: 9600 8N1**

Khi đang truy cập vào thiết bị thông qua serial console, nhấn tổ hợp phím Ctrl + A , Z để mở menu và chọn chuyển file như ftp bình thường.
