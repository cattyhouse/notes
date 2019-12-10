## /etc/nginx/nginx.conf

```ini
# main
worker_processes auto;
worker_cpu_affinity auto;
worker_rlimit_nofile 400000;
#pid /run/nginx.pid;
# include
include /etc/nginx/modules-enabled/*.conf;

# events
events {
    use epoll;
    worker_connections 65535;
    multi_accept on;
}

# http
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s; # rate limit 设置限制, 可以在 server, location 下面启用限制
    # refer: https://www.nginx.com/blog/rate-limiting-nginx/
    default_type application/octet-stream; # 不在 mime.types 里面的文件以二进制处理
    sendfile on;
    charset utf-8;
    types_hash_max_size 4096;
    client_header_buffer_size 4k;
    client_max_body_size 0;
    client_body_buffer_size 128k;
    server_tokens off;
    resolver 1.1.1.1 1.0.0.1;
    resolver_timeout 10s;

    ## adddtional
    aio on;
    directio 512;
    output_buffers 4 128k;
    client_body_timeout 10s;
    client_header_timeout 10s;
    keepalive_timeout 20s 20s;
    keepalive_requests 100;

    ## GZIP 压缩配置
    gzip on;                        # 开启 gzip 压缩功能
    gzip_min_length 50;             # 允许进行压缩的最小字节数
    gzip_comp_level 9;              # 压缩比(1-9)，级别越高占用 cpu 资源越多
    gzip_proxied any;               # 无条件压缩后端服务器返回的结果
    gzip_vary on;                   # 启用 gzip 压缩标识 "Vary: Accept-Encoding"
    gzip_disable "MSIE [1-6]\.";    # 禁用 IE6 以下的 gzip 压缩功能
    gzip_types text/xml text/rtf text/plain text/css text/javascript application/javascript application/ecmascript application/xml application/json image/gif image/svg+xml;
    # ssl 
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_buffer_size 4k;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_ciphers "TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES256-CCM8:ECDHE-ECDSA-AES256-CCM:ECDHE-ECDSA-AES128-CCM8:ECDHE-ECDSA-AES128-CCM:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA";
    ssl_prefer_server_ciphers on;
    ssl_early_data on; # 开启 tls 1.3 的 0-RTT
    # add header
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header Referrer-Policy no-referrer;
    add_header X-Robots-Tag "noindex, nofollow, nosnippet, noarchive"; # 禁止搜索引擎
    # log
    #access_log /var/log/nginx/access.log;
    access_log off;
    error_log /var/log/nginx/error.log;
    include /etc/nginx/mime.types;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

## /etc/nginx/site-available/example.com

```ini
server {
    root /var/www/html;
    index index.html index.htm;
    server_name example.com;
    listen 443 ssl http2 reuseport;
    # certs
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    # proxy
    proxy_http_version 1.1;
    proxy_connect_timeout 5s;
    proxy_read_timeout 3650d;
    proxy_send_timeout 3650d;
    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Early-Data $ssl_early_data; # 开启 tls 1.3 的 0-RTT
    #proxy_redirect          off;
    #proxy_buffering         off;

    # location
    location = /path {
        # = 表示精确匹配, 匹配完毕, 立即执行, 不再继续匹配
        proxy_pass              http://127.0.0.1:11111;
    }

    location /kernel {
        limit_req zone=mylimit; # 启用限制
        # 文件服务器, 打开 example.com/kernel 会访问 /var/www/html/kernel 下面的文件
        autoindex on; # 索引
        autoindex_exact_size off; # 显示文件大小
        autoindex_localtime on; # 显示文件时间
    }
    location / {
        # 网站镜像
        # 此时 root /var/www/html; 的内容不会再显示
        limit_req zone=mylimit; # 启用限制
        sub_filter mirror_domain.tld example.com;
        sub_filter_once off;
        sub_filter_types *;
        proxy_set_header Referer http://mirror_domain.tld;
        proxy_set_header Host mirror_domain.tld;
        proxy_pass http://mirror_domain.tld;
    }
}
server {
    listen 80;
    server_name example.com;
    return 301 https://example.com$request_uri;
}
```

```bash
# 建立软连接
sudo mkdir -p /etc/nginx/{site-available,sites-enabled}
ln -sf /etc/nginx/site-available/example.com /etc/nginx/sites-enabled/example.com
```

## TLS1.3 优先级

[source](https://github.com/openssl/openssl/issues/7562#issuecomment-461817236)

```bash
openssl version -a | grep OPENSSLDIR # 获取路径
vim /usr/lib/ssl/openssl.cnf
# 行首插入, 如果没有 openssl_conf 字段的话
openssl_conf = default_conf
# 行尾插入, 如果没有 default_conf, ssl_sect, system_default_sect 字段的话
[default_conf]
ssl_conf = ssl_sect
[ssl_sect]
system_default = system_default_sect
[system_default_sect]
## 如果有以上字段, 只需插入最下面这一行
Ciphersuites = TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256

systemctl restart nginx
```

## Certbot 证书

```bash
# 获取免费的证书
# standalone 方法, 运行前不能有 80 端口监听
certbot certonly -d example.com --standalone --agree-tos -m email_address
# webroot 方法, 必须有 80 端口监听, 以 nginx 为例, server block 里面必须有这句话:
location ^~ /.well-known/acme-challenge/ {
    root /var/www;
}
location = /.well-known/acme-challenge/ {
    return 404;
}
# webroot 的方法虽然不用暂停 nginx, 但是需要在证书更新后, 需要 reload nginx 以及其他使用此证书的服务.
# 从方便性角度讲, --standalone 是最方便的.
```

```bash
# 设置证书更新的时候的 hook
vim /etc/letsencrypt/cli.ini
# 内容
max-log-backups = 0
pre-hook = systemctl stop nginx.service # 更新前停止 nginx, 因为 certbot --standalone 模式需要临时监听 80
post-hook = systemctl start nginx.service # 更新后启动 nginx
```

```
# 确保 证书更新的 timer 已经启用
systemctl is-enabled certbot.timer | grep -q enabled && echo 'yes' || echo 'please run: systemctl enable certbot.timer'
```