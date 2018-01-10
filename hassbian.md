# Hassbian技巧作弊条

## 连接多个wifi的设置

如果你需要连接到多个wifi，可以通过在wpa_supplicant.conf中加入多个network来指定，比如

```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
        ssid="s1"
        psk="p1"
        key_mgmt=WPA-PSK
        priority=1
    }
network={
        ssid="s2"
        psk="p2"
        key_mgmt=WPA-PSK
        priority=10
    }
  ```

这里priority是优先级，哪个小先连接哪个。注意，这个文件放在sd卡根目录下，开机后会自动更新到/etc目录中去。


## 安装Node.js的不同版本

### Node.js v9.x:

```
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_9.x | bash -
apt-get install -y nodejs
```

### Node.js v8.x:

```
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_8.x | bash -
apt-get install -y nodejs
```

### Node.js v6.x:

```
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_6.x | bash -
apt-get install -y nodejs
```

## node.js的源设置

npm默认已经灰常慢了，用taobao的镜像吧

```
sudo npm config set registry https://registry.npm.taobao.org
```

使用

    npm info express

或

    npm config get registry

可以查看现在的源

## ssh的key认证设置

在自己的Mac上，打开终端，生成自己的私钥和公钥：

```
ssh-keygen -t rsa
```
这会在你的home目录中生成一个目录.ssh，这里有两个文件id_rsa是你的私钥，id_rsa.pub是你的公钥。

ssh到你的pi

```
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 644 ~/.ssh/authorized_keys
```

将你的id_rsa.pub文件中的内容复制到authorized_keys文件中去。接下来，配置pi上的sshd：

```
sudo vi  /etc/ssh/sshd_config
```

找到```PubkeyAuthentication yes```将前面的```#```去除。最后重启下sshd

```
sudo service ssh restart
```

## 反向SSH隧道

### 手工开洞

在pi上连接公网服务器，建立反向隧道。以下是用```r_user```连接到```1.1.1.1```，将远程的10022端口转发到localhost:22上，-R为反向转发，-fN为进入后台运行：

```
ssh -fN -R 10022:localhost:22 r_user@1.1.1.1
```

这样，你ssh到1.1.1.1，再ssh localhost的10022端口就回到你家里的pi了。

### 公网服务器打开绑定

如果你不想ssh到1.1.1.1就可以ssh回去，哪么在服务器上编辑```/etc/ssh/sshd_config```，加入以下内容：

```
GatewayPorts=yes
```

重启sshd后，你就可以直接ssh连接到服务器上的1022端口回到pi上了。可以通过这个命令来确认服务：

```
netstat -anp | grep 10022
```

### autossh保持连接

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

### 将autossh加入service

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


## 设置ssh ForwardAgent

### Mac客户端设置

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

### pi服务器设置

```
sudo vi /etc/ssh/ssh_config
```

在Host * 中加入

```
ForwardAgent yes
```
