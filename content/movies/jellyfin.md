+++
title = 'Jellyfin'
date = 2024-08-12T22:50:56+08:00
draft = false
+++

# Jellyfin Server Deploy Using Docker

```shell
# 有时候可能不是sdb1 你需要对比插拔硬盘前后/dev中的文件,看看你的硬盘到底是什么名字
mount /dev/sdb1 /mnt/usb

docker pull jellyfin/jellyfin
#                                                                                                      mounted device
docker run -d --volume /home/zebro/jellyfin/config:/config --volume /home/zebro/jellyfin/cache:/cache --volume /mnt/usb:/media --publish 8096:8096 jellyfin/jellyfin
```

```shell

# add these rules  in your machine ,  not docker,  in your server
sudo iptables -P FORWARD ACCEPT
sudo iptables -A FORWARD -i docker0 -j ACCEPT
sudo iptables -A FORWARD -o docker0 -j ACCEPT

# 注意,这一步只是你服务器上的docker变成了wireguard中继,如果想让你的服务器能启动很多服务被其他VPN中的节点访问到,还需要把这台服务器配置成wireguard的一个客户端
docker run -d --name=wg -e WG_HOST=124.78.9.226 -e PASSWORD=passwd123 -e WG_DEFAULT_ADDRESS=10.0.8.x -e WG_DEFAULT_DNS=114.114.114.114 -e WG_ALLOWED_IPS=10.0.8.0/24 -e WG_PERSISTENT_KEEPALIVE=25 -v ~/.wg-easy:/etc/wireguard -p 51820:51820/udp -p 51821:51821/tcp --cap-add=NET_ADMIN --cap-add=SYS_MODULE --sysctl="net.ipv4.conf.all.src_valid_mark=1" --sysctl="net.ipv4.ip_forward=1" --restart unless-stopped weejewel/wg-easy


# 配置ubuntu为wireguard客户端
# 不能直接scp 到/etc目录下面,会没有权限,先scp到/tmp再mv过去
scp -P 23333 /mnt/c/Users/Zebro/Desktop/server.conf zebro@192.168.1.106:/tmp/wg0.conf
mv /tmp/wg0.conf /etc/wireguard/wg0.conf

#启动 客户端
sudo wg-quick up wg0

# 腾讯云, 腾讯云的centos内核版本不支持wireguard,抽空把镜像换成ubuntu最新版本吧....

  ```


# install clash for docker pull
1. download this https://github.com/juewuy/ShellCrash/raw/master/bin/ShellCrash.tar.gz
2. send file from local machine to your server machine by scp command:
```shell 
scp -P 23333 /mnt/c/Users/zebro/Desktop/ShellCrash.tar.gz zebro@192.168.1.106:/tmp/ShellCrash.tar.gz


scp -P 23333 /mnt/c/Users/zebro/Desktop/ShellCrash.tar.gz zr:/tmp/ShellCrash.tar.gz
```

3. run this:
```shell
sudo -i #如提示输入密码，请输入用户密码
mkdir -p /tmp/SC_tmp && tar -zxf '/tmp/ShellCrash.tar.gz' -C /tmp/SC_tmp/ && bash /tmp/SC_tmp/init.sh && source /etc/profile >/dev/null

```

4. 运行crash 并选1开始生成配置文件, 直接输入这个链接  https://s1.trojanflare.one/clashx/01686572-9414-42a1-8492-385ef5f55c13  然后输入 1 .  配置好后 按照提示立即启动服务, 首次启动下载clash需要一点时间.

5. 按装crash面板  运行 crash  依次选 9   4

如果有问题 只能仔细看看 https://juewuy.github.io/bdaz/ 这个博客


#  部署nextCloud 搭建自己的nas

```shell
cd ~
# 创建应用文件夹
mkdir -p nextcloud_docker/app

# 创建数据文件夹
mkdir -p nextcloud_docker/db

# 创建docker-compose文件
touch nextcloud_docker/docker-compose.yml
```

docker-compose.yml 文件内容如下 (千万别用Maridb 会遇到权限不够的问题)
```yml
services:
  db:
    image: mysql:latest
    restart: always
    container_name: nextcloud_db
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    networks:
      - nextcloud_netbridge
    volumes:
      - /home/dev/nextcloud_docker/db:/var/lib/mysql:rw
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_PASSWORD=admin
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=admin

  app:
    image: nextcloud
    restart: always
    container_name: nextcloud_core
    ports:
      - 8090:80
    networks:
      - nextcloud_netbridge
    links:
      - db
    volumes:
      - /home/dev/nextcloud_docker/app:/var/www/html:rw
    environment:
      - MYSQL_PASSWORD=admin
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=admin
      - MYSQL_HOST=db

networks:
  nextcloud_netbridge:
    driver: bridge
version: '2.3'
volumes: {}

```


```shell
#安装docker compose plugin插件
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

运行服务
```shell
cd ~/nextcloud_docker
docker compose up 
docker logs -f nextcloud_core
```
