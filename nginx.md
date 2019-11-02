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
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header Referrer-Policy no-referrer;

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
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect          off;
        proxy_buffering         off;
        proxy_pass              http://127.0.0.1:11111;
    }	
}

server {
    listen 80;
    server_name example.com;
    return 301 https://example.com$request_uri;
}

```