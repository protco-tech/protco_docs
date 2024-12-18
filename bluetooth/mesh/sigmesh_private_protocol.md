# Sigmesh Private Communication Protocol - V0.1

This article defines the protocol format in the Protco Vendor Model to achieve private data transmission, which can achieve control logic or data transmission that cannot be achieved in the standard model of the Bluetooth Mesh Model.  

Considering the characteristics of Bluetooth MESH for transmitting small amounts of data, the protocol is based on the principle of simplification, that is, rules such as saving where possible and bit marking.  

## 1. Overview  

The main categories of Bluetooth mesh products include lighting, electrical, sensing, remote control, etc. When designing the Bluetooth mesh product system, the standard model of Bluetooth should be given priority to achieve its functions. For example, for the switch of the lighting category, the Generic On/Off Model is used.  

Due to the limited functionality of the standard Model, it is necessary for the manufacturer Model to define private features. For example, Bluetooth lighting products use the manufacturer Model to achieve scene configuration and switching.  

## 2. Vendor Model  

The vendor model must be configured on the first element of the device and support fast binding of App Key (shared binding) and fast subscription of subscription address (shared subscription). The details of sharing processing can be referred to " sigmesh node network access specification " "Other provisions" section  

### Model ID  

The Client model ID and server level model ID of the private model are described as follows:  

Vendor ID: 0x0211

- Server Model ID：0x0211 0000
- Client Model ID：0x0211 0001

### Model Cmd  

The command words supported by the private model are described as follows:  

Cmd | Opcode | Client：0x02110001 | Server：0x02110000
---- | ---- | ---- | ----
GET | 0xE80211 | Query data request | -
SET | 0xE90211 0XE90211 | Set up data requests with retransmission mechanism | -
SET_UNACK | 0xEA0211 | Setup data requests that do not require a response | -
STATUS | 0xEB0211 | - | Data response or proactive reporting
STC_REQ | 0xEC0211 | - | Server level data requests to clients
CTS_RSP | 0xED0211 | Client's data response to the server | -

### Command usage rules  

1. The private model must be configured on the first element of the node.  
2. The sender uses GET, SET, SET_UNACK model command words, and the receiver uses the STATUS model command word when sending a response  
3. The sender uses GET to send the private protocol, and the receiver needs to use STATUS when sending the private protocol response.  
4. The sender uses SET to send private protocols, and the receiver needs to use STATUS when sending private protocol responses.  
5. When the sender multicasts or broadcasts a private protocol, if SET_UNACK is used, the receiver cannot reply with STATUS after receiving it.  
6. When the sender multicasts or broadcasts a private protocol, if SET is used, the receiver needs to reply with STATUS after receiving it.  
7. If the server level needs to get data from the client, the server level uses the STC_REQ command to initiate the request, and the client uses the CTS_RSP to reply;  

### Package data length  

The Mesh protocol stipulates that the Access layer payload includes a maximum of 384 bytes in the TransMIC field, with a TransMIC length of 4 or 8 bytes, which is 4 bytes by default. The Access layer includes Opcode + payload, with Opcode lengths of 1, 2, or 3 bytes.  

When the length of Opcode is 3 bytes and the length of TransMIC is 8 bytes, the calculation formula for the maximum length of the load is as follows:  

Total data length before maximum subcontracting = 384 - 3 - 8 = 373  

Based on the above analysis, it is known that the maximum length of private Opcode transmission data is 373 bytes, excluding the number of bytes occupied by Opcode .  

## 3. Definition  

### Byte order  

IoT platforms mainly use DP to describe functions. When implementing functions using standard models, apps or gateways mainly do the following work:  

1. Query the standard DP and standard model mapping table, convert the DP into a standard model command, and send it to the Bluetooth mesh network.  
2. Convert the standard model command to DP and report to the cloud.  

When using the vendor model to implement DP data transmission function, the App or gateway will pass-through DP according to the following data structure:  

Field | Byte | Explanation
---- | ---- | -----
Dpid | 1 | DP function endpoint number, defined by the platform product
Type | 1 | Corresponding to the data type of Functional Button on the IoT platform, the specific definition is below
Len | 1 | Byte data length of the Data field. Note: The numeric types listed below do not have this field, only other types have this field. PROP_BOOL、PROP_INT、PROP_FLOAT、PROP_ENUM Such as type of PROP_BOOL, Data length is 1 by default. PROP_INT, Data length is 4 by default Data 1/4/N Hex indicates that numeric types larger than 1 Byte are transmitted using big endian

The definition of Type is as follows  

