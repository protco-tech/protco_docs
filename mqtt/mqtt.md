
# MQTT2.0  

## Description  

1. The device offline function is encapsulated and processed by the broker. When the device is successfully connected to the mqtt broker, the device goes online. After the device is disconnected, the detection device is offline according to the Client keep alive setting.  

2. `${pid}`: product ID/product model (third-party device model); `${uuid}`: triplet ID

## Device Connection Method  

Parameter name | Parameter Description
---------|----------
 Access domain name | Test environment: [testmqtt.protco.in](mqtt://testmqtt.protco.in) Formal environment: [mqtt.protco.in](mqtt://mqtt.protco.in)
 mqttClientId | Format: `rlink_${uuid}_V2` Note: clientId always ends with `_V2`
 username | Format: `${uuid}\|signMethod=${signMethod},ts=${ts}`
 password | hmacSha256(content, deviceSecret) content: `uuid=${uuid},ts=${ts}` deviceSecret: get the secret/key in the triple
 ts | timestamp in seconds
 signMethod | Signature algorithm, hmacSha1 or hmacSha256 can be selected according to hardware capabilities

### Example

Test address: testmqtt.protco.in  
username：`rn014ebe0C20E122|signMethod=hmacSha1,ts=1662544613`  
password(hmacSha1)：`ca4ff69d9408da837ed361bd5458b9c1f92c8cbb`  
password(hmacSha256): `833f4da1801f825232b5da71e1a965ac958181bb9b756934b3c3b05c8f600551`  
Triple:  
Uuid: `rn014ebe0C20E122` The first four digits are string type, and the last six digits are hexadecimal string.  
Secret/key: `ef958e891ffa4c72ba6c64e73c5dd422` length 32 string  
Mac: `7C87C94B97EC` 6-bit hexadecimal string  

Formal environment: mqtt.protco.in  
username：`rn010c0B9cB35EA9|signMethod=hmacSha1,ts=1662544613`  
password(hmacSha1)： `4f3bd5555cebe64ebb72ec846616251c032449b9`  
password(hmacSha256): `c98f698b9b86357f456daa46eee15d1fed82937eaa5671e24f4f987422ddc80b`  
Triple:  
Uuid: `rn010c0B9cB35EA9` The first four digits are string type, and the last six digits are hexadecimal string.
Secret/key: `adf2866e8ed94a5da5d45ee4e213f2a5` length 32 string  
Mac: `7CFC89B152C3` 6-bit hexadecimal string  

## App Connection Method  

Parameter name | Parameter description
----------- | ---------------
 Access domain name | xxx.xxx.xxx.xxx:1883
 mqttClientId | format: `app_${userId}\|${ts}`, for example: `app_C8NaEHry9Gjw0EM0\|1626197189`
 username | format: `${userId}\|signMethod=${signMethod},ts=${ts}`
 password | hmacSha256(content, secret) content: `uuid=${userId},ts=${ts}` secret: ap application id
 ts | timestamp (seconds)
 signMethod | signature algorithm, hmacSha1 or hmacSha256 can be selected according to hardware capabilities.

## Access Process diagram

**TODO:** _insert flow chart here!_

## 1. Device Level MQTT Messages

### Device Event Reporting to Cloud

Publish Topic:  `rlink/v2/${pid}/${uuid}/report`

```JSON
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"reset",//Event code, see device event list
    "data":{},//Report data structure See device event list->data structure definition
    "ack":0 //0: No reply required; 1: Reply required
}
```

Subscribe Topic: `rlink/v2/${pid}/${uuid}/report_response`

```JSON
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001", //Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"reset", //Event code, see device event list (consistent with the reported message ID)
    "data":{} //Reply data structure See device event list->data structure definition
}
```

### Device Command from cloud

Subscribe Topic: `rlink/v2/${pid}/${uuid}/issue`

```JSON
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"property_set",//Command code, see device command list
    "data":{} //Send data structure See device command list->data structure definition
}
```

Publish Topic: `rlink/v2/${pid}/${uuid}/issue_response`

```JSON
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001", //Message ID (consistent with the ID of the sent message)
    "ts":1626197189, //Timestamp (seconds)
    "code":"property_set", //Command code, see device command list (consistent with the reported message ID)
    "data":{} //Reply data structure See device command list->data structure definition
}
```

### Device event codes

SN | Code | Event name | Default ACK | Supported device type
---- | ---- | ---- | ---- | ----
 1 | reset | Device reset | 0 | Gateway, normal direct connection
 2 | time | Get cloud time | 1 | Gateway, normal direct connection
 3 | model | Acquisition model | 1 | Gateway, normal direct connection
 4 | config | Get remote configuration | 1 | Gateway, normal direct connection
 5 | bind | Device binding | 1 | Gateway, normal direct connection
 6 | info | Equipment information reporting (important, recommended docking) | 1 | Gateway, normal direct connection
 7 | property_report | Attribute reporting | 0 | Gateway, normal direct connection
 8 | ota_progress | OTA progress report | 0 | Gateway, normal direct connection
 9 | sub_bind | Child device binding | 1 | Gateway
 10 | sub_delete | Child device unbind | 1 | Gateway
 11 | sub_login | The child device is launched. | 0 | Gateway
 12 | sub_loginout | Child device offline | 0 | Gateway
 13 | sub_model | Obtain sub-device object model | 1 | Gateway
 14 | sub_property_report | Sub-device attribute reporting | 0 | Gateway
 15 | local_config | Notify the cloud to send the gateway to local groups/scenes/one-click execution | 1 | Gateway
 16 | sub_list | Get a list of child devices | 1 | Gateway
 17 | ipc_cloud | Obtain IPC cloud storage information | 1 | IPC equipment
 18 | p2p_config | Get P2P configuration information | 1 | IPC equipment
 19 | agora_token | Acquire Acoustic Tokens (RTM/RTC) Only for re-obtaining and using the token when it expires. | 1 | Gateway, normal direct connection
 20 | event | Custom event reporting (device alarms, etc.)  | 0 | Gateway, normal direct connection
 21 | ipc_token_bind | IPC token binding device | 1 | IPC equipment
 22 | find_alert | Find pop-up devices (with app) | 0 | Gateway, normal direct connection
 23 | find_report | The gateway reports the searched device | 0 | Gateway
 24 | ipc_live_get | IPC device obtains real-time stream push address | 0 | IPC equipment
 25 | pwd_sync | Device password synchronization (door lock type) | 1 | Door lock equipment
 26 | local_data_sync | Gateway requests local data synchronization | 0 | Gateway
 27 | webrtc_signal_exchange | WEBRTC signaling exchange | 0 | IPC equipment
 28 | update_properties_info | Update device properties | 0 | Gateway, normal direct connection
 29 | agora_ai_token | Get SoundNet AI token | 1 | IPC equipment
 30 | weather | Obtain weather information | 1 | Gateway, normal direct connection
 31 | sweeper_sts_token | Cleaner obtains upload temporary credentials | 1 | Cleaner
 32 | sweeper_map_list | Cleaner gets the full map | 1 | Cleaner
 33 | sweeper_map_add | Cleaner adds map | 1 | Cleaner
 34 | sweeper_map_update | Sweeper modify map | 1 | Cleaner
 35 | sweeper_map_del | Cleaner Delete Map | 1 | Cleaner
 36 | sweeper_record_add | Cleaner adds cleaning record | 1 | Cleaner
 37 | webrtc_config | Get webrtc config | 1 | Cleaner

#### 1. Device Reset

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"reset",//Event code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "cleanData":false //Whether to clear data
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"reset",//Event code (consistent with the reported event code)
}
```

#### 2. Get Cloud Time

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"time",//Event code
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"time",//Event code (consistent with the reported event code)
    "data":{
        "ts":1626197189, //Timestamp (seconds)
        "sys_tz":"America/New_York",
        "zone_offset":-14400, //Offset (seconds) DST offset processed
        "local_date_time": "2024-07-13 04:33:39",//Current local time (yyyy-MM-dd HH:mm:ss)
        "is_dst": true, //Is it currently in DST?
        "is_zone_dst": true, //Is this region using DST?
        //Daylight saving time start UTC timestamp (seconds), if there is no DST in this region, this field is empty
        "dst_start_ts": 1710054000,
        //Daylight saving time start local time (yyyy-MM-dd HH:mm:ss), if there is no DST in this region, this field is empty
        "dst_start_local_date_time": "2024-03-10 02:00:00",
        //Daylight saving time end UTC timestamp (second level), if there is no daylight saving time in this area, this field is empty
        "dst_end_ts": 1730617200,
        //Daylight saving time end local time (yyyy-MM-dd HH:mm:ss), if there is no daylight saving time in this area, this field is empty
        "dst_end_local_date_time": "2024-11-03 02:00:00"
    }
}
```

- How to determine the time
  - Method 1: Use `ts` and `sys_tz` to calculate daylight saving time
  - Method 2: Use `ts` and `zone_offset` to calculate daylight saving time
  - Method 3: Use `local_date_time` to convert daylight saving time
- How to Determine Daylight Saving Time Changes
  - Method 1: Use `dst_start_ts` or `dst_end_ts` based on timestamp processing. If entering or exiting daylight saving time, resend this command to calibrate the device time
  - Method 2: Use `dst_start_local_date_time` or `dst_end_local_date_time` processing and resend this command to calibrate device time if entering or exiting daylight saving time
- Notes:
  - Some areas do not have daylight saving time, you can judge whether there is daylight saving time in this area according to `is_zone_dst`  
  - When the user switches the time zone, the cloud may actively issue this response, and the device needs to process it ( `this logic has not yet been added` ).

#### 3. Get Object Model

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"model",//Event code
    "data":{
        //complete: full version, simple: simplified version, mini: mini version
        "format": "complete"
    },
    "ack":1 //0: no reply required; 1: reply required
}
```

```JSON
//Cloud reply (no reply when ack=0) complete: full version
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"model",//Event code (consistent with the reported event code)
    "data":{
        "format": "complete",
        "profile": {
            "version": "V1.0.0",//Firmware version
            "productId": "Current product id"
        },
        "properties": [{
            "identifier": "Property unique identifier (unique under the object model module)",
            "dpBusiId":"11", //Business id
            "name": "Property name",
            "accessMode": "Property read-write type: read-only (r) or read-write (rw).",
            "required": "Is it a required attribute for standard functions?",
            "dataTypeStr":"int",
            "dataType": {
                "type": "Property type: int (native), float (native), double (native), text (native), date (String type UTC milliseconds), bool (int type of 0 or 1), enum (int type, the enumeration item definition method is the same as the bool type definition of 0 and 1 value method), struct (structure type, can contain the previous 7 types, the following uses specs: [{}] to describe the contained objects), array (array type, supports int, double, float, text, struct)",
                "specs": {
                    "min": "Parameter minimum value (int, float, double type specific)",
                    "max": "Parameter maximum value (int, float, double type specific)",
                    "unit": "Property unit (int, float, double type specific, not required)",
                    "unitName": "Unit name (int, float, double type specific, not required)",
                    "size": "The number of array elements, up to 512 (array type specific). ",
                    "step": "Step length (text and enum types do not have this parameter)",
                    "length": "Data length, maximum 10240 (unique to text type)",
                    "item": {
                        "type": "Type of array element (array type specific)"
                    },
                    "enums":["1","2"]
                }
            }
        }],
        "events":[],
        ""
    }
}
```

```JSON
//Cloud reply (no reply when ack=0) simple simplified version
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"model",//Event code (consistent with reported event code)
    "data":{
        "format": "simple",
        "profile": {
            "version": "V1.0.0",//Firmware version
            "productId": "Current product id"
        },
        "properties": [{
            "identifier": "Property unique identifier (unique under the object model module)",
            "busiId":"11", //Business id
            "dataType":"int",//Property type: int (native), float (native), double (native), text (native), date (String type UTC milliseconds), bool (int type of 0 or 1), enum (int type, the enumeration item definition method is the same as the bool type definition of 0 and 1 value method), struct (structure type, can contain the previous 7 types, the following uses specs: [{}] to describe the included objects), array (array type, supports int, double, float, text, struct)
            "specs": {
                "min": "Parameter minimum value (int, float, double type specific)",
                "max": "Parameter maximum value (int, float, double type specific)",
                "unit": "Property unit (int, float, double type specific, not required)",
                "unitName": "Unit name (int, float, double type specific, not required)",
                "size": "The number of array elements, maximum 512 (array type specific).",
                "step": "Step length (text, enum type does not have this parameter)",
                "length": "Data length, maximum 10240 (text type specific). ",
                "item": {
                    "type": "Type of array element (specific to array type)"
                },
                "enums":["1","2"]
            }
        }],
        "events":[],
        ""
    }
}
```

```JSON
//Cloud reply (no reply when ack=0) mini: mini version
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"model",//Event code (consistent with reported event code)
    "data":{
        "format": "mini",
        "profile": {
            "version": "V1.0.0",//Firmware version
            "productId": "Current product id"
        },
        "properties": [{
            "i": "Property unique identifier (unique under the object model module)",
            "b":"11", //Business id
            "t": "int", //Property type: int (native), float (native), double (native), text (native), date (String type UTC milliseconds), bool (int type of 0 or 1), enum (int type, the enumeration item definition method is the same as the bool type definition of 0 and 1 value method), struct (structure type, can contain the previous 7 types, the following uses specs: [{}] to describe the contained objects), array (array type, supports int, double, float, text, struct)
        }]
    }
}
```

#### 4. Get Remote Configuration

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"config",//Event code
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0) simple simplified version
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"config",//Event code (consistent with the reported event code)
    "data":{
        "config":{"key":value}// map key-value pairs different devices The configuration is different, the key is String, and the value can be any type
    }
}
```

