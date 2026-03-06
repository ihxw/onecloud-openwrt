上游: https://github.com/coolsnowwolf/lede


1. 用于docker: openwrt-amlogic-meson8b-thunder-onecloud-rootfs.tar.gz 

2. 用于线刷: openwrt-amlogic-meson8b-thunder-onecloud-ext4-emmc.img.gz

# 线刷我没有刷过，我主要用来 docker 运行

# docker 部署记录
```shell
cat openwrt-amlogic-meson8b-thunder-onecloud-rootfs.tar.gz | docker import - onecloud-openwrt:latest
```


# 下面的内容有AI后续生成，不保证正确，但是上面你的镜像确实是可用的，我懒得再次部署做教程

 部署到玩客云作为旁路由
 在你的玩客云（已刷入底层系统，如 Armbian 或 CasaOS，并装好 Docker）上，执行以下操作来拉取并运行：

1. 开启网卡混杂模式（必须）
为了让 Docker 容器能作为路由器接管局域网流量，需要开启网卡（假设玩客云网卡名为 eth0）的混杂模式：

```sudo ip link set eth0 promisc on```

2. 创建 Docker Macvlan 网络
根据你的主路由网段调整（假设主路由 IP 是 192.168.1.1）：


```shell
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macnet

docker run -d \
  --name openwrt \
  --restart always \
  --network macnet \
  --privileged \
  onecloud-openwrt:latest \
  /sbin/init

docker exec -it openwrt sh
vi /etc/config/network # 将ip 192.168.1.1 改成和你的路由在同一个网段的 ip地址，不要和主路由的ip一样



```



# 如果出现无法打开web的情况(Web 默认：root 密码 password):

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