Type | Value | Length | Explanation
---- | ---- | ---- | ----
 Unknown | 0 | N | Unknown type
 BOOL | 1 | 1 | Boolean type, value range 0-1
 INT | 2 | 4 | Integer, need to be transmitted in the specified byte order, value range -2147483648~ 2147483647
 FLOAT | 3 | 4 | Floating-point type, needs to be transmitted in the specified byte order, with a value range of -2 ^ 128~ + 2 ^ 128, that is, -3.40E + 38~ + 3.40E + 38
 ENUM | 4 | 1 | Enumeration type, value range 0x00 - 0xFF
 FAULT | 5 | N | Error string type, length not exceeding 0xFF
 TEXT | 6 | N | Text string type, length not exceeding 0xFF
 RAW | 7 | N | Pass-through type, Hex (hexadecimal) ByteFlow data Example: For example, to transmit the string "123" "123" -> 0x31 0x32 0x33
 STRUCT | 8 | N | Structure type
 ARRAY | 9 | N |Array type

```C
typedef enum {
    PROP_UNKNOWN, //unknown type
    PROP_UNKNOWN, //unknown type
    PROP_BOOL, //1 byte numeric boolean
    PROP_BOOL, //1 byte numeric boolean
    PROP_INT, //4 byte numeric integer
    PROP_INT, //4 byte numeric integer
    PROP_FLOAT, //4 byte numeric floating point
    PROP_FLOAT, //4-byte numeric float
    PROP_ENUM, //1 byte numeric enumeration
    PROP_ENUM, //1 byte numeric enumeration
    PROP_FAULT, //error string
    PROP_FAULT, //Error string
    PROP_TEXT, //text string
    PROP_TEXT, //text string
    PROP_RAW, //raw hexadecimal byte array data
    PROP_RAW, //raw hexadecimal byte array data
    PROP_STRUCT, //Structure type
    PROP_STRUCT, //structure type
    PROP_ARRAY, //array type
    PROP_ARRAY, //array type
} DP_PROP_TP_E;
```

### Definition of Receiving and Receiving Objects  

Object | Explanation
---- | ----
 Gateway | Gateway
 CtrlNode | Control nodes mainly include switches, remote controls, sensors and other node devices
 TargetNode | Target nodes mainly include execution devices such as light bulbs and sockets

- The target node and the control node can coexist.
- When a Node sends a message to another Node, the Node that sends the message is called the controlling Node.
- When a node receives a message from a gateway or control node, it is called a target node.

## 4. Frame format  

In order to minimize data, it is compressed into a short broadcast single packet output, so the protocol is concise enough. The command length can be calculated and obtained based on the type of command received and the length of the received data.  

The frame format is: Model Cmd Opcode (3 Bytes) + Cmd (1 Byte) + Data (N Bytes).  

Field | Byte | Explanation
---- | ---- | ----
 Opcode  | 3 | Opcode for private Model Cmd, see the "Model Cmd" section for specific type definitions
 Cmd | 1 | Command types, for specific type definitions, please refer to the "Command Type Table" section
 Data | N | Command load data, for specific load format, please refer to the "Command Load Format" section

The maximum effective length of a single packet is 11 Bytes, so the maximum length of a single packet of Data is 7 Bytes  

## 5. Command type table  

### Command word table between gateway and node  

The command word range is \[0x00 - 0x3F].  

Model Cmd | Command | Value | Explanation
---- | ---- | ---- | ----
 SET_NOACK | PING | 0x00 | Heartbeat PING request, gateway PING node The SET_NOACK model command word is used because PING_ACK is not an immediate response and does not require the use of a reliably transmitted SET model command
 STATUS | PING_ACK | 0x01 | Response to heartbeat PING request, node response gateway
 GET | GET_DP | 0x02 | Query node DP
 SET | SET_DP | 0x03 | Set node DP
 STATUS | DP_ACK | 0x04 | Query/set the response of node DP
 STATUS | DP_REPORT | 0x05 | Node actively reports DP status
 SET | BIND_DP | 0x06 | Bind node DP, set the relationship between the control node DP and the controlled node DP
 STATUS | BIND_DP_ACK | 0x07 | Response of binding node DP
 SET | SET_SCENE | 0x08 | Set node scene
 STATUS | SET_SCENE_ACK | 0x09 | Set response for node scene
 SET | RECALL_SCENE | 0x0A | Recovery scenario
 STATUS | RECALL_SCENE_ACK | 0x0B | Respond to scene
 SET | SET_DP_GROUP | 0x0C | Set DP group
 STATUS | SET_DP_GROUP_ACK | 0x0D | Set response for DP group
 STC_REQ | GET_GW_TIME | 0x0E | Get the gateway time, the node actively gets the gateway time.
 CTS_RSP | GET_GW_TIME_ACK | 0x0F | Obtain the response of the gateway time, and the gateway passively responds to the node time
 SET | CLEAR_DATA | 0x10 | Clear node data
 STATUS | CLEAR_DATA_ACK | 0x11 | Response to clear node data
 SET | BIND_REMOTE_CTRL | 0x12 | Set up node binding remote control for bulb node, supporting at least 8 binding table entries
 STATUS | BIND_REMOTE_CTRL_ACK | 0x13 | Set the response of the node binding remote control
 SET | UNBIND_REMOTE_CTRL | 0x14 | Set node unbind remote control for bulb node
 STATUS | UNBIND_REMOTE_CTRL_ACK | 0x15 | Set the node to unbind the response of the remote control

