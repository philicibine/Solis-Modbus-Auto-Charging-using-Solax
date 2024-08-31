Copy below automations into your HA instance.

# Installation

1. Go to Settings > Automations & Scenes > "+ Create Automation" > Create new automation

2. Click the three dots top right and Choose Edit in YAML

3. Paste each individual automation below into the code field

4. Hit SAVE and allow the automations to be named exactly as detailed below.  This should happen automatically for you.

## Automation Code


### Flux - Discharge Cutout
```
alias: Flux - Discharge Cutout
description: >-
  Stops Flux battery discharging if the remaining SoC drops to 50% between the
  hours of 16:00 and 19:00 with Flux discharging enabled
trigger:
  - platform: state
    entity_id:
      - sensor.solax_battery_soc
    from: "51"
    to: "50"
condition:
  - condition: time
    after: "16:00:00"
    before: "19:00:00"
    weekday:
      - mon
      - tue
      - wed
      - thu
      - fri
      - sat
      - sun
  - condition: state
    entity_id: input_boolean.flux_discharge
    state: "on"
action:
  - service: input_boolean.turn_off
    data: {}
    target:
      entity_id: input_boolean.flux_discharge
mode: single
```


### Flux - Discharge Off
```
alias: Flux - Discharge Off
description: >-
  Sets the Battery to Not Discharge to the grid.  Both Solax discharge values
  set to 16:00
trigger:
  - platform: state
    entity_id:
      - input_boolean.flux_discharge
    from: "on"
    to: "off"
condition: []
action:
  - parallel:
      - service: number.set_value
        data:
          value: "16"
        target:
          entity_id: number.solax_timed_discharge_start_hours
      - service: number.set_value
        data:
          value: "0"
        target:
          entity_id: number.solax_timed_discharge_start_minutes
      - service: number.set_value
        data:
          value: "16"
        target:
          entity_id: number.solax_timed_discharge_end_hours
      - service: number.set_value
        data:
          value: "0"
        target:
          entity_id: number.solax_timed_discharge_end_minutes
  - delay:
      hours: 0
      minutes: 0
      seconds: 2
      milliseconds: 0
  - service: button.press
    data: {}
    target:
      entity_id:
        - button.solax_update_charge_discharge_times
mode: single
```


### Flux - Discharge On
```
alias: Flux - Discharge On
description: >-
  Sets the Battery to Discharge to the grid at full power between 16:00 and
  19:00
trigger:
  - platform: state
    entity_id:
      - input_boolean.flux_discharge
    from: "off"
    to: "on"
condition: []
action:
  - parallel:
      - service: number.set_value
        data:
          value: "16"
        target:
          entity_id: number.solax_timed_discharge_start_hours
      - service: number.set_value
        data:
          value: "0"
        target:
          entity_id: number.solax_timed_discharge_start_minutes
      - service: number.set_value
        data:
          value: "19"
        target:
          entity_id: number.solax_timed_discharge_end_hours
      - service: number.set_value
        data:
          value: "0"
        target:
          entity_id: number.solax_timed_discharge_end_minutes
  - delay:
      hours: 0
      minutes: 0
      seconds: 2
      milliseconds: 0
  - service: button.press
    data: {}
    target:
      entity_id:
        - button.solax_update_charge_discharge_times
mode: single
```


### Solar - Battery Charge Automation
```
alias: Solar - Battery Charge Automation
description: Main automatic battery charging function
trigger:
  - platform: time
    at: "23:55:00"
condition: []
action:
  - data:
      value: "{{ states('sensor.solax_house_load_today') }}"
    target:
      entity_id: input_number.expected_consumption
    action: input_number.set_value
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
  - parallel:
      - data:
          value: "{{ states('sensor.soc_charge_end_time_hhmm').split(':')[0] }}"
        target:
          entity_id: number.solax_timed_charge_end_h
        action: number.set_value
      - data:
          value: "{{ states('sensor.soc_charge_end_time_hhmm').split(':')[1] }}"
        target:
          entity_id: number.solax_timed_charge_end_m
        action: number.set_value
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
  - data: {}
    target:
      entity_id: button.solax_update_charge_discharge_times
    action: button.press
mode: single
```


