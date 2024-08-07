# Nacos 的PostgreSQL版本

---

当前pull:

```shell
docker pull yingtao-martin/nacos-server-pgsql-docker:v2.2.3
```

或

```shell
docker pull ghcr.io/yingtao-martin/nacos-server-pgsql-docker:v2.2.3
```


---
[原版Nacos链接](https://hub.docker.com/r/nacos/nacos-server)

具体的变量内容查看原版即可,只是我这边新增了几个变量 

用于适配PostgreSQL数据库

构建脚本是从[Nacos Docker](https://github.com/nacos-group/nacos-docker)克隆后

在build文件夹下的构建脚本构建的.进行过一些修改

数据库插件来自[官方收录的仓库中下载](https://github.com/nacos-group/nacos-plugin)

[当前镜像构建脚本仓库链接](https://github.com/yingtao-martin/nacos-server-pgsql-docker)

---

## 一 新增的环境变量

| 变量名            | 说明       | 示例                                                       |
|:---------------|:---------|:---------------------------------------------------------|
| PGSQL_URL      | JDBC URL | jdbc:postgresql://localhost:5432/db?currentSchema=schema |
| PGSQL_USERNAME | 数据库用户名   | postgres                                                 |
| PGSQL_PASSWORD | 数据库密码    | postgres                                                 |

## 二 docker-compose运行示例

#### docker-compose 文件

```yaml
version: "3"
services:
  nacos:
    image: yingtao-martin/nacos-service-pgsql:v2.2.3
    container_name: nacos-pgsql
    privileged: true
    env_file:
      - "/nacos/env/pgsql.env"
    network_mode: host
    volumes:
      - "/nacos/logs/:/home/nacos/logs"
```

#### pgsql.env

```yaml
MODE=standalone
PREFER_HOST_MODE=hostname
SPRING_DATASOURCE_PLATFORM=postgresql
PGSQL_URL=jdbc:postgresql://localhost:5432/db?currentSchema=schema
PGSQL_USERNAME=postgres
PGSQL_PASSWORD=postgres
```



## 三 常见问题

- caused: Incorrect result size: expected 1, actual 2;

目前发现这个问题应该是2.2.3版本的数据库有过改动,可以在[当前镜像构建脚本仓库链接](https://github.com/yingtao-martin/nacos-server-pgsql-docker)的schema文件夹下获取到pgsql的导入脚本,

具体步骤为: 

1. 在现有的nacos中把配置文件等内容导出,
2. 清空nacos连接的数据库,
3. 使用schema文件夹下的脚本进行初始化
4. 导入配置文件等内容

- No DataSource set

> PREFER_HOST_MODE=hostname;

需要设置为hostname,暂不清楚原因,在容器中会出现这个问题

猜测可能是设置为ip之后,不做处理,

不能用容器名或者ip访问到其他的容器或者主机的ip
