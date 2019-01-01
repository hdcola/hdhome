# Hack作弊条

## 私有网络作弊条

一直希望有一个好的方法来回到每台设备，用过反向ssh、frp，都感觉不好，最近看到了ZeroTier这个东东，感觉还不错，下面有三个作弊条都可以使用。

### ZeroTier

### 安装

* 去 ```https://my.zerotier.com/network``` 得到你的Network ID（注册什么的我就不多说了）
* 在pi上安装 ```curl -s https://install.zerotier.com/ | sudo bash```
* 加入系统自动启动 ```sudo systemctl enable zerotier-one```
* 使用 ```sudo zerotier-cli status``` 可以看到200 ONLINE状态
* 使用 ```sudo zerotier-cli join [Network ID]``` 加入网络，会看到 200 OK的反馈
* 去 ```https://my.zerotier.com/network/[Network ID]``` 在新的join请求前打钩
* 可以用 ```sudo zerotier-cli listnetworks``` 看到连接到的网络状态

### 反向SSH隧道

#### 手工开洞

在pi上连接公网服务器，建立反向隧道。以下是用```r_user```连接到```1.1.1.1```，将远程的10022端口转发到localhost:22上，-R为反向转发，-fN为进入后台运行：

```
ssh -fN -R 10022:localhost:22 r_user@1.1.1.1
```

这样，你ssh到1.1.1.1，再ssh localhost的10022端口就回到你家里的pi了。

#### 公网服务器打开绑定

如果你不想ssh到1.1.1.1就可以ssh回去，哪么在服务器上编辑```/etc/ssh/sshd_config```，加入以下内容：

```
GatewayPorts=yes
```

重启sshd后，你就可以直接ssh连接到服务器上的1022端口回到pi上了。可以通过这个命令来确认服务：

```
netstat -anp | grep 10022
```

#### autossh保持连接

在pi上使用ssh连接有时会中断，使用autossh能保持连接的持续。先安装autossh：

```
sudo apt-get install autossh
```

手工启动ssh

```
autossh -M 10900 -fN -o "PubkeyAuthentication=yes" -o "StrictHostKeyChecking=false" -o "PasswordAuthentication=no" -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3" -R 1.1.1.1:10022:localhost:22 relayserver_user@1.1.1.1
```

“-M 10900” 选项指定中继服务器上的监视端口，用于交换监视 SSH 会话的测试数据。中继服务器上的其它程序不能使用这个端口。

“-fN” 选项传递给 ssh 命令，让 SSH 隧道在后台运行。

“-o XXXX” 选项让 ssh：

* 使用密钥验证，而不是密码验证。
* 自动接受（未知）SSH 主机密钥。
* 每 60 秒交换 keep-alive 消息。
* 没有收到任何响应时最多发送 3 条 keep-alive 消息。
其余 SSH 隧道相关的选项和之前介绍的一样。

#### 将autossh加入service

在/etc/default/autossh中加入以下内容：

```
AUTOSSH_OPTS=-M 10900 -o "PubkeyAuthentication=yes" -o "StrictHostKeyChecking=false" -o "PasswordAuthentication=no" -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3"
```
建立/etc/systemd/system/autossh.service：

```
[Unit]
Description=Auto SSH Tunnel
After=network-online.target

[Service]
User=pi
Type=simple
EnvironmentFile=/etc/default/autossh
ExecStart=/usr/bin/autossh $AUTOSSH_OPTS -NR 1.1.1.1:10022:localhost:22 relayserver_user@1.1.1.1 >> /dev/null 2>&1
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -TERM $MAINPID
KillMode=process
Restart=no

[Install]
WantedBy=multi-user.target
WantedBy=graphical.target
```


### 设置ssh ForwardAgent

#### Mac客户端设置

```
sudo vi /etc/ssh/ssh_config
```

在Host * 中加入

```
ForwardAgent yes
```

将ssh key加入Keychain

```
ssh-add .ssh/id_rsa
ssh-add -L
```

#### pi服务器设置

```
sudo vi /etc/ssh/ssh_config
```

在Host * 中加入

```
ForwardAgent yes
```

## frp反向代理

### 服务器设置

```
wget https://github.com/fatedier/frp/releases/download/v0.18.0/frp_0.18.0_linux_amd64.tar.gz
tar xzvf frp_0.18.0_linux_amd64.tar.gz
sudo cp /frp_0.18.0_linux_amd64/frps /usr/local/bin/
sudo vi /etc/frp/frps.ini
```
frps.ini中

```
[common]
bind_port = 7000
vhost_http_port = 7080
dashboard_port = dashboard_port_number
dashboard_user = dashboard_user_name
dashboard_pwd = dashboard_pwd_value
privilege_token = privilege_token_value
```

