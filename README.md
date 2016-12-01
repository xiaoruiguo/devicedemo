# devicedemo

一个基于Openstack规范开发的API Demo

### 项目结构

```
.
├── AUTHORS      
├── ChangeLog    
├── devicedemo  
│   ├── api　　　　   
│   │   ├── app.py      
│   │   ├── config.py
│   │   ├── controllers
│   │   │   ├── __init__.py
│   │   │   ├── root.py
│   │   │   └── v1
│   │   │       ├── controller.py
│   │   │       ├── devices.py
│   │   │       └── __init__.py
│   │   ├── expose.py
│   │   ├── hooks.py
│   │   └── __init__.py
│   ├── cmd
│   │   ├── api.py
│   │   └── __init__.py
│   ├── db
│   │   ├── api.py
│   │   ├── __init__.py
│   │   ├── models.py
│   │   └── sqlalchemy
│   │       ├── alembic
│   │       │   ├── env.py
│   │       │   ├── README
│   │       │   ├── script.py.mako
│   │       │   └── versions
│   │       │       ├── e7f6a2fc5d53_create_device_table.py
│   │       │       └── ff7b0e8f7372_create_user_table.py
│   │       └── alembic.ini
│   └── __init__.py
├── devicedemo.db
├── doc
│   └── your_doc_file.md
├── etc
│   └── devicedemo
│       └── your_app.conf
├── LICENSE
├── README.md
├── requirement.txt
├── setup.cfg
├── setup.py
├── test_requirements.txt
└── tools
    └── your_bash_tools.sh
```

1. 项目打包相关文件

    AUTHORS：　 描述项目参与者大
    
    ChangeLog：  描述版本的ChangLog
    
    requirement.txt： 实现自动依赖安装
    
    setup.cfg：　setuptools打包时提供包的元数据
    
    setup.py:    setup命令运行接口

2. 项目配置文件目录

    etc/devicedemo：　devicedemo项目需要的配置文件

3. 项目说明文档目录

    doc/install:  项目安装说明文档
    
    doc/source:   项目产品说明文档

4. 项目源码目录

    + api
    
    > 提高API服务的目录, 整体框架采用OpenStack新项目, 比如Ceilometer和magnum采用的框架
    > 该框架基于对象分发方式进行路由, 是一个轻量级框架, 通过WSME进行请求参数的响应的类型检查
    > 通过Paste + PasteDeploy 完成 WSGI管理, 而关于WSGI应用的部署, 需要参考Pecan官方文档
    > 其中
    > app.py 一般包含了Pecan应用的入口，包含应用初始化代码
    > config.py 包含Pecan的应用配置，会被app.py使用
    > controllers/ 这个目录会包含所有的控制器，也就是API具体逻辑的地方
    > controllers/root.py 这个包含根路径对应的控制器
    > controllers/v1/ 这个目录对应v1版本的API的控制器。如果有多个版本的API，你一般能看到v2等目录
    
    + db
    
    > 提高数据层面API, 使用使用sqlarchemy 作为ORM系统，　基于Alembic进行数据库版本管理
    > 其中
    > api.py 实现一个Connection类，这个类封装了所有的数据库操作接口。我们会在这个类中实现对表的CRUD等操作
    > models.py　定义了所有的模型
    > sqlalchemy/alembic 基于Alembic的配置以及数据信息都存放在此目录中
    
    + cmd
    
    > service需要有一个命令行的管理接口, 这个是实现的用于管理API Service的一个命令行工具
    
5. 单元测试与覆盖率测试(比较复杂Demo未实现)

    在OpenStack的项目中, 这部分很全面复杂, 涉及到如下内容：
    
	测试环境管理: tox
	
	测试用例的运行和管理: testrepository, subunit, coverage
	
	测试用例的编写: unittest, mock, testtools, fixtures, testscena