### Command word table between nodes  

The command word range is \[0x40 - 0x7F].  

Model Cmd | Command | Value | Explanation
---- | ---- | ---- | ----
 SET | LOCAL_SYNC_DOUBLE_CTRL_STATE | 0x40 | Local synchronous dual-control status command for dual-control switching lamp equipment
 STATUS | LOCAL_SYNC_DOUBLE_CTRL_STATE_ACK | 0x41 | Local synchronous dual control status command response (reserved, no response between nodes for the time being)
 SET | LOCAL_RECALL_SCENE | 0x42 | Local restore scene for scene switch light device
 STATUS | LOCAL_RECALL_SCENE_ACK | 0x43 | Local recovery scene response (reserved, no response between nodes for the time being)
 SET | LOCAL_SET_DP | 0x44 | Local setting node DP for scene switching lamp equipment
 STATUS | LOCAL_SET_DP_ACK | 0x45 | Local setting node DP response (reserved, no response between nodes)
 SET_UNACK | LOCAL_REMOTE_CTRL_BIND | 0x46 | The local remote control is bound to a device in a bound state, and the bound device supports at least 8 bound table entries. <br/>For example: the light bulb will be in the binding and unbinding state within 16 seconds of restarting, and it will flash once after successful binding
 SET_UNACK | LOCAL_REMOTE_CTRL_UNBIND | 0x47 | Local remote control unbind device
 SET_UNACK | LOCAL_REMOTE_CTRL_XXX | 0x48-0x4F | Local remote control management commands are reserved for future use
 SET_UNACK | LOCAL_REMOTE_CTRL_ONOFF | 0x50 | Local remote control sets the switch of the target node
 SET_UNACK | LOCAL_REMOTE_CTRL_SET_LEVEL | 0x51 | Set the brightness of the target node with the local remote control
 SET_UNACK | LOCAL_REMOTE_CTRL_SET_COLOR_TEMP | 0x52 | Set the color temperature of the target node with the local remote control
 SET_UNACK | LOCAL_REMOTE_CTRL_SET_COLOR | 0x53 | Set the color of the target node with the local remote control
 SET_UNACK | LOCAL_REMOTE_CTRL_MOVE_LEVEL | 0x54 | Adjust the brightness of the target node with the local remote control
 SET_UNACK |LOCAL_REMOTE_CTRL_MOVE_COLOR_TEMP | 0x55 | Local remote control adjusts the color temperature of the target node
 SET_UNACK | LOCAL_REMOTE_CTRL_MOVE_HUE | 0x56 | Local remote control adjusts the chromaticity of the target node
 SET_UNACK | LOCAL_REMOTE_CTRL_SET_LEVEL_COLOR_TEMP | 0x57 | The local remote control sets the brightness and color temperature of the target node
 SET_UNACK | LOCAL_REMOTE_CTRL_SCENE_SWITCH | 0x58 | Local remote control switches target node default scene
 SET_UNACK | LOCAL_REMOTE_CTRL_SCENE_SAVE | 0x59 | The local remote control sets the target node to save the scene, and the target device supports at least 8 custom scene table entries.
 SET_UNACK | LOCAL_REMOTE_CTRL_SCENE_RECALL | 0x5A | Set target node recovery scene for local remote control
 SET_UNACK | LOCAL_REMOTE_CTRL_SCENE_DELETE | 0x5B | Set target node delete scene with local remote control

## 6. Interaction description  

### Local remote control interaction logic  

When the local remote control is bound to the target device, the binding table entry is only saved in the target device, and the remote control itself does not save any data. The key functions of the remote control send fixed commands in a broadcast address (0xFFFE) manner The target device receives the command and recognizes whether it is the short address of the bound remote control. If not, the command is discarded. If not, it is judged whether the specified key group is bound or not. If not, the command is discarded. If not, the command is executed.  

The command prefix " LOCAL_REMOTE_CTRL " in the command word table between the above nodes belongs to the command word described in this chapter.  

