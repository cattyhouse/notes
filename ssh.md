# ~/.ssh/config

```ini
#!/bin/bash
# 全局设置
Host *
# 定义加密方式优先级
Ciphers aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
# 开启压缩
Compression yes
# 关闭 tcp keep alive
TCPKeepAlive no
# 定义连接超时时间
ConnectTimeout 10
# 重连次数
ConnectionAttempts 2
# 发送保持连接代码的次数
ServerAliveCountMax 2
# 发送保持连接代码的间隔, 2x15=30, 表示30秒如果没有收到回应, ssh 连接已经 dead
ServerAliveInterval 15
# 不记录服务器连接记录
UserKnownHostsFile /dev/null
# 用hash加密服务器连接记录
HashKnownHosts yes
# 只用 rsa 连接, 其它方式弃用
IdentitiesOnly yes
# 
StrictHostKeyChecking no
# 不要 escape 任何字符
EscapeChar none
# 服务器端需要设置 #AcceptEnv LANG 才可以正常显示中文, 针对tmux需要这样设置
SendEnv LANG LC_*
LogLevel ERROR

Host xxx
#   用代理访问 xxx 
#   Using relay, 通常这个 relay 速度比较快
    ProxyCommand ssh -q -W %h:%p relay
#	Using netcat-bsd with shadowsocks proxy
    ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p

Host relay
User root
IdentityFile ~/.ssh/id_rsa
Port 22
Hostname example1.com

Host xxx
IdentityFile ~/.ssh/id_rsa
Hostname example2.com
User root
Port 22
```

# 让ssh安静点

`touch ~/.hushlogin` 

[参考](https://debian-administration.org/article/546/Giving_yourself_a_quieter_SSH_login)