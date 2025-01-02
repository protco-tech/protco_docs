
# WiFi device provisioning in BLE mode  

This method is for the devices with WiFi+BLE capability.

## Network configuration process  

1. App sends

    ```JSON
    {"type":"thing.network.getwifis","scan":true}
    ```

2. Device Scans and replies the list of APs to the App

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

3. App sends  

    ```JSON
    {"type":"device.information.get"} 
    ```

4. Device sends

    ```JSON
    {
        "type":"device.information.get.response",
        "ts":1721192794,
        "data":{
            "version":"1.0.0",
            "msgType":"poweron",
            "pid":"fFMes8kzaLRHMy",
            "bind":false,
            "gateway":0,
            "gatewayType":1,
            "sign":"b1dd387b1e7ec0b4804990775ffca0fe15186e2b",
            "wifi_mac":"44:4a:d6:0a:ba:a9",
            "ble_mac":"44:4a:d6:0a:ba:aa"
        }
    }
    ```

    Note:  
    hmacSHA1(content,secretKey)

    content: note: timestamp+1
    uuid=rn01b55A74fa0c16,ts=1721192795

    secretKey = device secret = bd2625eb1fa0499ea60aadd09ae0a184

5. App sends bind message  

    ```JSON
    {
        "type":"thing.network.set",
        "data":{
            "force_bind":false,
            "country":"US",
            "mq":"testmqtt.protco.in",
            "port":1883,
            "tz":"Asia/Kolkata",
            "pw":"123**4567",
            "bid":"1740300523660267520",
            "userId":"us1689940117281427456",
            "sid":"TplinkPT"
        }
    }
    ```

6. Device replies connecting progress status

    ```JSON
    {"ts":22,"code":1701,"msg":"success","type":"thing.network.set.response"}

    {"ts":22,"code":1006,"msg":"BLE_DATA_WIFI_GOT_IP","type":"thing.network.set.response"}

    {"ts":22,"code":1801,"msg":"bind success","type":"thing.network.set.response"}

    {"msgId":"1813440145827258368","ts":1721192798,"type":"sys.event.trigger.response"}

    {"ts":22,"code":1703,"msg":"Successfully connected to the server","type":"thing.network.set.response"}
    ```

    For detail status codes please refer to [BLE data transfer protocol documentation.](../bluetooth/ble_data_transfer_protocol.md)
