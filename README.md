# CentOS安装配置Shadowsock

1. 首先安装python2.x
```
easy_install pip
pip install shadowsocks
vi /etc/shadowsocks.json
```

加入以下配置
```
{
    "server":"xxxxxx",
    "port_password": {
        "xxx": "xxx",
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

启动
```
ssserver -c /etc/shadowsocks.json -d start
```
