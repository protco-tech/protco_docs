
# Bluetooth transmission protocol  

**TODO:** _Add pairing flow diagram_

The protocol data is in JSON format  
sign:
hmacSha256(content, deviceSecret)，
Bluetooth deviceSecret takes the first 8 encryption keys
Value of content uuid=6c828cba434ff40c074wF2, ts = 1607635284, mq = web.protco.com  
When there is no MQ parameter, use uuid=6c828cba434ff40c074wF2, ts = 1607635284
This order assembles the plaintext content, and deviceSecret encrypts the CDKEY according to the first 8 bits of the given string.

Single Bluetooth encryption:  
Use AES encryption algorithm, PKCS7, EBC.
Take the secret of the triplet for MD5 and then convert it to hex string, and then take 8-24 bits as AES key after MD5. The reference code is as follows:  
  String secretMd5 = SecureUtil.md5(secret).toLowerCase();  
    String hexStr = HexUtil.encodeHexStr(secretMd5);  
    String hexMd5 = SecureUtil.md5(hexStr).toLowerCase();  
    String result = StrUtil.sub(hexMd5,8,24);  
    return result;  

## Bluetooth Broadcasting data Format  

Bluetooth name: RLINK  
Broadcast data:
deviceType 1 byte 01 - Bluetooth Dual Mode Device (BK)  
isBind 1 byte, whether the user has been bound, 00-no 01-yes  
Uuid 10 bytes  
Pid XXX bytes  

## Transfer protocol between APP and device  

Byte | Data | Value
---- | ---- | ----
 1byte | Header | 0xFF
 1byte | Type | 1 = Normal command, 2 = OTA data
 2byte | Packet Serial Number | Starts with 0
 2byte | Total package number | Total number of packets for complete data
 2byte | Total length | Total data length, in all packets
 1byte | Data length in current packet | &nbsp;
 nbyte | Current Packet Data | &nbsp;
 1byte | Checksum | 1byte sum of all bytes except Header and Checksum

Note:

- The maximum length of BLE data transfer length is 128 bytes.  
Therefore, the maximum valid data length for each packet is 118 = 128 - (FF + type + sequence number + total number of packets + total length + data length + checksum).  
- Use big endian  

## Receiver response  

The data sent by the receiver to the sender after receiving the data is either successful or failed.  

```JSON
{
    "ts":33333,
    "msg":"error",
    "code": 0, //0: success, negative number means failure
    "cmd_type": 1, //1 represents command, 2 represents OTA
}
```

## Bluetooth distribution protocol (server level return)

Transmission SSID and PASSWORD ( dual mode time distribution network)

```JSON
//APP send：
{
    "type":"thing.network.set",
    "msgId":"45lkj3551234001",
    "ts":1626197189638,
    "data":{
        "force_bind":true, //Whether to force device binding
        "sid":"XXsmart-2.4G", //Router ssid
        "pw":"31415926", //Router password
        "mq":"mqtt.protco.in", //mqtt host
        "bid":"ssssss321345e", //bind id
        "port":1883, //mqtt port
        "country":"IN", //Country CN, US, JP, EP, AU
        "areaCode":"91", //Country code 91
        "tz":"Asia/Kolkata", //User time zone
        "userId":"cn1752575749181579264", // ipc network binding
    }
}
```

BLE binding (single mode, i.e. pure Bluetooth mode)  

```JSON
{
    "type":"thing.network.set",
    "msgId":"45lkj3551234001",
    "ts":1626197189638,
    "data":{
    "ble": "bind", //bind: bind, unbind: unbind, init: unbind and clear data
        "force_bind":true
    }
}
```

Response  

