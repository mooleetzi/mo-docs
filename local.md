# 支持load local

Developer: 李常亮

Date: 12/30/2022

Release: 0.7.0

Issues: https://github.com/matrixorigin/matrixone/issues/6665

## Introduction

目前mo尚不支持将client端的数据直接导入，需要先将数据上传到server或云，然后再通过load命令导入数据。因此，需要支持load
local，使用户可以直接从client端导入数据。

## Design

1. mo前端接收到load local命令后，生成两个channel保存到**proc**，分别用于接收发送localInfileRequest的信号和接收client端发送的文件数据。
2. mo前端监听localInfileRequest信号，当收到信号后，向client端发送localInfileRequest请求
4. 执行load local时，在prepare阶段给channel1发送localInfileRequest信号，前端监听到信号后向client端发送localInfileRequest请求
5. client端收到localInfileRequest请求后，向server端发送文件数据
6. mo前端收到文件数据后，将数据写入channel2
7. 在load call阶段，从channel2中读取数据，开始正常load流程

## Pseudo Code

```go
//pkg/vm/process/types.go
type Process struct{
	//...
    LocalInfileCh1 chan struct{}
    LocalInfileCh2 chan []byte
	
}

//mysql_cmd_executor.go doComQuery
switch st := stmt.(type) {
case *tree.Local
    if st.Local{
        // 创建两个channel
        // 1. 用于接收发送localInfileRequest的信号
        // 2. 用于接收client端发送的文件数据
        go processLocalInfileRequest(proc, protocol) // 监听channel1的localInfileRequest信号，从protocol读取client发来的文件数据并写入channel2
    }
}


func processLocalInfileRequest(proc *process.Process, protocol MysqlProtocol) {
    <-proc.LocalInfileCh1
	protocol.SendLocalInfileRequest()
	for { 
		// 从protocol读取client发来的文件数据并写入channel2 
		//读取到EOF后，停止读取，退出循环
		// pkt长度等于maxPayloadSize时，继续读取
    }
}
```