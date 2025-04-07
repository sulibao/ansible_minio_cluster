# ansible_minio_cluster

This document is used to indicate how to quickly deploy a minio high availability cluster with 4 replicas using ansible+docker+docker-compose.

# Example server list

https://i-blog.csdnimg.cn/direct/565b4753942c4cbd98bdd29f0cdcae1b.png

# Before installation

## Modify the variable file group_vars/all.yml

```
docker_data_dir: /app/docker_data   #docker data storage directory
minio_data: /app/minio_data    #minio data storage directory
minio_port: 9000              #minio page port
minio_console_port: 9001      #minio-console port
image_minio: "registry.cn-chengdu.aliyuncs.com/su03/minio:RELEASE.2024-05-28T17-19-04Z"
# minio image
minio_ak: "admin"    #minio-ak
minio_sk: "admin@2025"   #minio-sk
```

## Modify the ansible host manifest

```
[minio01]  #The following are the IP addresses of the four nodes used to deploy minio
192.168.2.190
[minio_others01]
192.168.2.191
[minio_others02]
192.168.2.192
[minio_others03]
192.168.2.193
```

## Modify the setup.sh installation script

```
vim setup.sh
export ssh_pass="sulibao"     #This should be the server root user password
```

# Installation

```
bash setup.sh
```

# Post-installation verification

- Command-line validation

```sh
docker exec -it minio_data-minio-1 bash   #Go to any node, any minio container
bash-5.1# mc alias set mycluster http://test1:9000 admin admin@2025   #Set an alias for any node
mc: Configuration written to `/tmp/.mc/config.json`. Please update your access credentials.
mc: Successfully created `/tmp/.mc/share`.
mc: Initialized share uploads `/tmp/.mc/share/uploads.json` file.
mc: Initialized share downloads `/tmp/.mc/share/downloads.json` file.
Added `mycluster` successfully. 
bash-5.1# mc admin info mycluster    #Check the cluster status, the following is the normal 4 replica online status
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

- The page upload file verifies that the data directory is synchronized（Access method: http:// Any node IP:9000）

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

