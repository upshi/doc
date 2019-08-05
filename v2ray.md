
## v2ray
### 安装v2ray
bash <(curl -L -s https://install.direct/go.sh) 

安装后的目录: /etc/v2ray/

配置: /etc/v2ray/config.json
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
service v2ray start

service v2ray stop

service v2ray restart


## HTTPS
### 准备好一个二级域名
wall.doaminname.com
### 安装HTTPS证书工具
curl  https://get.acme.sh | sh
### 安装工具
yum install socat
### 安装HTTPS证书
~/.acme.sh/acme.sh --issue -d wall.doaminname.com --standalone -k ec-256
### 安装证书到v2ray目录下
~/.acme.sh/acme.sh --installcert -d wall.doaminname.com --fullchainpath /etc/v2ray/v2ray.crt --keypath /etc/v2ray/v2ray.key --ecc 


## 安装nginx
创建文件/etc/yum.repos.d/nginx.repo
加入以下内容

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

安装
yum install -y nginx

启动
service nginx start

### 配置nginx
/etc/nginx/conf.d/default.conf
```
server {
  listen  443 ssl;
  ssl on;
  ssl_certificate       /etc/v2ray/v2ray.crt;
  ssl_certificate_key   /etc/v2ray/v2ray.key;
  ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers           HIGH:!aNULL:!MD5;
  server_name           wall.doaminname.com;
        location /ray {
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


