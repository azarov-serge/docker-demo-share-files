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

## bind mounts

!!! Use for bind config

Clear all, stop all or remove all:

```bash
docker container prune
docker volume prune
```

Binding to user directory (spaceman)

```bash
docker run --name share-files-v-2 -d -p 3003:3000 -v /home/spaceman/data:/opt/app/data share-files-v:latest
cd ~/
ls -l
```

Result:

```bash
drwxr-xr-x 2 root     root        4096 Jul 11 09:11 data
drwxrwxr-x 3 spaceman spaceman    4096 Jul 10 11:51 docker-demo-go
drwxrwxr-x 3 spaceman spaceman    4096 Jul 10 14:03 docker-demo-network
drwxrwxr-x 4 spaceman spaceman    4096 Jul 11 08:43 docker-demo-share-files
drwxrwxr-x 5 spaceman spaceman    4096 Jul 10 10:29 image-converter
```

## tmpfs

!!! Need for secret

```bash
docker run --name share-files-v-3 -d -p 3004:3000 --tmpfs /opt/app/data  share-files-v:latest
curl "127.0.0.1:3004/set?id=2292"
docker stop share-files-v-3
docker start share-files-v-3
curl "127.0.0.1:3004/get"
```

Result (error):

```bash
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Error: ENOENT: no such file or directory, open &#39;./data/req&#39;<br> &nbsp; &nbsp;at Object.openSync (fs.js:498:3)<br> &nbsp; &nbsp;at readFileSync (fs.js:394:35)<br> &nbsp; &nbsp;at /opt/app/src/index.js:13:14<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/opt/app/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/opt/app/node_modules/express/lib/router/route.js:137:13)<br> &nbsp; &nbsp;at Route.dispatch (/opt/app/node_modules/express/lib/router/route.js:112:3)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/opt/app/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at /opt/app/node_modules/express/lib/router/index.js:281:22<br> &nbsp; &nbsp;at Function.process_params (/opt/app/node_modules/express/lib/router/index.js:335:12)<br> &nbsp; &nbsp;at next (/opt/app/node_modules/express/lib/router/index.js:275:10)</pre>
</body>
</html>
```

Eq `docker run --name share-files-v-3 -d -p 3004:3000 --tmpfs /opt/app/data  share-files-v:latest`

```bash
docker run --name share-files-v-4 -d -p 3005:3000 --mount type=tmpfs,destination=/opt/app/data share-files-v:latest
```

## Copy data

Run:

```bash
docker run --name share-files-v-5 -d -p 3006:3000 share-files-v:latest
curl "127.0.0.1:3006/get"
```

Result (error):

```bash
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Error: ENOENT: no such file or directory, open &#39;./data/req&#39;<br> &nbsp; &nbsp;at Object.openSync (fs.js:498:3)<br> &nbsp; &nbsp;at readFileSync (fs.js:394:35)<br> &nbsp; &nbsp;at /opt/app/src/index.js:13:14<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/opt/app/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/opt/app/node_modules/express/lib/router/route.js:137:13)<br> &nbsp; &nbsp;at Route.dispatch (/opt/app/node_modules/express/lib/router/route.js:112:3)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/opt/app/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at /opt/app/node_modules/express/lib/router/index.js:281:22<br> &nbsp; &nbsp;at Function.process_params (/opt/app/node_modules/express/lib/router/index.js:335:12)<br> &nbsp; &nbsp;at next (/opt/app/node_modules/express/lib/router/index.js:275:10)</pre>
</body>
</html>
```

Run:

```bash
docker cp /home/spaceman/data share-files-v-5:/opt/app/data

curl "127.0.0.1:3006/set?id=909"
docker cp share-files-v-5:/opt/app/data /home/spaceman/test-data
cat /home/spaceman/test-data/req
```

Result:

```bash
909
```

Copy all:

```bash
docker cp share-files-v-5:/opt/app/data/. /home/spaceman/test-all-data
```
