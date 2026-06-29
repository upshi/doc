
## v2ray
### 安装v2ray

[bash <(curl -L -s https://install.direct/go.sh) ](https://github.com/v2fly/fhs-install-v2ray/blob/master/README.zh-Hans-CN.md

安装可执行文件和 .dat 数据文件
```bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)```


只更新 .dat 数据文件
```bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)```


安装后的目录: /usr/local/etc/v2ray/

配置: /usr/local/etc/v2ray/config.json
```
{
  "inbounds": [{
    "port": 10000,
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "your id",
          "alterId": 64
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
         "path": "/ray"
      }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

### 操作
systemctl enable v2ray;

systemctl restart v2ray;


## HTTPS
### 准备好一个二级域名
wall.doaminname.com
### 安装HTTPS证书工具
curl  https://get.acme.sh | sh
### 安装工具
yum install socat
### 注册zerossl账号
~/.acme.sh/acme.sh --register-account  -m xxx@mail.com --server zerossl
### 安装HTTPS证书
~/.acme.sh/acme.sh --issue -d wall.doaminname.com --standalone -k ec-256
### 安装证书到v2ray目录下
~/.acme.sh/acme.sh --installcert -d wall.doaminname.com --fullchainpath /usr/local/etc/v2ray/v2ray.crt --keypath /usr/local/etc/v2ray/v2ray.key --ecc 


## 安装nginx
安装
yum install -y nginx

启动
service nginx start

### 配置nginx
/etc/nginx/nginx.conf
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
}
```
/etc/nginx/conf.d/v2ray-nginx.conf
```
server {
  listen  443 ssl http2;
  ssl_certificate       /usr/local/etc/v2ray/v2ray.crt;
  ssl_certificate_key   /usr/local/etc/v2ray/v2ray.key;
  ssl_protocols         TLSv1.2 TLSv1.3;
  ssl_ciphers           HIGH:!aNULL:!MD5;
  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout  30m;
  ssl_prefer_server_ciphers off;
  server_name           wall.doaminname.com;
        location /index {
          proxy_redirect off;
          proxy_pass http://127.0.0.1:10000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
          proxy_set_header Host $http_host;

          # Show realip in v2ray access.log
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
  location / {
    default_type text/html;
    return 200 'go go go';
  }
}
```

service v2ray restart

service nginx restart


### 注意事项
v2ray目录: /usr/local/etc/v2ray/
v2ray日志: /var/log/v2ray

nginx目录: /etc/nginx/
nginx日志: /var/log/nginx

2026/06/29 14:20:23 127.0.0.1:39162 rejected  common/drain: common/drain: drained connection > proxy/vmess/encoding: invalid user: VMessAEAD is enforced and a non VMessAEAD connection is received. You can still disable this security feature with environment variable v2ray.vmess.aead.forced = false . You will not be able to enable legacy header workaround in the future.
为了解决上述问题，修改以下文件  /etc/systemd/system/v2ray.service
增加配置:  Environment="V2RAY_VMESS_AEAD_FORCED=false" 

修改完后执行命令:
systemctl daemon-reload
service v2ray restart