### Solar - Charge Current Settings
```
alias: Solar - Charge Current Settings
description: >-
  This Automation automatically sets the Solax Timed Charge Current to match the
  user inputs of battery capacity, overdischarge soc and forcecharge soc
trigger:
  - platform: state
    entity_id:
      - input_number.battery_capacity
      - input_number.overdischarge_soc
      - input_number.force_charge_soc
      - input_number.offpeak_window
condition: []
action:
  - service: number.set_value
    data:
      value: "{{ states('sensor.calculated_charge_current') }}"
    target:
      entity_id: number.solax_timed_charge_current
mode: single
```


### Solar - Expected Consumption Low State Tracker
```
alias: Solar - Expected Consumption Low State Tracker
description: >-
  If the number input number 'Expected Consumption' is overtaken by the Solis
  sensor 'Solax House Load Today', the value from the sensor is set to the
  number input to avoid peculiarities in battery SoC forecasting.
trigger:
  - platform: state
    entity_id:
      - sensor.solax_house_load_today
condition:
  - condition: numeric_state
    entity_id: sensor.solax_house_load_today
    above: input_number.expected_consumption
action:
  - service: input_number.set_value
    data:
      value: "{{ states('sensor.solax_house_load_today') }}"
    target:
      entity_id: input_number.expected_consumption
mode: single
```


### Solar - Restore Consumption Defaults
```
alias: Solar - Restore Consumption Defaults
description: >-
  Sets todays expected consumption and tomorrows expected consumption to 10,
  Target SoC to 4.5, base load to 0.23, and boost charge to 0
trigger:
  - platform: state
    entity_id:
      - input_button.reset_consumption_defaults
condition: []
action:
  - service: input_number.set_value
    data:
      value: 10
    target:
      entity_id: input_number.expected_consumption
  - service: input_number.set_value
    data:
      value: 10
    target:
      entity_id: input_number.expected_consumption_tomorrow
  - service: input_number.set_value
    data:
      value: 4.5
    target:
      entity_id: input_number.target_usable_soc
  - service: input_number.set_value
    data:
      value: 0.23
    target:
      entity_id:
        - input_number.base_load
  - service: input_number.set_value
    data:
      value: 0
    target:
      entity_id: input_number.boost_charge
  - service: number.set_value
    data:
      value: "55"
    target:
      entity_id: number.solax_timed_charge_current
mode: single
```


### Solar - Update Times
```
alias: Solar - Update Times
description: >-
  Manually updates Inverter Charge and Discharge Times from Solax Modbus
  Integration
trigger: []
condition: []
action:
  - parallel:
      - service: number.set_value
        data:
          value: "{{ states('sensor.soc_charge_end_time_hhmm').split(':')[0] }}"
        target:
          entity_id: number.solax_timed_charge_end_hours
      - service: number.set_value
        data:
          value: "{{ states('sensor.soc_charge_end_time_hhmm').split(':')[1] }}"
        target:
          entity_id: number.solax_timed_charge_end_minutes
  - delay:
      hours: 0
      minutes: 0
      seconds: 5
      milliseconds: 0
  - service: button.press
    data: {}
    target:
      entity_id:
        - button.solax_update_charge_discharge_times
mode: single
```


### Solcast - API Poll Schedule
```
alias: Solcast - API Poll Schedule
description: New API call Solcast
trigger:
  - platform: time
    at: "06:00:00"
  - platform: time
    at: "10:00:00"
  - platform: time
    at: "14:00:00"
  - platform: time
    at: "18:00:00"
  - platform: time
    at: "23:50:00"
condition: []
action:
  - service: solcast_solar.update_forecasts
    data: {}
mode: single
```
