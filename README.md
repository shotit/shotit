<p align="center">
  <a href="https://shotit.github.io/" target="_blank" rel="noopener noreferrer">
    <img width="180" src="https://shotit.github.io/shotit-frontend/img/logo.svg" alt="Shotit logo">
  </a>
</p>
<br/>
<p align="center">
  <a href="https://github.com/shotit/shotit-api"><img src="https://img.shields.io/docker/v/lesliewong007/shotit-api?style=flat-square" alt="Version"></a>
  <a href="https://github.com/shotit/shotit-api"><img src="https://img.shields.io/static/v1?label=Repo&message=Shotit-api&color=brightgreen&style=flat-square" alt="Shotit-api"></a>
  <a href="https://github.com/shotit/shotit-media"><img src="https://img.shields.io/static/v1?label=Repo&message=Shotit-media&color=brightgreen&style=flat-square" alt="Shotit-media"></a>
  <a href="https://github.com/shotit/shotit-worker"><img src="https://img.shields.io/static/v1?label=Repo&message=Shotit-worker&color=brightgreen&style=flat-square" alt="Shotit-worker"></a>
  <a href="https://github.com/shotit/shotit-sorter"><img src="https://img.shields.io/static/v1?label=Repo&message=Shotit-sorter&color=brightgreen&style=flat-square" alt="Shotit-sorter"></a>
</p>
<br/>

# Shotit ⚡

Shotit is a screenshot-to-video search engine tailored for TV & Film, blazing-fast and compute-efficient.

![Shotit DEMO](./shotit-demo.png)

## Quick Start 🚀

Docker Compose is required, Please install it first.

Minimum workload: 2v16G, 4v32G preferred.

```
git clone https://github.com/shotit/shotit.git
cd shotit
```

- Copy `.env.example` to `.env`
- Edit `.env` as appropriate for your setup, as is for the first time.
- Copy `milvus.yaml.example` to `milvus.yaml`
- Edit `milvus.yaml` as appropriate for your setup, as is for the first time.

Create these necessary folders.

```
mkdir -p volumes/shotit-hash
mkdir -p volumes/shotit-incoming
mkdir -p volumes/shotit-media
mkdir -p volumes/mycores
mkdir -p volumes/mysql
```

Set the user and group information of `mycores` to 8983, required by `liresolr`. 

```
sudo chown 8983:8983 volumes/mycores
```

Then, up docker-compose services.

```
(Windows or Mac):
docker compose up -d
(Linux):
docker-compose up -d
```

Once the cluster is ready, you can add your video files to the incoming folder. Take Blender's Big Buck Bunny as an example, whose imdb tag is tt1254207, the path should be:

```
./volumes/shotit-incoming/tt1254207/Big_Buck_Bunny.mp4
```

Restart `shotit-worker-watcher`, in case it doesn't catch the change of your files.

```
docker restart shotit-worker-watcher
```

When `shotit-worker-watcher` detects the existence of video files in the incoming folder, it would start uploading the videos to object-storage powered `shotit-media`. After the upload, the videos would be eliminated, then `shotit-worker-hasher` creates hash and `shotit-worker-loader` loads the hash to vector database. Use the following command to see whether the index process has been completed: 
```
docker logs -f -n 100 shotit-worker-loader
```

When the index process completes, you will notice a `Loaded tt1254207/Big_Buck_Bunny.mp4` log and you can search the videos by screenshot directly from the URL below. 

```
GET http://127.0.0.1:3311/search?url=https://i.ibb.co/KGwVkqy/big-buck-bunny-10.png
```

Response:

```
{
    "frameCount": 0,
    "error": "",
    "result": [
        {
            "imdb": "tt1254207",
            "filename": "Big_Buck_Bunny.mp4",
            "episode": null,
            "from": 473.75,
            "to": 479.17,
            "similarity": 0.9992420673370361,
            "video": "http://127.0.0.1:3312/video/tt1254207/Big%20Buck%20Bunny.mp4?t=476.46000000000004&now=1682985600&token=kc64vEWHPMsvu54Fpl1BrR7wz8",
            "image": "http://127.0.0.1:3312/image/tt1254207/Big%20Buck%20Bunny.mp4.jpg?t=476.46000000000004&now=1682985600&token=K0qxDPHhoviiexOyEvS9qHRim4"
        }
    ]
}
```

**Congratulations!** You have successfully deployed your `shotit` search engine.

Notice: the first time of api call should be longer since shotit has to load hash completely into RAM first.

## Documentation 📖