```JSON
{
    "type":"thing.network.set.response",
    "msgId":"45lkj3551234001",
    "ts":1626197189638,
    "code":0 // Reference unified code
    //1006 WiFi connection successful
    //1007 Network configuration error failed, username and password incorrect
    //1008 Network configuration error failed, no nearby routers found, signal or WiFi device problem
    //1499 BLE received a packet of data (not used for the time being)
    //1501 BLE received data analysis, packet header abnormality device.information.get
    //1502 BLE received data analysis, sn abnormality
    //1503 BLE received data analysis, crc abnormality
    //1504 BLE received data analysis, device cache 1 abnormality
    //1505 BLE received data analysis, device cache 2 abnormality
    //1506 BLE received data analysis, total data length abnormality
    //1507 Please send the next packet of data (Not used for the time being)
    //1700 json parsing failed
    //1701 json parsing succeeded, network connection started
    //1702 server connection failed
    //1703 server connection successful
    //1704 ble bind response
    //1801 bind successful
    //1802 bind failed
    //1803 unbind successful
    //1804 unbind failed
    //1805 init successful
    //1806 init failed
    //2000 physical model parsing successful
    //2001 physical model parsing failed
}
```

## Reconnect Mechanism  

When connected,
If no thing.network .set.response is received within 3 seconds, resend the thing.network data
When an error of 1500 to 1599, 1700 is received, resend thing.network

Reconnect (if disconnected) and resend distribution network data. When the distribution network page exits, cancel the reconnection mechanism

When receiving 1007, 1008, 1702, 1802, 1804, 1806, an error can be returned

Logs need to report thing.network .set.response all Bluetooth logs

## Get a list of nearby WIFI  

```JSON
{
    "type":"thing.network.getwifis",
    "scan": true //true: requires the device to scan WiFi immediately, false: obtain pre-scan data

    // If scan does not exist, obtain pre-scan data
}
```

Response  

```JSON
{
    "type":"thing.network.getwifis.response",
    "msgId":"45lkj3551234001",
    "ts":1626197189638,
    "data":{
    "wifis":[
        {
            "ssid":"xxxx",
            "rssi":"111",
            "security":true
        },
        {
            "ssid":"xxxx",
            "rssi":"111",
            "security":false
        }
        ]
    },
    "code":0 // Reference unified code
    //1007 Network configuration error failed, incorrect username and password
    //1008 Network configuration error failed, no nearby routers found, signal or wifi device problems
}
```

## APP settings properties  

```JSON
{
    "type":"thing.property.set",
    //Refer to MQTT protocol
    "sign":"5375a1a9275bafab52c21cc77de2cb99eaf784dd"
}

//Send data structure:

{"type":"thing.property.set","data":{"open":true}}
```

Response  

```JSON
/**{
"type":"thing.property.set.response",

"code":0 // Reference unified code
//1007 Network configuration error failed, incorrect username and password
//1008 Network configuration error failed, no nearby routers, signal or wifi device problems found
}*/

//Report data:
{"msgId":"bae1a747fc92499","data":{"open":{"value":true}},"type":"thing.property.report"}
```

## APP reads properties  

```JSON
{
    "type":"thing.property.get",
    //Refer to MQTT protocol
    "sign":"5375a1a9275bafab52c21cc77de2cb99eaf784dd",
}
```

Response  

```JSON
{
    "type":"thing.property.get.response",

    "code":0 // Reference unified code
    //1007 Network configuration error failed, incorrect username and password
    //1008 Network configuration error failed, no nearby routers, signal or wifi device problems found
}
```

## App reads device information  

```JSON
{
    "type":"device.information.get",
    "ts":1626197189638,
}

{
    "type":"device.information.get.response",
    "ts":1626197189638, //At least greater than and equal to the timestamp just sent
    "data": {
        "version": "1.1.2",
        "mcuVersion": "1.0.2",
        "msgType": "poweron",
        "pid": "2vRXuUhlmmDZCGUa",
        "bind": false,
        "gateway": 1,
        "gatewayType": 1,
        "bindId":"", //
        "ble_mac":"xxx:ddd",
        "wifi_mac": "sss",
        "thing_model": false, //Whether the device has a thing model,
        "get_thing_model": true, //The device needs the APP to send the thing model, sigmesh to ble Required for direct control
        "signMethod":"hmacSha1", // hmacSha1, hmacSha256, optional, default hmacSha1
        "sign":"encrypt according to triple = hmacSha1(content, deviceSecret)" //
    }
}

// Same as WIFI encryption rules,
// hmacSha256(content, deviceSecret), content value uuid=rn010f8c391Ff133,
// ts=(1626197189638+1) assemble plaintext content in this order, deviceSecret takes secret/key in triple
```

