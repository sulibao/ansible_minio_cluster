# ansible_minio_cluster

此文档用于指示如何使用ansible+docker+docker-compose快速部署4个副本的minio高可用集群。

# 示例服务器列表

https://i-blog.csdnimg.cn/direct/565b4753942c4cbd98bdd29f0cdcae1b.png

# 安装前

## 修改变量文件group_vars/all.yml

```
docker_data_dir: /app/docker_data   #docker数据存储目录
minio_data: /app/minio_data    #minio数据存储目录
minio_port: 9000              #minio页面端口
minio_console_port: 9001      #minio-console端口
image_minio: "registry.cn-chengdu.aliyuncs.com/su03/minio:RELEASE.2024-05-28T17-19-04Z"
# minio镜像
minio_ak: "admin"    #minio-ak
minio_sk: "admin@2025"   #minio-sk
```

## 修改ansible主机清单

```
[minio01]  #以下分别填写用于部署minio的4个节点IP地址
192.168.2.190
[minio_others01]
192.168.2.191
[minio_others02]
192.168.2.192
[minio_others03]
192.168.2.193
```

## 修改setup.sh安装脚本

```
vim setup.sh
export ssh_pass="sulibao"     #此项应为服务器root用户密码
```

# 用法演示

```
bash setup.sh
```

# 安装后验证

- 命令行验证

```sh
docker exec -it minio_data-minio-1 bash   #进入任意一个节点任意一个minio容器
bash-5.1# mc alias set mycluster http://test1:9000 admin admin@2025   #为任意一个节点设置别名
mc: Configuration written to `/tmp/.mc/config.json`. Please update your access credentials.
mc: Successfully created `/tmp/.mc/share`.
mc: Initialized share uploads `/tmp/.mc/share/uploads.json` file.
mc: Initialized share downloads `/tmp/.mc/share/downloads.json` file.
Added `mycluster` successfully. 
bash-5.1# mc admin info mycluster    #查看集群状态，以下为正常4副本online状态
●  test1:9000
   Uptime: 16 minutes 
   Version: 2024-05-28T17:19:04Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  test2:9000
   Uptime: 20 minutes 
   Version: 2024-05-28T17:19:04Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  test3:9000
   Uptime: 54 seconds 
   Version: 2024-05-28T17:19:04Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  test4:9000
   Uptime: 20 minutes 
   Version: 2024-05-28T17:19:04Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

┌──────┬───────────────────────┬─────────────────────┬──────────────┐
│ Pool │ Drives Usage          │ Erasure stripe size │ Erasure sets │
│ 1st  │ 0.5% (total: 200 GiB) │ 4                   │ 1            │
└──────┴───────────────────────┴─────────────────────┴──────────────┘

4 drives online, 0 drives offline, EC:2

```

- 页面上传文件验证数据目录是否同步

https://i-blog.csdnimg.cn/direct/3c026d9fc972413896c1805fd9f5329c.png

```
[root@test1 app]# ll minio_data/test/
total 0
drwxr-xr-x 2 root root 21 Apr  7 22:41 制作tomcat镜像.md
[root@test2 app]# ll minio_data/test/
total 0
drwxr-xr-x 2 root root 21 Apr  7 22:41 制作tomcat镜像.md
[root@test3 app]# ll minio_data/test/
total 0
drwxr-xr-x 2 root root 21 Apr  7 22:41 制作tomcat镜像.md
[root@test4 app]# ll minio_data/test/
total 0
drwxr-xr-x 2 root root 21 Apr  7 22:41 制作tomcat镜像.md
```

