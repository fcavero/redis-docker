# Redis en Docker

Levantamos un contenedor _Docker_ con una instancia de [Redis](https://redis.io).

La imagen empleada por defecto es `redis/redis-stack`, que incluye tanto el servidor de Redis como [RedisInsight](https://redis.com/redis-enterprise/redis-insight/); en palabras de los muchachos de Redis: _«This container is best for local development because you can use the embedded RedisInsight to visualize your data.»_

## Despliegue

Como es obvio, es posible usar los comandos de _Docker_ a calzón quitao, pero para facilitar la cosa existe la posibilidad de usar un fichero `Makefile` al efecto:

```shell
❯ make
usage: make [target]

targets:
help                  Show this help message
run                   Start the Redis container
stop                  Stop the Redis container
restart               Restart the Redis container
redis-cli             Connecting to the Redis instance via redis-cli
ssh-redis             SSH into the Redis container
ssh-redis-root        SSH into the Redis container as root
logs-redis            Show logs of the Redis container (with --follow option set)
```

Podemos personalizar los argumentos de `docker run` para adecuarlos a nuestras necesidades; dichos valores se indican en un fichero `.env` que se invoca desde el `Makefile`:

* `REDIS_IMAGE` → Nombre de la imagen de _Redis_ a emplear para construir el contenedor (por defecto, `redis/redis-stack`).
* `REDIS_NAME` → Nombre del contenedor (por defecto, `redis`).
* `REDIS_PORT` → Puerto de la instancia de _Redis_ (por defecto, `6379`).
* `REDISINSIGHT_PORT` → Puerto usado por el _RedisInsight_ de administración (por defecto, `8001`).

Con todo preparado, podremos lanzar la creación del contenedor.

¡Adelante las rotativas!

```shell
❯ make run
docker rm redis || true
Error: No such container: redis
docker run -d \
        --name=redis \
        -p 6379:6379 \
        -p 8001:8001 \
        -v `pwd`/data:/data \
        --restart=on-failure \
        redis/redis-stack:latest
Unable to find image 'redis/redis-stack:latest' locally
latest: Pulling from redis/redis-stack
3b65ec22a9e9: Already exists 
dcbc409fa5d8: Pull complete 
88313b7a4398: Pull complete 
d8eed86f0a38: Pull complete 
90c77c796690: Pull complete 
4b53fd6ea332: Pull complete 
d74081cb09e9: Pull complete 
76b2ab836ee2: Pull complete 
a59552340b30: Pull complete 
79fe4a4d2ca8: Pull complete 
abd50211249d: Pull complete 
f20b332fd641: Pull complete 
f09c82789094: Pull complete 
e3a5818e0bed: Pull complete 
d32b6ba2b19f: Pull complete 
efc5c4979f00: Pull complete 
Digest: sha256:bd350ac66f6e8a1a3cd42561cf2c8b094efb604207a51f61986059abd4d0c960
Status: Downloaded newer image for redis/redis-stack:latest
d40624272b65e1c8d0115d22fbef584912bd2b9799f4ffd0264e04dd8d55e746
```

Vamos echando un ojo al proceso de creación y arranque:

```shell
❯ make logs-redis
docker logs --follow redis
9:C 03 Oct 2022 11:11:47.235 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
9:C 03 Oct 2022 11:11:47.235 # Redis version=6.2.7, bits=64, commit=00000000, modified=0, pid=9, just started
9:C 03 Oct 2022 11:11:47.235 # Configuration loaded
9:M 03 Oct 2022 11:11:47.235 * monotonic clock: POSIX clock_gettime
9:M 03 Oct 2022 11:11:47.236 # A key '__redis__compare_helper' was added to Lua globals which is not on the globals allow list nor listed on the deny list.
9:M 03 Oct 2022 11:11:47.236 * Running mode=standalone, port=6379.
9:M 03 Oct 2022 11:11:47.236 # Server initialized
9:M 03 Oct 2022 11:11:47.236 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
9:M 03 Oct 2022 11:11:47.236 * <search> Redis version found by RedisSearch : 6.2.7 - oss
9:M 03 Oct 2022 11:11:47.236 * <search> RediSearch version 2.4.14 (Git=HEAD-20b84535)
9:M 03 Oct 2022 11:11:47.236 * <search> Low level api version 1 initialized successfully
9:M 03 Oct 2022 11:11:47.236 * <search> concurrent writes: OFF, gc: ON, prefix min length: 2, prefix max expansions: 200, query timeout (ms): 500, timeout policy: return, cursor read size: 1000, cursor max idle (ms): 300000, max doctable size: 1000000, max number of search results:  10000, search pool size: 20, index pool size: 8, 
9:M 03 Oct 2022 11:11:47.236 * <search> Initialized thread pool!
9:M 03 Oct 2022 11:11:47.237 * <search> Enabled diskless replication
9:M 03 Oct 2022 11:11:47.237 * <search> Enabled role change notification
9:M 03 Oct 2022 11:11:47.237 * Module 'search' loaded from /opt/redis-stack/lib/redisearch.so
9:M 03 Oct 2022 11:11:47.238 * <graph> Starting up RedisGraph version 2.8.19.
9:M 03 Oct 2022 11:11:47.238 * <graph> Thread pool created, using 12 threads.
9:M 03 Oct 2022 11:11:47.238 * <graph> Maximum number of OpenMP threads set to 12
9:M 03 Oct 2022 11:11:47.238 * Module 'graph' loaded from /opt/redis-stack/lib/redisgraph.so
9:M 03 Oct 2022 11:11:47.239 * <timeseries> RedisTimeSeries version 10617, git_sha=c32b0cb0d8ebc1d31b74c624b8b0424628ff7887
9:M 03 Oct 2022 11:11:47.239 * <timeseries> Redis version found by RedisTimeSeries : 6.2.7 - oss
9:M 03 Oct 2022 11:11:47.239 * <timeseries> loaded default CHUNK_SIZE_BYTES policy: 4096
9:M 03 Oct 2022 11:11:47.239 * <timeseries> loaded server DUPLICATE_POLICY: block
9:M 03 Oct 2022 11:11:47.239 * <timeseries> Setting default series ENCODING to: compressed
9:M 03 Oct 2022 11:11:47.240 * <timeseries> Detected redis oss
9:M 03 Oct 2022 11:11:47.240 * <timeseries> Enabled diskless replication
9:M 03 Oct 2022 11:11:47.240 * Module 'timeseries' loaded from /opt/redis-stack/lib/redistimeseries.so
9:M 03 Oct 2022 11:11:47.240 * <ReJSON> version: 20200 git sha: 7584706 branch: HEAD
9:M 03 Oct 2022 11:11:47.240 * <ReJSON> Exported RedisJSON_V1 API
9:M 03 Oct 2022 11:11:47.240 * <ReJSON> Exported RedisJSON_V2 API
9:M 03 Oct 2022 11:11:47.240 * <ReJSON> Enabled diskless replication
9:M 03 Oct 2022 11:11:47.240 * <ReJSON> Created new data type 'ReJSON-RL'
9:M 03 Oct 2022 11:11:47.240 * Module 'ReJSON' loaded from /opt/redis-stack/lib/rejson.so
9:M 03 Oct 2022 11:11:47.240 * <search> Acquired RedisJSON_V1 API
9:M 03 Oct 2022 11:11:47.240 * <graph> Acquired RedisJSON_V1 API
9:M 03 Oct 2022 11:11:47.240 * <bf> RedisBloom version 2.2.18 (Git=8b6ee3b)
9:M 03 Oct 2022 11:11:47.240 * Module 'bf' loaded from /opt/redis-stack/lib/redisbloom.so
9:M 03 Oct 2022 11:11:47.240 * Ready to accept connections
```

Probamos la conexión con el [redis-cli](https://redis.io/docs/manual/cli/) incluido en nuestro contenedor:

```shell
❯ make redis-cli
docker exec -it redis redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> 
```

Recordad que también tenemos disponible _RedisInsight_ ─si no hemos tocado la imagen por defecto, claro─, y podremos cargar el GUI en la URL `http://localhost:8001/`:

![Empty RedisInsight](./doc/redisinsigth-out-of-the-box.png?raw=true "RedisInsigth out of the box")