#### 5. Device Binding

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"bind",//Event code

    "data":{
        "userId":"11112333",//Network user ID
        "assetId": "ssssss321345e", //From ble (transmitted during first network configuration) asset id
        "version": "1.1.2",//Firmware version number Required
        "mcuVersion":"1.0.0",//MCU version number
        "cleanData":false //Whether to clear data, true: clear data, false: do not clear data
    },
    "ack":1 //0: no reply required; 1: reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success , others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001", //Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"bind", //Event code (consistent with the reported event code)
}
```

#### 6. Device Information Report

**TODO:** _Check what the below text note means!!_  

It is recommended to connect with this agreement. If this agreement is not connected, important legacy messages that have not been delivered to the cloud due to power outage/network disconnection will not be received  

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"info",//Event code
    "data":{
        "checkPwd":0,//Whether to detect undelivered password: 0: No; 1 Yes; Default 0
        "bindStatus":0, //Device binding status 0 Unbound; 1 Bound
        "version": "1.1.2",//Firmware version number Required
        "mcuVersion":"1.0.0",//MCU version number Optional
        //Device information Optional, key-value format json string, report as needed
        //currentSsid: Current wifi name;
        //wifiList: Alternate network list;
        "config":"{\"currentSsid\":\"PT_2.4G\",\"wifiList\":[{\"ssid\":\"wifi1\",\"pwd\":\"12345678\"}]}"
    },
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"info",//Event code (consistent with reported event code)
    "data": {
        "bindStatus":0, //Cloud binding status, 0 unbound; 1 bound
        "isClearData":0, //Has the data been cleared? 0: No; 1: Yes
        "logUploadUrl":"http://abcd.com/bcd", //Log upload address
        "tcpUrl":"xyz.abcd.com", //Low power consumption TCP address
        "logSwitch":{
            "enable":1, //0 off; 1 on
            "tttype":1, //Must transmit when enable=1, reporting mode: 1 single report; 2: cyclic report;
            "intervalTime":10 //Must transmit when enable=1, upload cycle; minutes; 1~360 minutes
        }
    }
}
```

#### 7. Attribute Report

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"property_report",//Event code
    "data":{
        "properties":{
            "color":"red", //Color Red
            "brightness":80 //Brightness 80
        }
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"property_report", //Event code (consistent with the reported event code)
}
```

#### 8. OTA Progress Report

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota_progress",//Event code
    "data":{
        //0, success; error code, -1: download timeout; -2: file does not exist; -3: signature expired; -4: MD5 does not match; -5: firmware update failed
        "resCode":0,
        "subPid":"xxxxx",//Sub-device product ID Not required If the gateway sub-device is upgraded, it is required to fill in the pid of the sub-device
        "subUuid":"xxxxxxxxxx",//Sub-device uuid Not required If the gateway sub-device is upgraded, it is required to fill in the uuid of the sub-device
        //downloading: downloading
        //burning: upgrading and burning
        //report: report version number (the gateway reports the version number for the sub-device when the sub-device upgrade is completed);
        //fail: failure
        "type":"downloading",
        "percent":80, //Progress 0~100 ；
        "version":"1.0.0", //Current version number of the device (reported when the sub-device upgrade is completed)
        "mcuVersion":"" //Current version number of the device (reported when the sub-device upgrade is completed)
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//When type=downloading
{
    "id":"45lkj3551234001", //Message ID
    "ts":1626197189, //Timestamp (seconds)
    "code":"ota_progress", //Event code
    "data":{
        "resCode":0,
        "subPid":"xxxxx",
        "subUuid":"xxxxxxxxxx",
        "type":"downloading",
        "percent":80, //Progress 0~100 ；
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//When type=burning
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota_progress",//Event code
    "data":{
        "resCode":0,
        "subPid":"xxxxx",
        "subUuid":"xxxxxxxxxx",
        "type":"burning"
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//When type=report (use when gateway sub-device upgrade is successful)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota_progress",//Event code
    "data":{
        "resCode":0,
        "subPid":"xxxxx",
        "subUuid":"xxxxxxxxxx",
        "type":"report",
        "version":"1.0.0",
        "mcuVersion":"1.0.0"
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota_progress",//Event code (consistent with the reported event code)
}
```

#### 9. Sub-Device Binding

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_bind",//Event code
    "data":{
        "devices":[
        {
            "uuid":"aaaaaxxxx", //Device UUID
            "trdUuid":"xxxxxx",//No real uuid when searching, used for binding result matching
            "pid": "2vRXuUhlmmDZCGUa", //Product ID Required
            "version": "1.1.2",//Firmware version number Required
            "mcuVersion":"1.0.0",//MCU version number
            "type":"rino" //rino: Xiyun, other third-party products such as: ry: Ruiying, default rino when not transmitted
        }
        ]
    },
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_bind",//Event code (consistent with reported event code)
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "pid":"xxxxxxxxxxxxx"
        }
        ] //List of successfully bound devices
    }
}
```

#### 10. Sub-Device Unbind

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_delete",//Event code
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "pid":"xxxxxxxxxxxxx"
        }
        ] //Unbind device list
    },
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_delete", //Event code (consistent with the reported event code)
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "pid":"xxxxxxxxxxxxx"
        }
        ] //List of devices that have been successfully unbound
    },
}
```

#### 11. Sub-Device Online

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_login",//Event code
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "pid":"xxxxxxxxxxxxx"
        }
        ] //Device list
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_login", // event code (consistent with the reported event code)
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "pid":"xxxxxxxxxxxxx"
        }
        ] // list of devices that have been successfully launched
    },
}
```

#### 12. Sub-Device Offline

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_loginout",//Event code
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "pid":"xxxxxxxxxxxxx"
        }
        ] //Device list
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_loginout", // event code (consistent with the reported event code)
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "pid":"xxxxxxxxxxxxx"
        }
        ] // uuid list of successful offline
    },
}
```

#### 13. Get Sub-Device Object Model

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_model",//Event code
    "data":{
        //simple: simplified version, mini: mini version
        "format": "simple",
        "subPid":"EEE01" //Product ID/or model
    },
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0) complete: Complete version
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_model",//Event code (consistent with the reported event code)
    "data":{
        "format": "complete",
        "subPid":"EEE01",
        "profile": {
            "version": "V1.0.0",//Firmware version
            "productId": "Current product id"
        },
        "properties": [{
            "identifier": "Property unique identifier (unique under the object model module)",
            "dpBusiId":"11", //Business id
            "name": "Property name",
            "accessMode": "Property read-write type: read-only (r) or read-write (rw).",
            "required": "Is it a required property for standard functions?",
            "dataTypeStr":"int",
            "dataType": {
                "type": "Property type: int (native), float (native), double (native), text (native), date (String type UTC milliseconds), bool (int type of 0 or 1), enum (int type, the enumeration item definition method is the same as the bool type definition of 0 and 1 value method), struct (structure type, can contain the previous 7 types, the following uses specs: [{}] to describe the contained objects), array (array type, supports int, double, float, text, struct)",
                "specs": {
                    "min": "Parameter minimum value (int, float, double type specific)",
                    "max": "Parameter maximum value (int, float, double type specific)",
                    "unit": "Property unit (int, float, double type specific, not required)",
                    "unitName": "Unit name (int, float, double type specific, not required)",
                    "size": "The number of array elements, up to 512 (array type specific). ",
                    "step": "Step length (text and enum types do not have this parameter)",
                    "length": "Data length, maximum 10240 (unique to text type)",
                    "item": {
                        "type": "Array element type (array type specific)"
                    },
                    "enums":["1","2"]
                }
            }
        }],
        "events":[],
        ""
    }
}
```

```JSON
//Cloud reply (no reply when ack=0) simple simplified version
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_model",//Event code (consistent with reported event code)
    "data":{
        "format": "simple",
        "subPid":"EEE01",
        "profile": {
            "version": "V1.0.0", //Firmware version
            "productId": "The id of the current product"
        },
        "properties": [{
            "identifier": "Property unique identifier (unique under the object model module)",
            "busiId":"11", //Business id
            "dataType":"int", //Property type: int (native), float (native), double (native), text (native), date (String type UTC milliseconds), bool (int type of 0 or 1), enum (int type, the enumeration item definition method is the same as the bool type definition of 0 and 1 value method), struct (structure type, can contain the previous 7 types, the following uses specs: [{}] to describe the included objects), array (array type, supports int, double, float, text, struct)
            "specs": {
                "min": "Parameter minimum value (int, float, double type specific)",
                "max": "Parameter maximum value (int, float, double type specific)",
                "unit": "Property unit (int, float, double type specific, not required)",
                "unitName": "Unit name (int, float, double type specific, not required)",
                "size": "The number of array elements, maximum 512 (array type specific).",
                "step": "Step length (text, enum type does not have this parameter)",
                "length": "Data length, maximum 10240 (text type specific). ",
                "item": {
                    "type": "Type of array element (specific to array type)"
                },
                "enums":["1","2"]
            }
        }],
        "events":[],
        ""
    }
}
```

```JSON
//Cloud reply (no reply when ack=0) mini: mini version
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_model",//Event code (consistent with reported event code)
    "data":{
        "format": "mini",
        "subPid":"EEE01",
        "profile": {
            "version": "V1.0.0",//Firmware version
            "productId": "The id of the current product"
        },
        "properties": [{
            "i": "Property unique identifier (unique under the object model module)",
            "b":"11", //Business id
            "t": "int", //Property type: int (native), float (native), double (native), text (native), date (String type UTC milliseconds), bool (int type of 0 or 1), enum (int type, the enumeration item definition method is the same as the bool type definition of 0 and 1 value method), struct (structure type, can contain the previous 7 types, the following uses specs: [{}] to describe the included objects), array (array type, supports int, double, float, text, struct)
        }]
    }
}
```

#### 14. Sub-Device Attribute Report

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_property_report",//Event code
    "data":{
        "devices":[
        {
            "pid":"xxxxxxx",
            "uuid":"uuid1",
            "properties":{
                "color":"red", //Color Red
                "brightness":80 //Brightness 80
            }
        }
        ]
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_property_report",//Event code (consistent with the reported event code)
}
```

