## wifi设置

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

## hassbian的ssh

可以ssh的用户是pi，密码是raspberry。我的mac可以用 hassbian.local 连接过去，不用去路由器查ip（按道理win也应该可以）。

## hassbian的更新源

修改 /etc/apt/sources.list

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
感受下舒爽的更新吧

## 安装Node.js

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
