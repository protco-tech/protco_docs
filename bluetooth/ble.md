
# BLE Overview  

## BLE Device Discovery  

### Adv data format

Broadcast data segment | Type | Description
---- | ---- | ----
 Physical connection identifier of BLE device | 0x01 | Length: 0x02;Type: 0X01; Data: 0x06
 Service UUID | 0x02 | Length: 0x03; Type: 0x02; Data: 0xA101
 Service Data | 0x16 | Length: 0x0C or 0x14 Type: 0x16 Data: 0x01, 0xA2, type (0-pid,1-product_key) PID or product_key (in 8 or 16 byte)

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
 manufacturer specific data | 0xff | Length: 0x19 Type: 0xff<br/>Date: COMPANY ID:0x0000 FLAG:0x00 Protocol version:0x03 Encryption method:0x00 Communication capacity: 0x0000 Reserved field: 0x00 ID field: 6 or 16 bytes

Encryption byte  

```C
#define PROTCO_BLE_SECURE_CONNECTION_WITH_AUTH_KEY     0x00
#define PROTCO_BLE_SECURE_CONNECTION_WITH_ECC          0x01
#define PROTCO_BLE_SECURE_CONNECTION_WTIH_PASSTHROUGH  0x02
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

**TODO:** _check and complete!_  
