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

