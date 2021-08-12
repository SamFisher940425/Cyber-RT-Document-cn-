# Python API 教程

## 1.背景

Cyber RT的核心功能是使用C++开发的，我们也提供了Python接口来帮助开发者构建自己特定功能的工程。

## 2.Cyber RT Python接口

Cyber RT Python接口封装了对应的C++接口，其实现并未依赖第三方工具，如swig等，这使得其更容易维护。

## 3.Cyber RT中的Python接口概览

目前，Python接口包括：

* 通道信息的访问
* 服务器（Server）/客户端（Client）的通信
* 数据记录文件（Record）中信息的查询
* 从数据记录文件中读取，或写入数据记录文件
* 时间（Time）/持续时间（Duration）/速率（Rate）相关操作
* 计时器（Timer）

### 3.1通道的读写

步骤如下：

1. 首先建立一个节点（Node）
2. 创建对应的读取者或写入者
3. 如果写入到通道，调用写入者的写接口
4. 如果是从通道读取，在节点中调用回传（Spin）接口，并在回调函数中处理消息

接口如下所示：

```python
class Node:
    """
    Class for cyber Node wrapper.
    """

    def create_writer(self, name, data_type, qos_depth=1):
        """
        create a topic writer for send message to topic.
        @param self
        @param name str: topic name
        @param data_type proto: message class for serialization
        """

   def create_reader(self, name, data_type, callback, args=None):
        """
        create a topic reader for receive message from topic.
        @param self
        @param name str: topic name
        @param data_type proto: message class for serialization
        @callback fn: function to call (fn(data)) when data is
                   received. If args is set, the function must
                   accept the args as a second argument,
                   i.e. fn(data, args)
        @args any: additional arguments to pass to the callback
        """
    def create_client(self, name, request_data_type, response_data_type):
    """
    """
    def create_service(self, name, req_data_type, res_data_type, callback, args=None):

    def spin(self):
        """
        spin in wait and process message.
        @param self
        """

class Writer(object):
    """
    Class for cyber writer wrapper.
    """
    def write(self, data):
        """
        writer msg string
        """
```

### 3.2数据记录接口

从数据记录文件（Record）读取：

1. 创建一个数据记录读取者（RecordReader）
2. 从数据记录文件中读取消息

写入到数据记录文件（Record）：

1. 创建一个数据记录写入者（RecordWriter）
2. 写入消息到数据记录文件

接口如下：

```python
class RecordReader(object):
    """
    Class for cyber RecordReader wrapper.
    """
    def read_messages(self, start_time=0, end_time=18446744073709551615):
        """
        read message from bag file.
        @param self
        @param start_time:
        @param end_time:
        @return: generator of (message, data_type, timestamp)
        """
    def get_messagenumber(self, channel_name):
        """
        return message count.
        """
    def get_messagetype(self, channel_name):
        """
        return message type.
        """
    def get_protodesc(self, channel_name):
        """
        return message protodesc.
        """
    def get_headerstring(self):
        """
        return message header string.
        """
    def reset(self):
        """
        return reset.
        ""
        return _CYBER_RECORD.PyRecordReader_Reset(self.record_reader)

    def get_channellist(self):
        """
        return channel list.
        """
        return _CYBER_RECORD.PyRecordReader_GetChannelList(self.record_reader)


class RecordWriter(object):
    """
    Class for cyber RecordWriter wrapper.
    """
    def open(self, path):
        """
        open record file for write.
        """
    def write_channel(self, channel_name, type_name, proto_desc):
        """
        writer channel by channelname,typename,protodesc
        """
    def write_message(self, channel_name, data, time, raw = True):
        """
        writer msg:channelname,data,time,is data raw
        """

    def set_size_fileseg(self, size_kilobytes):
        """
        return filesegment size.
        """

    def set_intervaltime_fileseg(self, time_sec):
        """
        return file interval time.
        """

    def get_messagenumber(self, channel_name):
        """
        return message count.
        """

    def get_messagetype(self, channel_name):
        """
        return message type.
        """

    def get_protodesc(self, channel_name):
        """
        return message protodesc.
        """
```

### 3.3时间（Time）接口

```python
class Time(object):
    @staticmethod
    def now():
        time_now = Time(_CYBER_TIME.PyTime_now())
        return time_now

    @staticmethod
    def mono_time():
        mono_time = Time(_CYBER_TIME.PyTime_mono_time())
        return mono_time

    def to_sec(self):
        return _CYBER_TIME.PyTime_to_sec(self.time)

    def to_nsec(self):
        return _CYBER_TIME.PyTime_to_nsec(self.time)

    def sleep_until(self, nanoseconds):
        return _CYBER_TIME.PyTime_sleep_until(self.time, nanoseconds)
```

