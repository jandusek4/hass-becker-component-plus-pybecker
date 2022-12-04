# Becker cover support for Home Assistant

A native home assistant component to control becker RF shutters with a Becker Centronic USB Stick.

It is based on the work of [ole](https://github.com/ole1986) and [Nicolas Berthel](https://github.com/nicolasberthel).

The becker integration currently support to
- Open a Cover
- Close a Cover
- Stop a Cover
- Set Cover position
- Track Cover position by travel time or template
- Track commands from Becker Remote

It tracks the position by using the travel time of your cover.

It support value template if you want to use sensors to set the current state of your cover.


## Installation

Add this repository to HACS custom repositories (preferred).

Alternatively copy this repository into the custom_components folder of your HA configuration directory

## Configuration

Once installed you can add a new cover configuration in your HA configuration

```yaml
cover:
  - platform: becker
    # Optional device path (useful when running from docker container)
    # Default device:
    # "/dev/serial/by-id/usb-BECKER-ANTRIEBE_GmbH_CDC_RS232_v125_Centronic-if00"
    device: "/dev/beckercentronicusb"
    # Optional database filename (database is stored in HA config folder)
    filename: "centronic-stick.db"
    covers:
      kitchen:
        friendly_name: "Kitchen Cover"
        channel: "1"
      bedroom:
        friendly_name: "Bedroom Cover"
        # Using Unit 1 - Channel 2
        channel: "2"
        value_template: "{{ states('sensor.bedroom_sensor_value') | float > 22 }}"
      livingroom:
        friendly_name: "Living room Cover"
        # Using Unit 2 - Channel 1
        channel: "2:1"
        # Optional Travel Time to track cover position by time
        # one time is sufficient if up and down travel time is equal
        travelling_time_up: 30
        # Optional Travel Time for direction down
        travelling_time_down: 27
        # Optional Remote ID from your Becker Remote, e.g. your master sender (multiple ID's separated by comma are possible)
        # to find out the Remote ID of your Becker Remote enable debug log for becker
        remote_id: "12345:2"
```

Note: The channel and remote_id needs to be a string!

## Note

To use your cover in HA you need to pair it first with the USB stick.

The pairing is always between your remote and the shutter. The shutter will react on the commands of all paired remotes.
Usually you already have programmed your original remote as the master remote. It is not recommended to program the USB stick as the master remote!
The USB stick is like an additional remote. Therefore the pairing procedure for the USB stick is the same as with additional remotes. Please refer to you manual for more details.

You have to put your shutter in pairing mode before. This is done by pressing the program button of your master remote until you hear a "clac" noise

To pair your shutter run the service becker.pair once (see HA Developer Tools -> Services). The shutter will confirm the successful pair with a single "clac" noise followed by a double "clac" noise.

Example data for service becker.pair:

```yaml
service: becker.pair
data:
  # Example data to pair your cover with USB stick unit 1 - channel 1
  channel: 1
  unit: 1
```

## Troubleshooting

If you have any trouble enable debug log for becker. Add the following lines to your configuration.yaml:

```yaml
logger:
  default: info
  logs:
    # This must be the folder name of your /config/custom_components/hass-becker-component folder
    custom_components.hass-becker-component: debug
```

This enable DEBUG messages to the home-assistant.log file in your config folder.

Also helpful to find out the Remote ID of your Becker Remote. The message will be something like below every time you press a key on your Remote:

... DEBUG ... \[custom_components.hass-becker-component...\] Received packet: **unit_id: 12345, channel: 2**, command: HALT, argument: 0, packet: ...
