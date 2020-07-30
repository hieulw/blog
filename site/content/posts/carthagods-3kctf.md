---
title: carthagods - 3kCTF
date: 2020-07-30T03:00:57.277Z
categories:
  - CTF
tags:
  - opcache
  - php
  - web
---
Nguồn: <https://www.sousse.love/post/carthagods-3kctf2020/index.html>

Source `index.php` được cho sẵn, mình sẽ lọc bớt đoạn không cần thiết

```phtml
<a href="./baal">Baal</a>
<a href="./tanit">Tanit</a>
<a href="./dido">Dido</a>
<a href="./caelestis">caelestis</a>
<a href="./info.php">phpinfo</a>
<a href="./flag.php">FLAG</a>
<textarea
  class="label-input100"
  style="color: black; width: 100%; height: 300px;"
>
<?php
if (@$_GET[*REDACTED*]) {
  $file = $_GET[*REDACTED*];
  $f = file_get_contents('thecarthagods/'.$file);
  if (!preg_match("/<\?php/i", $f)) {
    echo $f;
  } else {
    echo 'php content detected';
  }
}
?>
</textarea>
```

Và kèm theo là source `.htaccess` 

```apache
RewriteEngine on
options -indexes

RewriteRule ^([a-zA-Z0-9_-]+)$ index.php?*REDACTED*=$1 [QSA]
```

Như ta thấy trang web này dùng `mod_rewrite`. Và ta có thể hiểu dựa theo rewrite rule: (chỉ rewrite khi chứa kí tự chữ hoa, thường và số)

* *carthagods.3k.ctf.to:8039*/**baal** => *carthagods.3k.ctf.to:8039*/**index.php?REDACTED=baal**
  Và `index.php` sẽ lấy nội dung file **baal** đẩy ra `textarea` nếu có trong thư mục `thecarthagods/`

Bây giờ việc cần làm đầu tiên là leak ra `REDACTED`. Chúng ta không thể bruteforce để lấy `REDACTED` vì việc đó rất mất thời gian và phi tinh thần CTF. Chúng ta có thể thử bằng việc truyền vào một thư mục và không chứa `/` để rewrite rule được áp dụng.
Truy cập: <http://carthagods.3k.ctf.to:8039/thecarthagods>
Và trang tự chuyển hướng thành: [http://carthagods.3k.ctf.to:8039/thecarthagods/**?eba1b61134bf5818771b8c3203a16dc9=thecarthagods**](http://carthagods.3k.ctf.to:8039/thecarthagods/?eba1b61134bf5818771b8c3203a16dc9=thecarthagods)

Do không có điều kiện `RewriteCond %{REQUEST_FILENAME} !-d` kèm theo flag [`[QSA]`](https://httpd.apache.org/docs/current/rewrite/flags.html#flag_qsa) nên khi truy cập vào thư mục thì nó vẫn tiếp tục rewrite. Và đây là thư mục bị hạn chế quyền truy cập 403 Forbidden nên query string trong rewrite rule đã bị leak lên thanh địa chỉ. Vậy ta có được `REDACTED` là `eba1b61134bf5818771b8c3203a16dc9` cần tìm.
Dựa vào code ở trên thì flag nằm ở file `flag.php` nhưng ta khi truy cập <http://carthagods.3k.ctf.to:8039/?eba1b61134bf5818771b8c3203a16dc9=../flag.php> thì không thể lấy nội dung cuả flag.php do điều kiện `!preg_match("/<\?php/i", $f)` chặn lại nên ta không thể lấy source code của file php.
Sau 1 vòng tìm kiếm bên `info.php` ta biết được website dùng opcache để compile các file php thành bytecode giúp tăng performance.

![](/img/2020-07-30_11-37.png)

Ta có thể dùng [php7-opcache-override](https://github.com/GoSecure/php7-opcache-override) để khai thác php opcache. Sau khi tìm hiểu ta biết được file cache lưu tại **`[opcache.file_cache]`**/**`[system_id]`**/**`[path_to_file].bin`**

![](/img/2020-07-30_11-56.png)

Dùng script của [php7-opcache-override](https://github.com/GoSecure/php7-opcache-override) ta lấy được system_id. Vậy thế vào ta có đường dẫn file cache của `flag.php` là: **`/var/www/cache/e2c6579e4df1d9e77e36d2f4ff8c92b3/var/www/html/flag.php.bin`**
Payload: [http://carthagods.3k.ctf.to:8039/?eba1b61134bf5818771b8c3203a16dc9=**../../../../var/www/cache/e2c6579e4df1d9e77e36d2f4ff8c92b3/var/www/html/flag.php.bin**](http://carthagods.3k.ctf.to:8039/?eba1b61134bf5818771b8c3203a16dc9=../../../../var/www/cache/e2c6579e4df1d9e77e36d2f4ff8c92b3/var/www/html/flag.php.bin)
Và ta đã lấy được nội dung file `flag.php` dưới dạng bytecode và có chứa flag

![](/img/2020-07-30_12-13.png)