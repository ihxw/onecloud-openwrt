1. 用于docker: openwrt-amlogic-meson8b-thunder-onecloud-rootfs.tar.gz 

2. 用于线刷: openwrt-amlogic-meson8b-thunder-onecloud-ext4-emmc.img.gz

# 线刷我没有刷过，我主要用来 docker 运行

# docker 部署记录
```shell
cat openwrt-amlogic-meson8b-thunder-onecloud-rootfs.tar.gz | docker import - onecloud-openwrt:latest
```

```shell
docker run -d \
  --name openwrt \
  --restart always \
  --network macnet \
  --privileged \
  onecloud-openwrt:latest \
  /sbin/init
```
```shell
docker exec -it openwrt sh
vi /etc/config/network
```
# 如果出现无法打开web的情况:

openwrt 容器内执行:
```vi /etc/config/uhttpd```

查找并注释 HTTPS 监听行：
在文件中找到类似 list listen_https '0.0.0.0:443' 和 list listen_https '[::]:443' 的行。
按 i 进入编辑模式，在这些行前面加上 # 号：

Ini, TOML
# list listen_https '0.0.0.0:443'
# list listen_https '[::]:443'
禁用证书路径（可选）：
如果有 option cert 和 option key 的配置，也建议一并注释掉。

保存并退出：
按 Esc 键，输入 :wq 并回车。

🚀 重启服务
修改完成后，执行以下命令：
openwrt 容器内执行:
/etc/init.d/uhttpd restart
如果服务不再报错，再次执行 ps | grep uhttpd，你应该能看到它稳定地运行在进程列表中了。
