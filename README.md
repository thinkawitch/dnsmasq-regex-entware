## Dnsmasq with regex support for Entware

Original regex support patches got from https://github.com/lixingcong/dnsmasq-regex

Binary made with the instruction tested on Keenetic Giga KN-1010 with KeeneticOS 3.6.3 and Entware (mipsel).


#### Ready binary:
[dnsmasq-2.84-regex-entware.tar.gz](/dnsmasq-2.84-regex-entware.tar.gz)

### 1. Prepare build environment

Clean fresh debian 10.8 linux. Install dependencies.

```shell
sudo apt update

sudo apt install build-essential ccache ecj fastjar file g++ gawk \
gettext git java-propose-classpath libelf-dev libncurses5-dev \
libncursesw5-dev libssl-dev python python2.7-dev python3 unzip wget \
python3-distutils python3-setuptools rsync subversion swig time \
xsltproc zlib1g-dev 
```

Add quilt tool, optional (in case you want to make own patch series):
```shell
sudo apt install quilt
```

### 2. Prepare sources

Get main repository:
```shell
cd ~
git clone https://github.com/Entware/Entware.git && cd Entware
```

Append keenetic developers feed to feeds.conf:
```shell
echo "src-git keendev3x https://github.com/The-BB/keendev-3x.git" >> ./feeds.conf
```

Update packages:
```shell
make package/symlinks
```

Copy platform config (mipsel in our case):
```shell
cp configs/mipsel-3.4.config .config
```

Setup build config (just save & exit without any changes):
```shell
make menuconfig
```

### 3. Build the tools

```shell
make tools/install
make toolchain/install
make target/compile
```

### 4. Prepare patches

Original regex support patches got from https://github.com/lixingcong/dnsmasq-regex

Put patches for dnsmasq sources:

```
Copy files from patches/ dir of this repository into ./package/network/services/dnsmasq/patches
```

Put patch for entware package Makefile:

```
Copy entware-Makefile.patch into ./package/network/services/dnsmasq/
```

Apply the patch to entware package makefile:

```shell
cd ./package/network/services/dnsmasq/
patch < entware-Makefile.patch
```

### 5. Build dnsmasq

```shell
cd ~/Entware
make package/dnsmasq/compile
```

Get ready binary from something like ~/Entware/build_dir/target-mipsel_mips32r2_glibc-2.27/dnsmasq-full/dnsmasq-2.84/ipkg-mipsel-3.4/dnsmasq-full/opt/sbin/dnsmasq

### 6. Place binary into keenetic router

```
# upload file to router
scp -P 222 ./dnsmasq root@keenetic-ip:/opt/root/dnsmasq

# replace new binary
ssh root@keenetic-ip -p 222
/opt/etc/init.d/S56dnsmasq stop
cp /opt/sbin/dnsmasq /opt/sbin/dnsmasq.orig
cp /opt/root/dnsmasq /opt/sbin/dnsmasq
/opt/etc/init.d/S56dnsmasq start

# check version
/opt/sbin/dnsmasq --version
Dnsmasq version 2.84  Copyright (c) 2000-2021 Simon Kelley
Compile time options: IPv6 GNU-getopt no-RTC no-DBus no-UBus no-i18n regex(+ipset) no-IDN DHCP DHCPv6 no-Lua TFTP conntrack ipset auth no-cryptohash no-DNSSEC no-ID loop-detect inotify dumpfile
```

### Regexp config sample

```
# file /opt/etc/dnsmasq.conf
# link all *a90.sa.lan domains to .90 ip
address=/:.*a90\.sa\.lan:/192.168.101.90
```

Additional example https://github.com/lixingcong/dnsmasq-regex/blob/master/dnsmasq_regex_example.conf

---
### Useful links

How to install Entware on Keenetic:
- [Entware setup in russian](https://forum.keenetic.net/topic/4299-entware/)
- [Another Entware setup example in english](https://help.keenetic.com/hc/en-us/articles/360000264829-Installation-and-configuration-of-the-rTorrent-OPKG-package)

[Entware official packages mipsel k3.4](https://bin.entware.net/mipselsf-k3.4/Packages.html)

Build package from sources:
- [Build package from sources in russian](https://forum.keenetic.net/topic/1288-%D1%81%D0%B0%D0%BC%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F-%D1%81%D0%B1%D0%BE%D1%80%D0%BA%D0%B0-%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%BE%D0%B2/)
- https://github.com/Entware/Entware/wiki/Compile-packages-from-sources
- [OpenWRT build system setup](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)

