
# Zigbee Custom Protocol

## Frame Format  

Protocol composition: `Header + Payload`  
Note: `u16/u32` in the protocol are all little-endian  

The protocol frame is as follows.  

Field | Bytes | Description
--- | --- | ---
 Header | 1 | The header is fixed to 0xFF
 Command | 1 | Command word
 Sequence | 1 | command sequence number, increases 1 for every message
 Total packets | 1 | Total number of packets
 Packet serial | 1 | Current packet serial number, serial starts with 0
 Data | N | Protocol payload

List of Command  

Command | Description
--- | ---
 0x99 | Ping response: the response packet sent by the slave device when it receives a Ping request from the gateway
 0x01 | Ping request, Gateway sends ping request, device replies with CMD 0x99
 0x02 | Gateway Query device status
 0x03 | Gateway reports network status to sub-device
 0x04 | The gateway issues DP commands
 0x05 | Reserve for future use
 0x06 | Device reporting status (active)
 0x0b | MCU version query request (for RAW module)
 0x0c | MCU OTA upgrade notification (for RAW Module)
 0x0d | MCU_OTA firmware content request (for RAW module)
 0x0e | MCU_OTA upgrade result reporting (for RAW module)
 0x24 | Sync Clock Time (check!!)
 0x41 | Eight-group private protocol (KID GID SID used for RAW Module)

## Command Payload  

Protocol payload format is different for each Command.

### Ping request payload content (CMD 0x00)

Direction: `Gateway -> Sub-device`  

The gateway detects the online or offline status of a sub-device by sending a Ping request and checking whether the corresponding sub-device has a Ping response.  

Field | Byte | Description
 --- | --- | ---
 flag | 1 | Bitwise flag <br/> bit0 -> Network flag, 1: Connected to network, 0: not connected <br/> Remaining reserved bits are 0
 reserve | 1 | for future use, default 0
 ReportWaitTime | 1 | Fixed delay time for reporting device status after multicast/broadcast control status, unit: s
 ReportRandomTime | 1 | Random delay time for reporting device status after multicast/broadcast control status, unit: s (Need to convert to ms and use it as the maximum value of random number to get random delay time)
 Divisor | 1 | non 0 value
 Remainder | 1 | remainder
 Max | 1 | Maximum value
 BaseTime | 1 | Base Time, unit: ms
 UTC | 4 | UTC time, unsigned integer, starting from 2000-01-01 00:00:00, 0 indicates invalid value, unit: seconds
 ZoneOffset | 4 | Time zone offset, signed integer, unit: seconds

The processing mechanism for whether the receiving node responds to the request is as follows:  

    Response conditions: Device unique identifier % Divisor == Remainder,
    Delayed response time: BaseTime * random (0, Max)
Among them, random(0, Max) indicates to take a random number between 0 - Max, including 0 and Max  
random() Generates a random number using the device short address as a random number seed  

Node status report parameter description:  

After receiving the multicast/broadcast control status, the node needs to delay for a fixed waiting time ReportWaitTime before taking the maximum value. The latest status is reported after the random delay time of ReportRandomTime is reached; the fixed delay time is used to avoid short-term. A large number of multicast/broadcast controls lead to frequent reporting by multiple nodes, reducing the probability of network congestion.  

When the values of ReportWaitTime and ReportRandomTime are both 0xFF, the multicast/broadcast control state does not need to be changed. To be reported.  

When the values are all 0, it indicates that a multicast/broadcast control status change has been received and needs to be reported immediately.  

For example:  

    ReportWaitTime = 10ÿReportRandomTime = 20

Indicates that after receiving the multicast/broadcast control, the node needs to first delay for 10 seconds, and then take a random value between 0 and 20*1000 milliseconds.  

The value starts with a random delay, and the status is reported after the random delay.

**TODO:** _Add diagram!_

### Ping Response (CMD 0x99)

Direction: `Sub-Device -> Gateway`  

When the child node receives the gateway PING request, it needs to calculate the delayed response time according to the PING request and whether it needs to respond.  

Field | Byte | Description
--- | --- | ---
 Application Version | 1 | Module application version number. The value is the same as the value of Basic Cluster Application Version Attribute
 MCU Version | 0/1 | MCU version number, the value is the same as the value of Basic Cluster MCU Version Attribute; if the module is not RAW module, then this field does not exists.

The version format is as follows:  

    u8 type data uses the xx.xx.xxxx format version number bit by bit, 
    that is, 1.0.0 can be represented by 0x40. The maximum version that can be represented is 3.3.15.

### Query DP  

Direction: `Gateway -> Sub-Device`  

Query the DP status value related to the sub-device. After receiving it, the sub-device returns the corresponding DP status value.  

Field | Byte | Description
--- | --- | ---
 DP_Num | 1 | The number of DPIDs to be queried. a value of `0x00` or `0xFF` means that all DPs are queried, and there is no subsequent DPID field number.
 DPID_0 | 1 | 1st DPID
 -- | 0/1 | --
 DPID_n | 0/1 | nth DPID

