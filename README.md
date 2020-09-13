![OpenWrt logo](/logo.svg)

此固件用于个人用途，主要针对海外用户以及旁路网关/透明网关的使用场景做了优化，默认 IP 是 `192.168.1.101`，密码为空。

## 编译环境

- Ubuntu 16/18 或 Debian 8/9/10
- 4GB 以上内存
- 30GB 以上磁盘

```
sudo apt update

sudo apt install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget swig rsync
```

## 编译方法

```bash
git clone https://github.com/pupboss/openwrt.git
cd openwrt
```

### 首次编译

```
# 更新外部插件包
./scripts/feeds update -a
./scripts/feeds install -a

# 定制自己的固件并且生成 .config 文件，可以备份下来复用
make menuconfig

# 保存自定义的配置
./scripts/diffconfig.sh > diffconfig

# 下载依赖包
make -j8 download V=s

# 编译
make -j$(($(nproc) + 1)) V=s
```

### 二次编译

```
git pull

./scripts/feeds update -a
./scripts/feeds install -a

# 加载之前的自定义配置
cp diffconfig .config
make defconfig

make -j8 download
make -j$(($(nproc) + 1)) V=s
```

编译完成后可以在 `./bin/targets` 找到相关镜像。

## License

OpenWrt is licensed under GPL-2.0
