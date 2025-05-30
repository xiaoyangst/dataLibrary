目标：实现初始化和远程连接

&nbsp;

在主机上修改文件后挂载进去，这样即便下次重启就没必要重复设置

```shell
mkdir -p /mydata/mysql/conf

vi /mydata/mysql/conf/my.cnf

# 填写如下内容

[mysqld]
bind-address = 0.0.0.0
```

运行容器，挂载该配置文件

```shell
docker run -d -p 3306:3306 --name mysql57 \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf/my.cnf:/etc/mysql/conf.d/my_custom.cnf \
-e MYSQL_ROOT_PASSWORD=root \
--restart=always \
mysql:5.7
```

授权远程连接

```shell
docker exec -it mysql57 mysql -uroot -proot

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
FLUSH PRIVILEGES;

```

