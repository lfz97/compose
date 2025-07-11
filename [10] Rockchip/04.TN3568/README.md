# TN3568 armbian折腾笔记
```
硬件配置
✨ CPU：瑞芯微 rk3568 4核(Cortex-A55*4)
✨ RAM：4/8G
✨ ROM：eMMC(8G/16G/32G/64G), SATA *1
✨ 数据接口：USB2.0 *1
✨ 网络接口：1000G 网口 *1
```
[遇事不决问deepseek](https://chat.deepseek.com)

[armbian系统入门](https://github.com/FrozenGEE/compose/blob/main/%5B10%5D%20Rockchip/Armbian.md)

### ⭐修改root密码(armbina默认密码为1234，初始化会要求修改)
```
passwd root
```
### ⭐替换apt源为乌班图apt源
由于清华镜像站可能不提供 bionic/arm64 的完整支持，建议改用 Ubuntu 官方 ARM 镜像源（ports.ubuntu.com），它专门支持 ARM 架构
```
echo "[Info] 正在备份默认apt源..."
cp /etc/apt/sources.list /etc/apt/sources.list.bak
echo "[Info] 正在替换apt源为乌班图apt源..."
echo deb http://ports.ubuntu.com/ubuntu-ports/ bionic main restricted universe multiverse > /etc/apt/sources.list
echo deb http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted universe multiverse >> /etc/apt/sources.list
echo deb http://ports.ubuntu.com/ubuntu-ports/ bionic-backports main restricted universe multiverse >> /etc/apt/sources.list
echo deb http://ports.ubuntu.com/ubuntu-ports/ bionic-security main restricted universe multiverse >> /etc/apt/sources.list
echo "[Info] 正在更新源..."
apt update
echo "[Info] 正在更新软件..."
apt upgrade -y
```
### ⭐更新软件列表和软件源
```
apt update && apt upgrade -y
```
### ⭐安装常用软件(按需选择)
1、安装单个
```
apt install -y vim
# apt install -y 软件名
```
2、批量安装
```
apt install -y vim nano samba nfs-kernel-server rclone git pip clinfo neofetch btop ncdu dnsmasq avahi-daemon
# apt install -y 软件名1 软件名2 软件名3 软件名4 软件名5
```
3、安装deb软件包，使用 apt 会自动处理依赖问题，无需额外操作
```
apt install ./包名.deb
```
4、检查是否安装成功
```
5、dpkg -l | grep 包名.deb
```
6、卸载 .deb 包
```
apt remove 包名.deb
```
7、完全移除软件包及其配置文件
```
apt purge 包名.deb
```
### ⭐安装 x-cmd (个人自用)
```
eval "$(wget -O- https://get.x-cmd.com)"
```
### ⭐查看命令
```
ls：列出当前目录下的文件和目录
ls -l：以长格式列出文件和目录，显示详细信息(权限、所有者、大小、修改时间等)
# ll 命令是 ls -l 的别名，如果 ll 不能使用，见下操作
ls -a：列出所有文件和目录，包括隐藏文件(以 . 开头的文件)
ls -la 或 ls -al：以长格式列出所有文件和目录，包括隐藏文件
ls -h：与 -l 一起使用，以人类可读的格式显示文件大小(如 KB、MB)
ls -R：递归列出子目录中的内容
```
### ⭐清屏
```
clear
```
### ⭐查看内核的命令
```
uname -srm
hostnamectl | grep -i kernel
cat /proc/version
# 以上三条均可
```
### ⭐SATA挂载到本地/mnt中
用于单盘NAS挂载存储介质到本地，docker容器的配置文件夹目录均存储在此处，统一路径，此处以使用 SSD 进行SATA挂载为例

1、识别设备
```
lsblk
fdisk -l
# 得知 SATA 为/dev/sda
```
2、确认分区情况
```
lsblk | grep sda
# 得知SATA分区情况
```
3、删除现有分区 (如果有的话)
```
fdisk /dev/sda
# 在fdisk命令行界面中，执行以下操作：
# 输入 p 查看现有分区
# 输入 d 删除分区，根据提示选择要删除的分区编号
# 重复删除所有分区
# 输入 w 保存更改并退出
```
4、新建分区
```
fdisk /dev/sda
# 在fdisk命令行界面中，执行以下操作：
# 输入 n 创建新分区
# 选择 1 创建主分区
# 按回车键，接受默认的起始扇区
# 按回车键，接受默认的结束扇区(使用全部存储空间)
# 输入 w 保存更改并退出
```
5、格式化分区
```
mkfs.ext4 /dev/sda1
# 以上为格式化分区为ext4

mkfs.btrfs /dev/sda1
# 以上为格式化分区为Btrfs
```
6、检查分区大小
```
parted /dev/sda print
```
7、创建挂载点
```
mkdir /mnt/ssd
# 将挂载到 /mnt/ssd 路径
```
8、临时挂载
```
mount /dev/sda1 /mnt/ssd
```
9、验证挂载状态
```
df -h | grep ssd
```
10、获取文件系统UUID
```
blkid /dev/sda1 | awk -F'UUID="' '{print $2}' | awk -F'"' '{print $1}'
# SSD 为 8842afa5-57db-4859-95a9-57ef49b3d059
```
11、修改 /etc/fstab 设置开机自动挂载
```
nano /etc/fstab
# 文本末端添加以下内容，然后推出并保存(按ctrl+X，Y，回车键)
UUID=8842afa5-57db-4859-95a9-57ef49b3d059 /mnt/ssd ext4 defaults 0 0
# 开机自动挂载SSD
```
11、挂载并验证
```
mount -a
# 测试自动挂载，若无报错即配置成功
df -h | grep ssd
# 验证挂载状态
```
12、如果遇到以下报错内容，则执行 systemctl daemon-reload 一次
```
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```
### ⭐开启WebDAV

### ⭐使用WebDAV挂载其他服务器
1、安装软件包
```
apt update
apt install davfs2 -y
```
2、创建挂载点目录，例如 /mnt/emmc/webdav
```
mkdir /mnt/emmc/webdav
```
3、配置 WebDAV 凭据 (可选)

编辑 /etc/davfs2/secrets，添加 WebDAV 服务器的用户名和密码
```
nano /etc/davfs2/secrets
```
添加如下内容 (替换 WEBDAV_URL、USERNAME、PASSWORD)，然后保存并退出
```
https://example.com/webdav  USERNAME  PASSWORD
```
4、挂载 WebDAV
```
mount -t davfs https://example.com/webdav /mnt/webdav
```
如果未在 secrets 文件配置密码，会提示输入用户名和密码

5、验证挂载
```
df -h | grep webdav
ls /mnt/emmc/webdav
```
6、开机自动挂载
```
nano /etc/fstab
```
7、添加以下内容
```
https://example.com/webdav  /mnt/emmc/webdav  davfs  user,rw,noauto  0  0
```
8、挂载
```
mount -a
```
### ⭐docker 相关
1、docker 安装

通过apt安装
```
apt install -y docker.io
```
Docker官方一键安装脚本，使用官方源安装(国内直接访问较慢)
```
curl -fsSL https://get.docker.com | bash
```
使用阿里源安装
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
使用中国区Azure源安装
```
curl -fsSL https://get.docker.com | bash -s docker --mirror AzureChinaCloud
```
一键安装最新版Docker Compose
```
COMPOSE_VERSION=`git ls-remote https://github.com/docker/compose | grep refs/tags | grep -oP "[0-9]+\.[0-9][0-9]+\.[0-9]+$" | sort --version-sort | tail -n 1`
sh -c "curl -L https://github.com/docker/compose/releases/download/v${COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
chmod +x /usr/local/bin/docker-compose
```
2、设置自启动Docker
```
systemctl enable --now docker
```
3、配置国内镜像源

[国内 Docker 服务状态 & 镜像加速监控](http://status.kggzs.cn/status/docker)
```
#- 终端命令添加 Docker 镜像加速
#- 以部分镜像加速为例
#- 以下是一整条命令，一起复制到终端运行

# 清空 /etc/docker/daemon.json 文件，准备写入新内容
echo >/etc/docker/daemon.json

# 使用 cat 命令将以下内容写入 /etc/docker/daemon.json 文件
# 配置 Docker 镜像加速器，使用多个镜像源来提高镜像下载速度
cat >/etc/docker/daemon.json <<EOF
{
"registry-mirrors": [
"https://k-docker.asia",
"https://docker.ketches.cn",
"https://docker.1ms.run"
]
}
EOF

# 重启 Docker 服务以使配置生效
systemctl restart docker
```
4、查看docker和compose版本
```
docker version && docker compose version
```
5、docker 拉取镜像

格式：docker pull 镜像源/作者名/镜像名:标签
- lscr.io，docker.1ms.run，k-docker.asia 为镜像源，也可以叫镜像仓库，可以自建
- hslr/sun-panel 为作者名和镜像名，可以去[dockerhub](https://hub.docker.com/)上搜索，有的不在dockerhub仓库上，特别是ghcr.io开头的
- latest，beta 为镜像的标签，一般情况不写则会自动拉取latest，某些镜像的tag不存在latest，或者某些架构的tag不一样，具体去看镜像详细页面上的tag
```
docker pull lscr.io/linuxserver/qbittorrent:latest
docker pull docker.amingg.com/hslr/sun-panel:beta
docker pull docker.1ms.run/dpanel/dpanel:lite
```
6、将用户加入docker组

如果遇到权限问题，将用户添加到 docker 组
```
usermod -aG docker rk3588
# sudo usermod -aG docker 用户名
```
7、docker 命令
| 命令 | 含义 | 命令 | 含义 |
| :---- | :---- | :---- | :---- |
| docker ps | 查看部署的docker容器 | docker images ps | 查看本地镜像 |
| docker start 容器ID或容器名 | 启动某个容器 | docker stop 容器ID或容器名 | 停止某个容器 |
| docker restart 容器ID或容器名 | 重启某个容器 | docker kill 容器ID或容器名 | 直接关闭容器 |
| docker rm 容器ID或容器名 | 删除某个容器 | docker tag 旧镜像名 新镜像名 | 修改镜像名字<br>注意是完整的docker镜像名字 |
* stop和kill的主要区别：stop给与一定的关闭时间交由容器自己保存状态，kill直接关闭容器
### ⭐部署 dpanel —— docker管理器
```
sudo docker run -it -d --name dpanel --restart=always -p 8807:8080 -v /var/run/docker.sock:/var/run/docker.sock -v /mnt/emmc/docker/dpanel:/dpanel registry.cn-hangzhou.aliyuncs.com/dpanel/dpanel:lite
```
### ⭐dpanel 替换加速源
```
https://docker.1ms.run
https://docker.amingg.com
https://hub.rat.dev
```


### ⭐香橙派5Plus jellyfin硬件转码
- 参考资料：[Rockchip VPU jellyfin硬件转码](https://jellyfin.org/docs/general/administration/hardware-acceleration/rockchip)

1、确保设备中存在 mpp、rga、dri、dma_heap，否则请将 BSP 内核升级到 5.10 LTS 及更高版本，运行此命令
```
$ ls -l /dev | grep -E "mpp|rga|dri|dma_heap"

drwxr-xr-x  2 root       root          80 Jan  1  1970 dma_heap
drwxr-xr-x  3 root       root         140 Jan 18 18:50 dri
crw-rw----  1 root       video   241,   0 Jan 18 18:50 mpp_service
crw-rw----  1 root       video    10, 122 Jan 18 18:50 rga
```
2、在主机上安装 ARM Mali OpenCL 

对于Ubuntu-Rockchip和Armbian上的6.1 LTS内核以及旧版5.10 LTS内核，请安装v1.9-1-2d267b0

https://github.com/tsukumijima/libmali-rockchip/releases/download/v1.9-1-2d267b0/libmali-valhall-g610-g13p0-gbm_1.9-1_arm64.deb

对于其他 SBC 供应商制作的发行版上的 6.1 LTS 内核，请安装 v1.9-1-55611b0

https://github.com/tsukumijima/libmali-rockchip/releases/download/v1.9-1-55611b0/libmali-valhall-g610-g13p0-gbm_1.9-1_arm64.deb

以下为安装命令，请根据实际情况修改
```
mkdir -p ~/tmp/libmali && cd ~/tmp/libmali
wget 'https://github.com/tsukumijima/libmali-rockchip/releases/download/v1.9-1-2d267b0/libmali-valhall-g610-g13p0-gbm_1.9-1_arm64.deb'
sudo dpkg -i ./libmali-valhall-g610-g13p0-gbm_1.9-1_arm64.deb
```
3、用 clinfo 检查主机上的 OpenCL (GPU 固件)
```
apt update && sudo apt install -y clinfo && clinfo
```
4、部署 jellyfin
详情见 [compose模板](https://github.com/FrozenGEE/compose/blob/main/%5B10%5D%20Rockchip/02.%E9%A6%99%E6%A9%99%E6%B4%BE5plus/jellyfin-%E7%A7%81%E4%BA%BA%E5%AA%92%E4%BD%93%E5%BA%93.yml)

5、要验证 OpenCL 运行时在 docker 容器内是否正常工作，您可以运行此命令，第一个 jellyfin 为容器名字
```
docker exec -it jellyfin /usr/lib/jellyfin-ffmpeg/ffmpeg -v debug -init_hw_device rkmpp=rk -init_hw_device opencl=ocl@rk
```
视频转码时，使用命令检查RGA引擎的占用情况
```
watch -n 1 cat /sys/kernel/debug/rkrga/load
```
检查 OpenCL 运行时状态
```
sudo /usr/lib/jellyfin-ffmpeg/ffmpeg -v debug -init_hw_device rkmpp=rk -init_hw_device opencl=ocl@rk

arm_release_ver: g13p0-01eac0, rk_so_ver: 10
[AVHWDeviceContext @ 0xaaaae8321360] 1 OpenCL platforms found.
[AVHWDeviceContext @ 0xaaaae8321360] 1 OpenCL devices found on platform "ARM Platform".
[AVHWDeviceContext @ 0xaaaae8321360] 0.0: ARM Platform / Mali-G610 r0p0
[AVHWDeviceContext @ 0xaaaae8321360] cl_arm_import_memory found as platform extension.
[AVHWDeviceContext @ 0xaaaae8321360] cl_khr_image2d_from_buffer found as platform extension.
[AVHWDeviceContext @ 0xaaaae8321360] DRM to OpenCL mapping on ARM function found (clImportMemoryARM).
...
```
通过内核日志查看与VPU相关的驱动加载信息，驱动名称如 rkvdec(解码)或 rkvenc(编码)会显示在日志中
```
cat /sys/class/vpu/vpu/version
```
也可以通过查看cpu占用情况确认是否硬件转码，如果cpu 100%占用，则是cpu软件转码
