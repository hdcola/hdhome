# homebridge作弊条

## homebridge-config-ui-x

这是一个用下来感觉最好的一个管理界面了，可以在[这里](https://homebridge-ui-demo.oz.nu)使用admin用户名admin密码登录看看效果。

```
sudo npm install -g --unsafe-perm homebridge-config-ui-x
```

在config.json中加入

```
{
  "platform": "config",
  "name": "Config",
  "port": 8080,
  "sudo": true,
  "log": "systemd"
}
```

这里我们使用了sudo，所以在```/etc/sudoers.d/020_homeassistant_hassbian-scripts```最后一行加入

```
%homeassistant ALL=(ALL) NOPASSWD: ALL
```

这样做显然在安全性上还是有漏洞，晚些仔细研究下hb的多开和非root启动再修改这个吧，必竟我现在还是使用的内网。

通过8080端口可以访问到管理界面了，默认用户名和密码是admin/admin。
