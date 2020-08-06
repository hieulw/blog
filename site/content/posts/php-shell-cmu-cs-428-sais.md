---
title: php shell - CMU-CS 428 SAIS
date: 2020-08-06T07:56:37.880Z
draft: false
categories:
  - CTF
tags:
  - php
  - shell
---
Source Code:

```php
<?php
$target_dir = "uploads/";
$target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);
$uploadOk = 1;
$imageFileType = strtolower(pathinfo($target_file,PATHINFO_EXTENSION));

// Check if image file is a actual image or fake image
if(isset($_POST["submit"])) {
  $check = getimagesize($_FILES["fileToUpload"]["tmp_name"]);
  if($check !== false) {
    echo "File is an image - " . $check["mime"] . ".";
    $uploadOk = 1;
  } else {
    echo "File is not an image.";
    $uploadOk = 0;
  }
}

// Check if file already exists
if (file_exists($target_file)) {
  echo "Sorry, file already exists.";
  $uploadOk = 0;
}

// Check file size
if ($_FILES["fileToUpload"]["size"] > 500000) {
  echo "Sorry, your file is too large.";
  $uploadOk = 0;
}

// Allow certain file formats
if($imageFileType != "jpg" && $imageFileType != "png" && $imageFileType != "jpeg"
&& $imageFileType != "gif" && $imageFileType != "php") {
  echo "Sorry, only JPG, JPEG, PNG & GIF files are allowed.";
  $uploadOk = 0;
}

// Check if $uploadOk is set to 0 by an error
if ($uploadOk == 0) {
  echo "Sorry, your file was not uploaded.";
// if everything is ok, try to upload file
} else {
  if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"],$target_file)) {
    echo "The file ". $_FILES["fileToUpload"]["name"] . " has been uploaded.";
  } else {
    echo "Sorry, there was an error uploading your file.";
  }
}
?>
```

Điều kiện để upload file:

* Có header image
* Có đuôi `.jpg` `.png` `.jpeg` `.gif` `.php`

Như vậy ta cần có header image trong shell để bypass hàm `getimagesize()`

![](/img/2020-08-06_15-40.png)

![](/img/2020-08-06_15-45.png)

![](/img/2020-08-06_15-45_1.png)

![](/img/2020-08-06_15-47.png)

Source code bài 2 tương tự bài 1:

```php
<?php
// Check if $uploadOk is set to 0 by an error
if ($uploadOk == 0) {
  echo "Sorry, your file was not uploaded.";
// if everything is ok, try to upload file
} else {
  list($name,$ext)=explode(".", $target_file);
  $new_target=$name.".".$ext;
  if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"],$new_target)) {
    echo "The file ". $new_target . " has been uploaded.";
  } else {
    echo "Sorry, there was an error uploading your file.";
  }
}
```

Nhưng bây giờ không cho phép upload file `.php` lên nữa. Ta có thể upload lên file có dạng `shell.php.jpg` để bypass mà khi up lên server vẫn thành `shell.php` nhờ câu lệnh `$new_target=$name.".".$ext;`

Tức `explode('shell.php.jpg')` => `array('shell','php','jpg')` và tên file upload lên sẽ chỉ lấy 2 phần tử đầu. Nên file trên server vẫn là `shell.php`

![](/img/2020-08-06_15-50.png)

![](/img/2020-08-06_15-50_1.png)

![](/img/2020-08-06_15-51.png)



Con shell bài 2 làm đẹp đẹp tí :)