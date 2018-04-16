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
