---
title: win搭建protobuf python开发环境
date: '2017-12-16'
description: win环境搭建protobuf环境使用python开发
categories: 
- python

tags: 

- python
- protobuf

---

## 搭建环境

	python 2.7.9
	win10
	protobuf 3.5.0
	pycharm

## protobuf编译工具protoc安装

下载[ protoc-3.5.0-win32.zip](https://github.com/google/protobuf/releases/tag/v3.5.0).

解压缩到指定目录，配置环境变量PATH。

打开命令行窗口输入：protoc --version 测试。

## protobuf python安装

```shell
pip install protobuf
```

测试：

```python
python
>> import google.protobuf
```

## demo测试

1. 创建test.proto描述文件

```java
message CDevice
{
   optional int32 devId = 1;
   optional string name = 2;
}
```

2. 使用protoc编译test.proto为test_pb2.py文件

```shell
protoc -I ./origin --python_out=./test ./origin/test.proto
```

3. 编写测试python测试编译生成的py

```python
# -*- coding: UTF-8 -*-
import test_pb2
import traceback
import sys


try:
    sendData = test_pb2.CDevice()
    sendData.devId = 9
    sendData.name = 'USB'

    #Serialize
    sendDataStr = sendData.SerializeToString()
    #print serialized string value
    print 'serialized string:', sendDataStr
    #------------------------#
    #  message transmission  #
    #------------------------#
    receiveDataStr = sendDataStr
    receiveData = test_pb2.CDevice()

    #Deserialize
    receiveData.ParseFromString(receiveDataStr)
    print 'pares serialize string, return: devId = ', receiveData.devId, ', name = ', receiveData.name
except Exception, e:
    print Exception, ':', e
    print traceback.print_exc()
    errInfo = sys.exc_info()
    print errInfo[0], ':', errInfo[1]
```
