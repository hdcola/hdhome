# Home Assistant & HomeBridge 安装作弊条

## 选择使用的方案

我在Mac OSX上安装过，也使用过HASS.IO版本，最终使用了Hassbian版本，有几个比较主要的原因：

* Mac用电较多，而且休眠有问题，摆放也不方便
* HASS.IO版本在国内用那是真的慢
* Hassbian版本没有图型界面、默认开启ssh，玩起来方便很多

## 准备软件

下载相关软件

* 下载[Hassbian image文件](https://github.com/home-assistant/pi-gen/releases/latest)
* 下载[Etcher](https://etcher.io/)写卡工具

解压Hassbian image文件后，使用Etcher将image写入SD卡。

## 网络准备

如果你是使用wifi，哪么将自己的wpa_supplicant.conf放入sd卡根目录，启动后会更新到/etc/wpa_supplicant.conf。

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
```
## 启动

启动后，可以使用 hassbian.local 连接到Hassbian。原则上来讲，只要ping通后，就可以ssh到系统了，ssh的用户是pi，密码是raspberry。

系统启动后，Hassbian会去下载和安装hass相关软件，安装并启动完成后，你可以看到``` /home/homeassistant/.homeassistant/ ```目录中有了文件就代表安装完成了。如果一直无法安装完成，你也可以手工重新安装

```
sudo systemctl start install_homeassistant.service
```

一切正常你可以通过 [http://hassbian.local:8123](http://hassbian.local:8123)来访问Hassbian。

## Hassbian apt更新源修改


先修改```/etc/apt/sources.list ```为国内的源

```
deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main non-free contrib
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ stretch main non-free contrib
deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main non-free contrib
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ stretch main non-free contrib
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main non-free contrib
deb-src http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main non-free contrib
deb http://mirrors.sohu.com/raspbian/raspbian/ stretch main contrib non-free rpi
deb-src http://mirrors.sohu.com/raspbian/raspbian/ stretch main contrib non-free rpi
```

然后

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get update
```

upgrade不是必须，有时间的时候再慢慢更新也不迟。

## 安装Node.js

```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## NPM更新源修改

```
sudo npm config set registry https://registry.npm.taobao.org
```
## 安装libAvahi

```
sudo apt-get -y install libavahi-compat-libdnssd-dev
```

## 安装HomeBridge

先安装HomeBridge

```
sudo npm install -g --unsafe-perm homebridge
```

再安装HomeBridge-HomeAssistant

```
sudo npm install -g homebridge-homeassistant
```

## 配置Homebridge启动脚本

编辑homebridge配置

```
sudo vi /etc/default/homebridge
```

内容为

```
# Defaults / Configuration options for homebridge
# The following settings tells homebridge where to find the config.json file and where to persist the data (i.e. pairing and others)
HOMEBRIDGE_OPTS=-U /home/homeassistant/.homebridge

# If you uncomment the following line, homebridge will log more
# You can display this via systemd's journalctl: journalctl -f -u homebridge
# DEBUG=*
```


编辑homebridge.service文件
```
sudo vi /etc/systemd/system/homebridge.service
```

内容为

```
[Unit]
Description=Homebridge
After=home-assistant@homeassistant.service

[Service]
Type=simple
User=homeassistant
EnvironmentFile=/etc/default/homebridge
ExecStart=/usr/bin/homebridge $HOMEBRIDGE_OPTS
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

启用homebridge.service

```
sudo systemctl daemon-reload
sudo systemctl enable homebridge.service
```
