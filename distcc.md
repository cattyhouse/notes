## 配置情况
- 主编译机: N1, 4core aarch64
- 副编译机: N2, 4core aarch64
- 副编译机: 虚拟机 4core x86-64
- 所有主机都需要安装 distcc

## 主编译机 N1 设置:

```bash
# sudo vim /etc/makepkg.conf

MAKEFLAGS="-j28"
DISTCC_HOSTS="localhost/3 192.168.11.2/5 192.168.11.66/6"
BUILDENV=(distcc color !ccache check !sign)
```
说明
- / 后面的数字为对应的主机编译所使用的线程数量
- 加入 localhost 是为了让本机也参与编译, 而且是直接编译, 而不是通过 distcc, 所以不需要设置下面提到的软连接
- 但是因为本机还要做 Preprocess 预处理的工作, 所以本机给的编译线程数量最多为(核心数-1), 此处为 3
- 如果本机也设置为 5, 那么情况就是本机负载太重, 无法及时传输数据到副编译机上去, 反而拖慢节奏, 这一点我是不断测试发现的
- 而副编译机按照通常设置为 (核心数+1), 对于 x86_64 主机可以设置高一点没事
- -j 的参数设置为 所有主机加起来的线程数量x2, 所以是 (3+5+6)*2=28, 避免闲置.
- 查看编译分配情况可以运行 `distccmon-text 1` 表示每隔 1 秒刷新一次, 可以设置为任意数字

## 副编译机 N2 设置:

```bash
# vim /etc/conf.d/distccd

DISTCC_ARGS="--allow 192.168.11.12 --log-level error --jobs 10"
```

```bash
# 建立软连接

ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-gcc
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-c++
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-cc
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-cpp
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-g++
```
```bash
# 启动服务

systemctl start distccd.service
systemctl enable distccd.service
```

## 副编译机 x86_64 虚拟机设置:

```bash
# 因为是 x86_64 系统, 所以需要下载工具链

cd ~/ && curl -OL https://archlinuxarm.org/builder/xtools/x-tools8.tar.xz
tar -xvf x-tools8.tar.xz
```
```bash
# 设置 distcc

# sudo vim /etc/conf.d/distccd
DISTCC_ARGS="--allow 192.168.11.12 --log-level error --make-me-a-botnet --jobs 20"

# --make-me-a-botnet 参数忽略distcc的安全策略, 如果不使用这个参数, 那么需要建立软连接:
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-gcc
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-c++
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-cc
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-cpp
ln -s /usr/bin/distcc /usr/lib/distcc/aarch64-unknown-linux-gnu-g++

```
```bash
# 建立服务, 这里我们不使用自带的服务, 而是新建一个.

# vim /etc/systemd/system/distccd8.service

[Unit]
Description=A distributed C/C++ compiler (ARMv8)
Documentation=man:distccd(1)
After=network.target

[Service]
User=jst
EnvironmentFile=/etc/conf.d/distccd
Environment="PATH=/home/jst/x-tools8/aarch64-unknown-linux-gnu/bin:/usr/bin"
ExecStart=/usr/bin/distccd --no-detach --daemon $DISTCC_ARGS

[Install]
WantedBy=multi-user.target
```
```bash
# 启动服务

sudo systemctl start distcc8.service
sudo systemctl enable distcc8.service

```

## 参考

- [Distributed Compiling](https://archlinuxarm.org/wiki/Distributed_Compiling)
- [Distcc Cross-Compiling](https://archlinuxarm.org/wiki/Distcc_Cross-Compiling)
- `man distcc`
- `man distccd`




