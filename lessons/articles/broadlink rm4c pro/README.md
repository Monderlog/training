### [wifi пульт для IR и RF кодов Broadlink RM4C Pro, работа в Home Assistant - управляем кондиционером](https://youtu.be/mnUF-dpVvGo)

#### Команды и скрипты из обзора

:white_check_mark: Ручное добавление устройства (команды из консоли)    

```yaml
docker exec -it homeassistant /bin/bash

cd /./usr/local/lib/python3.8/site-packages/broadlink

vi __init__.py
```
:ballot_box_with_check: Режим редактирования - `i`    
:ballot_box_with_check: Вставляем строку - `0x6184: (rm4pro, "RM4 pro", "Broadlink"),`    
:ballot_box_with_check: Выход из режима редактирования - `escape`    
:ballot_box_with_check: Сохранение - `:w`    
:ballot_box_with_check: Выход из редактора - `:q!`    
:ballot_box_with_check: Выход из контейнера - `exit`    

:white_check_mark: Скрипты показанные в уроке    

```yaml

  broadlink_learn_rm4:
    alias: Обучение Broadlink RM4C pro
    sequence:
      - service: remote.learn_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: tv
          command: power
          

  broadlink_learn_rm4_2:
    alias: Обучение Broadlink RM4C pro
    sequence:
      - service: remote.learn_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: light
          command: power on
          command_type: rf

  broadlink_learn_tv_command:
    alias: Телевизионный пульт
    sequence:
      - service: remote.learn_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: tv
          command: 
             - power
             - volume +
             - volume -
             - menu
             
  tv_broadlink_power:
    alias: Питание телевизора
    sequence:
      - service: remote.send_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: tv
          command: power
          
  tv_broadlink_volume_up:
    alias: Звук телевизора + 5
    sequence:
      - service: remote.send_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: tv
          command: volume +
          num_repeats: 5
          
  tv_broadlink_power_volume:
    alias: Включить и добавить звук
    sequence:
      - service: remote.send_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: tv          
          command: 
             - power
             - volume + 
             - volume +
             - volume +
             
  broadlink_send_rm4:
    alias: Отправка Broadlink RM4C pro
    sequence:        
      - service: remote.send_command
        data:
          entity_id: remote.rm4c_pro_remote
          command:
            - b64:JgBoACQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAAFhJBEWERYtFS0WERYtFhEVLRYRFi0WLRYAAWEjEhURFi0WLRYQFi0WERYtFREWLRYtFgABYSQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAA0F==
            - b64:JgBoACQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAAFhJBEWERYtFS0WERYtFhEVLRYRFi0WLRYAAWEjEhURFi0WLRYQFi0WERYtFREWLRYtFgABYSQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAA0F==
            - b64:JgBoACQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAAFhJBEWERYtFS0WERYtFhEVLRYRFi0WLRYAAWEjEhURFi0WLRYQFi0WERYtFREWLRYtFgABYSQRFhAWLRYtFhEWLBYRFi0WERUtFi0WAA0F==

  air_conditioner:
    alias: Кондиционер
    sequence:
      - service: remote.learn_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: conditioner
          command: 
             - cold
             - power off             
             
  delete_command:
    sequence:
      - service: remote.delete_command
        target:
          entity_id: remote.rm4c_pro_remote
        data:
          device: light
          command: ON
```

:white_check_mark: Пакадж для управление кондиционером    

```yaml
broadlink:

    switch:
      - platform: template
        switches:

          air_conditioner:
            friendly_name: "Кондиционер"
            value_template: "{{ states('sensor.0x04cf8cdf3c764e0a_power') | float > 10 }}"
            turn_on:
              service: remote.send_command
              target:
                entity_id: remote.rm4c_pro_remote
              data:
                device: conditioner
                command: 
                   - cold
            turn_off:
              service: remote.send_command
              target:
                entity_id: remote.rm4c_pro_remote
              data:
                device: conditioner
                command: 
                   - power off
            icon_template: >-
              {% if is_state("switch.air_conditioner", "on") %}
              mdi:air-conditioner
              {% else %}
              mdi:power-off
              {% endif %}
              
    climate:
      - platform: generic_thermostat
        name: air_conditioner
        heater: switch.air_conditioner
        target_sensor: sensor.0x00158d000156e92e_temperature
        target_temp: 22
        min_temp: 21
        max_temp: 25
        ac_mode: true
        cold_tolerance: 0.5
        hot_tolerance: 0.5
        min_cycle_duration:
          minutes: 5
        keep_alive:
          minutes: 3
        initial_hvac_mode: "cool"
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