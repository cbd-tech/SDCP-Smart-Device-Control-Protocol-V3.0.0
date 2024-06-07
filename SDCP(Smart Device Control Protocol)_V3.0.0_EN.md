# SDCP(Smart Device Control Protocol)V3.0.0

Shenzhen CBD Technology Co.,Ltd

## Overview

The SDCP protocol is an application layer protocol for interaction between the client and the motherboard, which includes command control, file transfer, Status Monitoring, and other content.

Protocol communication is conducted using the JSON format for interaction, with video stream data employing the RTSP protocol for real-time communication. Specific commands should be referred to and interacted according to the definitions provided below.

## Device Discovery Description

The motherboard initiates a WebSocket service on port 3030 after powering on.

The client broadcasts the string "M99999" using the UDP protocol on port 3000 within the local area network segment. Upon receiving this command, the motherboard replies with the following content.

```json
{
    "Id": "xxx",  // Machine brand identifier, 32-bit UUID
    "Data": {
        "Name": "PrinterName",  // Machine Name
        "MachineName": "MachineModel",  // Machine Model
        "BrandName": "CBD",  // Brand Name
        "MainboardIP": "192.168.1.2",  // Motherboard IP Address
        "MainboardID": "000000000001d354",  // Motherboard ID(16bit)
        "ProtocolVersion": "V3.0.0",  // Protocol Version
        "FirmwareVersion": "V1.0.0"  // Firmware Version
    }
}
```

## Connecting Device Description

Client connects to the motherboard WebSocket address: ws://${MainboardIP}:3030/websocket

## Topic

Topic Explanation: (${MainboardID} refers to the motherboard ID)

```text
SDCP Control Request (Client -> Motherboard): sdcp/request/${MainboardID}

SDCP Control Response (Motherboard -> Client): sdcp/response/${MainboardID}

SDCP Status Information (Motherboard -> Client): sdcp/status/${MainboardID}

SDCP Attribute Information (Motherboard -> Client): sdcp/attributes/${MainboardID}

SDCP Error Message (Motherboard -> Client): sdcp/error/${MainboardID}

SDCP Notification Message (Motherboard -> Client): sdcp/notice/${MainboardID}
```

## Heartbeat

Request Format Definition

```text
"ping"
```

Response Format Definition

```text
"pong"
```

## Attribute Information

The motherboard ensures to report the machine attribute information to the client at the following circumstances:

1. When the attribute information changes.
2. Upon receiving the control command to report attribute information.

The attribute information definition object content includes:

