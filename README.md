# HAPT

Home Assistant Presence Tracker (HAPT) is an event-driven device presence tracker for [Home Assistant][homeassistant] on
an OpenWRT router or access point.

## Description

HAPT listens on association and disassociation events to wireless networks, using the hostapd control interface. It
keeps track of which device is connected to which networks. When a device connects to its first network, or disconnects
from its last network, a service call to Home Assistant is performed to mark the device as home or away. By tracking
active device connections, HAPT ensures that a device switching between different networks (e.g. the 2.4 GHz and 5 GHz
bands) is not marked as away.

## Usage

### Installation

- **25.12 and newer:** Download a package **with the** `.apk` **file extension** from the [releases][releases] page, and install it by running `apk add --allow-untrusted <file>` from a shell. Installation from the web interface is not currently possible.
- **Older than 25.12:** Download a package **with the** `.ipk` **file extension** from the [releases][releases] page, and install it either by uploading it in LuCi (System > Software) or running `opkg install <file>` from a shell.

### Configuration
Once the package is installed, you must update the configuration in `/etc/config/hapt`. At minimum the `host` option
should be set to the URL of your Home Assistant installation (including the scheme, e.g. `http://homeassistant:8123`),
and the `token` option to a Home Assistant [long-lived access token][token] (these can be generated in the Home
Assistant web interface, on the Security page of your user profile).

As `device_tracker.see` has been deprecated in Home Assistant, it is now required to create an input_boolean for each device that you want to track. Since that's the case, it is strongly advised that you whitelist the specific devices that you want to be tracked using the `track_mac_address` option, as otherwise hapt will send a bunch of invalid requests to Home Assistant. This is very much non-ideal, but since device_tracker got unreasonably complicated to use with the deprecation of the `.see` method, I think it's a fair trade-off.

The input_boolean's entity IDs should be formatted as such:
- If the device has a hostname in the network, "hapt_ + the hostname with all dots (.) replaced with an underscore (_)", e.g. for "pixel.lan" the entity ID will be `input_boolean.hapt_pixel_lan`
- If the device only has a MAC address, "hapt_ + MAC address with all colons (:) replaced with an underscore (_)", e.g. for "00:11:22:33:44:55" the entity ID will be `input_boolean.hapt_00_11_22_33_44_55`

With the `wifi_interfaces` option, it is possible to specify the wireless interfaces that must be monitored. This can be
used (for example) to ignore devices on a guest network.

By listing MAC addresses in the `track_mac_address` option, it is possible to whitelist MAC addresses which are tracked.
This prevents uninteresting devices from being synchronized with Home Assistant and cluttering the entity registry.

In previous versions there also used to be options for the Home and Away times (e.g. a timeout after disconnection, and a maximum allowed time for the device to be considered home without reconnecting). Both of these are now omitted, as they were there because of a requirement in the `device_tracker` API. Instead, you should use the "for" option in Home Assistant. For example, this is how I prevent Home Assistant from turning off the lights if my phone disconnects for just a few seconds:
<img width="1240" height="733" alt="image" src="https://github.com/user-attachments/assets/89321e29-2a4e-499b-9bed-1e87596514dd" />
A similar automation can be created to keep a "maximum considered home time" function, although I fail to see how that is particularly useful.

### Running
After modifying the configuration, you must restart the service by `service hapt restart`. This can also be done from
the LuCi interface (System > Startup). HAPT prints log messages to the system log, so that you can verify it is working
as expected.

It is possible to synchronize just the currently connected devices with Home Assistant by running `hapt` from the
command line. This can be especially useful to debug the connection with Home Assistant, as this will also print any
errors that occur.

## Development
You can build a custom package by running the `makepkg.sh` script, which will run the package build in a Docker
container and place the compiled package in the `build/bin` directory.

## Acknowledgments

This project has been inspired by the [openwrt_hass_devicetracker][hasstracker] package.

[homeassistant]: https://www.home-assistant.io/
[hasstracker]: https://github.com/mueslo/openwrt_hass_devicetracker
[releases]: https://github.com/oxan/hapt/releases
[token]: https://developers.home-assistant.io/docs/auth_api/#long-lived-access-token
