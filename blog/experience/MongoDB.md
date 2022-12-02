# 使用 docker 安装并启动 MongoDB

## 前期准备

- 安装 docker、一些 docker 前置知识。可以参考[菜鸟教程](https://www.runoob.com/docker/ubuntu-docker-install.html)
- 安装客户端 [Studio 3T](https://robomongo.org/) 有 Free 版本

下列安装命令适用于 MongoDB 6.0 版本。

## 开始安装

```shell
# 下载并启动 MongoDB 的docker
docker pull mongo
# -v 后面的参数是映射到本地的数据，按需修改，不需要可以不加
docker run -itd --name mongo -v /home/He-yu/mongo/data:/data/db -p 27017:27017 mongo --auth
# 进入 docker
docker exec -it mongo mongosh
# 进入用户表，开始创建用户
use admin
# 创建 root 用户并切换过去
db.createUser({ user:'root',pwd:'root',roles:['root']});
db.auth('root', 'root');
# 创建用户管理员用户 admin，该用户同时可以访问操作所有数据库
db.createUser({ user:'admin',pwd:'admin',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
# 退出 docker 到本机终端，重启镜像
docker restart 【CONTAINER ID】
```

然后就可以使用 root 用户或者 admin 用户，通过 Studio 3T 来访问数据库了。

## 使用前准备

上面的操作新建了 root 用户。但是一般来说不会使用 root 用户来进行操作，也不会使用 admin 数据库来进行数据存储，因此需要新建数据库、新建数据库管理员用户。

假设新数据库名称为 `study`。

### 新建数据库

参考资料

- [MongoDB 创建数据库](https://www.runoob.com/mongodb/mongodb-create-database.html)

```shell
docker exec -it mongo mongosh
# 切换到 admin 账户
use admin
db.auth('admin', 'admin');
# 查看现有数据库列表
show dbs
# 新建数据库
use study
# 必须插入一条数据才会真正创建
db.study.insert({"name":"first"})
# 查看新的数据库列表，可以发现创建成功
show dbs
```

### 新建数据库管理员

参考资料

- [MongoDB 添加用户](https://www.yiibai.com/mongodb/create-users.html)
- [MongoDB 权限介绍](https://cloud.tencent.com/developer/article/1955526)

```shell
docker exec -it mongo mongosh
# 切换到 admin 账户
use admin
db.auth('admin', 'admin');
# 切换到 study 数据库
use study
# 创建 study 数据库的具有读写权限的用户
db.createUser({user: "study-admin",pwd: "study-admin",roles:[{role: "readWrite" , db:"study"}]});
# 查看用户
show users
```

然后就可以使用新建的 study-admin 用户，通过 Studio 3T 来访问 study 数据库了。