```json
{
    "Attributes": {
        "Name": "PrinterName",  // Machine Name
        "MachineName": "MachineModel",  // Machine Model
        "BrandName": "CBD",  // Brand Name
        "ProtocolVersion": "V3.0.0",  // Protocol Version
        "FirmwareVersion": "V1.0.0",  // Firmware Version
        "Resolution": "7680x4320",  // Resolution
        "XYZsize": "210x140x100" ,  // Maximum printing dimensions in the XYZ directions of the machine, in millimeters.(mm)
        "MainboardIP": "192.168.1.1",  // Motherboard IP Address
        "MainboardID": "000000000001d354", // Motherboard ID(16)
        "NumberOfVideoStreamConnected": 1,  // Number of Connected Video Streams
        "MaximumVideoStreamAllowed": 1,  // Maximum Number of Connections for Video Streams
        "NetworkStatus": "'wlan' | 'eth'",  // Network Connection Status, WiFi/Ethernet Port
        "UsbDiskStatus": 0,  // USB Drive Connection Status. 0: Disconnected, 1: Connected
        "Capabilities":[                            
            "FILE_TRANSFER",  // Support File Transfer
            "PRINT_CONTROL",  // Support Print Control
            "VIDEO_STREAM"  // Support Video Stream Transmission
        ],  // Supported Sub-protocols on the Motherboard
        "SupportFileType":[
            "CTB"  // Supports CTB File Type
        ],
        //Device Self-Check Status
        "DevicesStatus":{
            "TempSensorStatusOfUVLED": 0, // UVLED Temperature Sensor Status, 0: Disconnected, 1: Normal, 2: Abnormal
            "LCDStatus": 0,  // Exposure Screen Connection Status, 0: Disconnected, 1: Connected
            "SgStatus": 0,  // Strain Gauge Status, 0: Disconnected, 1: Normal, 2: Calibration Failed
            "ZMotorStatus": 0,  // Z-Axis Motor Connection Status, 0: Disconnected, 1: Connected
            "RotateMotorStatus": 0,  // Rotary Axis Motor Connection Status, 0: Disconnected, 1: Connected
            "RelaseFilmState": 0,  // Release Film Status, 0: Abnormal, 1: Normal
            "XMotorStatus": 0  // X-Axis Motor Connection Status, 0: Disconnected, 1: Connected
        },
        "ReleaseFilmMax": 0,  // Maximum number of uses (service life) for the release film
        "TempOfUVLEDMax": 0,  // Maximum operating temperature for UVLED(℃)
        "CameraStatus": 0,  // Camera Connection Status, 0: Disconnected, 1: Connected 
        "RemainingMemory": 123455,  // Remaining File Storage Space Size(bit)
        "TLPNoCapPos": 50.0,  // Model height threshold for not performing time-lapse photography (mm)
        "TLPStartCapPos": 30.0,  // The print height at which time-lapse photography begins (mm)
        "TLPInterLayers": 20  // Time-lapse photography shooting interval layers    
    }, 
    "MainboardID":"ffffffff",  // Motherboard ID
    "TimeStamp":1687069655,  // Timestamp
    "Topic":"sdcp/attributes/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

## Status Information

The motherboard guarantees to report machine status information to the client at the following circumstances:

1. When status information changes.
2. Upon receiving the control command to report status information.

The status message definition object content includes:

```json
{
    "Status": {
        "CurrentStatus":[0,1,2,3],  // Current Machine Status
        "PreviousStatus": 0,  // Previous Machine Status
        "PrintScreen": 0,  // Total Exposure Screen Usage Time(s)
        "ReleaseFilm": 0,  // Total Release Film Usage Count
        "TempOfUVLED": 0,  // Current UVLED Temperature（℃）
        "TimeLapseStatus": 0,  // Time-lapse Photography Switch Status. 0: Off, 1: On
        "TempOfBox": 0,  // Current Enclosure Temperature（℃）
        "TempTargetBox": 0,  // Target Enclosure Temperature（℃）
        "PrintInfo":{
            "Status": 0,  // Printing Sub-status
            "CurrentLayer": 100,  // Current Printing Layer
            "TotalLayer": 1000,  // Total Number of Print Layers
            "CurrentTicks": 65535,  // Current Print Time (ms)
            "TotalTicks": 65535,  // Estimated Total Print Time(ms)
            "Filename": "HitWork.ctb",  // Print File Name
            "ErrorNumber": 1, // Refer to the following text
            "TaskId": "xxx"  // Current Task ID
        }
    }, 
    "MainboardID": "ffffffff",  // Motherboard ID
    "TimeStamp": 1687069655 ,  // Timestamp
    "Topic": "sdcp/status/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Machine Status Description

The machine's reply includes two top-level statuses: PreviousStatus and CurrentStatus, as well as the Status sub-status under PrintInfo.

### Differences Between Top-Level Status Changes and Sub-Statuses

1. The machine's response includes two top-level statuses: PreviousStatus and CurrentStatus, as well as the Status sub-status under PrintInfo.

2. The sub-status will always retain the most recent status, for example, after the print is completed, the sub-status value will continue to be maintained as "Print Completed".

The definition of PreviousStatus and CurrentStatus:

```text
typedef enum
{
    SDCP_MACHINE_STATUS_IDLE = 0  // Idle
    SDCP_MACHINE_STATUS_PRINTING = 1  // Executing print task
    SDCP_MACHINE_STATUS_FILE_TRANSFERRING = 2  // File transfer in progress
    SDCP_MACHINE_STATUS_EXPOSURE_TESTING = 3  // Exposure test in progress
    SDCP_MACHINE_STATUS_DEVICES_TESTING = 4  //Device self-check in progress
} sdcp_machine_status_t;
```

### PrintInfo

#### Status

```text
typedef enum
{
SDCP_PRINT_STATUS_IDLE = 0  // Idle
SDCP_PRINT_STATUS_HOMING = 1  // Resetting
SDCP_PRINT_STATUS_DROPPING = 2  // Descending
SDCP_PRINT_STATUS_EXPOSURING = 3  // Exposing
SDCP_PRINT_STATUS_LIFTING = 4  // Lifting
SDCP_PRINT_STATUS_PAUSING = 5  // Executing Pause Action
SDCP_PRINT_STATUS_PAUSED = 6  // Suspended
SDCP_PRINT_STATUS_STOPPING = 7  // Executing Stop Action
SDCP_PRINT_STATUS_STOPED = 8  // Stopped
SDCP_PRINT_STATUS_COMPLETE = 9  // Print Completed
SDCP_PRINT_STATUS_FILE_CHECKING = 10 // File Checking in Progress
} sdcp_print_status_t;
```

#### ErrorNumber

```text
typedef enum
{
    SDCP_PRINT_ERROR_NONE = 0  // Normal
    SDCP_PRINT_ERROR_CHECK = 1  // File MD5 Check Failed
    SDCP_PRINT_ERROR_FILEIO = 2  // File Read Failed
    SDCP_PRINT_ERROR_INVLAID_RESOLUTION = 3  // Resolution Mismatch
    SDCP_PRINT_ERROR_UNKNOWN_FORMAT = 4  // Format Mismatch
    SDCP_PRINT_ERROR_UNKNOWN_MODEL = 5  // Machine Model Mismatch
} sdcp_print_error_t;
```

## Request Message

### Terminate File Transfer(Cmd: 255)

Request Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 255,  // Request Command.
       "Data": {
            "Uuid": "ffffffffffffffffffffffffffffffff",    // UUID for File Sending
            "FileName": "xxxxx"  // File Name
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 255,  // Request Command.
        "Data": {
            "Ack" : 0  // 0 represents success, other details are provided in the following text.
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Ack Code

```text
typedef enum
{
    SDCP_FILE_TRANSFER_ACK_SUCCESS = 0  // Success
    SDCP_FILE_TRANSFER_ACK_NOT_TRANSFER = 1  // The printer is not currently transferring files.
    SDCP_FILE_TRANSFER_ACK_CHECKING = 2  // The printer is already in the file verification phase.
    SDCP_FILE_TRANSFER_ACK_NOT_FOUND = 3  // File not found.
} sdcp_file_transfer_ack_t;
```

From

> The "From" field is used to identify the source, We will not repeat the explanation in the following text.
>

```text
typedef enum
{
    SDCP_FROM_PC = 0  // Local PC Software Local Area Network
    SDCP_FROM_WEB_PC = 1  // PC Software via WEB
    SDCP_FROM_WEB = 2  // Web Client
    SDCP_FROM_APP = 3  // APP
    SDCP_FROM_SERVER = 4  // Server
} sdcp_from_t;
```



### Request for status refresh message(Cmd:0)

Upon receiving this command, the motherboard will report the latest status to the status topic again.

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 0,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 0,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Request for attribute message(Cmd: 1)

Upon receiving this command, the motherboard will report the latest attributes to the attribute topic again.

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 1,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 1,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Start Printing(Cmd: 128)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 128,  // Request Command.
        "Data": {
            "Filename": "hitwork.ctb",  // File Name or File Path
            "StartLayer": 0  // Start Printing Layer Number
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 128,  // Request Command.
        "Data": {
            "Ack" : 0  // 0 represents success, other details are provided in the following text.
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Ack Code

```text
typedef enum
{
    SDCP_PRINT_CTRL_ACK_OK = 0  // OK
    SDCP_PRINT_CTRL_ACK_BUSY = 1  // Busy
    SDCP_PRINT_CTRL_ACK_NOT_FOUND = 2  // File Not Found
    SDCP_PRINT_CTRL_ACK_MD5_FAILED = 3  // MD5 Verification Failed
    SDCP_PRINT_CTRL_ACK_FILEIO_FAILED = 4  // File Read Failed
    SDCP_PRINT_CTRL_ACK_INVLAID_RESOLUTION = 5 // Resolution Mismatch
    SDCP_PRINT_CTRL_ACK_UNKNOW_FORMAT = 6  // Unrecognized File Format
    SDCP_PRINT_CTRL_ACK_UNKNOW_MODEL = 7  // Machine Model Mismatch
} sdcp_print_ctrl_ack_t;
```

### Pause Printing(Cmd: 129)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 129,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 129,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Stop Printing(Cmd: 130)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 130,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 130,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Continue Printing(Cmd: 131)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 131,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 131,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Stop Feeding Material(Cmd: 132)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 132,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 132,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Skip Preheating(Cmd: 133)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 133,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 133,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Change Printer Name(Cmd: 192)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 192,  // Request Command.
        "Data": {
            "Name": "newName"  // New Printer Name
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 192,  // Request Command.
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Retrieve File List(Cmd: 258)

Request Parameters

> "/usb/" represents USB storage space.
>
> "/local/" represents onboard storage space.
>
> If there is no "/", the default is "/local/".

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 258,  // Request Command.
        "Data": {
            "Url":"/usb/yourPath"
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

> The motherboard will automatically filter out files that cannot be printed.

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 192,  // Request Command.
        "Data": {
            "Ack" : 0,
            "FileList":[
                {
                    "name":"/usb/xxx",  // Indicates the current file or folder path.
                    "usedSize": 123456,  // Used Storage Space
                    "totalSize": 123456,  // Total Storage Space
                    "storageType":0,  // 0: Internal Storage, 1: External Storage

                    "type":0  // 0: Folder 1: File
                }
            ]   
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Batch Delete Files(Cmd: 259)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 259,  // Request Command.
        "Data": {
            "FileList": ["/usb/xx", "/usb/xx/xx"],  //List of Files to Be Deleted
            "FolderList": ["/usb/xx", "/usb/xx/xx"]  //List of Folders to Be Deleted
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 259,  // Request Command.
        "Data": {
            "Ack" : 0,   
            "ErrData":["/xxx/xxx"]  // Files that failed to be deleted; if there are no failures, no return is necessary
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Retrieve Historical Tasks(Cmd: 320)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 320,  // Request Command.
        "Data": {},
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 320,  // Request Command.
        "Data": {
            "Ack": 0,
            "HistoryData": ["xxxxxxxxxxx","xxxxxxxxxxx"]  // An ordered list of historical records, where the array elements are the taskid (UUID) of the historical records.
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Retrieve Task Details(Cmd: 321)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 321,  // Request Command.
        "Data": {
            "Id": ["xxxxxxxxxxx","xxxxxxxxxxx"]   // Task ID List
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 321,  // Request Command.
        "Data": {
            "Ack" : 0,   
            "HistoryDetailList":[
                {
                    "Thumbnail" : "xxx",  // Thumbnail Address
                    "TaskName" : "xxx",  // Task Name
                    "BeginTime" : 1689217424,  // Start Time (Timestamp in Seconds)
                    "EndTime" : 1689221024,  // End Time (Timestamp in Seconds)
                    "TaskStatus" : 1,  // Task Status (0: Other Status, 1: Completed, 2: Exceptional Status, 3: Stopped)
                    "SliceInformation" : {},  // Slice Information
                    "AlreadyPrintLayer" : 2,  // Printed Layer Count
                    "TaskId" : "",  // Task ID
                    "MD5" : "",  // MD5 of the Sliced File
                    "CurrentLayerTalVolume" : 0.02,  // Total Volume of Printed Layers(ml)
                    "TimeLapseVideoStatus": 0,  // Time-lapse photography status, 0: Not shot, 1: Time-lapse photography file exists, 2: Deleted, 3: Generating, 4: Generation failed.
                    "TimeLapseVideoUrl": "xxxx",  // URL for the time-lapse photography video
                    "ErrorStatusReason": 0  // Status Code, refer to the subsequent text.
                }
            ] // Task Details
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

ErrorStatusReason

```text
SDCP_PRINT_CAUSE_OK = 0  // Normal
SDCP_PRINT_CAUSE_TEMP_ERROR = 1  // Over-temperature
SDCP_PRINT_CAUSE_CALIBRATE_FAILED = 2  // Strain Gauge Calibration Failed
SDCP_PRINT_CAUSE_RESIN_LACK = 3  // Resin Level Low Detected
SDCP_PRINT_CAUSE_RESIN_OVER = 4  // The volume of resin required by the model exceeds the maximum capacity of the resin vat
SDCP_PRINT_CAUSE_PROBE_FAIL = 5  // No Resin Detected
SDCP_PRINT_CAUSE_FOREIGN_BODY = 6  // Foreign Object Detected
SDCP_PRINT_CAUSE_LEVEL_FAILED = 7  // Auto-leveling Failed
SDCP_PRINT_CAUSE_RELEASE_FAILED = 8  // Model Detachment Detected
SDCP_PRINT_CAUSE_SG_OFFLINE = 9  // Strain Gauge Not Connected
SDCP_PRINT_CAUSE_LCD_DET_FAILED = 10  // LCD Screen Connection Abnormal
SDCP_PRINT_CAUSE_RELEASE_OVERCOUNT = 11  // The cumulative release film usage has reached the maximum value

SDCP_PRINT_CAUSE_UDISK_REMOVE = 12  // USB drive detected as removed, printing has been stopped
SDCP_PRINT_CAUSE_HOME_FAILED_X = 13  // Detection of X-axis motor anomaly, printing has been stopped
SDCP_PRINT_CAUSE_HOME_FAILED_Z = 14  // Detection of Z-axis motor anomaly, printing has been stopped
SDCP_PRINT_CAUSE_RESIN_ABNORMAL_HIGH = 15  // The resin level has been detected to exceed the maximum value, and printing has been stopped
SDCP_PRINT_CAUSE_RESIN_ABNORMAL_LOW = 16  // Resin level detected as too low, printing has been stopped
SDCP_PRINT_CAUSE_HOME_FAILED = 17  // Home position calibration failed, please check if the motor or limit switch is functioning properly
SDCP_PRINT_CAUSE_PLAT_FAILED = 18  // A model is detected on the platform; please clean it and then restart printing
SDCP_PRINT_CAUSE_ERROR = 19  // Printing Exception
SDCP_PRINT_CAUSE_MOVE_ABNORMAL = 20  // Motor Movement Abnormality
SDCP_PRINT_CAUSE_AIC_MODEL_NONE = 21  // No model detected, please troubleshoot
SDCP_PRINT_CAUSE_AIC_MODEL_WARP = 22  // Warping of the model detected, please investigate
SDCP_PRINT_CAUSE_HOME_FAILED_Y = 23  // Deprecated
SDCP_PRINT_CAUSED_FILE_ERROR = 24  // Error File
SDCP_PRINT_CAUSED_CAMERA_ERROR = 25  // Camera Error. Please check if the camera is properly connected, or you can also disable this feature to continue printing
SDCP_PRINT_CAUSED_NETWORK_ERROR = 26  // Network Connection Error. Please check if your network connection is stable, or you can also disable this feature to continue printing
SDCP_PRINT_CAUSED_SERVER_CONNECT_FAILED = 27 // Server Connection Failed. Please contact our customer support, or you can also disable this feature to continue printing
SDCP_PRINT_CAUSED_DISCONNECT_APP = 28  // This printer is not bound to an app. To perform time-lapse photography, please first enable the remote control feature, or you can also disable this feature to continue printing
SDCP_PIRNT_CAUSED_CHECK_AUTO_RESIN_FEEDER = 29  // lease check the installation of the "automatic material extraction / feeding machine"
SDCP_PRINT_CAUSED_CONTAINER_RESIN_LOW = 30  // The resin in the container is running low. Add more resin to automatically close this notification, or click "Stop Auto Feeding" to continue printing
SDCP_PRINT_CAUSED_BOTTLE_DISCONNECT = 31  // Please ensure that the automatic material extraction/feeding machine is correctly installed and the data cable is connected
SDCP_PRINT_CAUSED_FEED_TIMEOUT = 32  // Automatic material extraction timeout, please check if the resin tube is blocked
SDCP_PRINT_CAUSE_TANK_TEMP_SENSOR_OFFLINE = 33  // Resin vat temperature sensor not connected
SDCP_PRINT_CAUSE_TANK_TEMP_SENSOR_ERRO = 34  // Resin vat temperature sensor indicates an over-temperature condition
```

### Enable/Disable Video Stream(Cmd: 386)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 386,  // Request Command.
        "Data": {
            "Enable": 0 // 0: Disable 1: Enable
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 386,  // Request Command.
        "Data":{
            "Ack": 0,  // 0: Success 1: Exceeded maximum simultaneous streaming limit 2: Camera does not exist 3: Unknown error
            "VideoUrl": "xxxx"  //When opening the video stream, return the RTSP protocol address
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### Enable/Disable Time-lapse Photography(Cmd: 387)

Request Parameters

```json
{
    "Id": "xxx", //Machine brand identifier, 32-bit UUID
    "Data":{
        "Cmd": 387,  // Request Command.
        "Data": {
            "Enable" : 0  // 0: Disable 1: Enable
        },
        "RequestID": "000000000001d354",  // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655,  // Timestamp
        "From": 0  // Identify the source of the command.
    },
    "Topic": "sdcp/request/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

Response Parameters

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Cmd": 387,  // Request Command.
        "Data":{
            "Ack": 0,  // 0: Success 1: Unknown Error
        },
        "RequestID": "000000000001d354", // Request ID
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/response/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

## Error Message

When there is an error message on the motherboard, it will be proactively reported.

```json
{
    "Id": "xxx", // Machine brand identifier, 32-bit UUID
    "Data": {
        "Data": {
            "ErrorCode": "xxxxxx"  // Error Code, please refer to the error code definition.
        },               
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/error/${MainboardID}"  // Topic, used to distinguish the type of reported message
}
```

### ErrorCode

```text
typedef enum
{
    SDCP_ERROR_CODE_MD5_FAILED = 1,  // File Transfer MD5 Check Failed
    SDCP_ERROR_CODE_FORMAT_FAILED = 2,  // File format is incorrect
} sdcp_normal_error_t;
```

## Notification Message

When there is a need to proactively report information, the motherboard will report it actively

```json
{
    "Id": "xxx", //  Machine brand identifier, 32-bit UUID
    "Data":{
        "Data": {
            "Message": "xxxxxx",  // Can be a string, can be JSON
            "Type":1  // Used to distinguish what type of notification it is, 1: History synchronization successful
         },               
        "MainboardID": "ffffffff",  // Motherboard ID
        "TimeStamp":1687069655  // Timestamp
    },
    "Topic": "sdcp/notice/${MainboardID}"  // Topic, used to distinguish the types of reported messages
}
```

## Mainboard HTTP Server Interface Definition

The HTTP Server is a dedicated service for file transfer, with the mainboard acting as the HTTP Server.

### Send File Interface

File sending is carried out using a packet-by-packet method, with each packet being 1MB

#### Request Address

```text
http://${MainboardIP}:3030/uploadFile/upload
```

#### Request Method

```text
POST（Content-Type: multipart/form-data）
```

#### Request Parameters

```text
S-File-MD5: ffffffffffffffffffffffffffffffff  //The MD5 generated for the file is used to verify the correctness of the file.
Check: '1'  // Whether to enable file verification 0:disable 1:enable
Offset：0  // Offset
Uuid： xxxxxx  // The UUID for each packet is the same
TotalSize：123  //Total Size
File: (binary)
```

#### Response Parameters

##### Success

```json
{
    "code": "000000",
    "messages": null,
    "data": {},
    "success": true
}
```

##### Failure

```json
{
    "code": "111111",
    "messages": [
        {
            "field": "common_field",
            "message": 100001  // When the field name is set to "common_field", the value of the message is the error code.
        },
        {
            "field": "filename",
            "message": "Cannot be empty"  // Field Validation Failure Reasons
        }
    ],
    "data": null,
    "success": false
}
```

##### 错误码

| Error Code | Failure Reason | Description |
|:------|:-------:|------:|
| -1 | offset error | Illegal file offset value (less than 0) |
| -2 | offset not match | File offset does not match the current file |
| -3 | file open failed | File cannot be opened |
| -4 | unknow error | Other Unknown Errors |