### Gateway reports network status  

Direction: `Gateway -> Sub-Device`  

When the gateway detects changes in its own network, it actively reports the network status to the corresponding sub-device through this message, which can be unicast, multicast, or broadcast.  

Field | Byte | Description
--- | --- | ---
 State | 1 | 0: Not connected to the network, 1: Connected to the network

### Set DP

Direction: `Gateway -> Sub-Device`  

Gateway command to set the DP status of the sub-device  

Field | Byte | Description
--- | --- | ---
 DP_Num | 1 | number of DP data, if it is 0, there is no subsequent field
 DPs | 0/n | Array of DP structures. For the DP structure format, see DataPoint. The length of this field depends on the number of DPs.

### Device reports DP status

Direction: `Sub-Device -> Gateway`  

After receiving the query/set DP status from the gateway, the sub-device needs to report the DP status to the gateway. In addition, when the device's own status changes DP status also needs to be reported.  

Field | Byte | Description
--- | --- | ---
 DP_Num | 1 | number of DP data, if it is 0, there is no subsequent field
 DPs | 0/n | Array of DP structures. For the DP structure format, see DataPoint. The length of this field depends on the number of DPs.

## MCU Firmware Upgrade  

The OTA process of MCU is as follows  

- The cloud sends an OTA notification, the gateway forwards it to the Zigbee module, and the module transparently transmits the data to the MCU
- After receiving the information, the MCU replies to the module, which then converts it into a Zigbee packet and sends it to the gateway to reply to the cloud.
- The MCU keeps polling to issue a firmware download request for the specified offset until the firmware transfer is complete.
- The gateway receives the firmware data that sends the specified offset.

### Get MCU version  

Direction: `Gateway -> Sub-Device`  

The gateway queries the MCU firmware version  

    No-Payload

The submodule sends a query to the MCU for response  

Direction: `Sub-Device -> Gateway`  

Field | Byte | Description
--- | --- | ---
 MCU Version | 1 | u8 type data uses the version number in the format of xx.xx.xxx bit by bit, that is, 1.0.0 can be represented by 0x40. A maximum of 3.3.15 versions can be represented

### Notify OTA upgrade  

Direction: `Gateway -> Sub-Device`  

Notify MCU to start OTA upgrade  

Field | Byte | Description
--- | --- | ---
 PID_Len | 1 | PID length
 PID | n | PID String
 Version | 1 | Upgraded version number, using the xx.xx.xxx format version number bit by bit
 File_size | 4 | Upgrade firmware file length
 Add_Sum | 4 | Firmware checksum, byte-by-byte sum from the first byte of the firmware

Submodule sends notification response  

Direction: `Sub-Device -> Gateway`

Field | Byte | Description
--- | --- | ---
 Status | 1 | 0: Success, 1: Fail

### Request MCU firmware data  

Direction: `Sub-Device -> Gateway`  

Module requests MCU firmware data.  

Field | Byte | Description
--- | --- | ---
 PID_Len | 1 | PID length
 PID | n | PID String
 Version | 1 | Upgraded version number, using the xx.xx.xxx format version number bit by bit
 File_Offset | 4 | Upgrade firmware offset
 Max_Len | 2 | Maximum length of the requested firmware data

Gateway responds to MCU firmware data.  

Direction: `Gateway -> Sub-Device`  

Field | Byte | Description
--- | --- | ---
 Result | 1 | 0: Success, 1: Failure
 PID_Len | 1 | PID length
 PID | n | PID String
 Version | 1 | Upgraded version number, using the xx.xx.xxx format version number bit by bit
 File_Offset | 4 | Data offset for upgrading firmware
 File_Data | n | Upgrade firmware data

### Notification of firmware download results

Direction: `Sub-Device -> Gateway`  

Field | Byte | Description
--- | --- | ---
 Result | 1 | 0: Success, 1:Failure
 PID_Len | 1 | PID length
 PID | n | PID String
 Version | 1 | Upgraded version number, using the xx.xx.xxx format version number bit by bit

Gateway responds to firmware download result notification

Direction: `Gateway -> Sub-Device`  

Field | Byte | Description
--- | --- | ---
 State | 1 | 0: Success, 1:Failure

## Setting button groups and scenes  

Mainly used for binding trigger storage of private scene ID and group ID of scene trigger button.  

After the data is sent, the MCU or module needs to store it by itself. When the corresponding button is triggered, the data is sent to the corresponding GID and SID scene.  

After creating a group through panel interaction, the cloud generates group/scene binding information and forwards it to the gateway, which then sends it to the module. It should be noted that the storage information needs to be allowed to be modified and replaced. Each time a different scene and group is bound, a newly generated GID and SID.  

After receiving the message, the sub-device sends a public protocol group-adding response to the gateway.  

Field | Byte | Description
--- | --- | ---
 Key_Num | 1 | The number of bound key IDs issued in this message
 KID0 | 1 | Key ID, used to trigger the index of scenes and groups
 GID0 | 2 | group ID is the identifier of the zigbee group
 SID | 1 | Scene ID is the identifier of the Zigbee scene
 ... | ... | ...

