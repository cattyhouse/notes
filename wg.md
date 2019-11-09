# 服务器端

### 安装软件

```bash
pacman -Ss wireguard
# 搜索软件包
pacman -S wireguard-tools wireguard-dkms 
# 根据搜到的结果安装对应的工具
# wireguard-dkms 对应自定义内核, 比如 archlinux arm64
# wireguard-arch 对应默认的 linux 内核
# wireguard-lts 对应 linux-tls 内核
```
### 生成密匙

```bash
cd /etc/wireguard
# 生成私匙和公匙
wg genkey | tee Server-privatekey | wg pubkey > Server-publickey
# 生成共享密匙, 进一步加强保密, 这步骤是可选项目
wg genpsk > presharedkey
# 查看已经生成的各种密匙
cat Server-privatekey Server-publickey presharedkey
# 设置权限防止电脑上的其他用户获取你的密匙
chmod 600 Server-privatekey Server-publickey presharedkey
```

### 服务器配置

``` bash
touch wg0.conf
chmod 600 wg0.conf
vim wg0.conf # 编辑内容
# 内容为: 

[Interface]
# 设置 server 的 VPN IP 地址, 注意这个 **不是** 你的 server 的公网 ip, 可以自己定义
Address = 10.100.0.1/24
# 每接受一个连接都不保存配置
SaveConfig = false
# 服务器监听的端口
ListenPort = 55555
PrivateKey = 服务器的 Server-privatekey 里面的内容填入这里
# 开启自动网络地址转换, eth0 更换成对应的出口网卡地址, ip a 可以查看网卡名称
PostUp = iptables -t nat -I POSTROUTING -s 10.100.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 10.100.0.0/24 -o eth0 -j MASQUERADE

# 下面是定义客户端如何与服务器互联

[Peer]
# 客户端的公匙, 以 iOS 为例, 打开 iOS wireguard 客户端, 点右上角的 + , 然后 create from scratch
# 点 generate keypair, 将生成的 Public key 复制, 填入这里
PublicKey = xxxxx
PresharedKey = 前面定义好的 presharedkey 内容填入这里
# 定义客户端的 ip 地址, 可以随便写, 只要跟服务端的 VPN IP 在同一个网段
AllowedIPs = 10.100.0.9/32
```

### 启动服务端

```bash
# 手动启动, 测试用
wg-quick up wg0
# 手动停止
wg-quick down wg0
# systemd 启动和停止
systemctl start wg-quick@wg0.service
systemctl stop wg-quick@wg0.service
# 开机启动
systemctl enable wg-quick@wg0.service
```

# 客户端

> 以 iOS 客户端为例

1. 打开 `App Store`, 搜索 `wireguard`, 下载
1. 点击客户端右上角的 +, `Create from scratch`
1. 各个选项的意思:
    1. `Name`: 随便填, 我这里填 `HOME`
    1. `Private key` 和 `Public key`: 点 `Generate keypair` 生成, 注意生成完毕后, 需要将 `Public key` 更新到服务器的 `wg0.conf`, 切记
    1. `Address`, 这里填服务器那边配置好的 `10.100.0.9/32`
    1. `Listen Port`: 不用填
    1. `MTU`: 不用填
    1. `DNS Servers`, 这里填 `10.100.0.1`, 我们通过服务器做 DNS 解析
    1. 然后点 `Add Peer`
    1. `Public Key`: 把服务器那边生成的 `Server-publickey` 的内容填到这里
    1. `Preshared key`: 把服务器那边生成的 `presharedkey` 的内容填到这里
    1. `Endpoint`: 填入服务器的公网 ip (或者域名)和端口, `example.com:55555`
    1. `Allow IPs`: 这里的意思是允许哪些 ip 走 VPN 连接到服务器, 这里填 `0.0.0.0/0`, 表示所有都走 VPN
    1. `Persistent Keekalive`: 间隔多久检查一下连接状况, 这里填 `25` 就可以了, 代表 `25` 秒检查一次
    1. `ON-DEMAND ACTIVATION`, 这里把 `Wi-Fi` 和 `Cellular` 都选上, 表示任何网络下随时随地都连 `VPN`, 支持网络切换自动重连
1. 所有设置好了之后, 点连接, 便可以看到详细的连接信息

### 测试

1. 去服务端 `ping 10.100.0.9`, 看是否通
1. 用 `iPhone` 的 `Safari` 分别打开国内和国外网站, 看是否通.
