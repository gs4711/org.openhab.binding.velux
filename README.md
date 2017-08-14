
# velux Binding

This binding integrates the <B>Velux<B> devices with help of a special gateway, the <B>Velux Bridge KLF200</B>.
The Velux Binding interacts via the Velux Bridge with the Velux devices like controlling window openers, shutters and others.

For details about the feature, see the following website ![Velux](http://www.velux.com) 

## Binding Configuration

The binding can be configured in the file `services/velux.cfg`.

| Property       | Default                | Required | Description                                           |
|----------------|------------------------|:--------:|-------------------------------------------------------|
| bridgeURL      | http://velux.bridge:80 |   Yes    | Basic bridge URL for accessing the Velux Bridge.      |
| bridgePassword | velux123               |   Yes    | Password for authentication against the Velux Bridge. |
| timeoutMsecs   | 2000                   |    No    | Initial Connection timeout in milliseconds            |
| retries        | 6                      |    No    | Number of retries during I/O                          |

Advise: if you see a significant number of messages per day like
```
 "communicate(): socket I/O failed continuously (x times)."
```
please increase the parameters retries or/and timeoutMsecs.

## Supported Things

The Velux Bridge in API version 1 allows to activate a set of predefined actions, so called scenes.  Therefore one main thing exists, the scene element.

## Discovery

Unfortunatelly there is no way to discover the Velux bridge within the local network. Beware that all Velux scenes have to be added to the local Velux Bridge configuration as described in the Velux setup procedure.

## Bridge / Thing Configuration

The Velux Bridge requires the URL as a configuration value in order for the binding to know how to access it.
Additionally, a refresh interval, used to poll the Velux system, can be specified (in seconds).

In the thing file, this looks e.g. like
```
Bridge velux:klf200:home   [ bridgeURL="http://my-klf200.velux.home" ] {
 Thing scene karlsruhe     [  ]
}
```

| Thing type | Description                                                              |
|------------|--------------------------------------------------------------------------|
| scene      | Named ordered set of product states which can be activated for execution |


## Channels

The only currently available channel is the activation of some combined actions, in terms of <B>Velux</B> so-called scene.


| Channel Type ID | Item Type | Description                               | Thing types  |
|-----------------|-----------|-------------------------------------------|--------------|
| ACTION          | Switch    | Switch state to ON for activating a scene | Any          |

## Full Example

### Things

```
Bridge velux:klf200:home   [ bridgeURL="http://192.168.0.111:80", bridgePassword="velux123", timeoutMsecs=2000, retries=10 ] {
 Thing scene karlsruhe     [ refreshSecs=120 ]
}
```

### Items velux.items

```
Switch V_SCENE_01  "Velux DG Window Open"            { channel="velux:scene:home:karlsruhe:ACTION#V_DG_Window_Mitte_000" }
Switch V_SCENE_02  "Velux DG Window Closed"          { channel="velux:scene:home:karlsruhe:ACTION#V_DG_Window_Mitte_100" }

Switch V_SCENE_03  "Velux DG Shutter Open"           { channel="velux:scene:home:karlsruhe:ACTION#V_DG_Shutter_Mitte_000" }
Switch V_SCENE_04  "Velux DG Shutter Mostly Closed"  { channel="velux:scene:home:karlsruhe:ACTION#V_DG_Shutter_Mitte_085" }
Switch V_SCENE_05  "Velux DG Shutter Closed"         { channel="velux:scene:home:karlsruhe:ACTION#V_DG_Shutter_Mitte_100" }
```

### Sitemap velux.sitemap

```
sitemap demo label="Main Menu"
{
    Frame label="Velux Shutter and Window" {
     Switch item=V_SCENE_01
     Switch item=V_SCENE_02
     Switch item=V_SCENE_03
     Switch item=V_SCENE_04
     Switch item=V_SCENE_05
   
    }
}
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

During startup of normal operations, there should be only a few messages within the logfile:

```
2017-07-30 15:00:15.698 [INFO ] [b.v.handler.VeluxBridgeHandler:80   ] - Initializing Velux bridge handler for 'velux:klf200:home'.
2017-07-30 15:00:15.711 [INFO ] [nding.velux.bridge.VeluxBridge:175  ] - communicationSetup(): retries = 10 times, initial wait interval = 2000 msecs.
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
