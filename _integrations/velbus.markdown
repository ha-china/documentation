---
title: Velbus
description: Access and control your Velbus devices.
ha_category:
  - Binary sensor
  - Button
  - Climate
  - Hub
  - Light
  - Sensor
  - Switch
ha_iot_class: Local Push
ha_release: '0.50'
ha_config_flow: true
ha_codeowners:
  - '@Cereal2nd'
  - '@brefra'
ha_domain: velbus
ha_platforms:
  - binary_sensor
  - button
  - climate
  - cover
  - diagnostics
  - light
  - select
  - sensor
  - switch
ha_integration_type: hub
ha_quality_scale: bronze
---

The **Velbus** {% term integration %} is used to control [Velbus](https://www.velbus.eu/?lang=en) modules. It supports the Velbus USB, Velbus serial and a TCP/IP gateway.


{% include integrations/config_flow.md %}

## Configuration parameters

During the setup you will be shown 2 choices on ways to connect to the Velbus bus:

- USB
- TCP/IP

### USB

The USB connection is a way to connect to the Velbus bus. You will need a Velbus USB interface to connect to the bus. The USB interface is connected to the USB port of your Home Assistant device.
The interface USB devices are automatically detected and shown in a list.
Select the correct USB interface from the list and select **Submit**.

There will be a connection test to make sure the connection is working, and if it's working the integration will be added to Home Assistant.

### TCP/IP

The TCP/IP connection is a way to connect to the Velbus bus. You will need a Velbus TCP/IP interface available in your network.

There are a couple of parameters you need to fill in to connect to the Velbus bus:

- tls
- host
- port
- password

The `tls` parameter is optional and can be used to enable or disable the TLS connection.
The `host` parameter is the IP address of the Velbus TCP/IP interface.
The `port` parameter is the port number of the Velbus TCP/IP interface.
The `password` parameter is optional and can be used to authenticate to the Velbus TCP/IP interface.

#### Example: signum

- tls: yes
- host: your signum IP address
- port: 27015
- password: your signum password (if configured)

#### Example: velser

- tls: no
- host: your velser IP address
- port: 6000
- password: leave empty

#### Example: Home Assistant add-on

- tls: depending on your configuration
- host: your Home Assistant IP address
- port: 27015 if you kept the default
- password: leave empty

{% note %}
The pushbutton LEDs of input modules are disabled by default. These can be enabled from the **Devices** panel in the **Configuration** page of the web interface.
{% endnote %}

## Actions
- `velbus.sync clock`: Synchronize Velbus time to local clock.
- `velbus.scan`: Scan the bus for new devices.
- `velbus.set_memo_text`: Show memo text on Velbus display modules.
- `velbus.clear_cache`: Clear the full velbuscache or the cache for one module only.

### Action `velbus.sync_clock`

You can use the `velbus.sync_clock` action to synchronize the clock of the Velbus modules to the clock of the machine running Home Assistant. This is the same as the 'sync clock' button at the VelbusLink software.

| Data attribute | Optional | Description                              |
| ---------------------- | -------- | ---------------------------------------- |
| `interface`            | no       | The port used to connect to the bus (the same one as used during configuration). |
| `config_entry`         | no       | The config_entry to send the command to. |

### Action `velbus.scan`

You can use the `velbus.scan` action to synchronize the modules between the bus and Home Assistant. This is the same as the 'scan' button at the VelbusLink software.

| Data attribute | Optional | Description                              |
| ---------------------- | -------- | ---------------------------------------- |
| `interface`            | no       | The port used to connect to the bus (the same one as used during configuration). |
| `config_entry`         | no       | The config_entry to send the command to. |


### Action `velbus.set_memo_text`

You can use the `velbus.set_memo_text` action to provide the memo text to be displayed at Velbus modules like VMBGPO(D) and VMBELO.

| Data attribute | Optional | Description                              |
| ---------------------- | -------- | ---------------------------------------- |
| `interface`            | no       | The port used to connect to the bus (the same one as used during configuration). |
| `config_entry`         | no       | The config_entry to send the command to. |
| `address`              | no       | The module address in decimal format, which is displayed at the device list at the integration page. |
| `memo_text`            | yes      | Text to be displayed on module. When no memo text is supplied the memo text will be cleared. |

Example:

```yaml
script:
  trash_memo:
    alias: "Trash memo text"
    sequence:
    - action: velbus.set_memo_text
      data:
        address: 65
        memo_text: "It's trash day"
        config_entry: "01JGE8XB3MNPZFA836TTZ3KZ46"
```

### Action `velbus.clear_cache`

You can use the `velbus.clear_cache` action to clear the cache of one module or the full cache. Once the clear happens, the integration will start a new scan.
Use this action when you make changes to your configuration via velbuslink.

| Data attribute | Optional | Description                              |
| ---------------------- | -------- | ---------------------------------------- |
| `interface`            | no       | The port used to connect to the bus (the same one as used during configuration). |
| `config_entry`         | no       | The config_entry to send the command to. |
| `address`              | no       | The module address in decimal format, which is displayed on the device list on the integration page, if provided the service will only clear the cache for this model, without an address, the full velbuscache will be cleared. |


## VMB7IN and the Energy dashboard

The VMB7IN sensor can be integrated with Home Assistant's Energy dashboard to track your utility consumption.

In some cases, the VMB7IN sensor does not report what the counter is counting. If the counter is related to an energy device, everything will work out of the box.
But if the VMB7IN sensor is a water or gas counter, you will need to specify this in your configuration.yaml file.

```yaml
homeassistant:
  customize:
    sensor.eau_counter:
      device_class: water
```

The device_class attribute can have 2 values:
- gas: if the counter represents a gas meter
- water: if the counter represents a water meter


## Example automation

The Velbus {% term integration %} allows you to link a Velbus button (i.e., a button of a [VMBGPOD](https://www.velbus.eu/products/view/?id=416302&lang=en) module) to a controllable {% term entity %} of Home Assistant.
The actual linking can be realized by two automation rules. One rule to control the device using the push button and a second rule to update the LED state of the push button as soon as the {% term entity %} state changes.

```yaml
# Control light living from Velbus push_button_10
- alias: "Control light living using Velbus push_button_10"
  triggers:
    - trigger: state
      entity_id: binary_sensor.push_button_10
      to: "on"
  actions:
    - action: light.toggle
      entity_id: light.living
      

# Keep status LED push_button_10 in sync to status light living
- alias: "Update LED state of push_button_10"
  triggers:
    - trigger: state
      entity_id: light.living
      to: "on"
    - trigger: state
      entity_id: light.living
      to: "off"
  conditions:
    - condition: or
      conditions:
        - condition: and
          conditions:
          - condition: state
            entity_id: light.led_push_button_10
            state: "on"
          - condition: state
            entity_id: light.living
            state: "off"
        - condition: and
          conditions:
            - condition: state
              entity_id: light.led_push_button_10
              state: "off"
            - condition: state
              entity_id: light.living
              state: "on"
  actions:
    - action: light.toggle
      entity_id: light.led_push_button_10
      
```

## Removing the integration

The Velbus integration and its entities can be removed by following these steps:

{% include integrations/remove_device_service.md %}

Note: Removing the integration will delete all Velbus devices and their history from Home Assistant.