#### 15. Notify the cloud to send the gateway Local Configs (local group/scene/one-click)

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_config",//Event code
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_config"//Event code (consistent with the reported event code)
}
```

#### 16. Get Sub-Device List

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_list",//Event code
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_list",//Event code (consistent with the reported event code)
    "data":{
        "devices":[//Sub-device list
        {
            "pid":"",//Product ID or model
            "uuid":"ssssxxxx",//Device UUID
            "name":"aaaa",//Device name
            "assetId":"xxxx",//Room id
            "assetName":"xxxx"//Room name
        }
        ]
    }
}
```

#### 17. Get IPC Cloud Storage Information

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_cloud",//Event code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        /**
        * Request type
        * - ipc_cloud_detail Get cloud storage details
        * - put_object Get object upload information
        */
        "type":"ipc_cloud_detail",

        //Upload object parameters, type=put_object must be passed
        "putObjectParams":{
            //Recording type (record-continuous recording, event-event recording, alarm-alarm)
            "recordType":"record",
            //File attribution date (yyyyMMdd, only files from the past two days can be uploaded)
            "fileDate":"20230613",
            //Start time (required, format: HHmmss)
            "startTime":"200000",
            //End time (required for fileType=record|event|alarm, format: HHmmss)
            "endTime":"200959",
            /**
            * File type (record-continuous recording file, event-event recording file, alarm-alarm recording file, alarmimg-alarm image)
            * - When recordType=record, optional: record
            * - When recordType=event, optional: event
            * - When recordType=alarm, optional: alarm|alarmimg
            */
            "fileType":"record",
            /**
            * File name suffix, not required
            * - When recordType=record, optional: ts (default value: ts)
            * - When recordType=event, optional: ts (default value: ts)
            * - When recordType=alarm, no limit (default value when fileType=alarm: mp4, default value when fileType=alarmimg: png)
            */
            "fileSuffix":"ts",
            //Alarm type (must be transmitted when recordType=alarm)
            "alarmType":"xxxxx",
            //Video duration (milliseconds, 10 seconds=10000)
            "duration":10000,
            //Number of cameras, such as binoculars, 1|2. The default value is 1
            "cameraNum":1
        }
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
//Reply when type=ipc_cloud_detail
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_cloud",//Event code (consistent with reported event code)
    "data":{
        "type":"ipc_cloud_detail",
        "userId":"cn1619869251258736640",//User id
        "ossType":"S3",//COS or S3
        //Recording type record-continuous recording event-event recording alarm-alarm
        "ossRecordType":["record","event","alarm"],
        "ossExpireTime":1687847608,//Cloud storage expiration time, in seconds. null if cloud storage is not enabled
        "encrypt": {
            "isEncrypt": true,//Encryption or not
            "encryptType": "1",//Encryption type, 1:AES128-CBC
            "encryptSecret": "YEcOY60RsJxIX6FZh2jLdFzP1etyjGog"//Encryption key
        }
    }
}
```

```JSON
//Reply when type=put_object
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_cloud",//Event code (consistent with the reported event code)
    "data":{
        "filePath": "record/cn1622515857328451584/rn013ebf21086E3A/20230612/preview-event-141700-141759.png",//File path
        "method":"PUT", //Request method
        "requestUrl":"xxxxx",//Request address
        //Request headers
        "headers":{
            "key1":"value1",
            "key2":"value2",
            "key3":"value3",
            //...
        }
    }
}
```

#### 18. Get P2P Configuration Information

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"p2p_config",//Event code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "type":"shangyun" //p2p type (shangyun-Shangyun, agora-Audio Network, webrtc-WebRtc), Shangyun is the default if not transmitted
    }
}
```

```JSON
//Cloud reply when type=shangyun (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"p2p_config",//Event code (consistent with the reported event code)
    "data":{
        "type":"shangyun",//shangyun: Shangyun; Agora: agora; webrtc: WebRtc
        "config":{
            "productId":"avmkP4hdysUrGs",
            "did":"",
            "apiLicense":"",
            "crcKey":"",
            "initString":"",
            "p2pKey":"",
            "wakeupKey":""
        }
    }
}
```

```JSON
//Cloud response when type=agora (no response when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"p2p_config",//Event code (consistent with the reported event code)
    "data":{
        "type":"agora",//shangyun: Shangyun; Agora: agora;webrtc: WebRtc
        "agoraConfig":{
            "appId": "", //Audio network appId
            "apiPid": "", //Audio network pid
            "apiLicense": "", //Audio network license
            "apiLicenseKey": "", //Audio network license key
            "activeTime": 1688194925200, //Activation time (milliseconds)
            "expireTime": 1688194925200, //Audio network expiration time (milliseconds)
            "skuData": "" //sku data, json format
        }
    }
}
```

```JSON
//Cloud response when type=webrtc (no response when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"p2p_config",//Event code (consistent with reported event code)
    "data":{
        "type":"webrtc",//shangyun: Shangyun; Agora: agora; webrtc: WebRtc
        "webrtcConfig":{
            "stunDomain":"stun.l.google.com",
            "stunUrl": "stun:stun.l.google.com:19302", //stun server address
            "turnUrls": [
                "turn:turn.3721iot.com:19573?transport=udp" //turn server address
            ],
            "auth":{
                "userName":"1666534426:rn01xxxxx",
                "password":"xxxxfsafsf",
                "expireSecond":3600
            }

        }
    }
}
```

#### 19. Get Token (RTM/RTC)

For renewal only. For example, if the issued token expires, you can use this command to get the token again.  

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"agora_token",//Event code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "rtcNum":1,//Number of rtc Tokens to be obtained, 1 is obtained by default if not transmitted
        "channelName":""//Channel name
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"agora_token",//Event code (consistent with the reported event code)
    "data":{
        "uuid": "inv01De86b75d01bb",//Device uuid
        "rtmToken": {
            "account": "d_1666425405739651072", //RTM account
            "rtmToken": "", //RTM Token
            "expireSecond": 3600 //RTM Token expiration time (seconds)
        },
        "rtcTokens": [{
            "channelName": "1666425405739651072", //Audio network channel
            "uid": 1, //Audio network uid
            "rtcToken": "", //RTC Token
            "expireSecond": 3600 //RTC Token expiration time (seconds)
        }, 
        {
            "channelName": "1666425405739651072",
            "uid": 2,
            "rtcToken": "007eJxTYMjWqa3Uu8JazrZI+vOD5a02z8RzdcsdH7y6tXdTm8Slfw4KDKZJKQZpqRZGRhZmBiZmaSlJpoYWhpYGliY GSaaJFuZpylVzUwT4GBjmRYaJMTIwMrAAMYjPBCaZwSQLmBRmMDQzMzMxMjUxMDU3tjQzNTQwN2JkMAIAFoQgjg==", "expireSecond": 3600 }, { "channelName": "1666425405739651072", "uid": 3, "rtcToken": "007eJxTYFh04dlnla8mDQZuc67M/MZc0vbpP298jBNXhfL711u0lr5UYDBNSjFIS7UwMrIwMzAxS0tJMjW0MLQ0sDQ xSDJNtDBPU66amyLAx8AwN+YsOyMDIwMLEIP4TGCSGUyygElhBkMzMzMTI1MTA1NzY0szU0MDcyNGBmMA64EiUA==",
            "expireSecond": 3600 
        }] 
    }
}
```

#### 20. Customs event reporting (device alarms, etc.)

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"event",//Event code
    "data":{
        "identifier":"alarm",//Event identifier, event identifier defined by the product object model
        "output":{} //Output parameters, event output parameters defined by the product object model
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//For example:
//Equipment alarm event
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"event",//Event code
    "data":{
        "identifier":"alarm",//Event identifier, event identifier defined by the product object model
        "output":{
            "type": "ipc_motion", //Alarm type
            "imageUrl":"", //Thumbnail address
            "storage":"local", //local or cloud
            "starttime": "030502", //Recording start time HHmmss
            "endtime": "030512", //Recording end time HHmmss
            "startPts": 1702350598, //Recording start timestamp (seconds)
            "endPts": 1702350598, //Recording end timestamp (seconds)
        } //Output parameters, event output parameters defined by product object model
    },
    "ack":0 //0: No response required; 1: Required response
}
```

```JSON
//Cloud response (no response when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001", //Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"event", //Event code (consistent with the reported event code)
}
```

#### 21. IPC Token Binding

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_token_bind",//Event code
    "data":{
        "token":"63D3Qx",
        "cleanData":false //Whether to clear data, true: clear data, false: do not clear data
    },
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_token_bind", //Event code (consistent with the reported event code)
}
```

#### 22. Find Alert

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"find_alert",//Event code
    "data":{
        "subPid": "xxxxx", //Sub-device pid Required if it is a gateway sub-device
        "subUuid":"Sub-device uuid" //Sub-device uuid Required if it is a gateway sub-device
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"find_alert", //Event code (same as the reported event code)
}
```

#### 23. Gateway Report found devices

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"find_report",//Event code
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx",
            "trdUuid":"xxxxxx",//No real uuid when searching, used for binding result matching
            "pid":"xxxxxxxxxxxxx"
        }
        ] //Device list
    },
    "ack":0 //0: No reply required; 1: Reply required
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"find_report", //Event code (same as the reported event code)
}
```

#### 24. IPC device obtains real-time stream push address

```JSON
//Report data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_live_get",//Event code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "type":"rtsps" //Real-time stream type (rtsps, rtmps)
    }
}
```

```JSON
//Response data structure
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_live_get",//Event code (consistent with the reported event code)
    "data":{
        // {streaming method rtsps or rtmps must be consistent with the actual streaming method}/{device pid}/{device uuid}/{user id}/{number of cameras}/{number of seconds for video segmentation}
        "url": "rtsps://rtsps.protco.in:8322/rtsps/6ZLZwGoM6AEbs4/rn012FfD0B8beAfB/cn1622515857328451584/1/10"
    }
}
```

#### 25. Device password synchronization (door lock type)

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"pwd_sync",//Event code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "subPid":"xxxxx",//Gateway sub-device pid, this field is only available when it is a gateway sub-device
        "subUuid":"xxxxx",//Gateway sub-device uuid, this field is only available when it is a gateway sub-device
        "pwdList":[
        {
            "index":1,//Password ID
            "type":1,//Password type: 1 digital password; 2 fingerprint; 3 card
            "deviceUserId":"001",//User ID No user is associated with the user
            "timeType":1,//Time type: 1: Permanent; 2: Single; 3 Limited time;
            //The following fields are required when timeType=3 Limited time
            "activeTime":1626197189,//Effective timestamp (second level)
            "expireTime":1626197189,//Expiration timestamp (second level)
            "startTime":"12:00",//Start time (HH:mm)
            "endTime":"13:00",//End time (HH:mm)
            "loops":"1111111"//Cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on that day)
        },
        {
            "index":2,//Password ID
            "type":2,//Password type: 1 digital password; 2 fingerprint; 3 card
            "deviceUserId":"001",//User ID
            "timeType":1,//Time type: 1: permanent; 2: single; 3 limited time;
            //The following field timeType=3 is required when limited time
            "activeTime":1626197189,//Effective timestamp (second level)
            "expireTime":1626197189,//Expiration timestamp (second level)
            "startTime":"12:00",//Start time (HH:mm)
            "endTime":"13:00",//End time (HH:mm)
            "loops":"1111111"//Cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on that day)
        },
        {
            "index":3,//Password ID
            "type":3,//Password type: 1 digital password; 2 fingerprint; 3 card
            "deviceUserId":"001",//User ID
            "timeType":1,//Time type: 1: permanent; 2: single; 3 limited time;
            //The following field timeType=3 is required for limited time
            "activeTime":1626197189,//Effective timestamp (second level)
            "expireTime":1626197189,//Expiration timestamp (second level)
            "startTime":"12:00",//Start time (HH:mm)
            "endTime":"13:00",//End time (HH:mm)
            "loops":"1111111"//Cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on that day)
        }
        ]
    }
}
```

