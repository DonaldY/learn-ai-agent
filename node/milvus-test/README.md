Milvus 里查询知识，是根据语义匹配的，你可以用自然语言来检索。

-   Milvus 分为 database、collection、entity 这三级，collection 要指定数据结构也就是 schema。
-   vector 向量字段需要做索引，用来快速检索。

安装通过 docker，用 docker-compose，配置可以从这里找：<https://github.com/milvus-io/milvus/releases>

```yml
version: '3.5'

services:
  etcd:
    container_name: milvus-etcd
    image: quay.io/coreos/etcd:v3.5.25
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
      - ETCD_SNAPSHOT_COUNT=50000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/etcd:/etcd
    command: etcd -advertise-client-urls=http://etcd:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio:
    container_name: milvus-minio
    image: minio/minio:RELEASE.2024-12-18T13-15-44Z
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    ports:
      - "9001:9001"
      - "9000:9000"
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/minio:/minio_data
    command: minio server /minio_data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  standalone:
    container_name: milvus-standalone
    image: milvusdb/milvus:v2.6.12
    command: ["milvus", "run", "standalone"]
    security_opt:
    - seccomp:unconfined
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
      MQ_TYPE: woodpecker
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9091/healthz"]
      interval: 30s
      start_period: 90s
      timeout: 20s
      retries: 3
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - "etcd"
      - "minio"

networks:
  default:
    name: milvus
```

可以根据`docker-compose.yml`直接跑起来：

```shell
docker compose -f ./milvus-standalone-docker-compose.yml up -d
```

-   milvus 数据库是跑在 19530 这个端口。
-   访问这个 url 可以做健康度检查：http://localhost:9091/healthz

安装一个 GUI 工具：https://github.com/zilliztech/attu?tab=readme-ov-file[#quick]()-start

Attu 是 Milvus 生态最好的 GUI 工具：https://github.com/zilliztech/attu/releases

```shell
# 安装后打开出现报错：“Attu” can’t be opened because Apple cannot check it for malicious software.

# 如果它是从网上下载的 .app，有时是被加了隔离属性；终端可去掉：
# 可以尝试：
sudo xattr -dr com.apple.quarantine /Applications/Attu.app
```

```shell
npm init -y

pnpm install @zilliz/milvus2-sdk-node

pnpm install @langchain/openai dotenv
```