### 使用组件参考文档
1. Pecan

    中文文档较少, 这里是[官方文档](http://pecan.readthedocs.io/en/latest/)
    
    这里有一份基于Ceilometer的介绍,涉及对Pecan介绍[Ceilometer API调用流程分析](http://blog.csdn.net/s1234567_89/article/details/51890459)

2. WSME

    文档很少, 这里是[官方文档](https://pythonhosted.org/WSME/)

3. sqlarchemy

    文档很多

4. Alembic

     [基本使用1](http://blog.csdn.net/oranyujian/article/details/48464365)
     
     [基本使用2](http://www.codeweblog.com/%E4%BD%BF%E7%94%A8alembic/)
     
     [官网](http://www.alembic.io/)
    

### Demo安装与运行

- 安装依赖组件KeyStone
 
 因为DeviceDemo依赖KeyStone做为认证和服务注册框架所以第一步是按照好KeyStone, 这里以按照OpenStack Nowton为列做简要介绍
 关于KeyStone中的一些核心概念请参考[KeyStone架构说明](http://docs.openstack.org/developer/keystone/architecture.html)
 具体按照过程可以参考[OpenStackNewton官方安装文档](http://docs.openstack.org/newton/install-guide-ubuntu/)
 我本地环境是Ubuntu16.04, 以下为一个简要过程
    
```
# 添加OpenStack newton官方源
apt install software-properties-common
add-apt-repository cloud-archive:newton

# 更新源
apt update && apt dist-upgrade

# 按照依赖的库以及软件
apt install python-pip python-dev libffi-dev libssl-dev libxml2-dev libxslt1-dev libjpeg8-dev zlib1g-dev git tmux lrzsz vim python-pip python-tox uwsgi

# 按照apache2 并且启用wsgi模块
apt-get install apache2 libapache2-mod-wsgi
a2enmod wsgi

# 安装openstack客户端工具
apt install python-openstackclient

# 数据库安装
apt install mariadb-server python-pymysql

# 配置数据库:  /etc/mysql/mariadb.conf.d/50-server.cnf
[mysqld]
bind-address = 127.0.0.1
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

# 启动数据库和数据库安全加固
service mysql restart
mysql_secure_installation

# 安装消息队列与添加用户与授权
apt install rabbitmq-server
rabbitmqctl add_user openstack RABBIT_PASS
rabbitmqctl set_permissions openstack ".*" ".*" ".*"

# 安装和配置memcache, 默认监听127.0.0.1可以不用修改配置
apt install memcached python-memcache
-l 127.0.0.1

# 启动memcached
service memcached restart

# 为KeyStone准备数据库
$ mysql -u root -p
mysql> CREATE DATABASE keystone;
mysql> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
mysql> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
  
# 部署keyStone服务
apt install keystone

# 配置KeyStone, 配置文件位置：/etc/keystone/keystone.conf, 修改database和token段
[database]
...
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone

[token]
...
provider = fernet

# 初始化表结构
su -s /bin/sh -c "keystone-manage db_sync" keystone

# 初始化 Fernet Key 仓库
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

# Bootstrap Identity服务
# keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://127.0.0.1:35357/v3/ \
  --bootstrap-internal-url http://127.0.0.1:35357/v3/ \
  --bootstrap-public-url http://127.0.0.1:5000/v3/ \
  --bootstrap-region-id RegionOne

# 启动Keystone服务(由apache2代理)
# service apache2 restart
# rm -f /var/lib/keystone/keystone.db

# 配置管理员账号, 将其保持到一个配置里方便使用
root@yumaojun-virtual-machine:~# cat keystone_admin 
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_DOMAIN_NAME=default
export OS_AUTH_URL=http://127.0.0.1:35357/v3
export OS_IDENTITY_API_VERSION=3


# 测试KeyStone是否正常工作,安装完成后会有一个超级管理员admin
root@yumaojun-virtual-machine:~# source  keystone_admin 
root@yumaojun-virtual-machine:~# openstack user list
+----------------------------------+------------+
| ID                               | Name       |
+----------------------------------+------------+
| db2928a61fcb44b09dab7e6154130a67 | admin      |
+----------------------------------+------------+
root@yumaojun-virtual-machine:~# openstack project list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| f0ae5fb3f68b40f0beec4da06895c50d | admin   |
+----------------------------------+---------+

```

- Devicedemo注册为KeyStone的服务

    Identity服务需要为每个Openstack的服务提供认证功能, 身份验证服务使用域、项目、用户和角色的组合。
    预定俗称有2个项目service 和 admin, service包容所有内部服务账号,给内部服务使用比如glance, 
    而admin项目则是包含OpenStack管理员账号的, 也就是上面KeyStone安装完成后的那个admin账号，我们使用他来
    管理整套OpenStack服务
    
    因此这里我们需要为devicedemo服务创建服务用户和服务端点
    
```
# 创建service项目
$ openstack project create --domain default \
  --description "Service Project" service

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 24ac7f19cd944f4cba1d77469b2a73ed |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | default                          |
+-------------+----------------------------------+

# 第一阶段Devicedemo并没用使用MySQL作为数据库,而是使用sqlite做为数据库，所以这里不用为Devicedemo初始化数据库
# Demo测试通过后完善时，会添加MySQL作为数据库

# 为Devicedemo服务创建用户: devicedemo
$ openstack user create --domain default --password-prompt devicedemo

User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 3f4e777c4062483ab8d9edd7dff829df |
| name                | devicedemo                       |
| password_expires_at | None                             |
+---------------------+----------------------------------+

# 将devicedemo用户加入到service项目的管理员中
$ openstack role add --project service --user devicedemo admin

# 创建devicedemo服务
$ openstack service create --name devicedemo \
  --description "OpenStack Devicedemo" devicedemo

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Devicedemo             |
| enabled     | True                             |
| id          | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| name        | devicedemo                       |
| type        | devicedemo                       |
+-------------+----------------------------------+

# 为devicedemo服务创建端点（我devicedemo的服务端口为9511）
$ openstack endpoint create --region RegionOne \
  devicedemo public http://127.0.0.1:9511

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 340be3625e9b4239a6415d034e98aace |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | devicedemo                       |
| service_type | devicedemo                       |
| url          | http://127.0.0.1:9511            |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  devicedemo internal http://127.0.0.1:9511

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a6e4b153c2ae4c919eccfdbb7dceb5d2 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | devicedemo                       |
| service_type | devicedemo                       |
| url          | http://127.0.0.1:9511            |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  devicedemo admin http://127.0.0.1:9511

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 0c37ed58103f4300a84ff125a539032d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | devicedemo                       |
| service_type | devicedemo                       |
| url          | http://127.0.0.1:9511            |
+--------------+----------------------------------+
```


- 配置Devicedemo并运行

    准备好的KeyStone, 也将devicedemo注册到的KeyStone中, 剩下的就是配置Devicedemo使用KeyStone并且启动服务
    
```
# 配置devicedemo的Keystone相关服务账号, 上面为Devicedemo服务申请的账号是 devicedemo/password
# 修改devicedemo项目下面的 etc/devicedemo/devicedemo.conf为
[keystone_authtoken]
auth_uri = http://127.0.0.1:5000
auth_url = http://127.0.0.1:35357
memcached_servers = 127.0.0.1:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = devicedemo
password = password

[paste_deploy]
flavor = keystone

# 在devicedemo项目下 启动服务
 yumaojun@yumaojun-virtual-machine  ~/PycharmProjects/devicedemo   master ●✚  PYTHONPATH=. python3 devicedemo/cmd/api.py --config-dir etc/devicedemo
2016-12-01 10:59:40.617 78288 WARNING oslo_reports.guru_meditation_report [-] Guru meditation now registers SIGUSR1 and SIGUSR2 by default for backward compatibility. SIGUSR1 will no longer be registered in a future release, so please use SIGUSR2 to generate reports.
2016-12-01 10:59:40.618 78288 INFO devicedemo.api.app [-] Full WSGI config used: /home/yumaojun/PycharmProjects/devicedemo/etc/devicedemo/api-paste.ini
2016-12-01 10:59:40.727 78288 INFO __main__ [-] Starting server in PID 78288
2016-12-01 10:59:40.733 78288 INFO __main__ [-] Serving on http://0.0.0.0:9511
2016-12-01 10:59:40.738 78288 INFO werkzeug [-]  * Running on http://0.0.0.0:9511/ (Press CTRL+C to quit)
/usr/local/lib/python3.5/dist-packages/keystoneauth1/adapter.py:135: UserWarning: Using keystoneclient sessions has been deprecated. Please update your software to use keystoneauth1.
  warnings.warn('Using keystoneclient sessions has been deprecated. '
2016-12-01 10:59:48.770 78288 INFO werkzeug [-] 127.0.0.1 - - [01/Dec/2016 10:59:48] "GET /devices HTTP/1.1" 404 -
2016-12-01 14:06:07.788 78288 INFO werkzeug [-] 127.0.0.1 - - [01/Dec/2016 14:06:07] "GET / HTTP/1.1" 200 -
2016-12-01 14:07:18.544 78288 INFO werkzeug [-] 127.0.0.1 - - [01/Dec/2016 14:07:18] "GET /v1 HTTP/1.1" 200 -
2016-12-01 14:07:24.026 78288 INFO werkzeug [-] 127.0.0.1 - - [01/Dec/2016 14:07:24] "GET /v1/devices HTTP/1.1" 200 -

# 以上Demo服务启动完毕
```

### Demo测试样例

- 像Keystone申请token, 用于访问devicedemo服务

```
RAW_TOKEN=`curl -s -X POST http://127.0.0.1:5000/v2.0/tokens -H "Content-Type: application/json"  -d '{"auth": {"tenantName": "'"admin"'", "passwordCredentials":{"username": "'"admin"'", "password": "'"password"'"}}}'`
TOKEN=`echo $RAW_TOKEN | python -c "import sys; import json; tok = json.loads(sys.stdin.read()); print tok['access']['token']['id'];"`
root@yumaojun-virtual-machine:~# echo $TOKEN
5899f591e99b49e0b53710ba38576892
```

- 查看device列表

```
root@yumaojun-virtual-machine:~# curl -H "Content-Type: application/json" -H "X-Auth-Token: $TOKEN" -X GET http://127.0.0.1:9511/v1/devices | python -m json.tool
{
    "devices": [
        {
            "name": "test1",
            "type": "test",
            "uuid": "test1",
            "vendor": "test",
            "version": "111"
        },
        {
            "name": "tes1222",
            "type": "test",
            "uuid": "test1sdfdsdff",
            "vendor": "test",
            "version": "111"
        },
        {
            "name": "tes2222222",
            "type": "test",
            "uuid": "test1sdfdsdfdff",
            "vendor": "test",
            "version": "111"
        },
        {
            "name": "test_new",
            "type": "test_new",
            "uuid": "test_new",
            "vendor": "test",
            "version": "111"
        },
        {
            "name": "test_new2",
            "type": "test_new2",
            "uuid": "test_new2",
            "vendor": "test",
            "version": "111"
        },
        {
            "name": "test_new5",
            "type": "test_new2",
            "uuid": "test_new5",
            "vendor": "test",
            "version": "111"
        }
    ]
}
```

- 查看device详情

```
root@yumaojun-virtual-machine:~# curl -H "Content-Type: application/json" -H "X-Auth-Token: $TOKEN" -X GET http://127.0.0.1:9511/v1/devices/test1 | python -m json.tool
{
    "name": "test1",
    "type": "test",
    "uuid": "test1",
    "vendor": "test",
    "version": "111"
}
```

- 创建device

```
root@yumaojun-virtual-machine:~# curl -X POST http://localhost:9511/v1/devices -H "Content-Type: application/json" -H "X-Auth-Token: $TOKEN" -d '{"uuid": "test_new7", "name": "test_new7", "type":"test_new7", "vendor": "test", "version": "111"}' | python -m json.tool
{
    "data": "create success",
    "error": null
}
```

- 删除device

```
root@yumaojun-virtual-machine:~# curl -X DELETE -H "X-Auth-Token: $TOKEN" http://localhost:9511/v1/devices/test_new2 | python -m json.tool
{
    "data": "delete device test_new2 success",
    "error": null
}
```