```JSON
//Cloud response (no response when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"pwd_sync", //Event code (consistent with the reported event code)
}
```

#### 26. Gateway requests local data synchronization

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_data_sync",//Event code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "subUuids":["uuid1","uuid2"],//List of sub-devices in the current gateway
        "groupShortIds":[1,2,3],//Group shortId in the current network
        "dpGroupShortIds":[7,8,9],//Dp group shortId in the current gateway
        "sceneShortIds":[4,5,6],//Automation and one-click execution shortId in the current network
        "dpBinds":[ //Current gateway already has
        {
            "subUuid":"",//Binding device uuid
            "identifier":"",//Binding dp identifier
            "dpValue":""//Binding value
        }
        ]
    }
}
```

#### 27. WEB-RTC signaling exchange

```JSON
//Device reporting (SDP_ANSWER)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_signal_exchange",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "peerClientId": "peer_xxxxxxxxxxxxxxx",
        "method": "SDP_ANSWER",
        "params": "{\"type\": \"answer\", \"sdp\":\"\"}",
        "rawData":"{\"auto_play\": true}",
        "source":1,//1:app,2:alexa,3:google
        "error":-2, //Device busy
    }
}
```

```JSON
//Device reporting (ICE_CANDIDATE)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_signal_exchange",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "peerClientId": "peer_0xi681aspzmlbm9",
        "method": "ICE_CANDIDATE",
        "params":"{\"candidate\":\"candidate:2 1 udp 1694498815 183.11.68.22 20534 typ srflx raddr 0.0.0.0 rport 0 generation 0 network-cost 999\",\"sdpMid\":\"0\",\"sdpMLineIndex\":0}",
        "rawData":"{}",
        "error":-2 //Device busy
    }
}
```

#### 28. Update device properties

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"update_properties_info",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        //Device information key-value format json string is different for different devices, report as needed, only overwrite the reported properties
        "propertiesInfo": {
            "ipcMaxConnection":10,
            "ipcCurrentConnection":1
            //...
        }
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"update_properties_info", //Event code (consistent with the reported event code)
}
```

#### 29. Get Agora AI Token

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"agora_ai_token",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{}
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"agora_ai_token",//Event code (consistent with the reported event code)
    "data":{
        "appId":"",
        "channel_name":"",
        "token":"",
        "uid":1
    }
}
```

#### 30. Get Weather Information

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"weather",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{}
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"weather",//Event code (consistent with the reported event code)
    "data":{
        "sunriseTs":1626197189,//Sunrise timestamp (seconds)
        "sunsetTs":1626197189,//Sunset timestamp (seconds)
        "temperature":36//Temperature (Celsius)
    }
}
```

#### 31. Cleaner obtains upload temporary credentials  

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_sts_token",//Command code
    "ack":1,//0: No reply required; 1: Reply required
    "data":{
        "type":"map" //Upload type. (map: map, record: sweep record)
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_sts_token",//Event code (consistent with the reported event code)
    "data":{
        "type":"map",//Upload type.(map: map, record: cleaning record)
        "ossType": "COS", //oss type.COS|S3
        "bucketName": "sweeper-cloud-dev-1313015441",//Bucket name
        "region": "ap-guangzhou",//Storage pool region
        "accessKeyId": "xxxx",//COS secretId,S3 accessKeyId
        "secretAccessKey": "xxxx",//COS secretKey,S3 secretAccessKey
        "sessionToken": "xxxxx",//Temporary credentials
        "expiredTime": 1734318703,//Expiration time (seconds)
        "authDir": "map/az1598928553543086080/inv01cfbAaeb1E02E/"//Authorization directory
    }
}
```

#### 32. Cleaner gets the full map  

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_map_list",//Command code
    "ack":1,//0: No reply required; 1: Reply required
    "data":{
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_map_list",//Event code (consistent with the reported event code)
    "data":{
        "mapId": "1868505694547271680", //map id
        "mapFormat": "custom", //map format (device customization)
        "mapName": "First map", //map name
        "mapPath": "map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx", //map path
        "mapDesc": "map description", //map description,
        "createTime": 1734321447, //creation time (seconds)
        "mapSignPath": "https://xxxx/map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx" //map data access address

    }
}
```

#### 33. Cleaner adds map  

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_map_add",//Command code
    "ack":1,//0: No response required; 1: Required response
    "data":{
        "mapFormat":"custom", //Map format (device custom)
        "mapName":"First map", //Map name
        "mapPath":"map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx",//Map path
        "mapDesc":"Map description" //Map description
    }
}
```

```JSON
//Cloud response (no response when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_map_add",//Event code (consistent with the reported event code)
    "data":{
        "mapId": "1868505694547271680", //Map id
        "mapFormat": "custom",//Map format (device custom)
        "mapName": "First map",//Map name
        "mapPath": "map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx",//Map path
        "mapDesc": "Map description"//Map description
    }
}
```

#### 34. Sweeper modify map  

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_map_update",//Command code
    "ack":1,//0: No reply required; 1: Reply required
    "data":{
        "mapId": "1868505694547271680", //Map id
        "mapFormat":"custom", //Map format (device custom)
        "mapName":"First map", //Map name
        "mapPath":"map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx",//Map path
        "mapDesc":"Map description" //Map description
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success , others represent failure; see status code definition
    "msg":"success", //result description
    "id":"45lkj3551234001", //message ID (consistent with reported message ID)
    "ts":1626197189, //timestamp (seconds)
    "code":"sweeper_map_update", //event code (consistent with reported event code)
    "data":{
        "mapId": "1868505694547271680", //map id
        "mapFormat": "custom", //map format (device custom)
        "mapName": "first map", //map name
        "mapPath": "map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx", //map path
        "mapDesc": "map description" //map description
    }
}
```

#### 35. Cleaner Delete Map  

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_map_del",//Command code
    "ack":1,//0: No reply required; 1: Reply required
    "data":{
        "mapId": "1868505694547271680", //Map id
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_map_del", // event code (consistent with the reported event code)
    "data":{
        "result": true // delete result
    }
}
```

#### 36. Cleaner adds cleaning record  

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sweeper_record_add",//Command code
    "ack":1,//0: No reply required; 1: Reply required
    "data":{
        "mapId": "1868505694547271680", //Map id
        "recordPath":"map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx",//Sweep data path
        "recordDesc":"xxxx"//Sweep description information
    }
}
```

```JSON
//Cloud reply (no reply when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001", //Message ID (consistent with the reported message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"sweeper_record_add", //Event code (consistent with the reported event code)
    "data":{
        "recordId": "1868505694547271680", //Sweeping record id
        "mapId": "1868505694547271680", //Map id
        "recordPath":"map/az1598928553543086080/inv01cfbAaeb1E02E/xxxx", //Sweeping data path
        "recordDesc":"xxxx" //Sweeping description information
    }
}
```

#### 37. Get webrtc config  

```JSON
//Device report
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_config",//Command code
    "ack":1,//0: No response required; 1: Required response
    "data":{
    }
}
```

```JSON
//Cloud response (no response when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "msg":"success", //Result description
    "id":"45lkj3551234001",//Message ID (consistent with the reported message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_config",//Event code (consistent with the reported event code)
    "data":{
        "stunDomain":"stun.l.google.com",
        "stunUrl": "stun:stun.l.google.com:19302", //stun server address
        "turnUrls": [
        "turn:turn.3721iot.com:19573?transport=udp" //turn server address
        ],
        "auth":{
            "userName":"1666534426:rn01xxxxx",
            "password":"xxxxfsafsf",
            "expireSecond":3600
        }
    }
}
```

### Device Command Codes

Code | Command name | Default ACK | Support equipment
---- | ---- | ---- | ----
 reset | Reset device | 0 | Gateway, normal direct connection
 ota | Firmware upgrade | 0 | Gateway, normal direct connection
 ping | Detect equipment online status | 1 | Gateway, normal direct connection
 property_set | Set properties | 0 | Gateway, normal direct connection
 update_local_config | Local groups/scenes/one-click execution configuration distribution | 0 | Gateway
 find_bind | Search and bind sub-devices | 0 | Gateway
 sub_add | Specify adding child devices | 0 | Gateway
 sub_property_set | Set child device properties | 0 | Gateway
 sub_delete | Delete a child device | 0 | Gateway
 local_group_set | Local group control | 1 |Gateway
 local_rule_exec | Local one-click execution | 1 |Gateway
 wake_up | Wake up the device | 1 | Video doorbell, etc
 agora_join | Notify the device to join the sound network channel | 0 | IPC
 ipc_cloud_open | IPC cloud storage is open | | IPC
 agora_event | Acoustics event notification | 0 | IPC
 common_cmd | General pass-through instruction | 1 | IPC
 local_group_update | Local group change notification | 1 |Gateway
 local_scene_update | Local scene/one-click execution change notification | 1 | Gateway
 log_switch | Device log switch | 0 | Gateway, normal direct connection
 reboot | Device restart | 0 | Gateway, normal direct connection
 sub_replace | Subdevice replacement | 1 | Gateway
 property_get | Get device properties | 0 | Gateway, normal direct connection
 config_settings | Set device configuration | 1 | Gateway, normal direct connection
 set_pwd_list | Set password (door lock type) | 1 | Door lock type
 gateway_data_init | Gateway data Initialization | 0 | Gateway
 dp_bind | DP point binding local execution | 1 |Gateway
 local_dp_group_update | Local DP group change notification | 1 | Gateway
 device_info_update | Device information change notification | 0 | Gateway, normal direct connection
 group_info_update | Group information change notification | 0 | Gateway
 webrtc_signal_exchange | WebRTC signaling exchange | 0 | IPC equipment
 remote_unlock | Remote unlocking (door lock type) | 0 | Door lock type
 clear_data | Notify device to clear data | 0 | &nbsp;

#### Reset Device

```JSON
//data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"reset",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "clearData":true //Whether to clear data
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
"res":0, //Status code 0 represents success, others represent failure; see status code definition
"id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
"ts":1626197189,//Timestamp (seconds)
"code":"reset",//Event code (consistent with the sent command code)
}
```

#### Firmware Upgrade

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "subUuids":["xxx1","xxx2"],//Sub-device uuid Not required If the gateway sub-device is upgraded, it is required to fill in the sub-device uuid
        "firmwareType":2,//1: Factory firmware, 2: Standard firmware; 3: MCU firmware
        "fileSize": 708482,
        "silence":false,//Whether to upgrade silently, if true, trigger an ota progress report after the upgrade is completed, done is enough
        "md5sum": "36eb5951179db14a63****a37a9322a2",
        "url": "https://ota-1255858890.cos.ap.mycloud.com",
        "version": "0.2",//Upgraded firmware version number
        "options":{ //Upgrade parameters Non-mandatory parameters
            "stable":"usr" //Upgrade partition usr/recovery/kernel
        }
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota",//Event code (consistent with the sent command code)
}
```

#### Check device online status and network conditions

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ping",//Command code
    "ack":1 //0: No reply required; 1: Reply required
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"ping",//Event code (consistent with the sent command code)
    "data":{
        "currentSsid":"PT_4G",//Currently connected wifi name
        "wifiList":[//Available networks
            {
                "ssid":"TpLink2.4",
                "rssi":-50
            }
        ],
        "networkType":1,//Network type: 1:wifi, 2:wired
        //Good: Ping response time is usually in the range of 10 milliseconds (ms) to 50 milliseconds (ms)
        //Medium: Ping response time is usually in the range of 50 milliseconds (ms) to 100 milliseconds (ms).
        //Poor: Ping response time exceeds 100 milliseconds (ms), usually in the range of hundreds of   milliseconds
        "signal":1,//Signal strength: 1: good; 2: medium; 3: poor
        "signalValue":100 //0-100 0%-100%
    }
}
```

