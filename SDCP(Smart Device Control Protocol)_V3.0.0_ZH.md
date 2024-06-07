# SDCP(Smart Device Control Protocol)文档V3.0.0

深圳市创必得科技有限公司

## 概述

SDCP 协议是客户端与主板交互的应用层协议，包含命令控制、文件传输、状态监测等内容。

协议通讯均采用JSON格式进行交互，其中视频流数据采用RTSP协议进行实时通讯。具体命令参照下文的定义。

## 发现设备说明

主板端开机后主动在 3030 端口启动WebSocket服务；客户端在局域网广播网段使用UDP协议在 3000 端口广播M99999字符串。主板端监听到该命令后会回复以下内容。

```json
{
    "Id": "xxx",  // 机器品牌标识， 32 位UUID
    "Data": {
        "Name": "PrinterName",  // 机器名称
        "MachineName": "MachineModel",  // 机型名称
        "BrandName": "CBD",  // 品牌名称
        "MainboardIP": "192.168.1.2",  // 主板IP地址
        "MainboardID": "000000000001d354",  // 主板ID(16bit)
        "ProtocolVersion": "V3.0.0",  // 协议版本
        "FirmwareVersion": "V1.0.0"   // 固件版本
    }
}
```

## 连接设备说明

客户端连接主板WebSocket地址：ws://${MainboardIP}:3030/websocket

## 消息主题

Topic说明：（${MainboardID}是指主板ID）

```text
SDCP控制请求(客户端->主板): sdcp/request/${MainboardID}

SDCP控制响应(主板->客户端): sdcp/response/${MainboardID}

SDCP状态信息(主板->客户端)：sdcp/status/${MainboardID}

SDCP属性信息(主板->客户端)：sdcp/attributes/${MainboardID}

SDCP错误信息(主板->客户端)：sdcp/error/${MainboardID}

SDCP通知信息(主板->客户端)：sdcp/notice/${MainboardID}
```

## 心跳

请求格式定义

```text
"ping"
```

响应格式定义

```text
"pong"
```

## 属性信息

主板端以下时刻保证会向客户端上报机器属性信息：

1. 属性信息发生变更

2. 接收到上报属性信息控制命令

属性消息定义对象内容：

