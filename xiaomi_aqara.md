#米家Aqara ZigBee网关接入

## 接入准备

* 在米家 app 中与网关配对
* 进入网关页，点选右上角“……” —— 关于
* 在下面的空白处点多下
* 局域网通信协议 —— 打开并获取key

iOS 客户端获取的密码为大写，安卓客户端获取的密码为小写，配置时请保留大小写，原样照搬。

## 配置ha

单个网关

```
# 使用单网关前提下，可不填 mac
xiaomi_aqara:
  gateways:
    - mac:
      key: xxxxxxxxxxxxxxxx
```

多个网关

```
# 多个网关必须填入 mac
xiaomi_aqara:
  gateways:
    - mac: xxxxxxxxxxxx
      key: xxxxxxxxxxxxxxxx
    - mac: xxxxxxxxxxxx
      key: xxxxxxxxxxxxxxxx
```

配置变量说明

* mac (可选): 网关 mac 地址，使用多个网关则必须填写
* key (可选): 网关通信协议密码。如果想要控制网关灯和开关，则必须填写；传感器在无密码情况下仍可正常运作。
* discovery_retry (可选): 连接失败重试次数，默认为 3。
* interface (可选): 所使用的接口，默认为全部(all）。