#### Set Property

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"property_set",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "properties":{
            "color":"red", //Color Red
            "brightness":80 //Brightness 80
        }
    },
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"property_set", // event code (consistent with the command code)
    "data":{
        "properties":{
            "color":"red", // color red
            "brightness":80 // brightness 80
        }
    },
}
```

#### Local groups/scenes/one-click execution configuration

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"update_local_config",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "updates":["groups","scenes","executes"],//Update type list
        //Local group
        "groups":[
        {
            //Sub-device group ID (short ID 0~9999)
            "shortId": 1,
            //Sub-device UUID (add or edit the returned full quantum device UUID list)
            "deviceUuids":["xxxxxx","xxxxxxxx","xxxxxxxx"]
        },
        //....
        ],
        //Local scene
        "scenes":[
        {
            "shortId": 1,//ID
            "executeType": 1, //Meet any condition: 1, Meet all conditions: 2
            //Conditions
            "conditions": [
            {
                "uuid": "xxxxxx", //Device uuid
                "conditionColumn": "switch_led", //dp identifier
                "conditionExpress": "==", //Greater than: >; Equal to: ==; Less than: <
                "conditionValue": true
            }
            ],
            //Actions
            "actions": [
            {
                "orderNum": 1,
                "actionType": "device", //device: device, group: group
                "shortGroupId": 1,
                "uuid": "xxxxxx",
                //Execute action
                "actionData": {
                    "switch_led": false
                }
            }
            ]
        },
        //....
        ],
        //Local one-click execution
        "executes": [
        {
            "shortId": 1, //Rule ID
            "actions": [//Actions
            {
                "orderNum":1,
                "actionType":"device",//device: device, group: group
                "shortGroupId":1,
                "uuid": "xxxxxx",
                //Execute action
                "actionData": {
                    "switch_led":false
                }
            }
            ]
        },
        //....
        ]
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"update_local_config",//Event code (consistent with the sent command code)
}
```

#### Search and bind sub-devices

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"find_bind",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "second":60,//Search duration
        "bind":true,//Whether to bind directly The device determines the sub-device binding or report the nearby device operation based on this field
        "find":true, //true: Start searching false: Stop searching
        "uuids":["uuid1","uuid2"],//Specify uuid When replacing, only one new replacement uuid is transmitted in uuids
        "pids":["pid1","pid2"]//Specify pid
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001", //Message ID (consistent with the sent message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"find_bind", //Event code (consistent with the sent command code)
}
```

#### Set sub-device properties

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_property_set",//Command code
    "ack":0, //0: No response required; 1: Required response
    "data":{
        "devices":[
        {
            "uuid":"uuid1",//If it is a third-party device, the uuid of the third-party device will be sent
            "properties":{
                "color":"red", //Color red
                "brightness":80 //Brightness 80
            }
        }
        ]
    }
}
```

```JSON
//Device reply (no response required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_property_set",//Event code (consistent with the command code)
    "data":{
        "devices":[
        {
            "uuid":"uuid1",//If it is a third-party device, the uuid of the third-party device will be sent
            "properties":{
                "color":"red", //Color red
                "brightness":80 //Brightness 80
            }
        }
        ]
    }
}
```

#### Remove Sub-Device

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_delete",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "devices":[
        {
            "uuid":"xxxxxx"//If it is a third-party device, the uuid of the third-party device will be sent
        }
        ]//Unbind device list
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_delete", //Event code (consistent with the issued instruction code)
}
```

#### Local Group Control

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_group_set",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "shortId":"Group short ID",
        "properties":{
            "color":"red", //Color Red
            "brightness":80 //Brightness 80
        }
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_group_set", // event code (consistent with the command code)
    "data":{
        "shortId":"Group short ID",
        "properties":{
            "color":"red", // color red
            "brightness":80 // brightness 80
        }
    }
}
```

#### Local one-click execution

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_rule_exec",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "shortId":1
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_rule_exec",//Event code (consistent with the sent command code)
    "data":{
        "shortId":1
    }
}
```

#### Wake up the device

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"wake_up",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "subUuid":"xxxxxxxxxx",//Sub-device uuid Not required If it is a gateway sub-device, it is required to fill in the sub-device uuid
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"wake_up", // event code (consistent with the command code)
    "data":{
        "subPid":"xxxxxxxxxx", // sub-device pid optional. If it is a gateway sub-device, it must be filled in, and the sub-device pid is passed
        "subUuid":"xxxxxxxxxx", // sub-device uuid optional. If it is a gateway sub-device, it must be filled in, and the sub-device uuid is passed
    }
}
```

#### Notify the device to join the sound network channel

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"agora_join",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "uuid": "inv01De86b75d01bb",//Device uuid
        "agoraModel": "playback", //Audio network mode. Live-live broadcast. Playback-playback
        "connectModel":1, //Connection mode (1-pre-connection, 2-live broadcast push)
        "timeoutMilliSecond":60000, //Pre-connection timeout (milliseconds)
        //App transparent data
        "rawData":"{\"cmdType\":\"start_liveplay\",\"ack \":1,\"channel\":\"0\",\"streamtype\":0,\"data\":{\"width\":1920,\"height\":1080,\"format\":1}}",
        "rtmToken": {
            "account": "d_1666425405739651072", //RTM account
            "rtmToken": "", //RTM Token
            "expireSecond": 3600 //RTM Token expiration time (seconds)
        },
        "rtcTokens": [{
            "channelName": "1666425405739651072", //Audio network channel
            "uid": 1, //Audio network uid
            "rtcToken": "", //RTC Token
            "expireSecond": 3600 //RTC Token expiration time (seconds)
        },
        {
            "channelName": "1666425405739651072", "uid": 2, "rtcToken": "007eJxTYMjWqa3Uu8JazrZI+vOD5a02z8RzdcsdH7y6tXdTm8Slfw4KDKZJKQZpqRZGRhZmBiZmaSlJpoYWhpYGliY GSaaJFuZpylVzUwT4GBjmRYaJMTIwMrAAMYjPBCaZwSQLmBRmMDQzMzMxMjUxMDU3tjQzNTQwN2JkMAIAFoQgjg==", "expireSecond": 3600
        },
        {
            "channelName": "1666425405739651072", "uid": 3, "rtcToken": "007eJxTYFh04dlnla8mDQZuc67M/MZc0vbpP298jBNXhfL711u0lr5UYDBNSjFIS7UwMrIwMzAxS0tJMjW0MLQ0sDQ xSDJNtDBPU66amyLAx8AwN+YsOyMDIwMLEIP4TGCSGUyygElhBkMzMzMTI1MTA1NzY0szU0MDcyNGBmMA64EiUA==", "expireSecond": 3600
        }]
    }
}
```

#### IPC Cloud Storage is Open

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_cloud_open",//Cloud storage activation notification
    "ack":1, //0: No response required; 1: Required response
    "data":{
        "userId":"cn1619869251258736640",//User id
        "ossType":"S3",//COS or S3
        //Recording type record-continuous recording event-event recording alarm-alarm
        "ossRecordType":["record","event","alarm"],
        "ossExpireTime":1687847608, //Cloud storage expiration time, in seconds. It is null when cloud storage is not activated
        "encrypt": {
            "isEncrypt": true,//Whether to encrypt
            "encryptType": "1", //Encryption type, 1: AES128-CBC
            "encryptSecret": "YEcOY60RsJxIX6FZh2jLdFzP1etyjGog" //Encryption key
        }
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res": 0, //Status code 0 represents success, others represent failure; see status code definition
    "id": "45lkj3551234001", //Message ID (consistent with the message ID sent)
    "ts": 1626197189, //Timestamp (seconds)
    "code": "ipc_cloud_open", //Event code (consistent with the command code sent)
    "data": {
        //If the device does not reply or the reply is false, the cloud will continue to send at intervals of 10 minutes within one day of cloud storage activation,
        //Until the device replies to true
        "result": true //true - device successful processing,  false - device processing failure
    }
}
```

#### Acoustics event notification

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"agora_event",//Audio network event notification
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "eventType": 103, //Event type, 103-Anchor joins the channel, 104-Anchor leaves the channel
        "channelName": "1666425405739651072", //Channel name
        "channelUserNum":2, //Number of channel users, excluding devices
        "uid": "20230703", //User audio network uid
        "userId":"cn1658058388653281280" //User id
    }
}
```

#### General pass-through instruction

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"common_cmd",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "cmdType":"start_tall",//Command type (convention)
        "cmdData":{//Different command types have different contents
            "uid":0,//Add a uid, which is used for filtering on the APP side to support point-to-point interaction between devices and users
            "format":0,
            "channel":1,
            "rate":0,
            "volume":0,
            "Samples":16000
        }
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the message ID sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"common_cmd",//Event code (consistent with the command code sent)
    "data":{
        "respType":"start_tall"//Reply type (convention)
        "respData":{//Different reply types have different contents
            "uid":0,//Add a uid, which is used for filtering on the APP side to support point-to-point interaction between devices and users
            "format":0,
            "channel":1,
            "rate":0,
            "volume":0,
            "Samples":16000
        }
    }
}
```

#### Local group change notification

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_group_update",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data": {
        "shortId":1,//Sub-device group ID (short ID 0~9999)
        "name":"Group name",
        "assetId":"Room ID",
        "assetName":"Room name",
        "operatorType":1,//1: Add/modify, 2: Delete
        "bindUuid":"xxxx",//Binding device uuid
        "dpIdentifier":"switch_1",//Binding dp identifier
        //Full quantum device UUID under the group (optional: required when operatorType=1 or operatorType=3)
        "deviceUuids":["xxxxxx","xxxxxxxx","xxxxxxxx"]
    }
}
```

```JSON
//Device reply (no reply is needed when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the message ID sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_group_update",//Event code (consistent with the command code sent)
    "data": {
        "shortId":1,//Group ID (short ID 0~9999)
        "operatorType":1,//1: add/modify, 2: delete
        //When operatorType=1, reply to the device that has been successfully added, and when operatorType=2, reply to the device that has been successfully removed
        "devices":[
        {
            "pid":"Product id or model",
            "uuid":"xxxxxx1"
        }
        ],
        "result":"success"//success: success; fail: failure
    }
}
```

#### Local scene/one-click execution change notification

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_scene_update",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data": {
        "retry":0,//Whether to retry; 0 no; 1 yes
        "operatorType":1,//1: Add/modify, 2: Delete, 6: Disable, 7 Enable
        //Scene information (optional: required when operatorType=1 or operatorType=3)
        "scene":{
            "shortId": 1,//Scene ID (short ID 0~9999)
            "name":"Scene name",
            "executeType": 1,//Meet any condition: 1, Meet all conditions: 2
            //Conditions
            "conditions": [
            {
                "orderNum":1,
                "type":"device",//device: device; job: timing;
                "uuid": "xxxxxx",//device uuid
                "conditionColumn":"switch_led",//dp identifier
                "conditionExpress":"==",//greater than: >; equal to: ==; less than: <
                "conditionValue":true
            },
            {
                "orderNum":2,
                "type":"job",//device: device; job: timing;
                //cycle, 0000000 7 digits start from Monday, the value of 1 means that execution is allowed on that day;
                //If all are 0, the default is to execute only once
                "loops":"0000000",
                "time":"18:00",//time, 24-hour formatted time: such as: 18:00
                "startDate":"2023-01-01"//start date (yyyy-MM-dd)
            }
            ],
            //action
            "actions": [
            {
                "orderNum":1,
                "actionType":"device",//device: device, group: group
                "uuid": "xxxxxx",//must be sent when actionType=device
                "delayedTime":0,//delay time (seconds)
                //Execute action
                "actionData": {
                    "switch_led":false
                }
            },
            {
                "orderNum":2,
                "actionType":"group",//device: device, group: group
                "shortGroupId":1,//must be sent when actionType=group
                "delayedTime":60,//delay time (seconds)
                //Execute action
                "actionData": {
                    "switch_led":false
                }
            }
            ]
        }
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the message ID sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_scene_update",//Event code (consistent with the command code sent)
    "data": {
        "shortId":1,//Update type list
        "operatorType":1,//1: Add/Modify, 2: Delete, 6: Disable, 7 Enable
        "devices":[
        {
            "pid":"Product id or model",
            "uuid":"xxxxxx1"
        }
        ],
        "result":"success"//success success; fail: failure
    }
}
```