```json
{
    "Attributes": {
        "Name": "PrinterName",  // 机器名称
        "MachineName": "MachineModel",  // 机型名称
        "BrandName": "CBD",  // 品牌名称
        "ProtocolVersion": "V3.0.0",  // 协议版本
        "FirmwareVersion": "V1.0.0",  // 固件版本
        "Resolution": "7680x4320",  // 设备分辨率
        "XYZsize": "210x140x100" ,  // 设备XYZ方向的成型尺寸，单位为mm
        "MainboardIP": "192.168.1.1",  // 主板IP地址
        "MainboardID": "000000000001d354", // 主板ID(16)
        "NumberOfVideoStreamConnected": 1,  // 已连接视频流
        "MaximumVideoStreamAllowed": 1,  // 最多可连接的视频流
        "NetworkStatus": "'wlan' | 'eth'",  // 网络连接状态，wifi/网口
        "UsbDiskStatus": 0,  // U盘接入状态 0：未接入，1：已接入
        "Capabilities":[                            
            "FILE_TRANSFER",  // 支持文件传输
            "PRINT_CONTROL",  // 支持打印控制
            "VIDEO_STREAM"  // 支持视频流传输
        ],  // 主板端支持的子协议
        "SupportFileType":[
            "CTB"  // 支持CTB文件类型
        ],
        //设备自检状态
        "DevicesStatus":{
            "TempSensorStatusOfUVLED": 0, // UVLED温度传感器状态,0未接入，1正常，2异常
            "LCDStatus": 0,  // 曝光屏连接状态，0断开，1连接
            "SgStatus": 0,  // 应变片状态，0未接入，1正常 2校准失败
            "ZMotorStatus": 0,  // Z轴电机连接状态，0断开，1连接
            "RotateMotorStatus": 0,  // 旋转轴电机连接状态，0断开，1连接
            "RelaseFilmState": 0,  // 离型膜状态，0异常，1正常
            "XMotorStatus": 0  // X轴电机连接状态，0断开，1连接
        },
        "ReleaseFilmMax": 0,  // 离型膜最大次数（寿命）
        "TempOfUVLEDMax": 0,  // UVLED最大工作温度(℃)
        "CameraStatus": 0,  // 摄像头接入状态 0断开， 1连接 
        "RemainingMemory": 123455,  // 剩余的文件存储空间大小(bit)
        "TLPNoCapPos": 50.0,  // 不进行延时摄影拍摄的模型高度阈值(mm)
        "TLPStartCapPos": 30.0,  // 开始进行延时摄影拍摄的模型高度(mm)
        "TLPInterLayers": 20  // 延时摄影拍摄间隔层数 
    }, 
    "MainboardID":"ffffffff",  // 主板ID
    "TimeStamp":1687069655,  // 时间戳
    "Topic":"sdcp/attributes/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

## 状态信息

主板端以下时刻保证会向客户端上报机器状态信息

1. 状态信息发生变更

2. 接收到上报状态信息控制命令

状态消息定义对象内容

```json
{
    "Status": {
        "CurrentStatus":[0,1,2,3],  // 当前机器状态，部分状态可以共存。
        "PreviousStatus": 0,  // 上一次机器状态
        "PrintScreen": 0,  // 曝光屏使用时间（s）
        "ReleaseFilm": 0,  // 离型膜次数
        "TempOfUVLED": 0,  // 当前UVLED温度（℃）
        "TimeLapseStatus": 0,  // 延时摄影开关状态 0 关闭， 1 打开
        "TempOfBox": 0,  // 箱体当前温度（℃）
        "TempTargetBox": 0,  // 箱体目标温度（℃）
        "PrintInfo":{
            "Status": 0,  // 打印子状态
            "CurrentLayer": 100,  // 当前打印层数
            "TotalLayer": 1000,  // 打印任务总层数
            "CurrentTicks": 65535,  // 当前已打印时间(ms)
            "TotalTicks": 65535,  // 总打印时间(ms)
            "Filename": "HitWork.ctb",  // 打印文件名称
            "ErrorNumber": 1, // 错误码，参考后文
            "TaskId": "xxx"  // 当前任务ID
        }
    }, 
    "MainboardID": "ffffffff",  // 主板ID
    "TimeStamp": 1687069655 ,  // 时间戳
    "Topic": "sdcp/status/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 机器状态说明

机器回复的状态包括PreviousStatus与CurrentStatus两个顶层的状态，以及PrintInfo下的Status子状态

### 顶层状态变化与子状态的区别

1. 顶层状态是实时更新的，例如当打印完成后CurrentStatus会立即上报为空闲；PreviousStatus会变更为上一次的顶层状态

2. 子状态会一直保留最近的状态，例如打印完成后子状态的值会一直保持为打印已完成

PreviousStatus与CurrentStatus定义：

```text
typedef enum
{
    SDCP_MACHINE_STATUS_IDLE = 0  // 空闲
    SDCP_MACHINE_STATUS_PRINTING = 1  // 执行打印任务中
    SDCP_MACHINE_STATUS_FILE_TRANSFERRING = 2  // 文件传输中
    SDCP_MACHINE_STATUS_EXPOSURE_TESTING = 3  // 曝光测试
    SDCP_MACHINE_STATUS_DEVICES_TESTING = 4  //设备自检
} sdcp_machine_status_t;
```

### PrintInfo

#### Status

```text
typedef enum
{
    SDCP_PRINT_STATUS_IDLE = 0  // 空闲
    SDCP_PRINT_STATUS_HOMING = 1  // 归零中
    SDCP_PRINT_STATUS_DROPPING = 2  // 下降中
    SDCP_PRINT_STATUS_EXPOSURING = 3  // 曝光中
    SDCP_PRINT_STATUS_LIFTING = 4  // 抬升中
    SDCP_PRINT_STATUS_PAUSING = 5  // 正在执行暂停动作中
    SDCP_PRINT_STATUS_PAUSED = 6  // 已暂停
    SDCP_PRINT_STATUS_STOPPING = 7  // 正在执行停止动作中
    SDCP_PRINT_STATUS_STOPED = 8  // 已停止
    SDCP_PRINT_STATUS_COMPLETE = 9  // 打印完成
    SDCP_PRINT_STATUS_FILE_CHECKING = 10 // 文件检测中
} sdcp_print_status_t;
```

#### ErrorNumber

```text
typedef enum
{
    SDCP_PRINT_ERROR_NONE = 0  // 正常
    SDCP_PRINT_ERROR_CHECK = 1  // 文件MD5校验失败
    SDCP_PRINT_ERROR_FILEIO = 2  // 文件读取失败
    SDCP_PRINT_ERROR_INVLAID_RESOLUTION = 3  // 非法分辨率
    SDCP_PRINT_ERROR_UNKNOWN_FORMAT = 4  // 格式不匹配
    SDCP_PRINT_ERROR_UNKNOWN_MODEL = 5  // 机型不匹配
} sdcp_print_error_t;
```

## 请求消息

### 终止传输文件(Cmd: 255)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 255,  // 请求命令
       "Data": {
            "Uuid": "ffffffffffffffffffffffffffffffff",    // 发送文件时的uuid
            "FileName": "xxxxx"  // 文件名
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 255,  // 请求命令
        "Data": {
            "Ack" : 0  // 0代表成功，其他详见下文
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

Ack码

```text
typedef enum
{
    SDCP_FILE_TRANSFER_ACK_SUCCESS = 0  // 成功
    SDCP_FILE_TRANSFER_ACK_NOT_TRANSFER = 1  // 设备未在进行文件传输
    SDCP_FILE_TRANSFER_ACK_CHECKING = 2  // 设备已处于校验文件阶段
    SDCP_FILE_TRANSFER_ACK_NOT_FOUND = 3  // 文件找不到
} sdcp_file_transfer_ack_t;
```

From

> From字段用于标识来源，后文将不再重复说明。
>

```text
typedef enum
{
    SDCP_FROM_PC = 0  //本地PC软件局域网 
    SDCP_FROM_WEB_PC = 1  //PC软件通过WEB
    SDCP_FROM_WEB = 2  //网页端
    SDCP_FROM_APP = 3  //APP
    SDCP_FROM_SERVER = 4  //服务端
} sdcp_from_t;
```



### 请求状态刷新消息(Cmd:0)

主板接收到该命令后重新向状态主题上报最新状态

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 0,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 0,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 请求属性消息(Cmd: 1)

主板接收到该命令后重新向属性主题上报最新属性

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 1,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 1,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 开始打印(Cmd: 128)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 128,  // 请求命令
        "Data": {
            "Filename": "hitwork.ctb",  // 文件名或者文件路径
            "StartLayer": 0  // 从开始打印层数
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 128,  // 请求命令
        "Data": {
            "Ack" : 0  // 0代表成功，其他详见下文
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

Ack码

```text
typedef enum
{
    SDCP_PRINT_CTRL_ACK_OK = 0  // OK
    SDCP_PRINT_CTRL_ACK_BUSY = 1  // 设备忙
    SDCP_PRINT_CTRL_ACK_NOT_FOUND = 2  // 未找到目标文件
    SDCP_PRINT_CTRL_ACK_MD5_FAILED = 3  // MD5校验失败
    SDCP_PRINT_CTRL_ACK_FILEIO_FAILED = 4  // 文件读取失败
    SDCP_PRINT_CTRL_ACK_INVLAID_RESOLUTION = 5 // 文件分辨率不匹配
    SDCP_PRINT_CTRL_ACK_UNKNOW_FORMAT = 5  // 无法识别的文件格式
    SDCP_PRINT_CTRL_ACK_UNKNOW_MODEL = 6  // 文件机型不匹配
} sdcp_print_ctrl_ack_t;
```

### 暂停打印(Cmd: 129)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 129,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 129,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 停止打印(Cmd: 130)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 130,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 130,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 继续打印(Cmd: 131)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 131,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 131,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 停止进料(Cmd: 132)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 132,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 132,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 跳过预热(Cmd: 133)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 133,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 133,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 修改打印机名字(Cmd: 192)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 192,  // 请求命令
        "Data": {
            "Name": "newName"  // 打印机新名称
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 192,  // 请求命令
        "Data": {
            "Ack" : 0
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 获取文件列表(Cmd: 258)

请求参数

> "/usb/"代表USB存储空间
>
> "/local/"代表板载存储空间
>
> 如果没有"/"，默认是“/local/”

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 258,  // 请求命令
        "Data": {
            "Url":"/usb/yourPath"
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

> 主板会默认会过滤掉不能打印的文件

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 192,  // 请求命令
        "Data": {
            "Ack" : 0,
            "FileList":[
                {
                    "name":"/usb/xxx",  // 表示当前文件或者文件夹的路径；
                    "usedSize": 123456,  // 已使用的存储空间
                    "totalSize": 123456,  // 总存储空间
                    "storageType":0,  // 0:内部存储，1：外部存储
                    "type":0  // 0:文件夹 1：文件
                }
            ]   
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 批量删除文件(Cmd: 259)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 259,  // 请求命令
        "Data": {
            "FileList": ["/usb/xx", "/usb/xx/xx"],  //需要删除的文件列表
            "FolderList": ["/usb/xx", "/usb/xx/xx"]  //需要删除的文件夹列表
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 259,  // 请求命令
        "Data": {
            "Ack" : 0,   
            "ErrData":["/xxx/xxx"]  //删除失败的文件，如果没有失败的，可以不用返回
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 获取历史任务(Cmd: 320)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 320,  // 请求命令
        "Data": {},
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 320,  // 请求命令
        "Data": {
            "Ack": 0,
            "HistoryData": ["xxxxxxxxxxx","xxxxxxxxxxx"]  // 历史记录的Taskid列表。
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 获取任务详情(Cmd: 321)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 321,  // 请求命令
        "Data": {
            "Id": ["xxxxxxxxxxx","xxxxxxxxxxx"]   // TaskID列表
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 321,  // 请求命令
        "Data": {
            "Ack" : 0,   
            "HistoryDetailList":[
                {
                    "Thumbnail" : "xxx",  // 缩略图地址
                    "TaskName" : "xxx",  // 任务名称
                    "BeginTime" : 1689217424,  // 开始时间(时间戳/秒)
                    "EndTime" : 1689221024,  // 结束时间(时间戳/秒)
                    "TaskStatus" : 1,  // 任务状态(0：其他状态，1:完成, 2:异常状态，3：停止)
                    "SliceInformation" : {},  // 切片信息
                    "AlreadyPrintLayer" : 2,  // 已打印层数
                    "TaskId" : "",  // 任务ID
                    "MD5" : "",  // 切片文件的MD5
                    "CurrentLayerTalVolume" : 0.02,  // 已打印层数总体积(ml)
                    "TimeLapseVideoStatus": 0,  // 延时摄影状态 0:未拍摄延时摄影文件 1:存在延时摄影文件 2:延时摄影文件已删除 3:延时摄影生成中  4:延时摄影生成失败
                    "TimeLapseVideoUrl": "xxxx",  // 延时摄影视频的URL
                    "ErrorStatusReason": 0  // 状态码，参照下文
                }
            ] // 历史记录的详情
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

ErrorStatusReason

```text
SDCP_PRINT_CAUSE_OK = 0  // 正常
SDCP_PRINT_CAUSE_TEMP_ERROR = 1  // 温度过高
SDCP_PRINT_CAUSE_CALIBRATE_FAILED = 2  // 力学传感器校准失败
SDCP_PRINT_CAUSE_RESIN_LACK = 3  // 检测到树脂不足，请及时补充
SDCP_PRINT_CAUSE_RESIN_OVER = 4  // 模型所需树脂体积已超过料槽最大容积，请注意中途添加树脂
SDCP_PRINT_CAUSE_PROBE_FAIL = 5  // 未检测到树脂
SDCP_PRINT_CAUSE_FOREIGN_BODY = 6  // 检测到有异物，请排查
SDCP_PRINT_CAUSE_LEVEL_FAILED = 7  // 自动调平失败，请排查
SDCP_PRINT_CAUSE_RELEASE_FAILED = 8  // 检测到模型脱落，请确认是否继续打印
SDCP_PRINT_CAUSE_SG_OFFLINE = 9  // 力学传感器未接入
SDCP_PRINT_CAUSE_LCD_DET_FAILED = 10  // 检测LCD屏幕连接异常，请排查
SDCP_PRINT_CAUSE_RELEASE_OVERCOUNT = 11  // 累计离型次数达到最大值,请及时更换

SDCP_PRINT_CAUSE_UDISK_REMOVE = 12  // 检测到U盘拔出，已停止打印
SDCP_PRINT_CAUSE_HOME_FAILED_X = 13  // 检测X轴电机异常，已停止打印
SDCP_PRINT_CAUSE_HOME_FAILED_Z = 14  // 检测Z轴电机异常，已停止打印
SDCP_PRINT_CAUSE_RESIN_ABNORMAL_HIGH = 15  // 检测到树脂已超出最大值，已停止打印
SDCP_PRINT_CAUSE_RESIN_ABNORMAL_LOW = 16  // 检测到树脂过少，已停止打印
SDCP_PRINT_CAUSE_HOME_FAILED = 17  // 归零失败,请检查电机或限位开关是否正常
SDCP_PRINT_CAUSE_PLAT_FAILED = 18  // 检测到平台有模型附着,请清理后重新打印
SDCP_PRINT_CAUSE_ERROR = 19  // 打印异常
SDCP_PRINT_CAUSE_MOVE_ABNORMAL = 20  // 电机运动异常
SDCP_PRINT_CAUSE_AIC_MODEL_NONE = 21  // 未探测到模型，请排查
SDCP_PRINT_CAUSE_AIC_MODEL_WARP = 22  // 检测到模型翘边，请排查
SDCP_PRINT_CAUSE_HOME_FAILED_Y = 23  // 已弃用
SDCP_PRINT_CAUSED_FILE_ERROR = 24  // 错误文件
SDCP_PRINT_CAUSED_CAMERA_ERROR = 25  // 摄像头错误。请检查摄像头是否正确连接，或者您也可以禁用此功能以继续打印
SDCP_PRINT_CAUSED_NETWORK_ERROR = 26  // 网路连接错误。请检查网路连接是否稳定，或者您也可以禁用此功能以继续打印
SDCP_PRINT_CAUSED_SERVER_CONNECT_FAILED = 27 // 伺服器连接失败。请联络我们的客户支援，或者您也可以禁用此功能以继续打印
SDCP_PRINT_CAUSED_DISCONNECT_APP = 28  // 此打印机未绑定app，如要进行延时摄影，请先开启远程控制功能，或者您也可以禁用此功能以继续打印
SDCP_PIRNT_CAUSED_CHECK_AUTO_RESIN_FEEDER = 29  // 请检查「自动抽料 / 注料机」的安装
SDCP_PRINT_CAUSED_CONTAINER_RESIN_LOW = 30  // 容器中的树脂已经不足。添加更多树脂以自动关闭此通知，或点击「停止自动注料」以继续打印
SDCP_PRINT_CAUSED_BOTTLE_DISCONNECT = 31  // 请确保自动抽注料机已正确安装并连接数据线
SDCP_PRINT_CAUSED_FEED_TIMEOUT = 32  // 自动抽料超时， 请检查树脂管是否阻塞
SDCP_PRINT_CAUSE_TANK_TEMP_SENSOR_OFFLINE = 33  // 料槽温度传感器未接入
SDCP_PRINT_CAUSE_TANK_TEMP_SENSOR_ERRO = 34  // 料槽温度传感器温度过高
```

### 打开/关闭视频流(Cmd: 386)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 386,  // 请求命令
        "Data": {
            "Enable": 0 // 0：关闭 1：开启
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 386,  // 请求命令
        "Data":{
            "Ack": 0,  //0: 成功; 1: 超过最大同时拉流限制; 2: 摄像头不存在; 3: 未知错误
            "VideoUrl": "xxxx"  //打开视频流的时候返回RTSP协议地址
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### 打开/关闭延时摄影(Cmd: 387)

请求参数

```json
{
    "Id": "xxx", //机器品牌标识，32位UUID
    "Data":{
        "Cmd": 387,  // 请求命令
        "Data": {
            "Enable" : 0  // 0: 关闭; 1: 开启
        },
        "RequestID": "000000000001d354",  // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655,  // 时间戳
        "From": 0  // 标识命令来源
    },
    "Topic": "sdcp/request/${MainboardID}"  // 消息类型
}
```

响应参数

```json
{
    "Id": "xxx", // 机器品牌标识，32位UUID
    "Data": {
        "Cmd": 387,  // 请求命令
        "Data":{
            "Ack": 0  // 0: 成功; 1: 未知错误
        },
        "RequestID": "000000000001d354", // 请求ID
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/response/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

## 错误消息

主板端有错误信息时会主动上报

```json
{
    "Id": "xxx", //用于机型隔离
    "Data": {
        "Data": {
            "ErrorCode": "xxxxxx"  // 错误码, 请查询错误码定义
        },               
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/error/${MainboardID}"  // 主题，用于区分上报消息的类型
}
```

### ErrorCode

```text
typedef enum
{
    SDCP_ERROR_CODE_MD5_FAILED = 1,  // 文件传输MD5校验失败
    SDCP_ERROR_CODE_FORMAT_FAILED = 2,  // 文件传输格式不对
} sdcp_normal_error_t;
```

## 通知消息

有需要主动上报信息时，主板主动上报

```json
{
    "Id": "xxx", // 用于机型隔离
    "Data":{
        "Data": {
            "Message": "xxxxxx",  // 可以是字符串，可以是JSON.
            "Type":1  // 用于区分是什么类型的通知, 1:历史记录同步成功
         },               
        "MainboardID": "ffffffff",  // 主板ID
        "TimeStamp":1687069655  // 时间戳
    },
    "Topic": "sdcp/notice/${MainboardID}"  // 主题，用于区分上报的信息是属于什么类型
}
```

## 主板HTTP Server接口定义

Http Server是文件传输的专用服务，主板端作为Http Server。

### 发送文件接口

文件发送采用分包方式进行发送，最大每包1Mb

#### 请求地址

```text
http://${MainboardIP}:3030/uploadFile/upload
```

#### 请求方式

```text
POST（Content-Type: multipart/form-data）
```

#### 请求参数

```text
S-File-MD5: ffffffffffffffffffffffffffffffff  //文件生成的MD5，用于检验文件是否正确
Check: '1'  // 文件是否开启校验 “0”不开启 “1”开启
Offset：0  // 偏移量
Uuid： xxxxxx  // uuid,每包的uuid都是一样的
TotalSize：123  //总大小
File: (binary)
```

#### 响应参数

##### 成功

```json
{
    "code": "000000",
    "messages": null,
    "data": {},
    "success": true
}
```

##### 失败

```json
{
    "code": "111111",
    "messages": [
        {
            "field": "common_field",
            "message": 100001  // 字段名 field=common_field时，message的值是错误码
        },
        {
            "field": "filename",
            "message": "不能为空"  // 字段的校验失败原因
        }
    ],
    "data": null,
    "success": false
}
```

##### 错误码

| 错误码 | 失败原因 | 描述 |
|:------|:-------:|------:|
| -1 | offset error | 文件偏移值非法(小于0) |
| -2 | offset not match | 文件偏移与当前文件不匹配 |
| -3 | file open failed | 文件无法打开 |
| -4 | unknow error | 其他未知错误 |
