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

## 編註

* 這是一篇與 Android 開發較微無關的章節。屬於後端平台性的章節。
* 筆者是在 2013/11, 0.6.7 之後的版本接觸。
