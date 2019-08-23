## depends_on

在使用 Compose 时，最大的好处就是少打启动命令，但是一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。
 例如在没启动数据库容器的时候启动了应用容器，这时候应用容器会因为找不到数据库而退出，为了避免这种情况我们需要加入一个标签，就是 depends_on，这个标签解决了容器的依赖、启动先后的问题。
 例如下面容器会先启动 redis 和 db 两个服务，最后才启动 web 服务：

```
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

注意的是，默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系。

容器内部可以直接其它容器名的，比如

```yaml
external_links:
    - mysql57:mysql57
    - redis-master:redis
```

只有启动好之后才会给目标容器分配正确的IP，不然因为顺序问题导致你引用的服务无法找到。

这个简单的问题绝对浪费了超过我二个小时的时间，而且容器内部盲目对比，`docker logs`各种容器，头都大了。

