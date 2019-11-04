## /etc/nginx/nginx.conf

```ini

# main
worker_processes auto;
worker_cpu_affinity auto;
worker_rlimit_nofile 400000;

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
    sendfile on;
    charset utf-8;
    types_hash_max_size 4096;
    client_header_buffer_size 4k;
    client_max_body_size 0;
    client_body_buffer_size 128k;
    server_tokens off;
    resolver 1.1.1.1;
    resolver_timeout 10s;
    
    ## GZIP 压缩配置
    gzip on;                        # 开启 gzip 压缩功能
    gzip_min_length 1k;             # 允许进行压缩的最小字节数
    gzip_comp_level 6;              # 压缩比(1-9)，级别越高占用 cpu 资源越多
    gzip_proxied any;               # 无条件压缩后端服务器返回的结果
    gzip_vary on;                   # 启用 gzip 压缩标识 "Vary: Accept-Encoding"
    gzip_disable "MSIE [1-6]\.";    # 禁用 IE6 以下的 gzip 压缩功能
    # 要压缩的 MIME 文件类型，text/html 默认添加了，因此不能再指定 text/html
    gzip_types text/plain text/css text/javascript application/javascript application/x-javascript application/xml application/x-httpd-php image/jpeg image/png image/gif image/svg+xml font/ttf font/otf;

    # ssl 
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_buffer_size 4k;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_ciphers "TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES256-CCM8:ECDHE-ECDSA-AES256-CCM:ECDHE-ECDSA-AES128-CCM8:ECDHE-ECDSA-AES128-CCM:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA";
    ssl_prefer_server_ciphers on;

    # add header
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;

    # log
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # include
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
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # location
    location = /ray {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass              http://127.0.0.1:11111;
    }	
}

server {
    listen 80;
    server_name example.com;
    return 301 https://example.com$request_uri;
}

```

## TLS1.3 优先级

[source](https://github.com/openssl/openssl/issues/7562#issuecomment-461817236)

```bash
openssl version -a | grep OPENSSLDIR # 获取路径
vim /usr/lib/ssl/openssl.cnf
# 添加相应的位置
[system_default_sect]
MinProtocol = TLSv1.2
CipherString = DEFAULT@SECLEVEL=2
# 上面是默认的, 下面是我们要添加的
Ciphersuites = TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384

# CipherString 设置 tls 1.2 顺序
# Ciphersuites 设置 tls 1.3 顺序

systemctl restart nginx

```

