# DN11配网(ROS)

### Peer的几个要素:

1.**公钥**(我的是`+mP9GqXMo89YXaYp5lcCGFyP82qp/T0xdGej8+TAhnQ=`)

2.**域名+端口**(我的是`dorm.ec3o.fun:15000`)

3.**隧道IP**(去飞书组织时申请顺延网段,一般是申请到网段的.254地址:`172.16.35.254`)

4.**ASN**(去飞书组织申请网段时自定义,一般是421111开头,我的是`4211115536`)

### 目前的peer步骤:

1.去routing-bgp,connection里点开potat0那个，右边copy

![](C:\Users\24993\Desktop\博客文章\static\Potat0.png)

2.copy出来的新的，name自己改，然后remote address改成对方的隧道ip,remote as改成对方的asn,其他保持原样

3.去3个地方改东西：wireguard（2个面板）、ip-address、routing-bgp

4.完成后前往GitHub仓库提交metadata(如果是第一次配置)