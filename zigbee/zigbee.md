
# Protco Zigbee

## Zigbee Basic

Profile ID: 0x0104  
Endpoint: 1  
Endpoint 1 is used for data exchange between gateway and the device.  
Basic cluster (0x0000) configuration  

The Zigbee protocol stipulates that the maximum size of a single packet of air data transmission is 127 bytes, of which the fixed frame header and frame tail of the MAC, NWK and other layers are 45 bytes.  

The remaining ZCL layer data that can be effectively used is only 82 bytes. Try to limit the data within a single packet.  

The gateway has enabled the Zigbee unpacking and packaging function by default. The maximum length of the packet is 512, and the maximum effective length of a single data packet is 512byte.  

## Basic Cluster  

During the network access process, the gateway will query the relevant attribute values of the Basic Cluster of the sub-device; the specific attribute values used are defined as follows  

Attribute ID | Attribute | Type | Value/Description
--- | --- | --- | ---
 0x0000 | ZCL version | INT8U | 0x08 default
 0x0001 | Application version | INT8U  | See description below
 0x0002 | Stack version | INT8U | &nbsp;
 0x0003 | Hardware version | INT8U | &nbsp;
 0x0004 | Manufacturer name | CHAR_STRING | PROTCO
 0x0005 | Model Identifier | CHAR_STRING | &nbsp;
 0x0007 | Power source | ENUM8 | &nbsp;
 0xFFFD | Cluster revision | INT16U | &nbsp;
 0x0066 | Protco MCU version | INT8U | &nbsp;
 0x0067 | Protco License | STRUCT | &nbsp;
 0x0068 | Protco custom protocol report | OCTET_STRING | &nbsp;
 0x0069 | Protco gateway ping | INT32U | &nbsp;

Command ID | Command | Direction | Description
--- | --- | --- | ---
 0x00 | Reset to factory default | Client to server | &nbsp;
 0x68 | Protco custom protocol receive | Client to server | &nbsp;

Apart from the above cluster, standard Zigbee OTA cluster is also required. Other endpoints and clusters can be used as per the device requirement.  

## Protco Custom attributes/commands

Prtoco uses custom Zigbee attribute ID and command ID from RFU pool.  

Application version: Format BIT 7~0 ->  XX.XX.XXXX, MAX 3.3.15  
Model Identifier: PID or Model as in Protco iot platform. Gateway use this information to whitelist Protco devices.  

    - RNZ: Starting with the ZIGBEE public version device model (find the device through the product model on the cloud platform to access the network)
    - RNRAW_PID: It is the device for the ZIGBEE transparent transmission module to access the network. The PID is the product PID (the device is found through the PID to access the network). DP data is transparently transmitted and will not be transferred to public protocols.
    - RNCUS_PID: It is the device for ZIGBEE customized device to access the network. PID is the product PID

Protco MCU version: required only for MCU based product. Format BIT 7~0 -> XX.XX.XXXX, MAX 3.3.15  

Protco License: Gateway reads this attribute to validate devices license. It contains three values namely UUID, SECRET and MAC. These values are unique for every device, can be obtained by purchasing license from Protco.  

Character string bytes is formatted in the following order  

Parameter name | Length | Value
--- | --- | ---
 Total Length | 1 byte | (Part of OctateString)
 0x01 | 1 Byte | 0x01
 Text Data Type(0x06) | 1 byte | 0x06
 UUID length | 1 Byte | 0x10
 UUID | (UUID length) Byte | UUID
 0x02 | 1 Byte | 0x02
 Text Type(0x06) | 1 Byte | 0x06
 SECRET Length | 1 Byte | 0x20
 SECRET | Secret length Byte | SECRET
 0x03 | 1 Byte | 0x03
 Text Type(0x06) | 1 Byte | 0x06
 MAC length | 1 Byte | 0x0C
 MAC | MAC length Byte | MAC

Note: _If the device has not been flashed with a license, the gateway query needs to reply with a specified string data length 0._

Parameter name | Length | Value
--- | --- | ---
 Total Length | 1 byte | (Part of OctateString)
 0x01 | 1 Byte | 0x01
 Text Data Type(0x06) | 1 byte | 0x06
 UUID length | 1 Byte | 0x00
 ~~UUID~~ | 0 Byte | -
 0x02 | 1 Byte | 0x02
 Text Type(0x06) | 1 Byte | 0x06
 SECRET Length | 1 Byte | 0x00
 ~~SECRET~~ | 0 Byte | -
 0x03 | 1 Byte | 0x03
 Text Type(0x06) | 1 Byte | 0x06
 MAC length | 1 Byte | 0x00
 ~~MAC~~ | 0 Byte | -

Protco  custom protocol report: This attribute is used to send custom protocol packets to the gateway.  

Protco gateway ping: NA  

Protco custom protocol receive: This command ID is used by the gateway to send custom message to the device.  

## Other Supported Cluster

Currently supported Clusters include OnOff/Level/Color (Color Temperature)/Group Cluster  
