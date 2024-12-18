
# Zigbee Custom Protocol

**TODO:** _Complete it according to the latest docs received from the RINO team_  

Protocol composition: `Header + Payload`  
Note: `u16/u32` in the protocol are all little-endian  

Header:  

The protocol header fields are as follows.  

Field | Bytes | Description
--- | --- | ---
 Header | 1 | The header is fixed to 0xFF
 Command | 1 | Command word
 Sequence | 1 | command sequence number, increases 1 for every message
 Total packets | 1 | Total number of packets
 Packet serial | 1 | Current packet serial number, serial starts with 0

List of Command  

Command | Description
--- | ---
 0x99 | Heartbeat
 0x01 | Ping request, the device replies with heartbeat packet content after receiving ping request. 
 0x02 | Query device status
 0x03 | Gateway network status
 0x04 | The gateway issues DP commands
 0x05 | Device reporting status (passive)
 0x06 | Device reporting status (active)
 0x0b | MCU version query request
 0x0c | MCU OTA upgrade notification
 0x0d | MCU_OTA firmware content request
 0x0e | MCU_OTA upgrade result reporting
 0x24 | Sync Clock Time
 0x41 | KID GID SID

Payload:
Protocol payload format is different for each Command.
Ping request payload content(0x00)
UTC	4	Utc time, little endian
Divisor	1	divisor, non-zero
Remainder	1	remainder
Max	1	maximum value
BaseTime	1	Base time, unit: ms

 Whether the received node responds to the heartbeat request is calculated as follows:
Response conditions: 
Device_node_id % Divisor == Remainder ,
Delayed heartbeat response time:
BaseTime * random (0, Max)

Heartbeat response packet content(0x99)
Start	 CMD	SEQ	LEN	VER	MCU_VER _ _ _
0XFF	0x99	* *	**	Module version number	Optional

Gateway query device DP status(0x02)
DP_Num 	1	DP quantity, followed by list of DP points (0 or 0xFF means device should report all DP statuses)
Dpid_0 	1	1st dpid
Dpid_N  ..	1	Nth dpid (query up to 1 0 )

The payload format of the command 0x04, 0X05, 0x06 is as follows.
DP_Num 	1	Number of  DP function points
Dpid	1	Function point(DPID) number
Type	1	type of data
Len	2	Data length (little endian)
Data	n	valid data

DP Type is defined as:  

```C
typedef enum {
    PT_DP_TYPE_UNKNOWN,
    PT_DP_TYPE_BOOL,    //1byte
    PT_DP_TYPE_INT,     //4byte little endian
    PT_DP_TYPE_FLOAT,   //4byte Little endian
    PT_DP_TYPE_ENUM,    //1byte
    PT_DP_TYPE_FAULT, //1byte - fault is also enum in protco platform
    PT_DP_TYPE_TEXT,
    PT_DP_TYPE_RAW,
    PT_DP_TYPE_STRUCT,
    PT_DP_TYPE_ARRAY,
} TE_PT_DP_TYPE;
```