## OnOff Cluster Private Commands  

The following are new private commands created in OnOff Cluster, which are mainly used for switch or remote control related products. The specific instructions are as follows  

Command ID | Data Content | Data identifier (enum) | ID Identifier
---------- | ------------ | ---------------------- | -------
 0xFC | 0/1 | '0'/'1' | knob_switch_mode(n)
 0xFD | 1/2/3 | '0'/ '1'/ '2 ' | switch_type_(n)
 0xFD | 0x10 | scene | scene_(n)

This command is a hard-coded command and requires special attention during use.  

When the device uses the 0xFC command word to report 0/1/2 data, it means single click/double click/long press respectively. After the gateway receives the corresponding command, it
will convert it into a fixed dp identifier according to the definition and report it to the cloud. For example, as shown below.

Device report, endpoint_1 cluster onoff / command id 0xfc 0x00  

After the gateway receives it, it converts it into the identifier "knob_switch_mode_1". The data content is the enumeration type "0".

If there is relevant data in the object model and relevant content in the panel, it can be correctly identified.  
If not, it will fail. Multiple dp points are distinguished by sending multiple enpoints to achieve the purpose of multiple button control.

Device report, endpoint_1 cluster onoff / command id 0xfd 0x01  

After the gateway receives it, it concatenates and converts it into the identifier "switch_type_1" and the data content is the enumeration type "1"  

Device report, endpoint_1 cluster onoff / command id 0xfd 0x10  

After the gateway receives it, it concatenates and converts it into the identifier "scene_1" and the data content is the enumeration type "scene"  

## Mapping relationship between zigbee public attributes and DP  

When processing the public DP identifier, the gateway will first convert the command into a Zigbee protocol command for forwarding, and only send it through the private protocol
channel when it cannot be converted into the corresponding standard protocol. Therefore, when creating the function of the product, you should consider whether to convert it
into the Zigbee public protocol for sending. If the DP definition does not conform to the definition in the public DP table, the DP identifier cannot be defined as the public DP
identifier name listed below. In addition, when the gateway receives the public command report from the sub-device, it will also convert it into the corresponding identifier for
reporting. If there is no public identifier in the product creation, the public command report cannot be correctly recorded by the cloud, resulting in invalid data reporting.  

The public DP table is as follows  

Function ID | Data Type | Value Range | Description
----------- | ----------- | ----------- | ----------
 switch_led | PROP_BOOL | true/false | Public light bulb device switch control, the gateway converts the request into a public zigbee Switch control instructions are sent to subnodes
 switch_xx | PROP_BOOL | true/false | Public switch device switch control, xx means the element number starts from 1, the matching keyword is "switch_", for example, switch or switch1 are not considered public DP, while switch_1 is valid.
 bright_value | PROP_INT | 10 - 1000 | For brightness control of public light bulbs, the cloud sends 10 - 1000, and the gateway needs to convert it into the percentage value of brightness in the public protocol (10 - 1000 is to expand the control range and improve the dimming fineness of brightness setting)
 temp_value | PROP_INT | Color temperature data | Public light bulb equipment color temperature control, the gateway directly forwards the color temperature data in the cloud to the child node through the public protocol. The commonly used color temperature range is 2700K (cold) - 6500K (warm)
 battery_voltage | PROP_INT | 0x00-0xff | The device reports the battery voltage. For example, 29 represents 2.9V.
 battery_percentage | PROP_INT | 0x00-0xff | The device reports the remaining battery percentage. For example, 90 represents 90% remaining battery.

## Description of the associated device protocol

Knob dimmer controls the command of the light bulb device  

Cluster | CommandId | Value | Description
------- | --------- | ----- | ----------
 ONOFF(0x0006) | 0/1/2 | off/on/toggled | Public command control for turning on lights
 LEVEL_Control(0x0008) | ZCL_STEP_COMMAND(2) | 1-100 | Send brightness value percentage 1-100
 Color_Control(0x0300) | STEP_COMMAND(0x4c) | 150-500 | Color temperature command control step value (the light needs to be converted to 2700-6500)
 Color_Control(0x0300) | Color_Control_HUE(0) | 1-180 | RGB color control (H)
 Color_Control(0x0300) | Color_Control_SATURATION(1) | 1-250 | RGB color saturation control (S)

## DP Type is defined as  

```C

typedef enum {
    PT_DP_TYPE_UNKNOWN,
    PT_DP_TYPE_BOOL,    //1byte PROP_BOOL
    PT_DP_TYPE_INT,     //4byte  PROP_INT
    PT_DP_TYPE_FLOAT,   //4byte 
    PT_DP_TYPE_ENUM,    //1byte
    PT_DP_TYPE_FAULT, //1byte - fault is also enum in protco platform
    PT_DP_TYPE_TEXT,
    PT_DP_TYPE_RAW,
    PT_DP_TYPE_STRUCT,
    PT_DP_TYPE_ARRAY,
} TE_PT_DP_TYPE;

```
