# 米家Aqara ZigBee网关接入HomeAssistant

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

## 删除小米网关

你会发现，在configuration.yaml中注释xiaomi_aqara是没有用的，启动ha后还会出现相应的设备，名称后只是多了一个_2，不能做任何控制。斛的方法就是在```discovery: ```前加一个注释好了

# 米家Aqara ZigBee网关接入HomeBridge

## 安装hb插件

```
sudo npm install -g homebridge-mi-aqara
```

## config.json配置

这是一个config.json的配置示例，在gateways中成对的输入你所有的小米网关的mac和key。之后可以使用defaultValue来个性化小米网关下的每台设置的配置。

```
{
  "platform":"MiAqaraPlatform",
  "gateways":{
    "mac" : "key"
  },
  "defaultValue": {
    "158d0001f3f96f": {
      "Global": {
        "serviceType": "Lightbulb"
      },
      "DuplexSwitch_Switch_Left": {
        "name": "侧灯"
      },
      "DuplexSwitch_Switch_Right": {
        "name": "顶灯"
      }
    },
    "158d00020119fb":{
      "TemperatureAndHumiditySensor_TemperatureSensor":{
        "name": "温度"
      },
      "TemperatureAndHumiditySensor_HumiditySensor":{
        "name": "湿度"
      }
    },
    "7811dcb24eed":{
      "Gateway_LightSensor":{
        "name": "亮度"
      },
      "Gateway_Switch_JoinPermission":{
        "name": "允许连接"
      },
      "Gateway_Lightbulb":{
        "name": "夜灯"
      }
    }
  }
}
```

## 小米无线开关二代

### HomeBridge

homebridge-mi-aqara v0.6.8 支持无线开关，但是对新的带摇一摇的无线开关无法识别。我在米家中发现这种新的无线开关的version是aq3。通过网友 ```猿•いずみ``` 的提示把方法记录一下。

```
cd /usr/lib/node_modules/homebridge-mi-aqara/lib
vi ParseUtil.js
```

找到

```
'sensor_switch.aq2': new Button2Parser(platform), // 按钮 第二代
```

在它下面加入一行

```
'sensor_switch.aq3': new Button2Parser(platform), // 按钮 第2.5代
```

重启homebridge就能识别出无线开关了。

无线开关的配置：

```
"158d0001b96e7c":{
  "Button2_StatelessProgrammableSwitch":{
    "name": "无线开关"
  },
  "Button2_Switch_VirtualSinglePress":{
    "name": "单击无线开关"
  },
  "Button2_Switch_VirtualDoublePress":{
    "name": "双击无线开关"
  }
},
```

这个配置将会在你的苹果家庭中出现三个组件：一个是叫无线开关的可设置按一下、连按两下、长按操作的可编程开关。以及一个叫单击无线开关、一个叫双击无线开关的虚拟开关。

### HomeAssistant

HomeAssistant的homekit不支持可编程开会。想了半天用了一个笨办法，用input_boolean虚拟出一个开关，然后用事件来切换这个开关，这样就可以在家庭中用这个开关的状态做自己想做的自动化了。

先做两个虚拟开关（configuration.yaml）：

```
input_boolean:
  single_click:
    name: 单击无线开关
    initial: off
    icon: mdi:car
  double_click:
    name: 双击无线开关
    initial: off
    icon: mdi:car
```

再加两个automation（automations.yaml）：


```
- alias: single_click
  trigger:
    platform: event
    event_type: click
    event_data:
      entity_id: binary_sensor.switch_158d0001b96e7c
      click_type: single
  action:
    service: input_boolean.toggle
    entity_id: input_boolean.single_click


- alias: double_click
  trigger:
    platform: event
    event_type: click
    event_data:
      entity_id: binary_sensor.switch_158d0001b96e7c
      click_type: double
  action:
    service: input_boolean.toggle
    entity_id: input_boolean.double_click
```

收工！
