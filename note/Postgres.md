

### 安装PostgreSQL13

#### 在线安装

官网说明：https://www.postgresql.org/download/linux/redhat/

```sh
# Install the repository RPM:
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install PostgreSQL:
sudo yum install -y postgresql13-server

# Optionally initialize the database and enable automatic start:
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable postgresql-13
sudo systemctl start postgresql-13
```

#### 离线安装

下载离线安装包：https://www.postgresql.org/ftp/source/

建议将源码包放在/usr/local/src文件夹下，因为这个文件夹通常是系统管理员放置源码包的地方，约定俗成。

```
rpm -Uvh *.rpm --nodeps --force
```

#### 修改配置文件

```sh
#进入postgresql
su - postgres
#进入sql命令行
psql
#修改postgres密码
ALTER USER postgres WITH PASSWORD 'postgres123';
#创建用户
create user admin with password '123456' SUPERUSER;	//超级用户
create user test_user with password 'abc123';       //创建用户
```

配置远程连接：

```sh
#vim /var/lib/pgsql/13/data/pg_hba.conf

local   all  all    md5    
host all all 0.0.0.0/0 md5
```

```sh
#vim /var/lib/pgsql/13/data/postgresql.conf
listen_addresses = '*'
```

重启服务

```
sudo systemctl restart postgresql-13
```

#### 备份与还原

```sh
#相关参数参考：https://www.kancloud.cn/zzgjb/pgsql/76204
pg_restore -d nis -U postgres  /var/lib/pgsql/13/backups/nis0301.dmp
```