### 3.4计时器（Timer）接口

```python
class Timer(object):

    def set_option(self, period, callback, oneshot=0):
        """
        set the option of timer.
        @param period The period of the timer, unit is ms.
        @param callback The tasks that the timer needs to perform.
        @param oneshot 1:perform the callback only after the first timing cycle
        0:perform the callback every timed period
        """

    def start(self):

    def stop(self):
```

## 4.样例

### 4.1从通道中读取（cyber/python/examples/listener.py）

```python
import sys
sys.path.append("../")
from cyber_py import cyber
from modules.common.util.testdata.simple_pb2 import SimpleMessage

def callback(data):

    """
    reader message callback.
    """
    print "="*80
    print "py:reader callback msg->:"
    print data
    print "="*80

def test_listener_class():
    """
    reader message.
    """
    print "=" * 120
    test_node = cyber.Node("listener")
    test_node.create_reader("channel/chatter",
            SimpleMessage, callback)
    test_node.spin()

if __name__ == '__main__':

    cyber.init()
    test_listener_class()
    cyber.shutdown()
```

### 4.2写入到通道（cyber/python/examples/talker.py）

```python
from modules.common.util.testdata.simple_pb2 import SimpleMessage
from cyber_py import cyber
"""Module for example of talker."""
import time
import sys
sys.path.append("../")

def test_talker_class():
   """
   test talker.
   """
   msg = SimpleMessage()
   msg.text = "talker:send Alex!"
   msg.integer = 0
   test_node = cyber.Node("node_name1")
   g_count = 1
   writer = test_node.create_writer("channel/chatter",
                                    SimpleMessage, 6)
   while not cyber.is_shutdown():
       time.sleep(1)
       g_count = g_count + 1
       msg.integer = g_count
       print "="*80
       print "write msg -> %s" % msg
       writer.write(msg)

if __name__ == '__main__':
   cyber.init()
   test_talker_class()
   cyber.shutdown()
```

### 4.3从数据记录文件中读取或写入消息（cyber/python/examples/record.py）

```python
"""Module for example of record."""

import time
import sys

sys.path.append("../")
from cyber_py import cyber
from cyber_py import record
from google.protobuf.descriptor_pb2 import FileDescriptorProto
from modules.common.util.testdata.simple_pb2 import SimpleMessage

TEST_RECORD_FILE = "test02.record"
CHAN_1 = "channel/chatter"
CHAN_2 = "/test2"
MSG_TYPE = "apollo.common.util.test.SimpleMessage"
STR_10B = "1234567890"
TEST_FILE = "test.record"

def test_record_writer(writer_path):
    """
    record writer.
    """
    fwriter = record.RecordWriter()
    if not fwriter.open(writer_path):
        print "writer open failed!"
        return
    print "+++ begin to writer..."
    fwriter.write_channel(CHAN_1, MSG_TYPE, STR_10B)
    fwriter.write_message(CHAN_1, STR_10B, 1000)

    msg = SimpleMessage()
    msg.text = "AAAAAA"

    file_desc = msg.DESCRIPTOR.file
    proto = FileDescriptorProto()
    file_desc.CopyToProto(proto)
    proto.name = file_desc.name
    desc_str = proto.SerializeToString()

    fwriter.write_channel('chatter_a', msg.DESCRIPTOR.full_name, desc_str)
    fwriter.write_message('chatter_a', msg, 998, False)
    fwriter.write_message("chatter_a", msg.SerializeToString(), 999)

    fwriter.close()

def test_record_reader(reader_path):
    """
    record reader.
    """
    freader = record.RecordReader(reader_path)
    time.sleep(1)
    print "+"*80
    print "+++begin to read..."
    count = 1
    for channelname, msg, datatype, timestamp in freader.read_messages():
        print "="*80
        print "read [%d] msg" % count
        print "chnanel_name -> %s" % channelname
        print "msg -> %s" % msg
        print "msgtime -> %d" % timestamp
        print "msgnum -> %d" % freader.get_messagenumber(channelname)
        print "msgtype -> %s" % datatype
        count = count + 1

if __name__ == '__main__':
    cyber.init()
    test_record_writer(TEST_RECORD_FILE)
    test_record_reader(TEST_RECORD_FILE)
    cyber.shutdown()
```

