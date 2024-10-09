# 持久化
默认开启RDB持久化。

于`/etc/redis/redis.conf`配置文件添加`appendonly yes`即可开启AOF持久化。

可以通过查看`/data`下有什么文件来看开启了哪个持久化。