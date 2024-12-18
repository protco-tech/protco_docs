
# Protco Zigbee (v1.00)

**TODO:** _Complete it according to the latest docs received from the RINO team_  

## Zigbee Basic

Profile ID: 0x0104  
Endpoint: 1  
Endpoint 1 is used for data exchange between gateway and the device.  
Basic cluster (0x0000) configuration  

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
 0x0067 | Protco License | CHAR_STRING | &nbsp;
 0x0068 | Protco custom protocol report | CHAR_STRING | &nbsp;
 0x0069 | Protco gateway ping | &nbsp; | &nbsp;

Command ID | Command | Direction | Description
--- | --- | --- | ---
 0x00 | Reset to factory default | Client to server | &nbsp;
 0x68 | Protco custom protocol receive | Client to server | &nbsp;

Apart from the above cluster, standard Zigbee OTA cluster is also required. Other endpoints and clusters can be used as per the device requirement.  

## Protco Custom attributes/commands

Prtoco uses custom Zigbee attribute ID and command ID from RFU pool.  
Application version: Format BIT 7~0 ->  XX.XX.XXXX, MAX 3.3.15  
Model Identifier: PID or Model as in Protco iot platform. Gateway use this information to whitelist Protco devices.  
Protco MCU version: required only for MCU based product. Format BIT 7~0 -> XX.XX.XXXX, MAX 3.3.15  
Protco License: Gateway reads this attribute to validate devices license. It contains three values namely UUID, SECRET and MAC. These values are unique for every device, can be obtained by purchasing license from Protco.  

Character string bytes is formatted in the following order  

Parameter name | Length
--- | ---
 Total Length | 1 byte
 0x01 | 1 Byte
 Text Data Type(0x06) | 1 byte
 UUID length | 1 Byte
 UUID | (UUID length) Byte
 0x02 | 1 Byte
 Text Type(0x06) | 1 Byte
 SECRET Length | 1 Byte
 SECRET | Secret length Byte
 0x03 | 1 Byte
 Text Type(0x06) | 1 Byte
 MAC length | 1 Byte
 MAC | MAC length Byte

Protco  custom protocol report: This attribute is used to send custom protocol packets to the gateway.
Protco gateway ping: NA
Protco custom protocol receive: This command ID is used by the gateway to send custom message to the device.
