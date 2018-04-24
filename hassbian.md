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

## nodejs相关

### 查找node_modules目录

```
npm -g root
```

### 将nodejs与npm升级到最新版本

```
sudo npm install -g n
```

## 单用户模式

当你玩花系统使得无法恢复时，你可以这样做，取出sd卡，编辑sd卡根目录下的cmdline.txt文件将原来的

```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=c2009fe5-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
```

后面加入 init=/bin/sh

```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=c2009fe5-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait init=/bin/sh
```

将sd卡放回树莓派，加上显示器和键盘启动后

```
mount -rw -o remount /
```

去做你想做的事吧，最后(这里有一步忘记了，回头再查)

```
sync
exec /sbin/init
```

启动一切正常后，halt系统。把cmdline.txt的```init=/bin/sh```去除即可。

## 3.5寸LCD触摸屏

### 安装驱动

```
git clone https://github.com/hdcola/LCD-show.git
cd LCD-show
chmod +x *
sudo ./LCD35-show
```
如果一切都ok，你就可以用```sudo reboot```重启就ok了。

如果想切回HDMI

```
sudo ./LCD-hdmi
```

### 旋转屏幕

#### 旋转显示

##### 适用于 GPIO 接口型 LCD(2.4 寸,2.8 寸,3.2 寸,3.5 寸)

为了让电源冲上，立着看屏幕需要设置旋转

```
sudo vi /boot/config.txt
```

找到dtoverlay=tft35a，改为

```
dtoverlay=tft35a:rotate=270
```

270度是电源冲上的，到底是0、90、180合适，自己reboot后观察效果决定吧。

##### 适用于HDMI接口型LCD

```
sudo nano /boot/config.txt
```

找到```display_rotate```改为

```
display_rotate=0
```

这里0为0度、1为90度、2为180度、3为270度、0x10000为水平翻转、0x20000为垂真翻转。

#### 旋转触摸屏

```
sudo vi /etc/X11/xorg.conf.d/99-calibration.conf
```

默认的配置是这样（旋转0度，display_rotate=0）：

```
Section "InputClass"
  Identifier "calibration"
  MatchProduct "ADS7846 Touchscreen"
  Option "Calibration" "140 3951 261 3998 "
  Option "SwapAxes" "0"
EndSection
```

如果旋转90度，display_rotate=1，则更改为

```
Section "InputClass"
  Identifier "calibration"
  MatchProduct "ADS7846 Touchscreen"
  Option "Calibration" "261 3998 3951 140"
  Option "SwapAxes" "1"
EndSection
```

如果旋转180度，display_rotate=2，则更改为

```
Section "InputClass"
  Identifier "calibration"
  MatchProduct "ADS7846 Touchscreen"
  Option "Calibration" "3951 140 3998 261"
  Option "SwapAxes" "0"
EndSection
```

如果旋转270度，display_rotate=3，则更改为

```
Section "InputClass"
  Identifier "calibration"
  MatchProduct "ADS7846 Touchscreen"
  Option "Calibration" "3998 261 140 3951"
  Option "SwapAxes" "1"
EndSection
```

如此麻烦的东西，反正我是没有想改的心了。

## 桌面

安装

```
sudo apt-get install --no-install-recommends xserver-xorg
sudo apt-get install --no-install-recommends xinit
sudo apt-get install --no-install-recommends raspberrypi-ui-mods lxterminal gvfs
```

加入默认登录

```
sudo vi /etc/lightdm/lightdm.conf
```

找到

```
#autologin-user=
```

改为

```
autologin-user=pi
```
