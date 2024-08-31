Paste the following directly into your Configuration.yaml file

1. In File Editor, navigate to Configuration.yaml

2. If you already have any template sensors setup, simply paste the following underneath "template:".  If you have not yet set up any template sensors, you need to type "template:" on a new line in configuration.yaml, and paste the following underneath:

```
##Charge Calculators
  - sensor:
      - name: "soc_overdischarge_percent"
        unique_id: "soc_overdischarge_percent"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: "{{ (states('input_number.overdischarge_soc') | float(0) / 100) | round(2) }}"
  - sensor:
      - name: "soc_usable_percent"
        unique_id: "soc_usable_percent"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: "{{ (1 - states('sensor.soc_overdischarge_percent') | float(0)) | round(2) }}"
  - sensor:
      - name: "soc_forcecharge_percent"
        unique_id: "soc_forcecharge_percent"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: "{{ (states('input_number.force_charge_soc') | float(0) / 100) | round(2) }}"
  - sensor:
      - name: "soc_usableforcecharge"
        unique_id: "soc_usableforcecharge"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: "{{ (1 - states('sensor.soc_forcecharge_percent') | float(0)) * (states('input_number.battery_capacity') | float (0)) | round(2) }}"
  - sensor:
      - name: "soc_total_usable"
        unique_id: "soc_total_usable"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: "{{ (states('sensor.soc_usable_percent') | float(0) * states('input_number.battery_capacity') | float (0)) | round(2) }}"
  - sensor:
      - name: "soc_usable_kwh"
        unique_id: "soc_usable_kwh"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: "{{ ((states('sensor.solax_battery_soc') | float(0) / 100) * states('input_number.battery_capacity') | float (0)) - (states('input_number.battery_capacity') | float (0) * states('sensor.soc_overdischarge_percent') | float (0)) | round(2) }}"
  - sensor:
      - name: "soc_required_charge"
        unique_id: "soc_required_charge"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: |-
         {% set sum = (states('input_number.target_usable_soc') | float(0) - states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge') | float(0)) | round(2)%}
         {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
         {{ ([0, sum, max] | sort)[1] }}
  - sensor:
      - name: "soc_required_charge_plus_boost"
        unique_id: "soc_required_charge_plus_boost"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: |-
         {% set sum = (states('sensor.soc_required_charge') | float(0) + states('input_number.boost_charge') | float(0)) | round(2)%}
         {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
         {{ ([0, sum, max] | sort)[1] }}
  - sensor:
      - name: "SoC at Start of Offpeak Tonight"
        unique_id: "soc_start_of_offpeak_tonight"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: |-
         {% set sum = states('sensor.solcast_pv_forecast_remaining_today') | float(0) + states('sensor.soc_usable_kwh') | float(0) - (states('input_number.expected_consumption') | float(0) - states('sensor.solax_house_load_today') | float(0)) - (states('input_number.base_load') | float(0) * states('sensor.soc_charge_start_time_decimal') | float(0)) | round(2)%}
         {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
         {{ ([0, sum, max] | sort)[1] }}
  - sensor:
      - name: "SoC at End of Offpeak Tonight - No Charge"
        unique_id: "soc_end_of_offpeak_no_charge"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: |-
         {% set sum = states('sensor.soc_at_start_of_offpeak_tonight') |float(0) - (3 * states('input_number.base_load') |float(0)) + states('input_number.boost_charge') |float(0) | round(2)%}
         {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
         {{ ([0, sum, max] | sort)[1] }}
  - sensor:
      - name: "SoC at End of Offpeak Tonight - With Charge"
        unique_id: "soc_end_of_offpeak_with_charge"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: |-
         {% set sum = (states('sensor.soc_at_start_of_offpeak_tonight') | float(0) + states('sensor.soc_required_charge') | float(0) - (3 - states('sensor.soc_charge_time_decimal') | float(0)) * states('input_number.base_load') | float(0)) + states('input_number.boost_charge') | float(0)| round(2)%}
         {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
         {{ ([0, sum, max] | sort)[1] }}
  - sensor:
      - name: "SoC at Start of Offpeak Tomorrow - No Charge"
        unique_id: "soc_start_of_offpeak_tomorrow_no_charge"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: |-
         {% set sum = states('sensor.soc_at_end_of_offpeak_tonight_no_charge') | float(0) + states('sensor.solcast_pv_forecast_tomorrow') | float(0) - (states('input_number.expected_consumption_tomorrow') | float(0) - (3 * states('input_number.base_load') | float(0))) - (states('input_number.base_load') | float(0) * states('sensor.soc_charge_start_time_decimal') | float(0)) | round(2)%}
         {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
         {{ ([0, sum, max] | sort)[1] }}
  - sensor:
      - name: "SoC at Start of Offpeak Tomorrow - With Charge"
        unique_id: "soc_start_of_offpeak_tomorrow_with_charge"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: |-
         {% set sum = states('sensor.soc_at_end_of_offpeak_tonight_with_charge') | float(0) + states('sensor.solcast_pv_forecast_tomorrow') | float(0) - (states('input_number.expected_consumption_tomorrow') | float(0) - (states('input_number.offpeak_window') | float(0) * states('input_number.base_load') | float(0))) - (states('input_number.base_load') | float(0) * states('sensor.soc_charge_start_time_decimal') | float(0)) | round(2)%}
         {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
         {{ ([0, sum, max] | sort)[1] }}
  - sensor:
      - name: "SoC at Start of Offpeak Tomorrow Display"
        unique_id: "soc_start_offpeak_tomorrow_display"
        unit_of_measurement: 'kWh'
        device_class: energy
        state:  >
          {% if states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge')|float(0) == states('sensor.soc_at_start_of_offpeak_tomorrow_with_charge')|float(0) %}
            {{ states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge')|float(0) }}
          {% else %}
            {{ states('sensor.soc_at_start_of_offpeak_tomorrow_with_charge') }}
          {% endif %}
        availability: >
          {{ states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge')|is_number and
             states('sensor.soc_at_start_of_offpeak_tomorrow_with_charge')|is_number }}    
  - sensor:
      - name: "Battery Charge Power"
        unique_id: "battery_charge_power"
        unit_of_measurement: 'W'
        device_class: power
        state: "{{((states('number.solax_timed_charge_current') | float(0) * 55)) | round(2) }}"
  - sensor:
      - name: "calculated_charge_current"
        unique_id: "calculated_charge_current"
        unit_of_measurement: 'A'
        device_class: energy
        state: "{{ (states('sensor.soc_usableforcecharge') | float(0) / 3 * 1000 / 55) | round(0) }}"
  - sensor:
      - name: "Auto Charge Scheduled"
        unique_id: "auto_charge_scheduled"
        state:  >
          {% if states('sensor.soc_required_charge') | float(0) > 0.01 %}
            Charge Scheduled
          {% else %}
            Not Required
          {% endif %}
        availability: >
          {{ states('sensor.soc_required_charge')|is_number  }}
          
##Charge Times
  - sensor:
      - name: "soc_charge_time_decimal"
        unique_id: "soc_charge_time_decimal"
        state: "{{ (states('sensor.soc_required_charge_plus_boost')| float(0) / (states('sensor.battery_charge_power') | float(0) / 1000)) | round(2) }}"
  - sensor:
      - name: "soc_charge_start_time_decimal"
        unique_id: "soc_charge_start_time_decimal"
        state: "{{((states('number.solax_timed_charge_start_m')| float(0) / 6 / 10)) + (states('number.solax_timed_charge_start_h')| float(0)) | round(2) }}"
  - sensor:
      - name: "soc_charge_end_time_decimal"
        unique_id: "soc_charge_end_time_decimal"
        state: "{{((states('sensor.soc_charge_time_decimal')| float(0) + states('sensor.soc_charge_start_time_decimal') | float(0))) | round(2) }}"
  - sensor:
      - name: "soc_charge_time_hhmm"
        unique_id: "soc_charge_time_hhmm"
        state: >
           {{ (states('sensor.soc_charge_time_decimal') | float(0) * 3600)
             | timestamp_custom('%H:%M', false) }}
  - sensor:
      - name: "soc_charge_end_time_hhmm"
        unique_id: "soc_charge_end_time_hhmm"
        state: >
           {{ (states('sensor.soc_charge_end_time_decimal') | float(0) * 3600)
             | timestamp_custom('%H:%M', false) }}
  - sensor:
      - name: "soc_charge_end_time_hour"
        unique_id: "soc_charge_end_time_hour"
        state: >           
           {{ (states('sensor.soc_charge_end_time_decimal') | float(0) * 3600)
           | timestamp_custom('%H', false) }}
  - sensor:
      - name: "soc_charge_end_time_minute"
        unique_id: "soc_charge_end_time_minute"
        state: >           
           {{ (states('sensor.soc_charge_end_time_decimal') | float(0) * 3600)
           | timestamp_custom('%M', false) }}
  - sensor:
      - name: "Charge Start Time"
        unique_id: "charge_start_time"
        state: >
            {{0}}{{ states('number.solax_timed_charge_start_h') | round(0) }}:{{ states('number.solax_timed_charge_start_m') | round(0) }}
  - sensor:
      - name: "Charge End Time Set"
        unique_id: "charge_end_time_set"
        state: >
            {{0}}{{ states('number.solax_timed_charge_end_h') | round(0) }}:{{ states('number.solax_timed_charge_end_m') | round(0) }}

##Consumption
  - sensor:
      - name: "Remaining Consumption Today"
        unique_id: "remaining_consumption_today"
        unit_of_measurement: 'kWh'
        device_class: energy
        state: "{{ states('input_number.expected_consumption')| float(0) - states('sensor.solax_house_load_today')| float(0) | round(2) }}"     
  - sensor:
      - name: "String 1 Output"
        unit_of_measurement: 'W'
        device_class: power
        state: "{{ states('sensor.solax_pv_current_1')| float(0) * states('sensor.solax_pv_voltage_1')| float(0) | round(0) }}"
  - sensor:
      - name: "String 2 Output"
        unit_of_measurement: 'W'
        device_class: power
        state: "{{ states('sensor.solax_pv_current_2')| float(0) * states('sensor.solax_pv_voltage_2')| float(0) | round(0) }}"
```
