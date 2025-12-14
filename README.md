# 3-Speed Fan Convert

Watch the full tutorial here: https://youtu.be/vpCJXOjK4bU

Here is the ESPHome code you need to convert a 3-Speed Desk Fan using a #Sonoff Basic. 
```yaml
esphome:
  name: ${name}
  platform: ESP8266
  board: esp01_1m

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true

substitutions:
  name: small_fan

logger:
api:
ota:
  platform: esphom

binary_sensor:
  - platform: homeassistant
    entity_id: input_boolean.${name}
    id: ${name}
    on_press:
      then:
        switch.turn_on: relay1
    on_release:
          then:
            - switch.turn_off: relay1
            - switch.turn_off: relay2
            - switch.turn_off: relay3

sensor:
  - platform: homeassistant
    entity_id: sensor.thermostat
    id: thermo
    
  - platform: homeassistant
    entity_id: sensor.bedroom_temperature
    id: temp
    on_value:
      - if:
          condition:
            and:
              - binary_sensor.is_on: ${name}
              - lambda: 'return id(temp).state > (id(thermo).state + 0.0);'
              - lambda: 'return id(temp).state < (id(thermo).state + 3.0);'
          then:
            - switch.turn_on: relay1
      - if:
          condition:
            and:
              - binary_sensor.is_on: ${name}
              - lambda: 'return id(temp).state > (id(thermo).state + 3.1);'
              - lambda: 'return id(temp).state < (id(thermo).state + 5.0);'
          then:
            - switch.turn_on: relay2
      - if:
          condition:
            and:
              - binary_sensor.is_on: ${name}
              - lambda: "return id(temp).state > (id(thermo).state + 5.1);"
          then:
            - switch.turn_on: relay3
      - if:
          condition:
            or:
              - binary_sensor.is_off: ${name}
              - lambda: "return id(temp).state < (id(thermo).state - 0.5);"
          then:
            - switch.turn_off: relay1
            - switch.turn_off: relay2
            - switch.turn_off: relay3

switch:
  - platform: gpio
    name: "${name} 1"
    pin: GPIO12
    icon: mdi:fan
    id: relay1
    interlock: &interlock_group [relay1, relay2, relay3]
    interlock_wait_time: 300ms
  - platform: gpio
    name: "${name} 2"
    pin: GPIO1
    inverted: true
    icon: mdi:fan
    id: relay2
    interlock: *interlock_group
    interlock_wait_time: 300ms
  - platform: gpio
    name: "${name} 3"
    pin: GPIO3
    inverted: true
    icon: mdi:fan
    id: relay3 
    interlock: *interlock_group
    interlock_wait_time: 300ms
    
status_led:
  pin:
    number: GPIO13
    inverted: yes
```

Below is the 'custom Sensor' needed to get a thermostats attribute in to ESPHome

```yaml
- platform: template
  sensors:
    thermostat:
      value_template: "{{ state_attr('climate.bedroom_cooling', 'temperature') }}"
```

---

<div align="center">

### 💖 Support This Project

Found this useful? Want to say thanks and fuel future creations?

**Your support keeps the creativity flowing!** 🍺✨

<table>
  <tr>
    <td align="center">
      <a href="https://www.buymeacoffee.com/3ative">
        <img src="https://img.shields.io/badge/Buy%20Me%20A%20Coffee-Support-yellow.svg?style=for-the-badge&logo=buy-me-a-coffee&logoColor=white" alt="Buy Me A Coffee"/>
        <br/>
        <b>☕ Buy me a Coffee</b>
      </a>
    </td>
    <td align="center">
      <a href="https://www.patreon.com/3ative">
        <img src="https://img.shields.io/badge/Patreon-Become%20a%20Patron-red.svg?style=for-the-badge&logo=patreon&logoColor=white" alt="Patreon"/>
        <br/>
        <b>💖 Join on Patreon</b>
      </a>
    </td>
  </tr>
</table>

**Every contribution helps bring more awesome projects to life!** 🚀

</div>

---