* bind_port:服务器por
* vhost_http_port:虚拟服务器http服务映射端口
* dashboard_port:frp控制台端口
* dashboard_user:frp控制台用户名
* dashboard_pwd:frp控制台密码
* privilege_token:自己的token，当frps和frpc token相同时才会进行通讯

```
sudo vi /etc/systemd/system/frps.service
```

里面为：

```
[Unit]
Description=frps daemon
After=syslog.target  network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.ini
ExecStop=/bin/kill $MAINPID
TimeoutStartSec=30

[Install]
WantedBy=multi-user.target
```

加载和启用服务

```
sudo systemctl daemon-reload
sudo systemctl enable frps
sudo systemctl start frps
```

### pi客户端设置

```
wget https://github.com/fatedier/frp/releases/download/v0.18.0/frp_0.18.0_linux_arm.tar.gz
tar xzvf frp_0.18.0_linux_arm.tar.gz
sudo cp frp_0.18.0_linux_arm/frpc /usr/local/bin/
sudo vi /etc/frp/frpc.ini
```
frpc.ini中如下

```
[common]
server_addr = your_server_ip
server_port = 7000
privilege_token = privilege_token_value
login_fail_exit = false

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = remote_port_number

[haweb]
type = http
local_port = 8123
custom_domains = server_domain
```

* login_fail_exit:连接失败是否退出，如果为 false，则启动时连接服务器失败将不会退出，而是定期重连。
* remote_port_number:指定通过远程服务器的哪个端口来 ssh
* custom_domains:访问服务器的域名

```
sudo vi /etc/systemd/system/frpc.service
```

里面为：

```
[Unit]
Description=frpc daemon
After=syslog.target  network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/frpc -c /etc/frp/frpc.ini
ExecStop=/bin/kill $MAINPID
TimeoutStartSec=30

[Install]
WantedBy=multi-user.target
```
加载和启动服务

```
sudo systemctl daemon-reload
sudo systemctl start frpc
```

当frpc启动的时候，你就可以通过server上的remote_port_number连回到pi上了。你也可以使用http://custom_domains:vhost_http_port来访问家里的8123上的ha web管理界面了。为了配合后面的远程协助，不建议enable frpc。

### 安全内网连接

由于我使用frp都是为了远程协助（帮助我的朋友们维护和升级pi），所以说明一下如何使用ha和frp完成一个“远程协助”。

#### 设置PI

* frpc.ini

```
[common]
server_addr = your_server_ip
server_port = 7000
privilege_token = your_token
login_fail_exit = false

[haweb_djy]
type = stcp
sk = huangdong@djy
local_ip = 127.0.0.1
local_port = 8123

[ssh_djy]
type = stcp
sk = your_secretkey
local_ip = 127.0.0.1
local_port = 22

[hbweb_djy]
type = stcp
sk = your_secretkey
local_ip = 127.0.0.1
local_port = 8080

[cloud9_djy]
type = stcp
sk = your_secretkey
local_ip = 127.0.0.1
local_port = 8181
```

* ha的configuration.yaml

```
switch:
  - platform: command_line
    switches:
      remote_helper:
        command_on: "sudo systemctl start frpc"
        command_off: "sudo systemctl stop frpc"
        command_state: "systemctl is-active frpc.service"
        value_template: '{{ value == "active" }}'
        friendly_name: "远程协助"
```

如果你开了homekit，哪么你的家庭中就会多出一个远程协助的按钮。每当打开开关，远程协助开启，关闭谁都访问不进来了。

#### 管理机

如果从互联网上远程协助，如下配置：

* frpc.ini

```
[common]
server_addr = your_server_ip
server_port = 7000
privilege_token = your_token

[ssh_visitor]
type = stcp
role = visitor
server_name = ssh_djy
sk = your_secretkey
bind_addr = 127.0.0.1
bind_port = 8022

[haweb_visitor]
type = stcp
role = visitor
server_name = haweb_djy
sk = your_secretkey
bind_addr = 127.0.0.1
bind_port = 8123

[hbweb_visitor]
type = stcp
role = visitor
server_name = hbweb_djy
sk = your_secretkey
bind_addr = 127.0.0.1
bind_port = 8080

[cloud9_visitor]
type = stcp
role = visitor
server_name = cloud9_djy
sk = your_secretkey
bind_addr = 127.0.0.1
bind_port = 8181
```
启动frpc后，就可以通过localhost连到ssh(8022)、ha(8123)、hb(8080)、cloud9(8181)


## mitmproxy作弊条

安装容易

```
brew install mitmproxy
```

启动容易

```
mitmproxy
```

加证书容易，把你的代理设置为启动mitmproxy的机器，端口默认是8080。访问 ```mitm.it```，按图示点平台类型点下去就好。然后装好根证书就成了。
