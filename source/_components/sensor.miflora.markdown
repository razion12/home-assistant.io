---
layout: page
title: "Mi Flora plant sensor"
description: "Instructions on how to integrate MiFlora BLE plant sensor with Home Assistant."
date: 2016-09-19 12:00
sidebar: true
comments: false
sharing: true
footer: true
logo: miflora.png
ha_category: DIY
ha_release: 0.29
ha_iot_class: "Local Polling"
---

The `miflora` sensor platform allows one to monitor plant soil and air conditions. The [Mi Flora plant sensor](https://xiaomi-mi.com/sockets-and-sensors/xiaomi-huahuacaocao-flower-care-smart-monitor/) is a small Bluetooth Low Energy device that monitors the moisture and conductivity of the soil as well as ambient light and temperature. Since only one BLE device can be polled at a time, the library implements locking to prevent polling more than one device at a time.

# Install Bluetooth Backend
Before configuring Home Assistant you need a Bluetooth backend and the MAC address of your sensor. Depending on your operating system, you may have to configure the proper Bluetooth backend for your system:

- On [Hass.io](/hassio/installation/): Miflora will work out of the box.
- On a [generic Docker installation](/docs/installation/docker/): Works out of the box with `--net=host` and properly configured Bluetooth on the host.
- On other Linux systems:
    - Preferred solution: Install the `bluepy` library (via pip). When using a virtual environment, make sure to use install the library in the right one.
    - Fallback solution: Install `gatttool` via your package manager. Depending on the distribution, the package name might be: `bluez`, `bluetooth`, `bluez-deprecated`
- On Windows and MacOS there is currently no support for the [miflora library](https://github.com/open-homeautomation/miflora/).

# Scan for MAC address
Start a scan to determine the MAC addresses of the sensor (you can identify your sensor by looking for `Flower care` or `Flower mate` entries) using this command:

```bash
$ sudo hcitool lescan
LE Scan ...
F8:04:33:AF:AB:A2 [TV] UE48JU6580
C4:D3:8C:12:4C:57 Flower mate
[...]
```

Or, if your distribution is using bluetoothctl use the following commands:

```bash
$ bluetoothctl
[bluetooth]# scan on
[NEW] Controller <your Bluetooth adapter> [default]
[NEW] F8:04:33:AF:AB:A2 [TV] UE48JU6580
[NEW] C4:D3:8C:12:4C:57 Flower mate
```

If you can't use `hcitool` or `bluetoothctl` but have access to an Android phone you can try `BLE Scanner` or similar scanner applications from the Play Store to easily find your sensor MAC address.

# Configure
To use your Mi Flora plant sensor in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
sensor:
  - platform: miflora
    mac: 'xx:xx:xx:xx:xx:xx'
    monitored_conditions:
      - moisture
```

- **mac** (*Required*): The MAC address of your sensor.
- **monitored_conditions** array (*Optional*): The parameters that should be monitored (defaults to monitoring all parameters).
  - **moisture**: Moisture in the soil.
  - **light**: Brightness at the sensor's location.
  - **temperature**: Temperature at the sensor's location.
  - **conductivity**: Conductivity in the soil.
  - **battery**: Battery details.
- **name** (*Optional*): The name displayed in the frontend.
- **force_update** (*Optional*): Sends update events even if the value hasn't changed.
- **median** (*Optional*): Sometimes the sensor measurements show spikes. Using this parameter, the poller will report the median of the last 3 (you can also use larger values) measurements. This filters out single spikes. Median: 5 will also filter double spikes. If you never have problems with spikes, `median: 1` will work fine.
- **timeout** (*Optional*): Define the timeout value in seconds when polling (defaults to 10 if not defined)
- **retries** (*Optional*): Define the number of retries when polling (defaults to 2 if not defined)
- **cache_value** (*Optional*): Define cache expiration value in seconds (defaults to 1200 if not defined)
- **adapter** (*Optional*): Define the Bluetooth adapter to use (defaults to hci0). Run `hciconfig` to get a list of available adapters.

<p class='note warning'>
By default the sensor is only polled once every 20 minutes. So, if you set `median: 3` it will take _at least_ 40 minutes   before the sensor will report a value after a Home Assistant restart. Since the values usually change very slowly, this usually isn't a big problem. Keep in mind though that reducing polling intervals will have a negative effect on the battery life.
</p>

A full configuration example could look like the one below:

```yaml
# Example configuration.yaml entry
sensor:
  - platform: miflora
    mac: 'xx:xx:xx:xx:xx:xx'
    name: Flower 1
    force_update: false
    median: 3
    monitored_conditions:
      - moisture
      - light
      - temperature
      - conductivity
      - battery
```
