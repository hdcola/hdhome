# 小米机器人接入

## 获取token

* 用iTunes备份你的iPhone
* 使用iBackup Viewer 找到小米app（AppDomain-com.xiaomi.mihome），导出```*_mihome.sqlite```这样的一个文件
* 使用DB Browser for SQLite打开导出的sqlite文件
* 打开ZDEVICE表，从中查到你所使用的扫地机器人，得到对应iteam里的ZTOKEN字段内容
* 使用 ```echo '0: <你的原始ZTOKEN字符串>' | xxd -r -p | openssl enc -d -aes-128-ecb -nopad -nosalt -K 00000000000000000000000000000000```得到32位token

## 加入配置

在 configuration.yaml 中加入：

```
vacuum:
  - platform: xiaomi_miio
    host: 192.168.1.2
    token: YOUR_TOKEN
```

配置参数：

* host (Required): The IP of your robot.
* token (Required): The API token of your robot.
* name (Optional): The name of your robot.

## HomeKit的配置

### 开关扫地机器人

在 configuration.yaml 中加入：

```
switch:
  - platform: template
    switches:
      shitou:
        friendly_name: "石头"
        icon_template: mdi:robot-vacuum
        value_template: "{{ is_state('vacuum.xiaomi_vacuum_cleaner', 'on') }}"
        turn_on:
          service: vacuum.turn_on
          data:
            entity_id: vacuum.xiaomi_vacuum_cleaner
        turn_off:
          service: vacuum.turn_off
          data:
            entity_id: vacuum.xiaomi_vacuum_cleaner
```

这样就通过一个开关可以开启和停止石头机器人的扫地了。

### 用一个湿度传感器显示电量

在 configuration.yaml 中加入：
```
sensor:
  - platform: template
    sensors:
      shitoubattery_level:
        friendly_name: "石头电量"
        unit_of_measurement: "%"
        device_class: humidity
        value_template: '{{ states.vacuum.xiaomi_vacuum_cleaner.attributes.battery_level }}'
```
这样你就会发现一个石头电量的湿度传感器显示石头的电量了。
