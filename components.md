这里记录的都是homeassistant里有用的组件作弊条，活用这些组件，能带来很多奇妙的作用。

## Template 系列

这个系列组件有Template Switch、Template Binary Sensor、Template Cover、Template Light、Template Sensor。它的作用就是把现有的state或state_attributes变为你需要的组件。

举例，把一个开关变为灯：

```
light:
  - platform: template
    lights:
      living_room_light:
        friendly_name: living_room_light
        value_template: '{{ states.switch.living_room_light.state }}'
        turn_on:
          service: switch.turn_on
          entity_id: switch.living_room_light
        turn_off:
          service: switch.turn_off
          entity_id: switch.living_room_light
```

## saswell

有兄弟贡献了saswell的代码：[https://github.com/Yonsm/HAExtra/blob/master/custom_components/climate/saswell.py]

```
wget https://raw.githubusercontent.com/Yonsm/HAExtra/master/custom_components/climate/saswell.py
sudo mkdir -p /home/homeassistant/.homeassistant/custom_components/climate
sudo cp saswell.py /home/homeassistant/.homeassistant/
```

ha配置中加入

```
climate:
  - platform: saswell
    #name: Saswell
    username: username
    password: password
    devices: 1
    #scan_interval: 300
```

devices指定温控器的个数。
