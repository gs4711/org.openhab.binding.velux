
# velux Binding

This binding integrates the <B>Velux<B> devices with help of a special gateway, the <B>Velux Bridge KLF200</B>.
The Velux Binding interacts via the Velux Bridge with the Velux devices like controlling window openers, shutters and others.

For details about the feature, see the following website ![Velux](http://www.velux.com) 

## Binding Configuration

The binding can be configured by parameters in the definition of Bridge thing as shown in the example below - or within the file `services/velux.cfg`.

| Property       | Default                | Required | Description                                           |
|----------------|------------------------|:--------:|-------------------------------------------------------|
| bridgeIPAddress| 127.0.0.1              |   Yes    | Name of address for accessing the Velux Bridge.       |
| bridgeTCPPort  | 80                     |    No    | TCP port for accessing the Velux Bridge.              |
| bridgePassword | velux123               |   Yes    | Password for authentication against the Velux Bridge. |
| timeoutMsecs   | 2000                   |    No    | Initial Connection timeout in milliseconds            |
| retries        | 6                      |    No    | Number of retries during I/O                          |

Advise: if you see a significant number of messages per day like

```
 "communicate(): socket I/O failed continuously (x times)."
```

please increase the parameters retries or/and timeoutMsecs.

Additionaly each scene Thing can process the following parameters:

| Property       | Default                | Required | Description                                           |
|----------------|------------------------|:--------:|-------------------------------------------------------|
| sceneName      |                        |   Yes    | Name of the scene to be handled.                      |

## Supported Things

The Velux Bridge in API version One (firmware version 0.1.1.*) allows to activate a set of predefined actions, so called scenes. Therefore beside the bridge, only one main thing exists, the scene element. Unfortunatelly even the current firmware version 0.1.1.0.44.0 does not include enhancements on this fact. Please refer to https://updates2.velux.com/ for more informations about available firmware upgrades.

String caveat: during the upgrade procedure of the KLF200, you'll probably loose any configured settings.

| Firmware revision | Release date | Description                                                             |
|:-----------------:|:------------:|-------------------------------------------------------------------------|
| 0.1.1.0.41.0      | 2016-06-01   | Default factory shipping revision.                                      |
| 0.1.1.0.42.0      | 2017-07-01   | N/A                                                                     |
| 0.1.1.0.44.0      | 2017-12-14   | N/A                                                                     |

## Discovery

Unfortunatelly there is no way to discover the Velux bridge within the local network. Beware that all Velux scenes have to be added to the local Velux Bridge configuration as described in the Velux setup procedure.

## Bridge / Thing Configuration

The Velux Bridge requires the URL as a configuration value in order for the binding to know how to access it.
Additionally, a refresh interval, used to poll the Velux system, can be specified (in seconds).

In the thing file, i.e. velux.things, this looks at least like

```
Bridge velux:klf200:home   [ bridgeIPAddress="velux-klf200.smile.de" ]
```

which allows to discover all scenes and instantiate them manually with GUI.


A more complex preconfigured thing file this looks e.g. like

```
Bridge velux:klf200:home    [ bridgeIPAddress="velux-klf200.smile.de", bridgeTCPPort=15001 ] {
    Thing   scene   windowClosed    [ sceneName="V_DG_Window_Mitte_000" ]
    Thing   scene   windowUnlocked  [ sceneName="V_DG_Window_Mitte_005" ]
    Thing   scene   windowOpened    [ sceneName="V_DG_Window_Mitte_100" ]
}
```


| Thing type | Description                                                               |
|------------|---------------------------------------------------------------------------|
| klf200     | The Velux KLF200 represents a gateway to all Velux devices.               |
| scene      | Named ordered set of product states which can be activated for execution. |


## Channels

The only currently available channel is the activation of some combined actions, in terms of <B>Velux</B> so-called scene.


| Channel Type ID | Item Type | Description                                                     | Thing types  |
|-----------------|-----------|-----------------------------------------------------------------|--------------|
| action          | Switch    | Activates a set of predefined product settings                  | scene        |
| silentMode      | Switch    | Modification of the silent mode of the defined product settings | scene        |
| status          | String    | Current Bridge State                                            | klf200       |
| doDetection     | Switch    | Start of the product detection mode                             | klf200       |
| firmware        | String    | Software version of the Bridge                                  | klf200       |
| ipAddress       | String    | IP address of the Bridge                                        | klf200       |
| subnetMask      | String    | IP subnetmask of the Bridge                                     | klf200       |
| defaultGW       | String    | IP address of the Default Gateway of the Bridge                 | klf200       |
| DHCP            | Switch    | Flag whether automatic IP configuration is enabled              | klf200       |
| WLANSSID        | String    | Name of the wireless network                                    | klf200       |
| WLANPassword    | String    | WLAN Authentication Password                                    | klf200       |

## Full Example

### Things

```
Bridge velux:klf200:home   [ bridgeIPAddress="velux-klf200.smile.de", bridgeTCPPort=15001, bridgePassword="velux123", timeoutMsecs=2000, retries=10 ] {
 Thing   scene   windowClosed    [ sceneName="V_DG_Window_Mitte_000" ]
 Thing   scene   windowUnlocked  [ sceneName="V_DG_Window_Mitte_005" ]
 Thing   scene   windowOpened    [ sceneName="V_DG_Window_Mitte_100" ]
}
```

### Items velux.items

```
/* Velux Bridge and Devices */

//  Group for simulating push buttons
Group:Switch:OR(ON, OFF) gV "PushButton"

// Velux Bridge channels

String  V_BRIDGE_STATUS     "Velux Bridge Status"               { channel="velux:klf200:home:status" }
String  V_BRIDGE_FIRMWARE   "Velux Bridge Firmware version"     { channel="velux:klf200:home:firmware" }
String  V_BRIDGE_IPADDRESS  "Velux Bridge IP Address"           { channel="velux:klf200:home:ipAddress" }
String  V_BRIDGE_SUBNETMASK "Velux Bridge IP Subnet Mask"       { channel="velux:klf200:home:subnetMask" }
String  V_BRIDGE_DEFAULTGW  "Velux Bridge Default Gateway"      { channel="velux:klf200:home:defaultGW" }
String  V_BRIDGE_DHCP       "Velux Bridge DHCP Enabled"         { channel="velux:klf200:home:DHCP" }
String  V_BRIDGE_WLANSSID   "Velux Bridge SSID"                 { channel="velux:klf200:home:WLANSSID" }
String  V_BRIDGE_WLANPASSWD "Velux Bridge WLAN Password"        { channel="velux:klf200:home:WLANPassword" }
Switch  V_BRIDGE_DO_DETECTION "Velux Bridge Detection mode" (gV) { channel="velux:klf200:home:doDetection" }

// Velux Scene channels

Switch  V_DG_M_W_OPEN   "Velux DG Window open"     (gV)         { channel="velux:scene:home:windowOpened:action" }
Switch  V_DG_M_W_UNLOCKED "Velux DG Window 05% open" (gV)       { channel="velux:scene:home:windowUnlocked:action" }
Switch  V_DG_M_W_CLOSED "Velux DG Window closed"   (gV)         { channel="velux:scene:home:windowClosed:action" }
```

### Sitemap velux.sitemap

```
sitemap velux label="Velux Environment"
{
    Frame label="Velux Shutter and Window" {

        Switch  item=V_DG_M_W_OPEN
        Switch  item=V_DG_M_W_UNLOCKED
        Switch  item=V_DG_M_W_CLOSED
    }
    
    Frame label="Velux Bridge" {
        Text    item=V_BRIDGE_STATUS
        Text    item=V_BRIDGE_FIRMWARE
        Text    item=V_BRIDGE_IPADDRESS
        Text    item=V_BRIDGE_SUBNETMASK
        Text    item=V_BRIDGE_DEFAULTGW
        Switch  item=V_BRIDGE_DHCP
        Text    item=V_BRIDGE_WLANSSID
        Text    item=V_BRIDGE_WLANPASSWD    
        Switch  item=V_BRIDGE_DETECTION  
    }
    
}
```

### Rule velux.rules

```
/**
 * This is a rules to simulate the push button behaviour...
 */
rule "PushButton of group gV"
    when
        Item gV changed
    then
        // waiting a second.
            Thread::sleep(1000)
        // Foreach-Switch-is-ON
        gV.allMembers.filter( s | s.state == ON).forEach[i|
            // switching OFF
                sendCommand(i, OFF)
        ]
    end
```

### Debugging

For those who are interested in more detailed insight view of the processing of this binding, a deeper look can be achieved by increased loglevel.

Try to add the following lines to  `/var/lib/openhab2/etc/org.ops4j.pax.logging.cfg` for common standalone installation:

```
### 
### Velux binding logging
###
log4j.logger.org.openhab.binding.velux = INFO

### For activation of a specific debugging, choose the following subtopics
#
## In-depth-observation of the bridge communication.
# 
# log4j.logger.org.openhab.binding.velux.bridge = TRACE
# 
## Analysis of the communication message structures.
# 
# log4j.logger.org.openhab.binding.velux.bridge.comm = TRACE
# 
## Debugging of Velux-OpenHab integration.
#
# log4j.logger.org.openhab.binding.velux.handler = TRACE
#
## For debugging the initial integration of things and channels, uncomment the following
#
# log4j.logger.org.openhab.binding.velux.internal = TRACE
#
## For debugging of Velux things, uncomment the following
#
# log4j.logger.org.openhab.binding.velux.things = TRACE
#

```

For using an IDE like Eclipse please use the specific entries within  `logback_debug.xml` within the usual package:

```

    <logger name="org.openhab.binding.velux" level="INFO" />

    <!-- For activation of a specific debugging, choose the following subtopics  -->

        <!-- In-depth-observation of the bridge communication.  -->
        <!-- 
    	<logger name="org.openhab.binding.velux.bridge" level="TRACE" />
        -->

        <!-- Analysis of the communication message structures.  -->
        <!-- 
    	<logger name="org.openhab.binding.velux.bridge.comm" level="INFO" />
        -->

        <!-- Debugging of Velux-OpenHab integration -->
        <!-- 
        <logger name="org.openhab.binding.velux.handler" level="TRACE" />
        -->

        <!-- For debugging the initial integration of things and channels, change the following to TRACE -->
        <!-- 
        <logger name="org.openhab.binding.velux.internal" level="TRACE" />
        -->

        <!-- For debugging of Velux things, change the following to TRACE -->
        <!-- 
        <logger name="org.openhab.binding.velux.things" level="TRACE" />
        -->

```

The changes will adapt the loglevel without restarting the framework.

During startup of normal operations, there should be only some few messages within the logfile, like:

```
2017-07-30 15:00:15.698 [INFO ] [b.v.handler.VeluxBridgeHandler:80   ] - Initializing Velux bridge handler for 'velux:klf200:home'.
2017-07-30 15:00:21.243 [INFO ] [b.v.handler.VeluxBridgeHandler:282  ] - handleCommand() found scenes 17 members: Scene "V_DG_Shutter_Ost_090" (index 4) with silent mode and 1 actions,Scene "V_DG_Shutter_West_090" (index 1) with silent mode and 1 actions,Scene "V_DG_Shutter_Ost_000" (index 3) with silent mode and 1 actions,Scene "V_DG_Shutter_Ost_100" (index 5) with silent mode and 1 actions,Scene "Sommertag" (index 11) with silent mode and 4 actions,Scene "V_DG_Shutter_West_100" (index 0) with silent mode and 1 actions,Scene "V_DG_Window_Mitte_080" (index 13) with silent mode and 1 actions,Scene "V_DG_Window_Mitte_070" (index 15) with silent mode and 1 actions,Scene "V_DG_Shutter_West_000" (index 2) with silent mode and 1 actions,Scene "V_DG_Window_Mitte_090" (index 12) with silent mode and 1 actions,Scene "V_DG_Window_Mitte_000" (index 9) with silent mode and 1 actions,Scene "V_DG_Shutter_Mitte_085" (index 7) with silent mode and 1 actions,Scene "V_DG_Window_Mitte_100" (index 10) with silent mode and 1 actions,Scene "V_DG_Shutter_Mitte_100" (index 8) with silent mode and 1 actions,Scene "V_DG_Shutter_Mitte_000" (index 6) with silent mode and 1 actions,Scene "V_DG_Window_Mitte_060" (index 14) with silent mode and 1 actions.
2017-07-30 15:00:25.423 [INFO ] [b.v.handler.VeluxBridgeHandler:294  ] - handleCommand() found products 4 members: Product "Rolladen Büro"/ROLLER_SHUTTER (bridgeIndex 3),Product "Rolladen Bad"/ROLLER_SHUTTER (bridgeIndex 2),Product "Bad"/WINDOW_OPENER (bridgeIndex 0),Product "Rolladen Schlafzimmer"/ROLLER_SHUTTER (bridgeIndex 1).
```


Another way to see what’s going on in the binding, is to switch the loglevel to DEBUG in the Karaf console:

```
log:set DEBUG org.openhab.binding.velux
```

If you want to see even more, switch to TRACE to also see the detailled bridge request/response data:

```
log:set TRACE org.openhab.binding.velux
```

To reset the logging back to normal:

```
log:set INFO org.openhab.binding.velux
```

And finally to identify startup problems, try the following:

```
stop org.openhab.binding.velux
log:set TRACE org.openhab.binding.velux
start org.openhab.binding.velux
log:tail
```

## Famous last words

All known <B>Velux<B> devices can be handled by this binding. In fact, there might be some new one which will be reported within the logfiles. Therefore, if you recognize an error message like:

```
20:59:05.721 [ERROR] [g.velux.things.VeluxProductReference] - PLEASE REPORT THIS TO MAINTAINER: VeluxProductReference(3) has found an unregistered ProductTypeId.
```

please pass the appropriate log line back to the maintainer to incorporate the new <B>Velux<B> device type.

## Document Revision

2018-06-18 Adapted to OpenHAB-v2.4.0-SNAPSHOT.
