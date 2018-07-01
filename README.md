# OpenWebNet (BTicino/Legrand) Binding

This new binding integrates BTicino / Legrand MyHOME(r) BUS & ZigBee Radio devices using the **[OpenWebNet](https://en.wikipedia.org/wiki/OpenWebNet) protocol**.
It is the first known binding for openHAB 2 that **supports *both* wired BUS/SCS** as well as **wireless ZigBee setups**, all in the same biding. The two networks can be configured simultaneously.
It's also the first OpenWebNet binding with initial support for discovery of BUS/SCS devices.
Commands from openHAB and feedback (events) from BUS/SCS and ZigBee networks are supported.

Support for both numeric (`12345`) and alpha-numeric (`abcde` - HMAC authentication) passwords is included.

## Prerequisites

In order for this biding to work, an OpenWebNet gateway is needed in your home system to talk to devices.
Currently these gateways are supported by the binding:

- **IP gateways** or scenario programmers, such as BTicino 
[F454](http://www.homesystems-legrandgroup.com/BtHomeSystems/productDetail.action?productId=006), 
[MyHOMEServer1](http://www.bticino.com/products-catalogue/myhome_up-simple-home-automation-system/), 
[MH202](http://www.homesystems-legrandgroup.com/BtHomeSystems/productDetail.action?productId=059), 
[F455](http://www.homesystems-legrandgroup.com/BtHomeSystems/productDetail.action?productId=051),
[MH200N](http://www.homesystems-legrandgroup.com/BtHomeSystems/productDetail.action?productId=016), 
[F453](http://www.homesystems-legrandgroup.com/BtHomeSystems/productDetail.action?productId=027),  etc.
- **ZigBee USB gateways**, using USB/serial ports such as [BTicino 3578](http://www.catalogo.bticino.it/BTI-3578-EN) and [Legrand 088328](https://www.legrand.fr/pro/catalogue/35115-interface-openradio/interface-open-webnet-et-radio-pour-pilotage-dune-installation-myhome-play)

## Supported Things

The following Things and OpenWebNet `WHOs` are supported:
### BUS/SCS

| Category   | WHO   | Thing Type IDs                    | Discovery?          | Feedback from BUS?          | Description                                                                             | Status           |
| ---------- | :---: | :-------------------------------: | :----------------: | :----------------: | --------------------------------------------------------------------------------------- | ---------------- |
| Gateway    | `13`  | `bus_gateway`                     | *work in progress*                | n/a  | Any IP gateway supporting OpenWebNet protocol should work (e.g. F454/MyHOMEServer1/MH202/F455/MH200N,...) | Successfully tested: F454, MyHOMEServer1, MH202, F455  |
| Lightning | `1`   | `bus_on_off_switch`, `bus_dimmer` | Yes                | Yes                | BUS switches and dimmers                                                                 | Successfully tested: F411/2, F411/4, F411U2, F429  |
| Automation | `2`   | `bus_automation`                | Yes | Yes                  | BUS roller shutters, with position feedback and auto-calibration via a *UP >> DOWN >> Position%* cycle                                                                                       | Successfully tested: LN4672M2  |
| Temperature Control | `4`   | *work in progress*  |  |  |  |


### ZigBee (Radio)

| Category   | WHO   | Thing Type IDs                               |    Discovery?  | Feedback from Radio? | Description                                                 | Status                               |
| ---------- | :---: | :------------------------------------------: | :----------------: | :--------: | ----------------------------------------------------------- | ------------------------------------ |
| Gateway    | `13`  | `dongle`                                     |     Yes            | n/a         | ZigBee USB Dongle (BTicino/Legrand models: BTI-3578/088328) | Tested: BTI-3578                     |
| Lightning| `1`   | `dimmer`, `on_off_switch`, `on_off_switch2u` | Yes                | Yes        | ZigBee dimmers, switches and 2-unit switches                | Tested: BTI-4591, BTI-3584, BTI-4585 |
| Automation | `2`   | `automation`                           | Yes | Yes          | ZigBee roller shutters, with position feedback and auto-calibration via a *UP >> DOWN >> Position%* cycle                                                           | *To be tested*    |

## Installation

Since it's still in development stage and not part of the official distribution, to install the binding copy the jar file to your openHAB2 addons folder, on Linux or RaspberryPi is under: 

`/usr/share/openhab2/addons/`

Also, from **openHAB 2.3.0** onwards the serial feature dependency must be activated manually from Karaf console:

1. go to [Karaf console](https://www.openhab.org/docs/administration/console.html)
1. Type `feature:install openhab-transport-serial`

The binding should now be installed: check in *PaperUI > Configuration > Bindings*.

### Upgrade from previous release version

The previous version of the binding must be uninstalled using Karaf console (is not enough update the binding jar file).
It is suggested also to remove BTicino Things before uninstalling the old binding, and  discover/configure them again after binding has been updated.

from Karaf console:

1. `bundle:list` to list all bundles and take note of the bundle ID for OpenWebNet Binding
1. `bundle:uninstall <ID>` to remove previous version of the binding
1. copy the new binding jar file to `addons/` folder

The new version of the binding should now be installed, check the version number in *PaperUI > Configuration > Bindings*.

## Discovery

Things discovery is supported using PaperUI by activating the discovery ("+") button form Inbox.

### BUS/SCS Discovery

- Gateway discovery using UPnP is *under development* and will be available only for IP gateways supporting UPnP.
- For the moment the OpenWebNet IP gateway should be added manually (see BUS/SCS Gateway configuration below), configuring the password if needed.
- Once the gateway is added manually as a Thing, a second discovery request from Inbox will discover its devices.
- BUS/SCS Dimmers must be ON and dimmed (20-100%) at time of discovery, otherwise they will be discovered as simple On/Off switches.

### ZigBee Discovery

- The USB dongle must be inserted in one of the USB ports of the openHAB computer before discovery is started
- ***IMPORTANT NOTE:*** As for the OH serial binding, on Linux the `openhab` user must be member of the `dialout` group, to be able to use USB/serial port:

    ```
    $ sudo usermod -a -G dialout openhab
    ```

    + The user will need to logout and login to see the new group added. If you added your user to this group and still cannot get permission, reboot Linux to ensure the new group permission is attached to the `openhab` user.
- Once the gateway is discovered and added, a second discovery request from Inbox will discover devices. Because of the ZigBee radio network, discovery will take ~40-60 sec. Be patient!
- Radio devices must be part of the same ZigBee network of the USB dongle to discover them. Please refer to [this guide by BTicino](http://www.bticino.com/products-catalogue/management-of-connected-lights-and-shutters/#installation) to setup a ZigBee network which includes the USB dongle
- Only powered radio devices part of the same ZigBee network and within radio coverage of the USB dongle will be discovered. Unreachable or not powered devices will be discovered as *GENERIC* devices and cannot be controlled. Control units cannot be discovered by the USB dongle and therefore are not supported


## Thing Configuration

### BUS/SCS Gateway

To configure the gateway: go to Inbox > "+" > OpenWebNet > click `ADD MANUALLY` and then select `OpenWebNet BUS Gateway` device, with this configuration:

- `host` : IP address / hostname of the BUS/SCS gateway (*mandatory*). Example: `192.168.1.35`
- `port` : port (*optional*, default: `20000`)
- `passwd` : gateway password (*optional*). Example: `abcde`
   - if the BUS/SCS gateway is configured to accept connections from the openHAB computer IP address, no password should be required

#### Example

```
Bridge openwebnet:bus_gateway:myBridge1 [ host="192.168.1.35", passwd="abcde" ]
```

**HELP NEEDED!!!**
Start a gateway discovery, and then send your (DEBUG-level) log file to the openHAB Community OpenWebNet thread to see if UPnP discovery is supported by your BTicino IP gateway.


### ZigBee USB Dongle

The ZigBee USB dongle is discovered automatically and added in Inbox. Manual configuration is not supported at the moment.

### Devices

For all OpenWebNet devices it must be configured:

- the associated gateway (`Bridge Selection` menu)
- the `where` config parameter (`OpenWebNet Device Address`):
  - example for BUS/SCS: Point to Point `A=2 PL=4` --> `where="24"`
  - example for BUS/SCS: Point to Point `A=6 PL=4` on local bus --> `where="64#4#01"`
  - example for ZigBee/Radio: use decimal format address without the UNIT part and network: ZigBee `WHERE=414122201#9` --> `where="4141222"`


## Channels

Devices support some of the following channels:

| Channel Type ID               | Item Type          | Description                                                            | Read/Write |
|-------------------------------|--------------------|------------------------------------------------------------------------|:----------:|
| switch                        | Switch             | To switch the device `ON` and `OFF`                                  |     R/W    |
| brightness                    | Dimmer             | To adjust the brightness value (Percent)                               |    R/W     |
| shutter                       | Rollershutter      | To activate roller shutters (`UP`, `DOWN`, `STOP`, Percent). For Percent and position feedback to work the `shutterRun` parameter must be configured equal to the time (in ms) to go from full UP to full DOWN. Use `shutterRun=AUTO` (default) to calibrate the shutter automatically the first time a Percent command is sent to the shutter: a *UP >> DOWN >> Position%* cycle will be performed automatically to calibrate the shutter. Before adding/configuring roller shutter Things (or installing a binding update) it is suggested to have all roller shutter `UP`, otherwise the Percent command won’t work until the roller shutter is fully rolled up |     R/W     |


## Full Example

### demo.things:

```
Bridge openwebnet:bus_gateway:mybridge1 [ host="192.168.1.35", passwd="abcde" ] {
      bus_on_off_switch   myswitch   [ where="64#4#01" ]
      bus_dimmer          mydimmer   [ where="24" ]
      bus_automation      myshutter  [ where="53", shutterRun="12000"]
}
``` 

```
<TODO----- ZigBee Dongle>
Bridge openwebnet:dongle:mydongle2  [serialPort="kkkkkkk"] {
      dimmer          myzigbeedimmer [ where="xxxxx"]
      on_off_switch   myzigbeeswitch [ where="yyyyy"]
}
```

### demo.items:

```
Switch BUS_Light           { channel="openwebnet:bus_on_off_switch:mybridge1:myswitch:switch" }
Dimmer BUS_Dimmer          { channel="openwebnet:bus_dimmer:mybridge1:mydimmer:brightness" }
Rollershutter BUS_Shutter  { channel="openwebnet:bus_automation:mybridge1:myshutter:shutter" }
```


### demo.sitemap

```
<example not available yet>
```

## Disclaimer

- This binding is not associated by any means with BTicino or Legrand companies
- The OpenWebNet protocol is maintained and Copyright by BTicino/Legrand. The documentation of the protocol if freely accessible for developers on the [MyOpen Community website - https://www.myopen-legrandgroup.com/developers](https://www.myopen-legrandgroup.com/developers/)
- OpenWebNet and MyHOME are registered trademarks by BTicino/Legrand
- This binding uses `openwebnet-lib 0.9.x`, an OpenWebNet Java lib partly based on [openwebnet/rx-openwebnet](https://github.com/openwebnet/rx-openwebnet) client library by @niqdev, to support:
  - gateways and OWN frames for ZigBee
  - frame parsing
  - monitoring events from BUS

  The lib also uses few classes from the openHAB 1.x BTicino binding for socket handling and priority queues.

## Changelog

**v2.4.0-b6** - 02/07/2018

- updated to openHAB 2.4.0 dev branch. The bundle should work with both 2.2 and 2.3 versions of openHAB.
- **[FIX #7]** added support for inverted UP/DOWN automation commands for older USB ZigBee dongles
- **[BUG]** some switches were wrongly discovered as dimmers (now use only commands for device discovery)
- **[FIX]** added support for SCS/ZIGBEE_SHUTTER_SWITCH (515/513) device types
- **[FIX]** added support for F455 gateways using `*99*0##` command session

**v2.3.0-b5** - 26/05/2018

- [BUG #1] state monitoring from BUS (feedback) is no longer stopped if unsupported messages are received from BUS
- [FIX] automatic reconnect to BUS when connection is lost
- [NEW] support for Gateways with string passwords (HMAC authentication), like MyHOMEServer1 
- [NEW] support for `WHO=2` Automation (shutters), both on BUS and ZigBee, with position feedback and  goto Percent. It requires setting the shutter run-time in the thing configuration. Experimental auto-calibration of the run-time is also supported!

**v2.3.0-b4** - 09/04/2018

- first public release