Please see [here](https://shotit.github.io/) for full documentation on:

- Getting started (installation, hands-on demo guide, cloud-native S3 configuration)
- Reference (full API docs, limitation)
- Resources (explanation of core concepts)

## Architecture ⛪

### In a nutshell

`Shotit` is composed of these docker images.

| Docker Image           | Docker CI Build | Image Size |
| ---------------------- | --------------- | ---------- |
| [shotit-api](https://github.com/shotit/shotit-api)| [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/shotit/shotit-api/docker-image.yml?style=flat-square)](https://github.com/shotit/shotit-api/actions) | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/shotit-api/v0.9.3?style=flat-square)](https://hub.docker.com/r/lesliewong007/shotit-api) |
| [shotit-media](https://github.com/shotit/shotit-media) | [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/shotit/shotit-media/docker-image.yml?style=flat-square)](https://github.com/shotit/shotit-media/actions) | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/shotit-media/v0.9.3?style=flat-square)](https://hub.docker.com/r/lesliewong007/shotit-media) |
| [shotit-worker-watcher](https://github.com/shotit/shotit-worker)  | [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/shotit/shotit-worker/docker-image.yml?style=flat-square)](https://github.com/shotit/shotit-worker/actions) | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/shotit-worker-watcher/v0.9.3?style=flat-square)](https://hub.docker.com/r/lesliewong007/shotit-worker-watcher) |
| [shotit-worker-hasher](https://github.com/shotit/shotit-worker)   |  [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/shotit/shotit-worker/docker-image.yml?style=flat-square)](https://github.com/shotit/shotit-worker/actions) | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/shotit-worker-hasher/v0.9.3?style=flat-square)](https://hub.docker.com/r/lesliewong007/shotit-worker-hasher) |
| [shotit-worker-loader](https://github.com/shotit/shotit-worker)   |  [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/shotit/shotit-worker/docker-image.yml?style=flat-square)](https://github.com/shotit/shotit-worker/actions) | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/shotit-worker-loader/v0.9.3?style=flat-square)](https://hub.docker.com/r/lesliewong007/shotit-worker-loader) |
| [shotit-worker-searcher](https://github.com/shotit/shotit-worker) |  [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/shotit/shotit-worker/docker-image.yml?style=flat-square)](https://github.com/shotit/shotit-worker/actions) | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/shotit-worker-searcher/v0.9.3?style=flat-square)](https://hub.docker.com/r/lesliewong007/shotit-worker-searcher) |
| [shotit-sorter](https://github.com/shotit/shotit-sorter)          |   [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/shotit/shotit-sorter/docker-image.yml?style=flat-square)](https://github.com/shotit/shotit-sorter/actions) | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/shotit-sorter/v0.9.3?style=flat-square)](https://hub.docker.com/r/lesliewong007/shotit-sorter) |
| [liresolr](https://github.com/Leslie-Wong-H/liresolr) | | [![Docker Image Size](https://img.shields.io/docker/image-size/lesliewong007/liresolr/latest?style=flat-square)](https://hub.docker.com/r/lesliewong007/liresolr) |
| [minio](https://min.io/) |                 | [![Docker Image Size](https://img.shields.io/docker/image-size/minio/minio/RELEASE.2022-03-17T06-34-49Z?style=flat-square)](https://hub.docker.com/r/minio/minio) |
| [etcd](https://etcd.io/)            |                 |  [![Docker Image Size](https://img.shields.io/docker/image-size/bitnami/etcd/3.5?style=flat-square)](https://quay.io/coreos/etcd:v3.5.0) |
| [mariadb](https://mariadb.org/)                |           | [![Docker Image Size](https://img.shields.io/docker/image-size/_/mariadb/latest?style=flat-square)](https://hub.docker.com/r/_/mariadb) |
| [adminer](https://www.adminer.org)                |        | [![Docker Image Size](https://img.shields.io/docker/image-size/_/adminer/latest?style=flat-square)](https://hub.docker.com/r/_/adminer) |
| [redis](https://redis.io/)                  |         | [![Docker Image Size](https://img.shields.io/docker/image-size/_/redis/latest?style=flat-square)](https://hub.docker.com/r/_/redis) |
| [milvus-standalone](https://milvus.io/)      |         | [![Docker Image Size](https://img.shields.io/docker/image-size/milvusdb/milvus/v2.2.2?style=flat-square)](https://hub.docker.com/r/milvusdb/milvus) |

### Go deeper

![Shotit architecture](./architecture.png)

### Deep dive

![Shotit deep architecture](./deep-architecture.png)

## Benchmarks

| Dataset | Episode number | Vector volume |  Search time |
| ---------------------- | --------------- |  ---| --- | 
| [Blender Open Movie](https://studio.blender.org/films/) | 15 | 55,677 | within 5s |
| Proprietary genre dataset | 3,734 | 53,339,309 | within 5s |


## Live Demo


[https://shotit.github.io/shotit-frontend/demo](https://shotit.github.io/shotit-frontend/demo)


## Acknowledgment

`Shotit` significantly adopts its system design pattern from [trace.moe](https://github.com/soruly/trace.moe). The vision of `Shotit` is to make screenshot-to-video search engine genre-neutral, ease-of-use, compute-efficient and blazing-fast.

## Contribution

See [Contributing Guide](https://github.com/shotit/shotit/blob/main/CONTRIBUTING.md).

## License

[Apache-2.0](https://github.com/shotit/shotit/blob/main/LICENSE)
