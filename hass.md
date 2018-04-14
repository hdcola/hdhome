
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
