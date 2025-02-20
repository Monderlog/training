### [Home Assistant. Урок 8.4 – Виртуализация. Изучаем возможности платформы template, практика](https://youtu.be/7jbtovItjVQ)

#### Код из урока в текстовом виде - 

```yaml
unit_8_4:

    binary_sensor:
      - platform: template
        sensors:

          room_motion:
            friendly_name: "Движение в комнате"
            device_class: motion
            value_template: >-
              {{ is_state('binary_sensor.0x00158d0001e547a3_occupancy', 'on') }}
            icon_template: >-
              {% if is_state('binary_sensor.room_motion', 'on') %}
                mdi:motion-sensor
              {% else %}
                mdi:motion-sensor-off
              {% endif %}  

          flat_motion:
            friendly_name: "Движение в квартире"
            device_class: motion
            value_template: >-
              {{ is_state('binary_sensor.0x00158d0001e547a3_occupancy', 'on') 
              or is_state('binary_sensor.0x00158d00016d56f5_occupancy', 'on')  
              or is_state('binary_sensor.0x00158d0001a66222_occupancy', 'on')  }}
            icon_template: >-
              {% if is_state('binary_sensor.flat_motion', 'on') %}
                mdi:account-group
              {% else %}
                mdi:account-group-outline
              {% endif %}
              
          flat_windows:
            friendly_name: "Окна в квартире"
            device_class: window
            value_template: >-
              {{ is_state('binary_sensor.0x00158d000445206b_contact', 'on') 
              or is_state('binary_sensor.0x00158d00013ed373_contact', 'on')  
              or is_state('binary_sensor.0xec1bbdfffedf6a6a_contact', 'on')  }}
            icon_template: >-
              {% if is_state('binary_sensor.flat_windows', 'on') %}
                mdi:window-open-variant
              {% else %}
                mdi:window-closed-variant
              {% endif %}

          room_occupancy:
            friendly_name: "Присутствие в комнате"
            device_class: occupancy
            value_template: >-
              {{ is_state('binary_sensor.0x00158d0001e547a3_occupancy', 'on') 
              or is_state('light.yeelight_ceiling4_0x00000000049c726b', 'on')  
              or (states('sensor.0x000d6f0014bb14b4_power') | float > 10)  
              or is_state('binary_sensor.notebook', 'on')  }}    
            icon_template: >-
              {% if is_state('binary_sensor.room_occupancy', 'on') %}
                mdi:account-multiple-check
              {% else %}
                mdi:account-multiple-check-outline
              {% endif %}

          moving_dark:
            friendly_name: "Движение в темноте"
            device_class: motion
            value_template: >-
              {{ is_state('binary_sensor.0x00158d0001e547a3_occupancy', 'on') 
              and (states('sensor.0x04cf8cdf3c7cf19e_illuminance') | float < 5000 ) }}    
            icon_template: >-
              {% if is_state('binary_sensor.moving_dark', 'on') %}
                mdi:motion-sensor
              {% else %}
                mdi:motion-sensor-off
              {% endif %}

          flat_windows_delay:
            friendly_name: "Окна в квартире c задержкой"
            device_class: window
            delay_on:
                seconds: 30           
            value_template: >-
              {{ is_state('binary_sensor.0x00158d000445206b_contact', 'on') 
              or is_state('binary_sensor.0x00158d00013ed373_contact', 'on')  
              or is_state('binary_sensor.0xec1bbdfffedf6a6a_contact', 'on')  }}
            icon_template: >-
              {% if is_state('binary_sensor.flat_windows_delay', 'on') %}
                mdi:window-open-variant
              {% else %}
                mdi:window-closed-variant
              {% endif %}

          flat_motion_delay:
            friendly_name: "Движение в квартире c задержкой"
            device_class: motion
            delay_off:
                minutes: 5
            value_template: >-
              {{ is_state('binary_sensor.0x00158d0001e547a3_occupancy', 'on') 
              or is_state('binary_sensor.0x00158d00016d56f5_occupancy', 'on')  
              or is_state('binary_sensor.0x00158d0001a66222_occupancy', 'on')  }}
            icon_template: >-
              {% if is_state('binary_sensor.flat_motion_delay', 'on') %}
                mdi:account-group
              {% else %}
                mdi:account-group-outline
              {% endif %}


          light_nomotion:
            friendly_name: "Свет без движения"
            delay_on:
                minutes: 5
            value_template: >-
              {{ is_state('binary_sensor.0x00158d0001e547a3_occupancy', 'off') 
              and is_state('light.yeelight_ceiling4_0x00000000049c726b', 'on') }}    
            icon_template: mdi:lightbulb-on-outline

    sensor:
      - platform: template
        sensors:

          socket_power:
            friendly_name: 'Мощность нагрузки'
            device_class: power
            unit_of_measurement: 'Вт'
            value_template: "{{ (states('sensor.0x000d6f0014bb14b4_power') | float) | round(3)}}"
            icon_template: >-
              {% if states('sensor.socket_power') | float < 1 %}
                mdi:gauge-empty
              {% elif states('sensor.socket_power') | float < 500 %}
                mdi:gauge-low
              {% elif states('sensor.socket_power') | float < 1000 %}
                mdi:gauge
              {% else %}
                mdi:gauge-full
              {% endif %}
              
              
          room_total_power:
            friendly_name: 'Общее потребление'
            unit_of_measurement: "Вт"
            device_class: power
            value_template: >-
              {{ (
                  (states('sensor.0x588e81fffed4af56_power') | float) +
                  (states('sensor.0x04cf8cdf3c764e0a_power') | float) +
                  (states('sensor.0x04cf8cdf3c788a1b_power') | float) 
                 ) | round(3)
                }}
            icon_template: >-
              {% if states('sensor.socket_power') | float < 1 %}
                mdi:gauge-empty
              {% elif states('sensor.socket_power') | float < 500 %}
                mdi:gauge-low
              {% elif states('sensor.socket_power') | float < 1000 %}
                mdi:gauge
              {% else %}
                mdi:gauge-full
              {% endif %}               
              
          room_median_temperature:
            friendly_name: 'Средняя температура'
            unit_of_measurement: "C"
            device_class: temperature
            icon_template: mdi:temperature-celsius
            value_template: >-
              {{ ((
                  (states('sensor.0x5c0272fffe0a4711_temperature') | float) +
                  (states('sensor.0x00124b0022659c04_temperature') | float) +
                  (states('sensor.0x00158d0001dcd47e_temperature') | float) 
                 ) / 3) | round(2)
                }}              
              
          real_mmhg_pressure:
            friendly_name: "Давление мм рт. ст. факт"
            unit_of_measurement: 'mmHg'
            device_class: pressure
            value_template: "{{ (states('sensor.0x00158d0001a4b9da_pressure')|float * 0.7500637)|round(2) }}"
            icon_template: mdi:gauge              
              
          static_power:
            friendly_name: "Статичная мощность"
            unit_of_measurement: "Вт"
            device_class: power
            icon_template: mdi:flash
            value_template: >-
              {% if is_state('switch.0x00158d00010ec4b8_switch', 'on') %}
              65,5
              {% else %}
              0
              {% endif %}              
              
          humidifier_state:
            friendly_name: "Увлажнитель - "
            value_template: >-
              {% if is_state('switch.0x000d6f0014bb14b4_switch', 'off') %}
              Выключен
              {% elif is_state('switch.0x000d6f0014bb14b4_switch', 'on') 
                 and (states('sensor.0x000d6f0014bb14b4_power') | float > 20)  %}
              Увлажнение
              {% elif is_state('switch.0x000d6f0014bb14b4_switch', 'on') 
                 and (states('sensor.0x000d6f0014bb14b4_power') | float < 20)  %}
              Закончилась вода
              {% else %}
              Ошибка
              {% endif %}
            icon_template: >-
              {% if is_state("sensor.humidifier_state", "Выключен") %}
              mdi:air-humidifier-off
              {% elif is_state("sensor.humidifier_state", "Увлажнение") %}
              mdi:air-humidifier
              {% elif is_state("sensor.humidifier_state", "Закончилась вода") %}
              mdi:water-off
              {% else %}
              mdi:alert-circle
              {% endif %}              
              
    switch:              
      - platform: template
        switches:  
          united_switch:
            friendly_name: "Розетки"
            value_template: >-
              {{ is_state('switch.0x00158d00010ec4b8_switch', 'on')  
                 and is_state('switch.0x00158d0001a2ccab_switch', 'on') }}
            turn_on:
              - service: switch.turn_on
                entity_id: 
                  - switch.0x00158d00010ec4b8_switch
                  - switch.0x00158d0001a2ccab_switch
            turn_off:
              - service: switch.turn_off
                entity_id: 
                  - switch.0x00158d00010ec4b8_switch
                  - switch.0x00158d0001a2ccab_switch
            icon_template: >-
              {% if is_state('switch.united_switch', 'on') %}
                mdi:power-plug
              {% else %}
                mdi:power-plug-off
              {% endif %}
              
              
          television:
            friendly_name: "Телевизор"
            value_template: "{{ states('sensor.0x588e81fffed4af56_power') | float > 10 }}"
            turn_on:
              - service: remote.send_command
                data:
                  entity_id: remote.broadlink_remote
                  command:
                   - b64:JgBoACQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAAFhJBEWERYtFS0WERYtFhEVLRYRFi0WLRYAAWEjEhURFi0WLRYQFi0WERYtFREWLRYtFgABYSQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAA0F==
            turn_off:
              - service: remote.send_command
                data:
                  entity_id: remote.broadlink_remote
                  command:
                    - b64:JgCCACQRFiQWEBYbFSQWERURFhoWLRYkFhoWAAGcJBEWJBYQFhsVJBYRFhAWGxUtFiQWGhYAAZwkERYkFhEVGxYjFhEWEBYbFS0WJBYaFgABnCUQFiQWERUbFiMWERYQFhsWLBYkFhoWAAGdJBEVJBYRFhoWIxYRFhEVGxYtFiMWGhYADQUAAAAAAAA==
                    - b64:JgCCACQRFiQWEBYbFSQWERURFhoWLRYkFhoWAAGcJBEWJBYQFhsVJBYRFhAWGxUtFiQWGhYAAZwkERYkFhEVGxYjFhEWEBYbFS0WJBYaFgABnCUQFiQWERUbFiMWERYQFhsWLBYkFhoWAAGdJBEVJBYRFhoWIxYRFhEVGxYtFiMWGhYADQUAAAAAAAA==
                    - b64:JgCCACQRFiQWEBYbFSQWERURFhoWLRYkFhoWAAGcJBEWJBYQFhsVJBYRFhAWGxUtFiQWGhYAAZwkERYkFhEVGxYjFhEWEBYbFS0WJBYaFgABnCUQFiQWERUbFiMWERYQFhsWLBYkFhoWAAGdJBEVJBYRFhoWIxYRFhEVGxYtFiMWGhYADQUAAAAAAAA==
                    - b64:JgCCACQRFiQWEBYbFSQWERURFhoWLRYkFhoWAAGcJBEWJBYQFhsVJBYRFhAWGxUtFiQWGhYAAZwkERYkFhEVGxYjFhEWEBYbFS0WJBYaFgABnCUQFiQWERUbFiMWERYQFhsWLBYkFhoWAAGdJBEVJBYRFhoWIxYRFhEVGxYtFiMWGhYADQUAAAAAAAA==
                    - b64:JgBoACQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAAFhJBEWERYtFS0WERYtFhEVLRYRFi0WLRYAAWEjEhURFi0WLRYQFi0WERYtFREWLRYtFgABYSQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAA0F==
            icon_template: >-
              {% if is_state("switch.television", "on") %}
              mdi:television
              {% else %}
              mdi:television-off
              {% endif %}              
                   
```

____
### Как поддержать развитие проекта?
* [Стать спонсором моего Youtube](http://kvazis.link/sponsorship)
* [Подписаться на Patreon](http://kvazis.link/patreon)
* [Перевод через Paypal](http://kvazis.link/paypal)
* Webmoney - Z243592584952
* BTC - 1Gzr7WQugfnPuWVawu47EiCMTDUBqCAshj
* ETH - 0xa0ce3E29Cf537013649Ae9cdbc08C4853fF91FAc
* LTC - ltc1qs493yk2wk9ywx5h6aruk4p9zm75hx42ekv4ym2
* TRX - TFTCLqvS1tMBwokRHBwz1TCDJ4oD1Z5zPk