## App downlink model  

When the device is a universal firmware, the App simulates the cloud response device and sends a list of object models, model structure (1 = simplified, 2 = mini-simplified, 3 = super-simplified, 0 = full version), App usage 3 = super-simplified, type reference MQTT  

```JSON
//  app send
{
    "type": "thing.model.get.response",
    "ts": 1626197189638,
    "data": {
        "properties": [
        {
            "m": "rw",
            "b": "101",
            "i": "current_mode",
            "s": {
                "enums": [
                "manual",
                "rhythm"
                ]
            },
            "t": "enum"
        },
        {
            "m": "rw",
            "b": "20",
            "i": "switch",
            "s": {
                "true": "open",
                "false": "close"
            },
            "t": "bool"
        },
        {
            "m": "rw",
            "b": "102",
            "i": "manual_mode_brightness_set",
            "s": {
                "unitName": "%",
                "min": 30,
                "max": 1000,
                "step": 1,
                "accuracy": 0
            },
            "t": "int"
        }
        ]
    }
}

// Device response
{
    "code":2000, // Other data unknown
}
//2000 Object model parsing successful
//2001 Object model parsing failed
```

## App issue clear data  

```JSON
//app sends
{
    "type":"device.data.clear",
    //Refer to MQTT protocol
    "data":{
        "flag": 0, //0: Clear all application data but do not unbind, that is, do not clear the binding relationship data
    }
}

//Send data structure:
{"type":"device.data.clear","data":{"flag":0}}

//Device response
{
    "type": "device.data.clear.response",
    "ts": 1626200000,
    "data": {
        "status": "0",//0 Success Others: Failure
    }
}
```

## Get user time and time zone  

```JSON
//app sends
{
    "type": "device.timezone.get",
    "data":{
        "ts":1626197189, //Timestamp (seconds)
        "tz":"Asia/Shanghai",//Transmit according to the user's time zone
        "offset":28800 //Offset (seconds) Asia/Shanghai offset 8 hours = 8*60*60 = 28800
    }
}

//Device response
{
    "type": "device.timezone.get.response",
    "ts": 1626200000,
    "data": {
        "status": "0",//0 success others: failure
    }
}
```

## Aggregation protocol: Get device attributes, OTA version, time zone information  

Device active acquisition  

```JSON
//Device request
{
    "type": "time",
    "ts":1626197189638,
}

//app reply
{
    "type":"time.response",
    "data":{
        "ts":1626197189, //Timestamp (seconds)
        "sys_tz":"Asia/Shanghai",
        "zone_offset":28800 //Offset (seconds) Asia/Shanghai offset 8 hours = 8*60*60 = 28800
    }
}
```

```JSON
//app sends
{
    "type": "device.aggregation.get",
    "data":{
        "ts":1626197189, //Timestamp (seconds)
        "tz":"Asia/Shanghai",//Transmit according to the user's time zone
        "offset":28800 //Offset (seconds) Asia/Shanghai offset 8 hours = 8*60*60 = 28800
    }
}

//Device response
{
    "type": "device.aggregation.get.response",
    "ts": 1626200000,
    "code": 0,
    "data": {
        "version": "1.1.2",
        "mcuVersion": "1.0.2",
        "property":{"open":true}//{"open":true,"xxx":"xxx"}
    }
}
```

## OTA Upgrade  

### OTA upgrade initialization request  

```JSON
//app sends
{
    "type": "ota.upgrade.initiate",
    "ts": 1626200000000,
    "data": {//If there is no data, the default is the module standard firmware
        "firmwareType": 2, //1: factory firmware, 2: standard firmware, 3MCU firmware,
    }
}

//Device response
{
    "type": "ota.upgrade.initiate.response",
    "ts": 1626200000100,
    "data": {
        "status": "0", //0 OK ota 1 Reject //The device rejects the following data and will not send it
        "max": 512, // Maximum data that can be written at one time
        "v":"1.0.2", //Current module firmware version
        "mv":"1.0.2" //Current mcu firmware version, if not, will not send it
        //Only the corresponding version number is returned each time, that is, the module firmware returns the module version, and the MCU returns the MCU version number
    }
}
```

