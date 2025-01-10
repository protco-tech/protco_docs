
# WiFi AP mode network configuration  

Method: Connect the phone to the 2.4G WiFi network, fill in the WiFi password, click "Next", and connect to the device hotspot "protcosmart-XXXX".  

## Configuration Process  

1. Device in AP mode, broadcasts the following JSON data in UDP port `22222`

    ```JSON
    {
        "type":"thing.info.broadcast",
        "msgId":"45lkj3551234001",
        "ts":1626197189638,
        "data":{
            "name":"protco-zigbee-gateway",
            "uuid": "rn135adawdasd",
            "pid":"S8rHEJgnoOkntd",
            "udpPort": 30000, // Device listening port
        }
    }
    ```

2. App queries for list of visible APs to the device

    App sends the following data  

    ```JSON
    {
        "type":"thing.network.getwifis",
        "scan":true
    } 
    ```

3. Device Scans and replies the list of APs to the App

    ```JSON
    {
        "type":"thing.network.getwifis.response",
        "msgId":"f4ab750d1f334df",
        "ts":1626197189638,
        "code":0,
        "data":{
            "wifis":[
                {"ssid":"TplinkPT","rssi":-51,"security":true},
                {"ssid":"PTJ_4G","rssi":-52,"security":true},
                {"ssid":"TplinkPT","rssi":-63,"security":true},
                {"ssid":"Airtel_pint_3661","rssi":-63,"security":true},
                //...
                ]}
    }
    ```

4. App sends connection parameter to the device  

    ```JSON
    {
        "type":"thing.network.set",
        "ts":1721374457673,
        "data":{
            "bid":"1740300523660267580",
            "country":"IN",
            "force_bind":true,
            "mq":"testmqtt.protco.in",
            "port":1883,
            "pw":"******",
            "sid":"TplinkPT",
            "userId":"us1689940117281427456"
        }
    }
    ```

JSON data format is similar to BLE data transfer protocol. Please refer to the [BLE data transfer protocol documentation.](../bluetooth/ble_data_transfer_protocol.md)