#### Device log switch

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"log_switch",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "enable":0, //0 off; 1 on
        "type":1,//Must transmit when enable=1, reporting method: 1 single report; 2: cyclic report;
        "intervalTime":10, //Must transmit when enable=1, upload cycle; minutes; 1~360 minutes
        "level":1 //Log level: 1debug; 2info; 3warn; 4error
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001", //Message ID (consistent with the message ID sent)
    "ts":1626197189, //Timestamp (seconds)
    "code":"log_switch", //Event code (consistent with the instruction code sent)
}
```

#### Device restart

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"reboot",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "subUuid":"xxx1"//Must be transmitted when the gateway sub-device restarts
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"reboot",//Event code (consistent with the sent command code)
}
```

#### Sub-device replacement

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_replace",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "uuid":"Replaced uuid",
        "newUuid":"New uuid"
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_replace",//Event code (consistent with the sent command code)
    "data":{
        "pid":"Product ID/Model",
        "uuid":"Replaced uuid",
        "newUuid":"New uuid",
        "result":"success"//success: success; fail: failure;
    }
}
```

#### Get device properties

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"property_get",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        "subUuids":["uuid1","uuid2"],//Must be transmitted when the gateway obtains sub-device properties
        "reportAll":true,//Report all properties when true
        "dps":["swidth","color"]//Must be transmitted when specifying dp reportAll=false
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"property_get", //Event code (consistent with the issued instruction code)
}
```

#### Set device configuration

Allowlist, backup network, connection settings, etc.  

```JSON
//===========Whitelist========================================
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "type":"white_list" //white_list whitelist
        "settings":{
            "operatorType":1,//1: Add, 2: Delete
            "uuids":["uuid1","uuid2"]
        }
    }
}
```

```JSON
//Whitelist reply (no reply is needed when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the message ID sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Event code (consistent with the instruction code sent)
    "data":{
        "type":"white_list", //white_list
        "settings":{
            "operatorType":1,//1: add, 2: delete
            "uuids":["uuid1","uuid2"]
        },
        "result":"success"//success success; fail: failure
    }
}
```

```JSON
//==========Operate the backup network list=======================================
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "type":"wifi_list", //wifi_list backup network list
        "settings":{
            "operatorType":1,//1: Add, 2: Delete
            "wifis":[{
                "ssid":"wifi_1",
                "pwd":"123456"//Required when adding
            }]
        }
    }
}
```

```JSON
//Operation backup network list device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the message ID sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Event code (consistent with the instruction code sent)
    "data":{
        "type":"wifi_list", //wifi_list backup network list
        "settings":{
            "operatorType":1,//1: add, 2: delete
            "wifis":[{
                "ssid":"wifi_1",
                "pwd":"123456"//Required when adding
            }]
        },
        "result":"success"//success success; fail: failure
    }
}
```

```JSON
//==========Network connection settings (change the network currently connected to the device)========================================
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "type":"wifi_set", //wifi_set network connection settings
        "settings":{
            "ssid":"wifi_1",
            "pwd":"123456"
        }
    }
}
```

```JSON
//Network connection settings reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the message ID sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Event code (consistent with the command code sent)
    "data":{
        "type":"wifi_set", //wifi_set network connection settings
        "settings":{
            "ssid":"wifi_1",
            "pwd":"123456"
        },
        "result":"success"//success success; fail: failure
    }
}
```

```JSON
//==========Tmall Genie serial port configuration=========================================
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "type":"aligenie_sp_config", //aligenie_sp_config Tmall Genie serial port configuration
        "settings":{
            //Device mapping
            "deviceMapping":[
            {
                "uuid":"", //Device UUID
                "name":"", //Device name
                "aliasList":[], //Device alias
                "commandMapping":{ //Command mapping
                    "turnOn":{ //Open command
                        "dpIdentifier":"", //Device DP point name
                        "dpValue":"" //Device DP point value
                    },
                    "turnOff":{ //Close command
                        "dpIdentifier":"", //Device DP point name
                        "dpValue":"" //Device DP point value
                    }
                }
            }
            ],
            //Scene mapping
            "sceneMapping":[
            {
                "name":"", //Scene name
                "aliasList":[], //Scene alias
                "shortId":"" //Local scene ID
            }
            ],
            //Infrared mapping
            "irMapping":{
                //Infrared air conditioner
                "air":{
                    "remoteId":"",
                    "codeUrl":"" //Code library address
                },
                //Infrared TV
                "tv":{
                    "remoteId":"",
                    "codeUrl":"" //Code library address
                }
            }
        }
    }
}
```

```JSON
//Network connection setting reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the ID of the sent message)
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings",//Event code (consistent with the command code issued)
    "data":{
        "type":"aligenie_sp_config", //aligenie_sp_config Tmall Genie serial port configuration
        "settings":{
            //Device mapping
            "deviceMapping":[
            {
                "uuid":"", //Device UUID
                "name":"", //Device name
                "aliasList":[], //Device alias
                "commandMapping":{ //Command mapping
                "turnOn":{ //Open command
                        "dpIdentifier":"", //Device DP point name
                        "dpValue":"" //Device DP point value
                    },
                    "turnOff":{ //Close command
                        "dpIdentifier":"", //Device DP point name
                        "dpValue":"" //Device DP point value
                    }
                }
            }
            ],
            //Scene mapping
            "sceneMapping":[
            {
                "name":"", //Scene name
                "aliasList":[], //Scene alias
                "shortId":"" //Local scene ID
            }
            ],
            //Infrared mapping
            "irMapping":{
                //Infrared air conditioner
                "air":{
                    "remoteId":"",
                    "codeUrl":"" //Code library address
                },
                //Infrared TV
                "tv":{
                        "remoteId":"",
                        "codeUrl":"" //Code library address
                }
            }
        },
        "result":"success"//success success; fail: failure
    }
}
```

#### Set password (door lock type)

```JSON
//Device password (door lock type)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"pwd_set",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "subUuid":"xxxxx",//Gateway sub-device uuid, this field is only available for gateway sub-devices
        "operatorType":1,//1 add/update; 2 delete;
        "pwdData":{
            "index":1,//Password ID
            //The following fields operatorType=1 only need to be paid attention to when adding passwords
            //Type: 1 digital password; 2 fingerprint; 3 card; 4 face; 5 iris; 6 palm print; 7 finger vein; 8 temporary digital password; 9 remote unlock password
            "type":1,
            "deviceUserId":"001",//User ID may be empty
            "pwd":"123456",//Required when password type=1 or 8
            "timeType":1,//Time type: 1: Permanent; 2: One-time; 3 Limited time;
            //The following fields only need attention when timeType=3 is limited time
            "activeTime":1626197189,//Effective timestamp (second level)
            "expireTime":1626197189,//Expiration timestamp (second level)
            "startTime":"12:00",//Start time (HH:mm)
            "endTime":"13:00",//End time (HH:mm)
            "loops":"1111111"//Cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on the same day)
        }
    }
}
```

```JSON
//Device reply (door lock type)
{
    "id":"45lkj3551234001",//Message ID (Consistent with the message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"pwd_set",//Command code (Consistent with the command code)
    "data":{
        "subPid":"xxxxx",//Gateway sub-device pid, required for gateway sub-device
        "subUuid":"xxxxx",//Gateway sub-device uuid, required for gateway sub-device
        "operatorType":1,//1 add/update; 2 delete;
        "pwdData":{
            "index":1,//Password ID
            //The following fields operatorType=1 are required when adding passwords
            //Type: 1 digital password; 2 fingerprint; 3 card; 4 face; 5 iris; 6 palm print; 7 finger vein; 8 temporary digital password; 9 remote unlock password
            "type":1,
            "deviceUserId":"001",//User ID
            "timeType":1, //Time-limit type: 1: permanent; 2: one-time; 3: limited time;
            //The following field timeType=3 is required when limited time
            "activeTime":1626197189, //Effective timestamp (second level)
            "expireTime":1626197189, //Expiration timestamp (second level)
            "startTime":"12:00", //Start time (HH:mm)
            "endTime":"13:00", //End time (HH:mm)
            "loops":"1111111" //Cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on that day)
        }
    }
}
```

#### Gateway data Initialization

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"gateway_data_init",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data":{
        //Sub-device uuid list
        "subUuids":[
            "xxxx1",
            "xxx2",
            "xxx3"
        ],
        "subDevices":[
        {
            "uuid":"xxx",
            "pid":"xxx",//Product ID or model
            "name":"aaaa",//Device name
            "assetId":"xxxx",//Room id
            "assetName":"xxxx"//Room name
        }
        ],
        //Local group list
        "localGroups":[
        {
            "shortId":1,//Local group id
            "name":"Group name",
            "assetId":"Room ID",
            "assetName":"Room name",
            "deviceUuids":[
                "xxxxxx",
                "xxxxxxxx",
                "xxxxxxxx"
            ]
        }
        ],
        //Local scene list
        "localScenes":[
        {
            "shortId":2,//Local scene ID
            "name":"Group name",
            "executeType":1,//Meet any condition:1,Meet all conditions:2
            //Conditions
            "conditions": [
            {
                "orderNum":1,
                "type":"device",//device: device; job: timing;
                "uuid": "xxxxxx",//Device uuid
                "conditionColumn":"switch_led",//dp identifier
                "conditionExpress":"==",//Greater than:>;Equal to:==;Less than:<
                "conditionValue":true
            },
            {
                "orderNum":2,
                "type":"job",//device: device; job: timing;
                //cycle, 0000000 7 digits start from Monday, value 1 means execution is allowed on that day;
                //If all are 0, it will be executed only once by default
                "loops":"0000000",
                "time":"18:00",//time, 24-hour formatted time: such as: 18:00
                "startDate":"2023-01-01"//start date (yyyy-MM-dd)
            }
            ],
            //action
            "actions": [
            {
                "orderNum":1,
                "actionType":"device",//device: device, group: group
                "uuid": "xxxxxx",//must be passed when actionType=device
                "delayedTime":0,//delay time (seconds)
                //execute action
                "actionData": {
                    "switch_led":false
                }
            },
            {
                "orderNum":2,
                "actionType":"group",//device: device, group: group
                "shortGroupId":1,//must be transmitted when actionType=group
                "delayedTime":60,//delay time (seconds)
                //Execute action
                "actionData": {
                    "switch_led":false
                }
            }
            ]
        }
        ]
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "id":"45lkj3551234001",//Message ID (consistent with the ID of the message sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"gateway_data_init"//Command code (consistent with the command code sent)
}
```

#### DP point binding local execution

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"dp_bind",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "subUuid":"",//Sub-device uuid Not required, if the value is passed, the sub-device dp is bound, if no value is passed, it means the gateway dp is bound
        "dpBinds":[
        {
            "identifier":"switch_type_1",//dp identifier
            "valueBinds":[
            {
                "dpValue":"1",
                "actions":[
                {
                    "type":1,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":"uuid",//Controlled device uuid
                    "delayedTime":0,//Delay time (seconds)
                    "actionData":{
                        "color":"red",
                        "switch":true
                    }
                },
                {
                    "type":1,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":"uuid",//controlled device uuid
                    "delayedTime":10,//delay time (seconds)
                    "actionData":{
                        "color":"red",
                        "switch":true
                    }
                },
                {
                    "type":2,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":1,//local group short id
                    "delayedTime":0,//delay time (seconds)
                    "actionData":{
                        "color":"red",
                        "switch":true
                    }
                },
                {
                    "type":3,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":1,//local one-click execution short id
                    "delayedTime":0,//delay time (seconds)
                    "actionData":null//One-click execution does not require action content, which has been set in advance
                }
                ]
            },
            {
                "dpValue":"2",
                "actions":[]//Empty means clearing the binding
            }
            ]
        }
        ]
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"dp_bind",//Command code (consistent with the sent command code)
    "data":{
        "subPid":"",//Sub-device pid has more reply value than sent.
        "subUuid":"",//Sub-device uuid is optional. If the value is passed, the sub-device dp is bound. If no value is passed, it means the gateway dp is bound.
        "dpBinds":[
        {
            "identifier":"switch_type_1",//dp identifier
            "valueBinds":[
            {
                "dpValue":"1",
                "actions":[
                {
                    "type":1,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":"uuid",//controlled device uuid
                    "delayedTime":0,//delay time (seconds)
                    "actionData":{
                        "color":"red",
                        "switch":true
                    }
                },
                {
                    "type":1,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":"uuid",//controlled device uuid
                    "delayedTime":10,//delay time (seconds)
                    "actionData":{
                        "color":"red",
                        "switch":true
                    }
                },
                {
                    "type":2,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":1,//local group short id
                    "delayedTime":0,//delay time (seconds)
                    "actionData":{
                        "color":"red",
                        "switch":true
                    }
                },
                {
                    "type":3,//1 local device; 2: local group; 3: local one-click execution
                    "targetId":1,//local one-click execution short id
                    "delayedTime":0,//delay time (seconds)
                    "actionData":null//One-click execution does not require action content, which has been set in advance
                }
                ]
            },
            {
                "dpValue":"2",
                "actions":[]//Empty means clearing the binding
            }
            ]
        }
        ]
    }
}
```

