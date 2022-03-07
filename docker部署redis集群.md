# docker 部署redis集群 

## 1、创建redis网卡

~~~shll
docker network create redis --subnet 172.38.0.0/16

查看网卡信息
docker network ls 
docker network inspect redis
~~~

## 2、创建redis配置

~~~shell
#使用脚本创建6个redis配置

for port in $(seq 1 6); \
do \
mkdir -p /Users/nlsg/sinan/workCode/docker/redis/node-${port}/conf
touch /Users/nlsg/sinan/workCode/docker/redis/node-${port}/conf/redis.conf
cat << EOF >>/Users/nlsg/sinan/workCode/docker/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done


~~~

启动redis

 ```shell
 docker run -p 6371:6379  -p 16371:16379 --name redis-1 \
 -v /Users/nlsg/sinan/workCode/docker/redis/node-1/data:/data \
 -v /Users/nlsg/sinan/workCode/docker/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
 -d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
 
 
 
 #使用命令启动六个
 for port in $(seq 1 6); \
 do \
 docker run -p 637${port}:6379  -p 1637${port}:16379 --name redis-${port} \
 -v /Users/nlsg/sinan/workCode/docker/redis/node-${port}/data:/data \
 -v /Users/nlsg/sinan/workCode/docker/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
 -d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
 done
 
 ```

## 3、创建redis集群

#### 	使用cluster 集群配置

```shell
#进入某一个redis容器后执行
docker exec -it redis-1 /bin/sh   #/bin/bash

redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
```



###  测试

```shell
redis-cli -c #连接redis集群
cluster info #查看集群信息
cluster nodes #查看主从信息
set a 11  #查看存入到那个ip中  停掉服务后 在get
get a
```





 