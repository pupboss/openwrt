![OpenWrt logo](/logo.svg)

The firmware is for personal use, maily optimized for oversea users and bypass-gateway use scenarios. The default IP is `192.168.1.101`, default password is `password`.

The code is on top of Lean's LEDE code repository.

## Requirements

- Ubuntu 16/18 or Debian 8/9/10
- 4GB memory
- 30GB disk space

```
sudo apt update

sudo apt install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget swig rsync
```

## Quickstart

```bash
git clone https://github.com/pupboss/openwrt.git
cd openwrt
```

### Build the image

```bash
# Update dependencies
./scripts/feeds update -a
./scripts/feeds install -a

# Customize your .config, it can be reused the next time
make menuconfig

# Download
make -j8 download V=s

# Compile
make -j$(($(nproc) + 1)) V=s
```

You will see the image file in `gz` format in `./bin/targets`.

## Few Insights

Why to reinvent the wheel? On GitHub, actually, there are few active distributions already exist. I found that large volume of improvements have been done there, but rare documents to record those improvements for future refrerence, especially for greebirds of OpenWrt, such as the underlying reason why they did modification and which part that they modified. It's known that all widely participated open-source projects must have stand-out documentations like user manual or development manual. 

Few examples are listed as below:

- [openwrt/openwrt](https://github.com/openwrt/openwrt)
- [Alamofire/Alamofire](https://github.com/Alamofire/Alamofire)
- [nextcloud/server](https://github.com/nextcloud/server)

For those projects with less people to maintain, their documentations are less up to standard as most are copied or drafted without thoughtful thinking, making it even more difficult to maintain with time goes by.

Meanwhile, in the new forked distribution, which part has been modified, the reason why to modify and even no comparision are all made unclear to the audience. Without these details but all non-sense issues raised, how come maintainers can keep contributing codes with patience in the long term? In this case, it cannot be community's all faults that no people contribute codes.

Let's come back to code! OpenWrt Chinese community is the most disordered one that I have ever seen. You can found severval distributions but for same function and a lot plugins become garbage as nobody maintains. 

More worse, Some maintainers scratch a few lines of code here and a few lines of configuration files there, only making plugin meets the minimum requirement that is able to run, not able to solve any complex situations from root. 

Also, most developers always would like to do addition but not subtraction, which means to refactor and weight down, inevitably making most softwares contain unexposed issues that cannot be debug.

So, that's why I would like to kick off a fresh space here rather than moving forward on old roads, tied up with briars and thorns. ~~Starting from the official build, making lean additions on that, avoiding invasions as much as possible, and to make it shine to the public. We can assume here that the official OpenWrt is trusted and simple as it is maintained globally. ~~


~~In this case, if anything related to root system goes wrong, you can directly raise issues in OpenWrt community to find solutions after excluding yourself modifications. Otherwise, it may happen due to inherent issues of plugins.~~

The way to go is absoultely hard but worthwhile, so there are few questions listed below to be solved out:

1. Find out the best practices of seperating original configuration and customized configurations without modification to official documents
2. Rewrite some plugins with simplest principles and cover border tesitng
3. Cautiously making any additon while targeting best practices

## License

OpenWrt is licensed under GPL-2.0