#### Local DP group change notification

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_dp_group_update",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data": {
        "shortId":1,//Sub-device group ID (short ID 0~9999)
        "operatorType":1,//1: Add/Modify, 2: Delete
        "deviceDps":[ // (Optional: Required when operatorType=1)
        {
            "uuid":"Device uuid",
            "dpIdentifier":"DP point identifier"
        }
        ]
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the ID of the issued message)
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_dp_group_update",//Event code (consistent with the code of the issued command)
    "data": {
        "shortId":1,//Group ID (short ID 0~9999)
        "operatorType":1,//1: add/modify, 2: delete
        "result":"success"//success; fail: failure
    }
}
```

#### Device information change notification

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"device_info_update",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "subUuid":"xxxx",//Sub-device uuid, must be transmitted when gateway is a sub-device
        "deviceInfo":{
            "assetId":"Room ID",
            "assetName":"Room name",
            "name":"Device name"
        }
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189, //Timestamp (seconds)
    "code":"device_info_update" //Event code (consistent with the issued instruction code)
}
```

#### Group information change notification

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"group_info_update",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "shortId":1,//Group short id
        "groupInfo":{
            "assetId":"Room ID",
            "assetName":"Room name",
            "name":"Group name"
        }
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"group_info_update"//Event code (consistent with the issued instruction code)
}
```

#### WEBRTC signaling exchange

```JSON
//Send data structure (SDP_OFFER)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_signal_exchange",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "peerClientId": "peer_0xi681aspzmlbm9",//Live_ indicates live broadcast, reply_ indicates playback
        "method": "SDP_OFFER",
        "params": "{\"type\": \"answer\", \"sdp\":\"\"}",
        "rawData":"{}",
        "autoPlay":false,//Whether to play automatically
        "source":1,//1:app,2:alexa,3:google
        "auth":{
            "userName":"1666534426:rn01xxxxx",
            "password":"xxxxfsafsf",
            "expireSecond":3600
        }
    }
}
```

```JSON
//Send data structure (ICE_CANDIDATE)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_signal_exchange",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "peerClientId": "peer_0xi681aspzmlbm9",
        "method": "ICE_CANDIDATE",
        "params":"[{\"candidate\":\"candidate:3621809022 1 udp 2113937151 192.168.26.71 57926 typ host generation 0 ufrag T3TR network-cost 999\",\"sdpMid\":\"0\",\"sdpMLineIndex\":0,\"usernameFragment\":\"T3TR\"}]",
        "rawData":"{}"
    }
}
```

```JSON
//Device reply (no reply is required when ack=0) --SDP_ANSWER
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the ID of the sent message)
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_signal_exchange"//Event code (consistent with the code of the sent command)
}
```

#### Remote unlocking (door lock type)

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"remote_unlock",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "subUuid":"",//Gateway sub-device uuid, this field is only available for gateway sub-devices
        "type":9,//Password type: currently fixed at 9
        "index":901,
        "pwd":"70f05e984557d8b7184e78a8554ec32b59581031"
    }
}
```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the message ID sent)
    "ts":1626197189,//Timestamp (seconds)
    "code":"remote_unlock"//Event code (consistent with the command code sent)
}
```

#### Notify device to clear data

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"clean_data",//Command code
    "ack":0, //0: No reply required; 1: Reply required
    "data": {
        "subUuid":""//Gateway sub-device uuid, this field is only available for gateway sub-device
    }
}

```

```JSON
//Device reply (no reply required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the sent message ID)
    "ts":1626197189,//Timestamp (seconds)
    "code":"clean_data"//Event code (consistent with the sent command code)
}
```

## 2. Product Level MQTT Messages

Subscribe Topic: `product/v2/${pid}/issue`  

```JSON
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"silence_ota",//Command code, see product command list
    "ack":1, //0: No reply required; 1: Reply required
    "data":{},//Send data structure See product command list->Data structure definition
}
```

Publish Topic: `product/v2/${pid}/issue_response`  

```JSON
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"silence_ota",//Command code, see product command list
    "data":{},//Reply data structure See product-level command list->data structure definition
}
```

### Product level instruction list

Code | Event name
---- | ----
 silence_ota | Whole product silent upgrade

#### Full product silent upgrade

Note: equipment should not be docked temporarily, the info protocol will actively detect when the equipment is powered on  

```JSON
//Send data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"silence_ota",//Command code
    "ack":1, //0: No reply required; 1: Reply required
    "data":{
        "taskId":"Upgrade task ID",
        "pid":"xxxxxx",//Product pid
        "pModel":"Product model",//Product model, used for third-party sub-devices
        "isSub":false, // Is it a sub-device product; false: no; true: yes
        "fileSize": 708482,
        "firmwareType":2,//1: factory firmware, 2: standard firmware; 3: mcu firmware
        "md5sum": "36eb5951179db14a63****a37a9322a2",
        "url": "https://ota-1255858890.cos.ap.mycloud.com",
        "version": "ALL",//Device version to be upgraded ALL means all versions
        "targetVersion":"1.0.1",//Target version number for upgrade
        "options":{ //Upgrade parameters Non-mandatory parameters
            "stable":"usr" //Upgrade partition usr/recovery/kernel
        }
    }
}
```

```JSON
//Device reply (no reply is required when ack=0)
{
    "res":0, //Status code 0 represents success, others represent failure; see status code definition
    "id":"45lkj3551234001",//Message ID (consistent with the ID of the issued message)
    "ts":1626197189,//Timestamp (seconds)
    "code":"silence_ota",//Event code (consistent with the code of the issued command)
    "data":{
        "taskId":"Upgrade task ID",
        "devices":[
        {
            "pid":"xxxx",//product id or model
            "uuid":"device uuid"
        }
        ]
    }
}
```

## 3. App Level MQTT Messages

Topic 1: `app/v2/${assetId}/notify`  
Topic 2: `app/v2/${userId}/userNotify`  

```JSON
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota_progress",//Event code, see device event list
    "data":{},//Data structure, see APP message list->data structure definition
}
```

### App Message Codes

Serial | Code | Description
---- | ----- | -----
 1 | ota_progress | Equipment Upgrade Progress Notification
 2 | property_change | Device attribute change notification
 3 | status | Device status change notification
 4 | bind_result | Binding result notification
 5 | ipc_token_used | IPC token usage success notification
 6 | wake_up_success | Device wake-up success notification
 7 | alert | Pop-up device event
 8 | common_cmd_resp | General instruction reply notification
 9 | find | The gateway searches for nearby device notifications
 10 | sub_replace_result | Subdevice replacement result notification
 11 | matter_find | Matter device search
 12 | matter_pair | Matter Device Pairing Notification
 13 | local_group_result | Local group save result notification
 14 | local_scene_result | Local scene save result notification
 15 | config_settings_resp | Set device configuration result notification
 16 | pwd_list_resp | Set password result notification (door lock type)
 17 | signal_check_result | Signal detection result notification
 18 | webrtc_signal_exchange | WEBRTC signaling exchange notification
 19 | device_properties_update | Device attribute update notification
 20 | sys_notice | System message notification

#### 1. Equipment Upgrade Progress Notification

Data Structure defination  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ota_progress",//Notification code
    "data":{
        //0, success; error code, -1: download timeout; -2: file does not exist; -3: signature expired; -4: MD5 does not match; -5: firmware update failed
        "resCode":0,
        "resMsg":"",//Error message
        "deviceId":"Device ID",
        "type":"download",//download: downloading, burning: upgrade burning, done: upgrade successful, fail: failure
        "percent":80, //Download progress 0~100;
        "version":"1.0.0"//Upgrade version number
    }//Data structure
}
```

#### 2. Device attribute change notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"property_change",//Notification code
    "data":{
        "devices":[
        {
            "deviceId":"Device ID",
            "gatewayId":"Gateway ID",//Required when reporting on a sub-device
            "properties":{
                "color":"red", //Color Red
                "brightness":80 //Brightness 80
            }
        }
        ],
        "groups":[
        {
        "groupId":"Group ID",
        "properties":{
            "color":"red", //Color Red
            "brightness":80 //Brightness 80
        }
        }
        ]
    }//Data structure
}
```

#### 3. Device status change notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"status",//Notification code
    "data":{
        "deviceId":"Device ID",
        "online":1, //Online status 0: offline; 1: online
        "tcpOnlineStatus":1 //Low power TCP online status 0: offline; 1: online Only low power devices pay attention to this field. Corresponding device tcpOnlineStatus
    }//Data structure
}
```

#### 4. Binding result notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"bind_result",//Notification code
    "data":{
        "devices":[
        {
            "deviceId":"Device ID",
            "uuid":"Device uuid",
            "trdUuid":"xxxxxx",//No real uuid when searching, used for binding result matching
            "gatewayId":"Gateway device ID",
            "type":"add",//add: bind; remove: unbind
            "result":"success", //success: success; fail: failure
            "errCode":0,//0: success; 17032: Device has been bound, others: binding failed
            "errMsg":""//Error description
        }
        ]
    }//Data structure
}
```

#### 5. IPC token usage success notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"ipc_token_used",//Notification code
    "data":{
        "token":"3rgfG5",
        "uuid":"xxxxxxxx"//UUID of the current scanning device
    }//Data structure
}
```

#### 6. Device wake-up success notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"wake_up_success",//Notification code
    "data":{
        "deviceId":"Device ID"
    }//Data structure
}
```

#### 7. Device pop-up notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"alert",//Notification code
    "data":{
        "deviceId":"Device ID"
    }//Data structure
}
```

#### 8. General instruction reply notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"common_cmd_resp",//Notification code
    "data":{
        "deviceId":"",//Device ID
        "respType":"",//Reply type
        "respData":{//Different reply types have different contents
        }
    }//Data structure
}
```

#### 9. The gateway searches for nearby device notifications

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"find",//Notification code
    "data":{
        "devices":[
        {
            "pid":"Product ID",
            "uuid":"Device uuid",
            "trdUuid":"xxxxxx"//No real uuid when searching, used for binding result matching
            "gatewayId":"Gateway device ID",
            "productName":"Name",
            "imageUrl":"Image",
            "switchDpList":["switch_1","switch_2"]//Quick switch dp
        }
        ]
    }//Data structure
}
```

#### 10. Subdevice replacement result notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sub_replace_result",//Notification code
    "data":{
        "deviceId":"Replaced device ID",
        "newUuid":"Replaced new uuid",
        "result":"success"//success: success; fail: failure;
    }//Data structure
}
```

