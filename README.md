# docker-demo-share-files (docker image)

Clone project and create image:

```bash
git clone https://github.com/azarov-serge/docker-demo-share-files.git
cd docker-demo-share-files/
docker build -t share-files:latest .
```

Create volume:

```bash
docker volume create share-files-volume
```

Result:

```bash
DRIVER    VOLUME NAME
local     share-files-volume
```

Inspect volume

```bash
docker volume inspect share-files-volume
```

Result:

```bash
[
    {
        "CreatedAt": "2023-07-11T07:24:38Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/share-files-volume/_data",
        "Name": "share-files-volume",
        "Options": null,
        "Scope": "local"
    }
]
```

Run:

```bash
docker run --name share-files-1 -v share-files-volume:/opt/app/data -p 3000:3000 -d share-files:latest
curl "127.0.0.1:3000/set?id=1234"
```

Result:

```bash
done!
```

Run:

```bash
sudo cat /var/lib/docker/volumes/share-files-volume/_data/req
```

Result:

```bash
1234
```

Run:

```bash
docker run --name share-files-2 -v share-files-volume:/opt/app/data -p 3001:3000 -d share-files:latest
curl "127.0.0.1:3001/get"
```

Result:

```bash
1234
```

Clear all, stop all or remove all:

```bash
docker container prune
docker volume prune
```

For add volume (Dockerfile)

```Dockerfile
FROM node:14-alpine as build
WORKDIR /opt/app
ADD *.json ./
RUN npm install
ADD . .
VOLUME ["/opt/app/data"]
CMD ["node", "./src/index.js"]
```

Run:

```bash
docker build -t share-files-v:latest .
docker volume inspect share-files-volume
docker run --name share-files-3 -v share-files-volume:/opt/app/data -p 3002:3000 -d share-files-v:latest
```
