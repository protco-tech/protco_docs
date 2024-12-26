
# BLE Overview  

## BLE Device Discovery  

### Adv data format

Broadcast data segment | Type | Description
---- | ---- | ----
 Physical connection identifier of BLE device | 0x01 | Length: 0x02;Type: 0X01; Data: 0x06
 Service UUID | 0x02 | Length: 0x03; Type: 0x02; Data: 0xA101
 Service Data | 0x16 | Length: 0x0C or 0x14<br/> Type: 0x16<br/> Data: 0x01, 0xA2,<br/> type (0-pid,1-product_key) <br/>PID or product_key (in 8 or 16 byte)

Example:  

```TEXT
02 01 06 
03 02 01A1
14 16 01A1 00 42704D46336D71554672687374370000
```

### Scan Response data format

Response data segment | Type | Description
--- | --- | ---
 Complete Local Name | 0x09 | Length: 0x03; Type:0x09; Data: 0x52,0x59
 manufacturer specific data | 0xff | Length: 0x19 Type: 0xff<br/>Data: COMPANY ID:0x0000<br/> FLAG:0x00<br/> Protocol version:0x03<br/> Encryption method:0x00<br/> Communication capacity: 0x0000<br/> Reserved field: 0x00<br/> UUID field: 6 or 16 bytes

**TODO:** _Verify if the app supports any encryption! I don't think there is any!_

Encryption byte  

```C
#define PROTCO_BLE_SECURE_CONNECTION_WITH_AUTH_KEY                  0x00
#define PROTCO_BLE_SECURE_CONNECTION_WITH_ECC                       0x01
#define PROTCO_BLE_SECURE_CONNECTION_WTIH_PASSTHROUGH               0x02
#define PROTCO_BLE_SECURE_CONNECTION_WITH_AUTH_KEY_DEVCIE_ID_20     0x03
```

Communication capacity definition  

```C
#define PROTCO_BLE_DEVICE_COMMUNICATION_ABILITY_BLE                0x0000
#define PROTCO_BLE_DEVICE_COMMUNICATION_ABILITY_REGISTER_FROM_BLE  0x0001
#define PROTCO_BLE_DEVICE_COMMUNICATION_ABILITY_MESH               0x0002
#define PROTCO_BLE_DEVICE_COMMUNICATION_ABILITY_WIFI_24G           0x0004
#define PROTCO_BLE_DEVICE_COMMUNICATION_ABILITY_WIFI_5G            0x0008
#define PROTCO_BLE_DEVICE_COMMUNICATION_ABILITY_ZIGBEE             0x0010
#define PROTCO_BLE_DEVICE_COMMUNICATION_ABILITY_NB                 0x0020
```

```C
// Network flags BLE
#define DVE_BLE_DIS_NETWORK_FLAG                0x01
#define DVE_BLE_DIS_NETWORK_GENERAL_FLAG        0x11
#define DEV_BLE_NETWORK_FLAG                    0xC1
#define DEV_BLE_NETWORK_GENERAL_FLAG            0xD1

//Flag used in WiFi device
/*
E1-> connected via WiFi
01-> not connected / init
C1-> Connected via Bluetooth
*/
```

Example:  

```TEXT
03 09 5259
19 FF 0000 D1 04 00 0001 00 726E3031373233613835373737443641
```

## Custom Service UUID  

Protco uses custom service UUIDs for App to Device communication  

Service UUID | Characteristic UUID | Properties | Security Permissions
---- | ---- | ---- | ----
 1910 | 2b10 | Notify | None
 &nbsp; | 2b11 | Write,write without response. | None

## MTU  

Minimum required MTU is 128  

Protco App requests MTU of 517  

Recommended device MTU is 512  

## Data transfer over GATT  

Check [BLE Data transfer protocol](ble_data_transfer_protocol.md) for the details.  
