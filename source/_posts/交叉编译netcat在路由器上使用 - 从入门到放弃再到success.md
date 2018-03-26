---
title: 交叉编译netcat在路由器上使用 - 从入门到放弃再到success
date: 2017-11-27 22:25:51
categories:
   - 技术相关
tags: 
   - 交叉编译
   - netcat
   - 路由器
---
<!-- more -->
Author : https://sp4rr0w.github.io 
Time : 2017-11-27 22:25:51
Router Model : RT-N12
Router CPU : MIPS32

## 1. 路由器架构

```
admin@RT-N12:/tmp/home/root
cat /proc/cpuinfo
system type             : Broadcom BCM53572 chip rev 1 pkg 8
processor               : 0
cpu model               : MIPS 74K V4.9
BogoMIPS                : 149.91
wait instruction        : no
microsecond timers      : yes
tlb_entries             : 32
extra interrupt vector  : no
hardware watchpoint     : yes
ASEs implemented        : mips16 dsp
shadow register sets    : 1
VCED exceptions         : not available
VCEI exceptions         : not available

unaligned_instructions  : 51078121
dcache hits             : 2147483648
dcache misses           : 0
icache hits             : 2147483648
icache misses           : 0
instructions            : 2147483648
```
    

## 2. 先编译buildroot 
可知`路由器CPU`为 `MIPS`。由同事告知可以先编译`buildroot`生成的gcc之后再指定编译`netcat`，这样可以在路由器上运行。

`折腾`之路由此开始




下载buildroot

