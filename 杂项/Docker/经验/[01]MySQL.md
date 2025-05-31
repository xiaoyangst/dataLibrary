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

## 远程连接失败？

```shell
docker stop mysql57
docker rm mysql57
docker run -d --name mysql57 --network host -e MYSQL_ROOT_PASSWORD=root mysql:5.7
```

注：mysql57 是你的容器名称，记得更改

用 `--network host` 模式：

- **优点**：容器直接使用宿主机网络，端口不需要映射，网络更直通，远程访问问题少。
- **缺点**：容器和宿主机共享网络命名空间，可能带来安全风险，也不适合多个相同端口的容器同时运行。
