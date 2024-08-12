+++
title = 'Jellyfin'
date = 2024-08-12T22:50:56+08:00
draft = true
+++

# Jellyfin Server Deploy Using Docker

```shell
docker pull jellyfin/jellyfin
#                                                                                                      mounted device
docker run -d --volume /home/zebro/jellyfin/config:/config --volume /home/zebro/jellyfin/cache:/cache --volume /mnt/usb:/media --publish 8096:8096 jellyfin/jellyfin
```