### OTA file information transfer  

```JSON
{
    "type": "ota.file.info",
    "ts": 1626200000200,
    "data": {
        "pid": "awaxfas",//Reserved, not transferred if not available
        "size": 1048576,//File length
        "crc": "File crc32",
        "ver":"1.0.0" //Version number of the firmware to be upgraded
    }
}

{
    "type": "ota.file.info.response",
    "ts": 1626200000300,
    "data": {
        "status": "0",//Normal upgrade, 1.pid is inconsistent 2. Version is lower than or equal to the current version 3. File size exceeds the range
        "s_len" :"", //Stored file length
        "s_crc" :"", //Stored file crc32
    }
}

//In order to support breakpoint resume, the file information stored on the device needs to be returned here.
//After receiving the data, the app intercepts the file data and calculates crc32, then compares the crc value
//If the crcs are consistent, the transmission starts from the offset, if not, the transmission starts from 0
```

### OTA upgrade offset  

```JSON
//app sends
{
    "type": "ota.file.offset",
    "ts": 1626200000400,
    "data": {
        "offset": 100
    }
}

//Device response
{
    "type": "ota.file.offset.response",
    "ts": 1626200000500,
    "data": {
        "offset": 0 //The file transfer offset required by the device. The actual transfer is subject to the device requirements
    }
}
```

### OTA file data transfer  

```JSON
{
    "type": "ota.file.data",
    "ts":1713260670,
    "data":"Package data"
}
//Package data: package number + current package data length + data + crc16
// 2 (bytes) + 2 (bytes) + n (bytes) + 2 (bytes)
// max of ota.upgrade.initiate.response

{
    "type": "ota.file.data.response",
    "ts": 1626200000500,
    "data": {
        "status":0 //0 success 1 abnormal package number 2 length error 3 crc check error 4
    }
}
```

### OTA upgrade is over  

```JSON
{
    "type": "ota.complete",
    "ts": 1626200000400,
}

{
    "type": "ota.complete.response",
    "ts": 1626200000500,
    "data": {
        "status":0 //0 success 1 total data length error 2 total data crc check failed 3 other
    }
}
```

## Door Lock  

### App Edit Password  

```JSON
{
    "type":"thing.pwd.set",
    "ts":1626197189638,
    "data":{
        "code":"add/remove/edit",
        "type":1,//1-digit password; 2: fingerprint; 3: card
        "index":901,//Serial number
        "pwd":"123456",//Required when password type=1
        "active":1626197189,//Effective timestamp (second level)
        "expire":1626197189,//Expiration timestamp (second level)
        "cycle":false,//Whether to cycle
        "start":"12:00",//Start time (HH:mm)
        "endT":"13:00",//End time (HH:mm)
        "loops":"1111111"//Cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on that day)
    }
}
```

Response  

```JSON
{
    "type":"thing.pwd.set.response",
    "ts":1626197189638
}
```

### App Read Password  

```JSON
{
    "type":"thing.pwd.getList",
    "ts":1626197189638
}
```

Response  

```JSON
{
    "type":"thing.pwd.getList.response",
    "ts":1626197189638,
    "data":{
        "pwds":[
        {
            "type":1,//1-digit password; 2: fingerprint; 3: card
            "index":901,//serial number
            "pwd":"123456",//password required when type=1
            "active":1626197189,//effective timestamp (second level)
            "expire":1626197189,//expiration timestamp (second level)
            "cycle":false,//whether to cycle
            "start":"12:00",//start time (HH:mm)
            "endT":"13:00",//end time (HH:mm)
            "loops":"1111111"//cycle (0000000, 7 digits starting from Monday, value 1 means execution is allowed on that day)
        }
        ]
    }
}
```

### App Password Status Report  

```JSON
{
    "type":"thing.pwd.status.response",
    "ts":1626197189638,
    "data":{
        // Fingerprint password progress report
        "max":4,
        "current":1
    }
}
```
