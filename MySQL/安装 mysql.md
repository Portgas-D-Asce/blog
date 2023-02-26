# 安装

官网下载

# 配置环境变量

```bash
MYSQL_HOME=/usr/local/mysql
export MYSQL_HOME
export PATH=${PATH}:${MYSQL_HOME}/bin
```



# 连接/退出数据库

```bash
mysql -u root -p

exit
```



# 命令

## 查询所有数据库

```bash
# show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```

## 创建/删除数据库

```bash
create database blog;
```

## 进入数据库

```bash
use blog
```

```bash
```

