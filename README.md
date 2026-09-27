# ThermoWorks Cloud for Home Assistant
![GitHub branch check runs](https://img.shields.io/github/check-runs/a2hill/ha-thermoworks-cloud/main)
[![License](https://img.shields.io/github/license/a2hill/ha-thermoworks-cloud)](https://raw.githubusercontent.com/a2hill/ha-thermoworks-cloud/refs/heads/main/LICENSE)

## About
This integration allows [Home Assistant](https://www.home-assistant.io/) to pull data (temperature, battery, signal strength) from ThermoWorks Cloud connected devices.

### Supported Devices
See [Discussions - Device Interoperability](https://github.com/a2hill/ha-thermoworks-cloud/discussions/6)

## Installation
### HACS
If you have [HACS](https://hacs.xyz/) installed in your Home Assistant instance

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=a2hill&repository=ha-thermoworks-cloud&category=integration)

### Manual Installation
Copy the [custom_components/thermoworks_cloud](custom_components/) folder into the `config/custom_components` folder of your Home Assistant instance

## Usage
1. With this custom component installed into your HA instance, you will now see the ThermoWorks Cloud integration available as an integration that can be added (settings > Devices & services > Add Integration)
1. After adding the integration, be sure to set the [scan interval](#scan-interval) which will tell HA how often to request new data from ThermoWorks  
    * The default is 1,800 seconds (30 minutes), however this may be too slow for real-time applications like grilling with RFX

## Scan Interval
The scan interval is how often Home Assistant polls ThermoWorks Cloud for new data. It is a single setting for your account: every device and channel on it is refreshed together on the same schedule. There is no per-device interval, so a fast interval you set for a cook also applies to an always-on device such as a freezer monitor while it is in effect.

* Default: 1,800 seconds (30 minutes)
* Allowed range: 5 to 86,400 seconds (24 hours)

### Changing the scan interval
There are two ways to change it. Both edit the same stored value, so a change made in one place shows up in the other.

1. **Integration options**: Settings > Devices & services > ThermoWorks Cloud > **Configure**, then set **Scan interval (seconds)**.
1. **Scan interval entity**: the integration creates a `number` entity named **Scan interval**, which you can change from the UI or from automations and scripts.
    * Its entity ID is `number.thermoworks_cloud_scan_interval`, or `number.eti_cloud_scan_interval` for an ETI Cloud account.
    * It appears on the integration page under a **ThermoWorks Cloud** (or **ETI Cloud**) service entry, not on any individual thermometer, because it applies to every device on the account.
    * It is a configuration entity, so Home Assistant does not add it to auto-generated dashboards. You can still add it to a dashboard card yourself.

When the value changes, the integration reloads to apply it. ThermoWorks entities may show as unavailable for a moment, then an update is fetched right away and polling continues at the new interval. Setting the value it already has does nothing and does not cause a reload. If ThermoWorks Cloud can't be reached during the reload, Home Assistant keeps retrying in the background, and the integration's entities (including **Scan interval**) are unavailable until it succeeds.

### Tips
* Polling more often than your device sends readings to the cloud won't give you fresher data. If your device has a **Transmit Interval** sensor, it gives a rough idea of how often the device sends readings.
* Because each change reloads the integration, change the interval when something meaningful happens (a cook starts or ends), not on a short repeating timer.
* Short intervals mean more requests to ThermoWorks Cloud. Consider using a fast interval only while you actually need it.

### Example: poll faster during a cook
This automation polls every 15 seconds while an `input_boolean.cook_mode` helper is on and goes back to every 30 minutes when it is turned off. You can use any trigger that suits your setup instead of the helper, such as a probe temperature, a device's **Last Seen** time, or whether your gateway is on the network.

```yaml
alias: ThermoWorks - fast polling during a cook
triggers:
  - trigger: state
    entity_id: input_boolean.cook_mode
actions:
  - action: number.set_value
    target:
      entity_id: number.thermoworks_cloud_scan_interval
    data:
      value: "{{ 15 if is_state('input_boolean.cook_mode', 'on') else 1800 }}"
mode: restart
```

### Checking the current interval
To confirm how often the integration is polling, enable debug logging for ThermoWorks Cloud (Settings > Devices & services > ThermoWorks Cloud > ⋮ > Enable debug logging). Every poll then logs `Polling ThermoWorks Cloud API (interval: N seconds)`.

### More than one account
If you have added more than one ThermoWorks Cloud or ETI Cloud account, each account has its own scan interval and its own **Scan interval** entity. Changing one account's interval, from its entity or from **Configure** on its entry, reloads and affects only that account.

A second account with the same provider gets a numbered entity ID such as `number.thermoworks_cloud_scan_interval_2`, and both entities share the name **ThermoWorks Cloud Scan interval**. To tell them apart, rename one account's service device on the ThermoWorks Cloud integration page (for example to "ThermoWorks Cloud - Home"); Home Assistant offers to update the entity ID to match.

### Upgrading from an earlier version
Restart Home Assistant after updating so the new entity is created. Any scan interval you set before is kept, and the new entity starts with that value. Intervals are now limited to 24 hours: a longer one you set earlier stays in effect until you next save the integration options, which then reduce it to 24 hours.

## Also
To pull data, this integration uses [python-thermoworks-cloud](https://github.com/a2hill/python-thermoworks-cloud)