# Hass.io 安装作弊条

## 准备工作

* 去[https://www.raspberrypi.org/downloads/raspbian/]下载最新的 RASPBIAN STRETCH LITE 版
* 下载一个Etcher[https://etcher.io/]
* 使用Etcher将raspbian恢复到SD卡上

### 网络准备

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

### 开启ssh

在sd卡根目录建立一个文件名为```ssh```

### 启动

启动树莓派后，你可以使用```pi```用户ssh到```raspberrypi.local```，密码是 ```raspberry```。记得登录上去后修改密码哟。

## 初始化系统

### 改apt源

修改 ```/etc/apt/sources.list```为

```
deb http://mirrors.aliyun.com/raspbian/raspbian/  stretch main non-free contrib
deb-src http://mirrors.aliyun.com/raspbian/raspbian/  stretch main non-free contrib
```

### 支持https的apt仓库

```
sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
```

### 安装Docker

加入Docker的GPG key

```
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
```

添加Docker CE仓库

```
echo "deb [arch=armhf] https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list
```

安装Docker CE

```
sudo apt-get update
sudo apt-get install docker-ce
sudo usermod -aG docker $USER
```

增加Docker仓库镜像 ```/etc/docker/daemon.json```

```
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

### 更新源

更换pip源 ```/etc/pip.conf```

```
[global]
trusted-host=mirrors.aliyun.com
index-url=http://mirrors.aliyun.com/pypi/simple
```

更新下apt和源列表

```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade
```

### 初始化Hass.io安装环境

安装hassio所需要的基础支持

```
sudo apt-get install bash socat jq -y
```

拉取Docker UI界面镜像

```
docker pull portainer/portainer:latest
```

这时重启下系统会有益于身心健康。重启后，启动Docker的管理界面：

```
docker run -d -p 9000:9000 --name portainer --restart=always -e TZ="Asia/Shanghai" -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```

启动后，用浏览器连接到 http://raspberrypi.local:9000 ，设置管理员用户名和密码后，选择 ```Local Manage the local Docker environment```

拉取Homeassistant库 ```docker pull homeassistant/armhf-homeassistant:latest```
拉取Hass.io Supervisor库 ```docker pull homeassistant/armhf-hassio-supervisor:latest```

sudo到root  ```sudo su```

安装Hass.io

```
curl -sL https://raw.githubusercontent.com/home-assistant/hassio-build/master/install/hassio_install | bash -s -- -m raspberrypi3
```

可以通过

```
sudo journalctl -fu hassio-supervisor.service
```

查看hassio的运行状态

#### s3被墙怎么办

不知道为什么，我这里的 https://s3.amazonaws.com/hassio-version/stable.json 不能访问。自己想办法访问到这个文件，看下supervisor的版本号。然后将hassio_install中的HASSIO_VERSION设置为这个版本号就好。

## 配置Hass.io

安装好Hassio
