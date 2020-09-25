![OpenWrt logo](/logo.svg)

[English README](README.en.md)

[详细使用说明](https://github.com/pupboss/openwrt/wiki)

此固件用于个人用途，主要针对海外用户以及旁路网关/透明网关的使用场景做了优化，默认 IP 自编译的是 `192.168.1.1` 我编译好的是 `192.168.1.101`，密码为 `password`。

> 默认 IP 的配置最好是自己在根目录创建 `./files/etc/config/network`，这样编译结束之后就跟以前的配置一模一样。

代码修改自 Lean 的 LEDE，主要修改内容就是让整个代码变得更精简，又不缺失什么功能，精简的定义就是没有多余的功能，最好连多余的依赖包都没有，并且所有功能都可以用，最后就是内核日志不报错。

大家可能不知道的一点是，如果你在 LuCI 取消选中某个功能，它并不会自动删除依赖包的勾选，要避免这个问题的方法就是 `make menuconfig` 只选自己需要的。第二种办法就是在固件里固定下来需要的包，编译的时候直接 `make menuconfig` 不作任何修改然后保存。

>  按照官方的推荐做法，应该是用 `make defconfig` 功能来生成默认配置文件，官方版确实也可以这么做。LEDE 似乎是 hook 了配置的相关函数，导致这条命令工作不正常，后面我会尽可能还原出来。

在这个版本中，固件大小选择了 160，因为过大的会导致内核像下面一样无限报错。VPN 服务器因为 [IPSec 方式总是有问题](https://github.com/coolsnowwolf/lede/issues/5262)，所以只安装了一个 SoftEther，虽然配置起来比其他的麻烦，但是挺好用。最后就是网易云音乐变灰解锁功能，Nodejs 编译需要很久，并且单独一个 Go 版本也够用了，所以直接去除了支持，然后换上了自己签名的证书。

```bash
[ 4546.035204] print_req_error: operation not supported error, dev loop0, sector 132936
[ 1884.659316] blk_update_request: I/O error, dev loop0, sector 16855008
```

其他的改动如下：

- 默认的 app 做了微调，简单说就是去除了 DDNS 还有文件传输 FTP，增加 nano 编辑器等等
- 默认打包 gzip 版的 img，可以用于直接在线刷机升级，并且体积只有 65MB
- 时区改成海外的，并且删掉国内的 OpenWrt mirror
- 调整了默认包的位置，比如 `luci-app` 开头的全部放在全局 [target](/include/target.mk) 里，系统相关的放在 [x86](/target/linux/x86/Makefile) 下。

具体使用方法是这样的，如果你的需求和我差不多，基本上 `make menuconfig` 之后什么也不用改直接保存即可，我个人在编译的时候会新增一个包 `open-vm-tools`。选包的时候尽可能一次选对，否则处理多余的依赖包很麻烦。

## 编译环境

- Ubuntu 16/18 或 Debian 8/9/10
- 4GB 以上内存
- 30GB 以上磁盘

```bash
sudo apt update

sudo apt install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget swig rsync curl
```

## 编译方法

```bash
git clone https://github.com/pupboss/openwrt.git
cd openwrt
```

### 首次编译

```bash
# 更新外部插件包
./scripts/feeds update -a
./scripts/feeds install -a

# 定制自己的固件并且生成 .config 文件，可以备份下来复用
make menuconfig

# 下载依赖包
make -j8 download V=s

# 编译
make -j$(($(nproc) + 1)) V=s
```

### 二次编译

```bash
git pull

./scripts/feeds update -a
./scripts/feeds install -a

make menuconfig

make -j8 download
make -j$(($(nproc) + 1)) V=s
```

编译完成后可以在 `./bin/targets` 找到相关镜像。

## 一些废话

关于为什么再造一个轮子，确实目前市面上已经有好几个发行版本，我也重点研究了几个，他们都做了大量的优化工作，这些工作本身都是很好的，唯一的缺陷就是没有文档，初次接触 OpenWrt 的开发者根本不知道他们到底修改了什么，为什么这么改。能被开发者们广泛参与的开源项目，无一例外的就是他们的文档都写的非常棒，不管是使用文档还是开发贡献文档，这里我给几个例子：

- [openwrt/openwrt](https://github.com/openwrt/openwrt)
- [Alamofire/Alamofire](https://github.com/Alamofire/Alamofire)
- [nextcloud/server](https://github.com/nextcloud/server)

对于那些寥寥数人维护的项目，绝大多数的文档可以用“不讲究”一词来形容，要么是不知道从哪复制粘贴的，要么是跟画龙一样随便画几句，新 fork 的发行版本也不会写明改了哪里，不会给出一个对比图，让用户知道改了哪里以及为什么这么改，一切全靠猜。尤其是在面对众多无脑 issue 的时候，维护者会更加容易失去耐心，所以在这种情况下，没什么人贡献代码也不是完全是社区的问题。

再回到代码本身，OpenWrt 中文社区是我见过最混乱的社区没有之一，相同功能的插件发行好几个，还有一堆插件没人维护成为垃圾，有的维护者这里扒拉几行代码，那里扒拉几行配置文件，拼出来的插件也只是满足了能够运行这个最低条件，说直白点就是一坨能动的废铁，出了疑难杂症根本没有能力解决。大多数开发者只会无限的做加法，很少有人愿意重构以及做减法，这就导致不少软件实际上隐含着非常多的难以 debug 的问题。

所以这就是我为什么再挖一个新坑，~~与其背着历史包袱前行，还不如在最纯净的版本上面做加法，尽可能的减少侵入性，并且公示所有和官方系统不一样的地方。我们可以假定官方 OpenWrt 是可信任的，因为它由全球的开发者共同维护，并且原生系统真的很简洁。在这种情况下，如果路由系统本身出了问题，排除掉自己修改的地方，就可以去 OpenWrt 社区提 issue 寻找解决方案，否则的话应该是插件本身存在问题，不至于 debug 的时候一头雾水。~~

经过两天的探索，我发现官方版的用起来问题更多，解决完依赖问题和补丁问题之后，编译是可以成功，但是安装好之后随便点几下整个页面就会崩掉。还有就是有时候 `make menuconfig` 也会报错，虽然不影响编译，总的来说还是相当不稳定。基于 Lean 的 LEDE 做改动会省去很多麻烦。

后面大概有如下问题需要解决：

1. 找到 C++ 程序最小侵入性的最佳实践
2. 或许会重写部分插件，用最简单的原理来实现，并且覆盖好边缘测试
3. 在尽可能无侵入性的情况下，谨慎的做加法

共勉。

## License

OpenWrt is licensed under GPL-2.0
