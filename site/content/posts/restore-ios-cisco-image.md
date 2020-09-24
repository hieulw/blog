---
title: Phục hồi Cisco IOS Image
date: 2020-05-13T22:26:17+07:00
draft: false
categories:
  - NET
tags:
  - cisco
  - ios
---
Truy cập vào [tfr.org](http://tfr.org/cisco-ios/) để tìm image. Tải phiên bản nào cũng được nhưng phải có dạng *.bin

Sau đó sử dụng minicom hoặc phần mềm tương tự có hỗ trợ transfer file thông qua xmodem trên linux.

Vào Serial port setup cấu hình **Serial Device: /dev/ttyUSB0** (ls -A /dev | grep ttyUSB) và **Bps/Par/Bits: 9600 8N1**

Khi đang truy cập vào thiết bị thông qua serial console, nhấn tổ hợp phím Ctrl + A , Z để mở menu và chọn chuyển file như ftp bình thường.