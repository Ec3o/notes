## CTF中常见php-MD5()函数漏洞

### 1.数字与字符串之间的比较

```php
var_dump( 0 == "a" );
var_dump( "0" == "a" );
```

第一个返回的是 true ，第二个返回的是 false
因为php把字母开头的转化为整型时，转化为0， 前面数字后面字母的话就只取到第一个字母出现的位置之前（如intval(’'123abd45gf)结果为123）

### 2.MD5函数漏洞

```php
$_GET['name'] != $_GET['password']
MD5($_GET['name']) == MD5($_GET['password'])
```

要求满足上述条件则
那么要求name和password数值不同但是MD5相同，在这里可以利用绕过。
PHP在处理哈希字符串时，它把每一个以“0E”开头的哈希值都解释为0，所以如果两个不同的密码经过哈希以后，其哈希值都是以“0E”开头的，那么PHP将会认为他们相同，都是0。

**以下值在md5加密后以0E开头：**

- QNKCDZO
- 240610708
- s878926199a
- s155964671a
- s214587387a
- s214587387a

**以下值在sha1加密后以0E开头：**

- sha1(‘aaroZmOk’)
- sha1(‘aaK1STfY’)
- sha1(‘aaO8zKZF’)
- sha1(‘aa3OFF9m’)

GET传入`a=QNKCDZO&b=240610708`就能绕过了

### 3.php特性

```php
if($_POST['param1']!==$_POST['param2'] && md5($_POST['param1'])===md5($_POST['param2'])){die("success!");}
```

在php中===为完全等于运算，不仅比较**值**，而且还比较值的**类型**，只有**两者一致**才为真。再次使用a=QNKCDZO&b=240610708就不行了，因为a和b类型不同。
**PHP中md5的函数特性**

```php
md5([1,2,3]) == md5([4,5,6]) == NULL
```

[1] !== [2] && md5([1]) === md5([2])
所以GET传入`a[]=1&b[]=2`就能够绕过了。

### 4.MD5碰撞

```php
if((string)$_POST['param1']!==(string)$_POST['param2'] && md5($_POST['param1'])===md5($_POST['param2'])){die("success!);
}
```

要求构造param1和param2不同，但是MD5相同，也就是说要求传入两个MD5相同的不同字符串。

```
Param1=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2
Param2=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2
```

MD5值相同使用谷歌可以搜到相当多被巧妙构造出的二进制文件，其MD5相同，注意一点，post时一定要urlencode！！！