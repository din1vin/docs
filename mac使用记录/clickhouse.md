### 安装

```shell
docker pull yandex/clickhouse-client

docker pull yandex/clickhouse-server
```



### 使用

```shell
cd Personal/docker
 
docker run -d --name ch-server --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 -p 9009:9009 yandex/clickhouse-server

cli

clickhouse默认用户名是defult 没有密码,端口是8123

```