The explanation of binding table entry fields is as follows:  

Field | Byte | Explanation
---- | ---- | ----
 Group_ID | 2 | Group ID for grouping
 Short_Addr | 2 |  Short address of the bound remote control
 Key_DPID | 1 | The grouping button DPID of the remote control is obtained from the product function interface of the IOT platform
 DPID_Num | 1 | Number of DPID bindings, If it is 0, it represents all, and the length of the DPIDs field is 0. If it is not 0, it means that only some DPs specified by this node are bound.
 DPIDs | 0/N | Array of DPIDs, with a single DPID length of 1 byte. The length of this field is determined by the number of DPIDs. For example, dpid0, dpid1, dpid2...

Key scene table entry field description is as follows:  

Field | Byte | Explanation
---- | ---- | ----
 Short_Addr | 2 | Short address of the bound remote control
 Flag | 1 | 8-Bit unsigned data, value by bit. Bit0: Key_DPID valid flag to specify whether key grouping or all groupings 1: valid, 0: invalid. <br/>When it is 0, the target node does not need to determine whether the specified Key_DPID has been bound, only to identify whether the short address of the remote controller in the binding table. <br/>Other bits are reserved and filled with 0.
 Key_DPID | 1 |  DPID values of grouped buttons are obtained from the product function interface of the IOT platform. <br/> When bit0 of Flag is 0, any value can be filled at will
 Key_Scene_ID | 1 | Key value of scene key, value range 0x00 - 0xFF
 DP_Num | 1 | Record the number of DP in the current state
 DPs | N | DP structure array, DP structure format view DataPoint <br/>The length of this field depends on the number of DP. <br/>For example: DP0, DP1, DP2...

When deleting the binding table, it is also necessary to synchronously delete the corresponding key scene table.  

### Set the interaction logic for adding or deleting groups on a node

Fully use standard public protocols to set nodes to add or delete groups, that is, subscribe to group addresses or delete subscribed group addresses  

## 7. Command payload format between gateway and node  

Please refer to the "Command Type Table" section for specific command type definitions  

### PING  

Direction: Gateway - > TargetNode  

Gateway PING node request refers to the request of the gateway to actively query whether the long power (non-low power) node is online. By controlling whether the node responds and the response delay time through request parameters, it avoids the situation that the receiving node sends a response to the gateway at the same time, causing network congestion. This command is mainly used for gateway broadcast transmission (or unicast). By controlling the change of divisor and remainder, all sub-nodes can respond in batches and with random delay. Please refer to the offline detection mechanism chapter of " sigmesh node network access specification " for the offline detection mechanism of non-low power devices and low power devices.  

Field | Byte | Explanation
---- | ---- | ----
 Divisor | 1 | Unsigned integer; divisor, non-zero value;
 Remainder | 1 | Unsigned integer; remainder;
 Max | 1 | Unsigned integer; maximum delay, sub-node needs to take a random number Random between \[0 - Max], including 0 and Max <br/>(Max value needs to consider that the calculated random delay time must be less than the PING request interval and cannot be equal to it. Otherwise, in extreme cases, it may cause the gateway to receive a response from the sub-device after executing the offline detection mechanism.)
 BaseTimeMs | 1 | Unsigned integer; delay reference time, unit: ms; delay response calculation formula is: Random * BaseTimeMs
 ReportWaitTime | 1 | Unsigned integer; fixed delay time for reporting device status after multicast/broadcast control status, unit: seconds <br/>(Fixed delay time is to solve the problem of network congestion caused by frequent reporting of status due to receiving multiple commands in a short period of time.)<br/>Value selection description: <br/> 0X00 - Fixed delay time is 0, no delay is required. <br/>0xFF - Permanent delay, which means that the device status is not reported after multicast/broadcast control. <br/>Other values - delay time;
 ReportRandomTime | 1 | Unsigned integer; Random delay time for reporting device status after multicast/broadcast control status, unit: seconds <br/>(Need to convert seconds to milliseconds and treat it as the maximum random number to obtain the random delay time) <br/>Value selection description: <br/>0X00 - Random delay time is 0, no delay is required.<br/>Other values - delay time;

Node status delay report parameter description:  

After the node receives the multicast/broadcast control status command, it needs to delay the fixed waiting time ReportWaitTime first, and then expand the ReportRandomTime seconds by 1000 times and convert it into a millisecond value. The random value between \[0- (ReportRandomTime * 1000) ] is taken as the random delay millisecond number. When the random delay expires, it reports the latest status of the device. The role of the fixed delay time is to avoid the occurrence of network congestion caused by frequent reporting by multiple nodes due to a large number of multicast/broadcast controls in a short time.  

For example:  
