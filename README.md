---
order: 6
title: gRPC 入门
type: 入门
---


## 什么是gRPC

`gRPC`客户端如同调用本地方法一样可以调用不在同一台物理机器上的远端服务器的方法，使用`gPRC`可以很方便的创建分布式应用和服务。
`gRPC`旨在定义方法可以通过传递参数被远程调用返回确定的类型，在服务器端只需要实现接口处理客户端的请求。客户端通过`stub`实现
调用远程方法调用。gRPC可以跨语言，所以你可以在服务端使用Java,在客户端使用python，go，ruby等。


## Protocol Buffer

`Protocol buffer`是一个灵活高效的序列化结构数据的工具，你只需定义一次你的数据，就可以使用多种语言来
读取和写入数据。

message.proto

```
syntax = "proto2";
package tutorial;

// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  required string name = 1;
}

// The response message containing the greetings
message HelloReply {
  required string message = 1;
}

```
执行如下命令`protoc --python_out=./  ./message.proto` 会生成对应的`message_pb2`代码，当然还支持
`Java`,`Go`, `Dart`, `C++`等其他语言

```
# Generated by the protocol buffer compiler.  DO NOT EDIT!
# source: message.proto

from google.protobuf import descriptor as _descriptor
from google.protobuf import message as _message
from google.protobuf import reflection as _reflection
from google.protobuf import descriptor_pb2
# @@protoc_insertion_point(imports)




DESCRIPTOR = _descriptor.FileDescriptor(
  name='message.proto',
  package='tutorial',
  serialized_pb='\n\rmessage.proto\x12\x08tutorial\"\x1c\n\x0cHelloRequest\x12\x0c\n\x04name\x18\x01 \x02(\t\"\x1d\n\nHelloReply\x12\x0f\n\x07message\x18\x01 \x02(\t2E\n\x07Greeter\x12:\n\x08SayHello\x12\x16.tutorial.HelloRequest\x1a\x14.tutorial.HelloReply\"\x00')




_HELLOREQUEST = _descriptor.Descriptor(
  name='HelloRequest',
  full_name='tutorial.HelloRequest',
  filename=None,
  file=DESCRIPTOR,
  containing_type=None,
  fields=[
    _descriptor.FieldDescriptor(
      name='name', full_name='tutorial.HelloRequest.name', index=0,
      number=1, type=9, cpp_type=9, label=2,
      has_default_value=False, default_value=unicode("", "utf-8"),
      message_type=None, enum_type=None, containing_type=None,
      is_extension=False, extension_scope=None,
      options=None),
  ],
  extensions=[
  ],
  nested_types=[],
  enum_types=[
  ],
  options=None,
  is_extendable=False,
  extension_ranges=[],
  serialized_start=27,
  serialized_end=55,
)


_HELLOREPLY = _descriptor.Descriptor(
  name='HelloReply',
  full_name='tutorial.HelloReply',
  filename=None,
  file=DESCRIPTOR,
  containing_type=None,
  fields=[
    _descriptor.FieldDescriptor(
      name='message', full_name='tutorial.HelloReply.message', index=0,
      number=1, type=9, cpp_type=9, label=2,
      has_default_value=False, default_value=unicode("", "utf-8"),
      message_type=None, enum_type=None, containing_type=None,
      is_extension=False, extension_scope=None,
      options=None),
  ],
  extensions=[
  ],
  nested_types=[],
  enum_types=[
  ],
  options=None,
  is_extendable=False,
  extension_ranges=[],
  serialized_start=57,
  serialized_end=86,
)

DESCRIPTOR.message_types_by_name['HelloRequest'] = _HELLOREQUEST
DESCRIPTOR.message_types_by_name['HelloReply'] = _HELLOREPLY

class HelloRequest(_message.Message):
  __metaclass__ = _reflection.GeneratedProtocolMessageType
  DESCRIPTOR = _HELLOREQUEST

  # @@protoc_insertion_point(class_scope:tutorial.HelloRequest)

class HelloReply(_message.Message):
  __metaclass__ = _reflection.GeneratedProtocolMessageType
  DESCRIPTOR = _HELLOREPLY

  # @@protoc_insertion_point(class_scope:tutorial.HelloReply)


# @@protoc_insertion_point(module_scope)

```

## gRPC示例

使用`python3 -m grpc_tools.protoc -I./ --python_out=./ --grpc_python_out=./ ./message.proto`命令生成
python接口文件`message_pb2.py`，和gprc接口文件 `message_pb2_grpc.py`

**Server 端**

```
"""The Python implementation of the GRPC helloworld.Greeter server."""

from concurrent import futures
import time
import logging

import grpc

import message_pb2
import message_pb2_grpc

_ONE_DAY_IN_SECONDS = 60 * 60 * 24


class Greeter(message_pb2_grpc.GreeterServicer):

    def SayHello(self, request, context):
        return message_pb2.HelloReply(message='Hello, %s!' % request.name)


def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    message_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    try:
        while True:
            time.sleep(_ONE_DAY_IN_SECONDS)
    except KeyboardInterrupt:
        server.stop(0)


if __name__ == '__main__':
    logging.basicConfig()
    serve()
```

**Client 端**

```
"""The Python implementation of the GRPC helloworld.Greeter client."""

from __future__ import print_function
import logging

import grpc

import message_pb2
import message_pb2_grpc


def run():
    # NOTE(gRPC Python Team): .close() is possible on a channel and should be
    # used in circumstances in which the with statement does not fit the needs
    # of the code.
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = message_pb2_grpc.GreeterStub(channel)
        response = stub.SayHello(message_pb2.HelloRequest(name='you'))
    print("Greeter client received: " + response.message)


if __name__ == '__main__':
    logging.basicConfig()
    run()
```

## 启动server端等待远程调用

```
python3 server.py
```

## 启动客户端调用远端接口

```
python3 client.py
Greeter client received: Hello, you!
```

[Github gRPC Exampl](https://github.com/JetQin/grpc_example.git)