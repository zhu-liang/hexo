---
title: Docker使用之Host本地一个简单的Wordpress服务器
date: 2016-12-18 22:45:07
tags: Docker
---

------

首先我们需要陈列所需要的components
> * MySQL 数据库
> * WordPress Instance

所以我们需要先 pull 这两个components各自的image
```Bash
docker pull mysql
```
```Bash
docker pull wordpress
```

可以看到结果
```Bash
PowerEngine:~ zl$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
wordpress           latest              d1d84847b56e        3 days ago          400.2 MB
mysql               latest              594dc21de8de        4 days ago          400.2 MB
```

运行mysql服务器，并使用-e选项传入相应的参数
```Bash
PowerEngine:~ zl$ docker run -e MYSQL_ROOT_PASSWORD=12345 -e MYSQL_USER=zhuliang -e MYSQL_PASSWORD=54321 -e MYSQL_DATABASE=newname_db  -v /Users/zl/develop/tool/docker/mysql:/var/lib/mysql --name wordpress_db -d mysql
b2e5631bf826322a91eb0fb1c494956c83e4216cc7eec5794395e1b7ea33607f
PowerEngine:~ zl$ 
```

运行wordpress 镜像，同样使用-e传入相应参数
```Bash
PowerEngine:~ zl$ docker run -e WORDPRESS_DB_USER=zhuliang -e WORDPRESS_DB_PASSWORD=54321 -e WORDPRESS_DB_NAME=newname_db -p 8088:80 -v /Users/zl/develop/tool/docker/wordpress/html:/var/www/html --link wordpress_db:mysql --name word_press_instance -d wordpress
22c05b98f19d0459466c9f2f29b96322d5938d2e1cdeb972d2026fafea874643
PowerEngine:~ zl$ 
```
根据我们上一条中expose出来的端口号8088，访问
```Bash
localhost:8088
```
则可以进入到熟悉的wordpress配置页面
```Bash
Welcome

Welcome to the famous five-minute WordPress installation process! Just fill in the information below and you’ll be on your way to using the most extendable and powerful personal publishing platform in the world.

Information needed

Please provide the following information. Don’t worry, you can always change these settings later.
```
