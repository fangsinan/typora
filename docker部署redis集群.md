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

for port in$(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
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
 -d --net redis --ip 172.38.0.11 redis:5.0.0-alpine3.11 redis-server /etc/redis/redis.conf \
 ```

## 3、创建redis集群

#### 	使用cluster

