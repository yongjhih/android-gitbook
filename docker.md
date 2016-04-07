# Docker

## 常用指令

開放使用者權限

```bash
sudo usermod -aG docker andrew
```

啟動 imag:

```bash
docker run -it ubuntu /bin/bash
```

指定 image 版本啟動 (-i interactive, -t tty):

```bash
docker run -it ubuntu:14.04 /bin/bash
```

背景啟動 image (-d dettach):

```
docker run -d ubuntu
```

進入執行中的 container:

```bash
docker exec -it c007a10e4 bash
```

更新 image:

```bash
docker pull ubuntu
```

停止執行中的 container:

```bash
docker stop c007a10e4
```

啟動停止執行中的 container:

```bash
docker start c007a10e4
```

列出執行中的 containers:

```bash
docker ps
```

列出包括停止的 containers (-a all):

```bash
docker ps -a
```

列出下載的 images:

```bash
docker images
```

掛目錄進去 (-v volume):

```java
docker run -d ubuntu -v /home:/var/home
````


設定 port (-p port):

```java
docker run -d ubuntu -p 80:3000
````

設定環境變數 (-e env):

```java
docker run -d ubuntu -e "http_proxy=http://192.168.1.254:3128"
````

## docker-gen

透過 container 資訊生成設定檔

以前透過 `docker inspect 6680cc9d6d9a` 取得資訊參考來寫設定檔，現在透過樣板語言來生成。

例如：

```sh
docker-gen nginx.tmpl nginx.conf
```

延伸用法產生設定檔後自動重啟指定 container ：

```sh
docker-gen -notify-sighup nginx -watch -only-exposed -wait 5s:30s nginx.tmpl nginx.conf
```

案例 - letencrypt 自動生成 nginx 與 proxy：

```yml
simple-site:
  image: nginx
  container_name: simple-site
  ports:
      - "8080:80"
  volumes:
    - "./volumes/examples/simple-site/conf.d/:/etc/nginx/conf.d"
  environment:
    - VIRTUAL_HOST=site.example.com
    - LETSENCRYPT_HOST=site.example.com
    - LETSENCRYPT_EMAIL=email@example.com

nginx:
  image: nginx
  container_name: nginx
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - "/etc/nginx/conf.d"
    - "/etc/nginx/vhost.d"
    - "/usr/share/nginx/html"
    - "./volumes/proxy/certs:/etc/nginx/certs:ro"
nginx-gen:
  image: jwilder/docker-gen
  container_name: nginx-gen
  volumes:
    - "/var/run/docker.sock:/tmp/docker.sock:ro"
    - "./volumes/proxy/templates/nginx.tmpl:/etc/docker-gen/templates/nginx.tmpl:ro"
  volumes_from:
    - nginx
  entrypoint: /usr/local/bin/docker-gen -notify-sighup nginx -watch -only-exposed -wait 5s:30s /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
letsencrypt-nginx-proxy-companion:
  image: jrcs/letsencrypt-nginx-proxy-companion
  container_name: letsencrypt-nginx-proxy-companion
  volumes_from:
    - nginx
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
    - "./volumes/proxy/certs:/etc/nginx/certs:rw"
  environment:
    - NGINX_DOCKER_GEN_CONTAINER=nginx-gen
```

* jrcs/letsencrypt-nginx-proxy-companion 啟動後會產生憑證
* nginx 等待生成設定檔與憑證產生
* nginx-gen 等待憑證產生，並依據 simple-site 接著產生 proxy 設定檔，重啟 nginx 生效

使用者只要維護 simple-site 的部份就好，就會自動產生憑證，且 proxy 也會幫你設定好。

共享 certs 憑證目錄

## 編註

* 這是一篇與 Android 開發較微無關的章節。屬於後端平台性的章節。
* 筆者是在 2013/11, 0.6.7 之後的版本接觸。