#### 11. Matter device search  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"matter_find",//Notification code
    "data":{
        "clientId":"xxxx",//Unique identifier of the device that initiated the search If it is the current device, no processing is required
        "userId":"Search user ID" //If processing is required, and userId is the current logged-in user, the shared device needs to be transferred when calling the interface
    }//Data structure
}
```

#### 12. Matter Device Pairing Notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"matter_pair",//Notification code
    "data":{
        "clientId":"xxxx",//Unique identifier of the paired terminal. Only process if it is the current device
        "devices":[
        {
            "deviceId":"Device ID",
            "payload":"Pairing information"
        }
        ]
    }//Data structure
}
```

#### 13. Local group save result notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_group_result",//Notification code
    "data":{
        "groupId":"xxxx",//Group ID
        "devices":[
        {
            "deviceId":"Device ID",
            "status":1,//0 No response; 1 Success; 2: Failure
        }
        ]
    }//Data structure
}
```

#### 14. Local scene save result notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"local_scene_result",//Notification code
    "data":{
        "instanceId":"xxxx",//Scene ID
        "devices":[
        {
            "deviceId":"Device ID",
            "status":1,//0 No response; 1 Success; 2: Failure
        }
        ]
    }//Data structure
}
```

#### 15. Set device configuration result notification  

```JSON
//Data structure (whitelist)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"config_settings_resp",//Notification code
    "data":{
        "deviceId":"xxxx",//Device ID
        "type":"white_list", //white_list whitelist
        "settings":{
            "operatorType":1,//1: Add, 2: Delete
            "uuids":["uuid1","uuid2"]
        },
        "result":"success"//success success; fail: Failure
    }//Data structure
}
```

```JSON
//Data structure (backup network)
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//timestamp (seconds)
    "code":"config_settings_resp",//notification code
    "data":{
        "deviceId":"xxxx",//device ID
        "type":"wifi_list", //wifi_list backup network list
        "settings":{
            "operatorType":1,//1: add, 2: delete
            "wifis":[{
                "ssid":"wifi_1",
                "pwd":"123456"//required when adding
            }]
        },
        "result":"success"//success success; fail: failure
    }//data structure
}
```

```JSON
//data structure (network connection settings (change the network currently connected to the device))
{
    "id":"45lkj3551234001",//message ID
    "ts":1626197189,//timestamp (seconds)
    "code":"config_settings_resp",//Notification code
    "data":{
        "deviceId":"xxxx",//Device ID
        "type":"wifi_set", //wifi_set network connection settings
        "settings":{
            "ssid":"wifi_1",
            "pwd":"123456"
        },
        "result":"success"//success success; fail: failure
    }//Data structure
}
```

#### 16. Set password result notification (door lock type)  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"pwd_set_result",//Notification code
    "data":{
        "deviceId":"xxxx",//Device ID
        "operatorType":1,//1 add; 2 delete
        "pwdData":{
            "index":1,//Password ID is required, generated by the device when adding, to ensure uniqueness
            //The following fields are required when operatorType=1 is added
            "type":1,//1 digital password; 2: fingerprint; 3: card
            "deviceUserId":"001",//User ID
            "timeType":1,//Time type: 1: permanent; 2: single; 3 limited time;
            //The following fields are required when timeType=3 is limited time
            "activeTime":1626197189,//Effective timestamp (second level)
            "expireTime":1626197189,//Expiration timestamp (second level)
            "startTime":"12:00",//Start time (HH:mm)
            "endTime":"13:00",//End time (HH:mm)
            "loops":"1111111"//Cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on that day)
        }
    }
}
```

#### 17. (Obsolete) Signal Detection Result Notification  

(See: Device Property Update Notification )  

#### 18. WEBRTC signaling exchange notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"webrtc_signal_exchange",//Notification code
    "data":{
        "deviceId":"xxxx",//Device ID
        "peerClientId": "peer_0xi681aspzmlbm9",
        "method": "SDP_ANSWER", //SDP_ANSWER, ICE_CANDIDATE
        "params":"[{\"candidate\":\"candidate:3621809022 1 udp 2113937151 192.168.26.71 57926 typ host generation 0 ufrag T3TR network-cost 999\",\"sdpMid\":\"0\",\"sdpMLineIndex\":0,\"usernameFragment\":\"T3TR\"}]",
        "rawData":"{}" 
    } 
}
```

#### 19. Device attribute update notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"device_property_update",//Notification code
    "data":{
        "deviceId":"xxxx",//Device ID
        "propertiesInfo":{
            "temperatureUnit":null,//Temperature unit 1 degree Celsius 2 degrees Fahrenheit
            "amountUnit":null,//Cost unit CNY: RMB USD: US dollar EUR: Euro
            "amountRateShow":null,//Unit rate
            "currentSsid":"rino",//Currently connected wifi name
            "signal":null,//Signal strength: 1: good; 2: medium; 3: poor
            "networkType":null,//Network type: 1wifi; 2wired
            "signalValue":null,//Signal value (%)
            "ipcCurrentConnection":null,//Current number of ipc connections
            "ipcMaxConnection":null,//Maximum number of ipc connections
            "wifiList":[//Available networks
            {
                "ssid":"rino2.4",
                "rssi":-50
            }
            ],
            //...
        }
    }
}
```

#### 20. System message notification  

```JSON
//Data structure
{
    "id":"45lkj3551234001",//Message ID
    "ts":1626197189,//Timestamp (seconds)
    "code":"sys_notice",//Notification code
    "data":{
        "type":"message",//Type: message: message notification; homeShare: home sharing notification; deviceShare device sharing notification
        "messageInfo":{//Message notification content
            "msgType":"device_alarm",
            "msgCode":"alarm",
            "title":"Device alarm",
            "body":"Device sends alarm"
        },
        "shareInfo":{//Share notification content
            //To be determined
        }
    }
}
```

## 4. App Level REST API for device control

### Set Device Properties

url: `{host}/business-app/v1/device/command/propsIssue`  
method: POST  
content-type: application/JSON

Request parameter:  

```JSON
{
    "deviceId":"xxx", //Device ID
    "data":{
        "color":"red", //Color Red
        "brightness":80 //Brightness 80
        //......
    }
}
```

### Batch Set Device Properties  

url: `{host}/business-app/v1/device/command/batchPropsIssue`  
method: POST  
content-type: application/JSON

Request parameter:  

```JSON
{
    "deviceIds":["xxx"], //Device ID list
    "data":{
        "color":"red", //Color red
        "brightness":80 //Brightness 80
        //......
    }
}
```

### Device attribute reporting  

TODO: check use case!  

url: `{host}/business-app/v1/device/command/propsReport`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId":"xxx", //Device ID
    "data":{
        "color":"red", //Color Red
        "brightness":80 //Brightness 80
        //......
    }
}
```

### Firmware Upgrade API  

TODO: check use case!  

url: `{host}/business-app/v1/device/command/otaIssue`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "taskId":"xxxxxxx",//Upgrade task ID
    "deviceId":"xxx",//Device ID
    "firmwareType":2,//1: factory firmware, 2: standard firmware; 3: mcu firmware
    "fileSize": 708482,
    "md5sum": "36eb5951179db14a63****a37a9322a2",
    "url": "https://ota-1255858890.cos.ap.mycloud.com",
    "version": "0.2"//Upgraded firmware version number
}
```

### Batch firmware upgrade  

url: `{host}/business-app/v1/device/command/otaBatchIssue`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
[
    {
        "taskId":"xxxxxxx",//Upgrade task ID
        "deviceId":"xxx",//Device ID
    },
    {
        "taskId":"xxxxxxx",//Upgrade task ID
        "deviceId":"xxx",//Device ID
    }
]
```

### Start/stop searching and binding devices  

url: `{host}/business-app/v1/device/command/scanIssue`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId":"xxx",//Gateway device ID
    "second":60,//Search duration
    "bind":true,//true: Search and bind; false: Search only Default: true
    "find":true, //true: Start searching false: Stop searching Default: true
    "uuids":["uuid1","uuid2"],//Specify uuid, search all uuids without passing
    "pids":["pid1","pid2"]//Specify pid, search all pid without passing
}
```

### GROUP Control  

url: `{host}/business-app/v1/device/command/groupPropsIssue`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "groupId":"xxx",//Group ID
    "data":{
    "color":"red", //Color Red
    "brightness":80 //Brightness 80
    //......
}
}
```

### One-click execution  

url: `{host}/business-app/v1/device/command/onekeyExecuteIssue`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "sceneId":"xxx", //Scene ID, obtain the relevant rule ID through the scene ID and send it to the gateway
}
```

### Wake up the Device  

url: `{host}/business-app/v1/device/command/wakeUpIssue`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId": "xxx" //Device ID
}
```

### Notify the device to join sound network channel  

url: `{host}/business-app/v1/device/command/notifyDeviceJoin`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId":"xxx", //Device ID
    "agoraModel":"live"//Audio network mode, live-live broadcast, playback-playback
}
```

### General pass-through command  

url: `{host}/business-app/v1/device/command/commonCmd`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId":"xxx", //Device ID
    "cmdType":"", //Command type (convention)
    "cmdData":{//Different command types have different contents

    }
}
```

### Device Restart  

url: `{host}/business-app/v1/device/command/reboot`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId": "xxx" //Device ID
}
```

### Sub-Device replacement  

url: `{host}/business-app/v1/device/command/subReplace`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId":"Device ID to be replaced", //Device ID
    "newUuid":"Uuid to be replaced"
}
```

### Get Device Properties  

url: `{host}/business-app/v1/device/command/propsGet`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId":"The replaced device ID", //Device ID
    "reportAll":true, //Whether to report all attributes
    "dps":["swidth","color"] //Required when specifying dp reportAll=false
}
```

### Set Device Configuration  

allowlist, backup network, network connection, etc.

url: `{host}/business-app/v1/device/command/configSettings`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
//Whitelist settings
{
    "deviceId":"Device ID", //Device ID
    "type":"white_list", //white_list whitelist
    "settings":{
        "operatorType":1, //1: add, 2: delete
        "uuids":["uuid1","uuid2"] //If it is a third-party device, pass the third-party uuid
    }
}
```

```JSON
//Backup network settings
{
    "deviceId":"Device ID", //Device ID
    "type":"wifi_list", //wifi_list backup network
    "settings":{
        "operatorType":1, //1: add, 2: delete
        "wifis":[{
            "ssid":"wifi_1",
            "pwd":"123456"//Required when adding
        }]
    }
}
```

```JSON
//Network connection settings (change the network currently connected to the device)
{
    "deviceId":"Device ID", //Device ID
    "type":"wifi_set", //wifi_set network connection settings
    "settings":{
        "ssid":"wifi_1",
        "pwd":"123456"
    }
}
//......
```

### Detection equipment signal strength  

url: `{host}/business-app/v1/device/command/checkSignal`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId": "Device ID" //Device ID
}
```

### WEBRTC Signaling Exchange  

url: `{host}/business-app/v1/device/command/webrtcSignalExchange`  
method: POST  
content-type: application/JSON

Request parameter:

```JSON
{
    "deviceId":"Device ID", //Device ID
    "peerClientId": "peer_0xi681aspzmlbm9",
    "method": "SDP_OFFER",//SDP_OFFER、ICE_CANDIDATE
    "params":"[{\"candidate\":\"candidate:3621809022 1 udp 2113937151 192.168.26.71 57926 typ host generation 0 ufrag T3TR network-cost 999\",\"sdpMid\":\"0\",\"sdpMLineIndex\":0,\"usernameFragment\":\"T3TR\"}]",
    "rawData":"{}"
}
```

## Status Codes

Status code | Description
----------- | ---------
 0 | Default state, represents success
 1001 | Service exception
 1002 | Invalid parameter
 1003 | Message format error
 1004 | Device does not exist
 1005 | Product does not exist
 1006 | Service not activated
 17032 | Device is bound
 24029 | Scene does not exist
 24030 | One-click execution is not supported.
 24034 | Device group does not exist
