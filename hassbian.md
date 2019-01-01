# Hassbian技巧作弊条

## 设置以太网为静态IP

```ls /sys/class/net/```

看看系统的以太网卡名字，在 /etc/dhcpcd.conf 中加入

```
interface ethxxxxxx
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8 8.8.4.4
```

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
## hass的常用信息

Configuration dir: ```/home/homeassistant/.homeassistant/```
Start service: ```sudo systemctl start home-assistant@homeassistant.service```
Stop service: ```sudo systemctl stop home-assistant@homeassistant.service```
Restart service: ```sudo systemctl restart home-assistant@homeassistant.service```
Service status: ```sudo systemctl status home-assistant@homeassistant.service```

## hass升级

升级homeassistant

```
sudo hassbian-config upgrade homeassistant
```

升级homeassistant中的软件包

进入homeassistant env：

```
sudo su -s /bin/bash homeassistant
source /srv/homeassistant/bin/activate
```

查看所有软件包

```
pip list
```

查看所有过期软件包

```
pip list --outdated
```

升级指定软件包

```
pip install --upgrade 库名
```

升级所有可升级的包

```
pip freeze --local | grep -v '^-e' | cut -d = -f 1  | xargs -n1 pip install -U
```

```
for i in `pip list -o --format legacy|awk '{print $1}'` ; do pip install --upgrade $i; done
```

## hassctl

hassctl是一个hass的简单操作工具

### 安装：

```
sudo curl -o /usr/local/bin/hassctl https://raw.githubusercontent.com/dale3h/hassctl/master/hassctl && sudo chmod +x /usr/local/bin/hassctl
```