* [https://buildroot.org/downloads/buildroot-2017.08.tar.gz](https://buildroot.org/downloads/buildroot-2017.08.tar.gz)

<!-- more -->

后编译
```
make menuconfig
```
出现
```
'make menuconfig' requires the ncurses libraries 
```

Centos 下需要安装
```
yum install ncurses-devel -y 
```

若出现这个
```
Your Perl installation is not complete enough; at least the following
modules are missing:
         Data::Dumper
         ExtUtils::MakeMaker
         Thread::Queue
```

解决
```
yum install 'perl(Data::Dumper)' -y 
yum install 'perl(ExtUtils::MakeMaker)' -y 
yum install 'perl(Thread::Queue)' -y 
```

运行
```
cd buildroot-2017.08
make manuconfig 
```

**Buildroot Configuration**
/img/Buildroot_Configuration.png

**选择 Target options**
/img/Target_options.png
```
Target Architecture		---> 	`MIPS (little endian)` 
Target Architecture Variant	---> 	`Generic MIPS32 ` 
````
make 三十分钟之后ok.
猜测`MIPS (little endian)` ，因为尝试编译了很多次才发现的，血泪教训。而路由器使用`MIPS`编译buildroot的时候会有很多选择，各自生成的gcc也不同。如下：


```
mips64 big endian       mips64 little endian
mips64-linux-gcc        mips64el-linux-gcc

mips big endian         mips little endian    
mips-linux-gcc          mipsel-linux-gcc    
```
选择mips32 little endian 即会生成mipsel-linux-gcc。

**Toolchian 选择3.2 (里面最低内核版本)**
/img/Toolchain.png
```
Kernel Headers	---> 	`Linux 3.2.x kernel headers`  
````

开始编译
```
make
```
编译完成结尾显示
```
.....
/usr/bin/install -m 0644 support/misc/target-dir-warning.txt /root/Desktop/6_mipsel_little/buildroot-2017.08/output/target/THIS_IS_NOT_YOUR_ROOT_FILESYSTEM
```

编译之后生成
``` 
./buildroot-2017.08/output/host/bin/    ：
    mipsel-linux-gcc
    mipsel-linux-ranlib 
    mipsel-linux-ar
    mipsel-linux-ld 
    mipsel-linux-strip 
and
./buildroot-2017.08/output/host/mipsel-buildroot-linux-uclibc/sysroot/lib/  ：
    ld-uClibc.so.0 -> ld-uClibc.so.1
    ld-uClibc.so.1 -> ld-uClibc-1.0.26.so
    ld-uClibc-1.0.26.so

    libc.so.0 -> libuClibc-1.0.26.so
    libc.so.1 -> libuClibc-1.0.26.so
    libuClibc-1.0.26.so
```


## 3. 编译netcat

下载netcat
```
wget http://sourceforge.net/projects/netcat/files/netcat/0.7.1/netcat-0.7.1.tar.gz/download -O netcat-0.7.1.tar.gz
```
编译netcat
```
cd netcat-0.7.1
./configure
make CC=/root/Desktop/buildroot-2017.08/output/host/bin/mips64el-linux-gcc
```

不行的话就全部指定
```
cd netcat-0.7.1 
CC=/root/Desktop/6_mipsel_little/buildroot-2017.08/output/host/bin/mipsel-linux-gcc RANLIB=/root/Desktop/6_mipsel_little/buildroot-2017.08/output/host/bin/mipsel-linux-ranlib AR=/root/Desktop/6_mipsel_little/buildroot-2017.08/output/host/bin/mipsel-linux-ar LD=/root/Desktop/6_mipsel_little/buildroot-2017.08/output/host/bin/mipsel-linux-ld STRIP=/root/Desktop/6_mipsel_little/buildroot-2017.08/output/host/bin/mipsel-linux-strip ./configure --host=mipsel-linux
make
```

编译之后生成
```
cd src 
file ./netcat 
    ./netcat: ELF 32-bit LSB executable, MIPS, N32 `MIPS64` version 1 (SYSV), dynamically linked (uses shared libs), with unknown capability 0xf41 = 0x756e6700, with unknown capability 0x70100 = 0x3040000, not stripped
./netcat 
    bash: ./netcat: cannot execute binary file
```



`telnet`登录路由器，使用wget 下载我的netcat（不可以传到`https的网络盘`，因为路由上的wget不支持https。例如`https://dropfile.to`或者其他，一定传http类型网站例如`http://s.dropcanvas.com`）

之后发现`./netcat` 可以运行，但是 ：
```
admin@RT-N12:/tmp/home/root# ./netcat
Cmd Line : -lvvp 5555
segmentation fault

admin@RT-N12:/tmp/home/root# ./netcat_mips_little -h
GNU netcat 0.7.1, a rewrite of the famous networking tool.
Basic usages:
connect to somewhere:  ./netcat_mips_little [options] hostname port [port] ...
listen for inbound:    ./netcat_mips_little -l -p port [options] [hostname] [port] ...

admin@RT-N12:/tmp/home/root# ./netcat_mips_little 192.168.19.113 5555 < 1.txt
segmentation fault
```

只有-h参数可用（心中万只草泥马飞奔而过）,不然都是`segmentation fault`

无法知道错误，大神又告诉我，`Qemu`可以模拟MIPS运行netcat



## 转到Ubuntu
```
sudo apt-get install qemu
    ==>
        /usr/bin/qemu-system-i386
        /usr/bin/qemu-mipsel
        /usr/bin/qemu*
```


出错
```
sudo /usr/bin/qemu-mipsel netcat_mipsel_little
    =>  /lib/ld-uClibc.so.0: No such file or directory
```

解决（`ld-uClibc-1.0.26.so`在上面已经提到了 在`./buildroot-2017.08/output/host/mipsel-buildroot-linux-uclibc/sysroot/lib/`里）
```
sudo mv ld-uClibc-1.0.26.so /lib/
sudo chown -R root:root /lib/ld-uClibc-1.0.26.so
sudo ln -s /lib/ld-uClibc-1.0.26.so /lib/ld-uClibc.so.0
```

出错
```
sudo /usr/bin/qemu-mipsel netcat_mipsel_little
    =>  /home/db/Desktop/netcat_mipsel_little: can't load library 'libc.so.0'
```

解决
```
sudo mv libuClibc-1.0.26.so /lib/            
sudo chown -R root:root /lib/libuClibc-1.0.26.so
sudo ln -s /lib/libuClibc-1.0.26.so /lib/libc.so.0
```

但是但是 运行
```
sudo /usr/bin/qemu-mipsel netcat_mipsel_little --help
GNU netcat 0.7.1, a rewrite of the famous networking tool.
Basic usages:
connect to somewhere:  netcat_mipsel_little [options] hostname port [port] ...
listen for inbound:    netcat_mipsel_little -l -p port [options] [hostname] [port] ...
tunnel to somewhere:   netcat_mipsel_little -L hostname:port -p port [options]

Mandatory arguments to long options are mandatory for short options too.
Options:
  -c, --close                close connection on EOF from stdin
  -e, --exec=PROGRAM         program to exec after connect
  -g, --gateway=LIST         source-routing hop point[s], up to 8
  -G, --pointer=NUM          source-routing pointer: 4, 8, 12, ...
  -h, --help                 display this help and exit
  -i, --interval=SECS        delay interval for lines sent, ports scanned
  -l, --listen               listen mode, for inbound connects
  -L, --tunnel=ADDRESS:PORT  forward local port to remote address
.....
  
sudo /usr/bin/qemu-mipsel netcat_mipsel_little -lvvp 555
or
sudo /usr/bin/qemu-mipsel netcat_mipsel_little 192.168.19.113 5555 < /home/db/Desktop/netcat_mipsel_little
    ==> 
        Unsupported setsockopt level=65535 optname=128
        Error: Couldn't create connection (err=-2): Protocol not available
        
```

`????????`  --help 没问题，其他参数就不行?




`然后大神告诉我，可能路由器的Linux内核太低了,处理器mipsel也不支持buildroot编译内核3.2的,我看了一下是2.6.32的，而我编译的buildroot选择Linux内核是3.2，最高可选4.4。
妈的 buildroot可选内核都没2.6.32，这让我怎么搞 !`



`Fri 24 Nov 2017 07:19:57 AM EST`

周末玩荒野行动PC版两局都是第二，差点吃鸡....气死我了 

周一再搞 ， 发现
....
https://buildroot.org/downloads/buildroot-2009.02.tar.gz
最低 -> Linux 2.6.35.x kernel header -> 无法编译

但是 ！！！！
这个版本
```
https://buildroot.org/downloads/buildroot-2012.05.tar.gz
```
最低 -> `Linux 2.6.35.x kernel header` -> `编译成功`
编译之后上传路由器 成功监听端口了,其它参数皆可正常使用，不会出现 `segmentation fault`.
http://s.dropcanvas.com/1000000/923000/922746/netcat_2011_05

Nice!!!! 

**running netcat**
/img/router_netcat.png
(AC66U and RT-N12 一樣的CPU)

这里还有个小点，放到Ubuntu的时候 使用qemu-mipsel执行netcat 依旧无法使用，猜测qemu版本问题，
我这个是2.5,官网已经2.11了，但是我apt-get install qemu时说此版本是最高了......估计从官网下载编译可以运行的。
```
# sudo /usr/bin/qemu-mipsel netcat_2011_05 -l -p 555
Unsupported setsockopt level=65535 optname=128
Error: Couldn't setup listening socket (err=-2)
```

卸载重装qemu
```
sudo apt-get remove --auto-remove qemu
wget https://download.qemu.org/qemu-2.10.1.tar.xz
tar xvJf qemu-2.10.1.tar.xz
cd qemu-2.10.1
./configure
make

# cd qemu-2.10.1/mipsel-linux-user
# mipsel-linux-user ./qemu-mipsel /home/db/Desktop/netcat_2011_05
qemu: uncaught target signal 11 (Segmentation fault) - core dumped
[1]    61221 segmentation fault (core dumped)  ./qemu-mipsel /home/db/Desktop/netcat_2011_05
....还有错误  不管这个了
```


