# frp

##### linux 下载frp

```
wget https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_linux_amd64.tar.gz
```

##### 解压

```
tar -zxvf frp_0.20.0_linux_amd64.tar.gz
```

##### 删除

```
rm -f frpc
rm -f frpc.ini
```

##### 编辑frps.ini

```
vi frps.ini
```

一、服务端

```
[common]
bind_port = 7000
vhost_http_port = 80
dashboard_port = 7500
dashboard_user = xxx
dashboard_pwd = xxxxx
token = xxx
max_pool_count = 10
log_file = ./frps.log
log_level = info
log_max_days = 3
```

启动，进入相应的文件夹

```
nohup ./frps -c frps.ini >/dev/null 2>&1 &
```

二、客户端

```
[common]
server_addr = xxxx
server_port = xxxx
token = xxxx
user = xxx

[web]
type = tcp
local_ip = 192.168.2.1
local_port = xxxx
remote_port = xxxx
use_encryption = true
use_compression = true

[aria]
type = tcp
local_ip = 192.168.2.1
local_port = xxxx
remote_port = xxxx
use_encryption = true
use_compression = true

[transmission]
type = tcp
local_ip = 192.168.2.1
local_port = xxxx
remote_port = xxxx
use_encryption = true
use_compression = true

[cloud]
type = tcp
local_ip = 192.168.2.1
local_port = xxxx
remote_port = xxxx
use_encryption = true
use_compression = true
```

#### 开机自启动

```
vi /etc/systemd/system/frpc.service



[Unit]
Description=frpc service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
#启动服务的命令（frpc的实际安装目录）
ExecStart=/usr/local/bin/frpc -c /etc/frpc/frpc.ini

[Install]
WantedBy=multi-user.target
```

#### 启动命令

```
systemctl enable frpc
systemctl start frpc
systemctl status frpc


如果要重启应用，可以这样，systemctl restart frpc
如果要停止应用，可以输入，systemctl stop frpc
如果要查看应用的日志，可以输入，systemctl status frpc
```
