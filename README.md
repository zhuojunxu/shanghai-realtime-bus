# Shanghai Realtime Bus

「上海公交」APP 的 Python 实现。

好用请 Star~

## Installation

`pip install shbus`

## Usage

### 线路查询 (修理版，之后和何大佬一起讨论弄一个pull request)

```
from shbus import realtime, LineSequence
# Direction 代表上下行，默认为上行（方向=0），83路为例子，长清北路枢纽站往三林世博家园为下行，所以为routes[1], 反方向为上行，所以改routes[0]
response = realtime.get_realtime_bus([LineSequence(line='83路', info=True)])

print(response[0].info.routes[1].time.early)
print(response[0].info)
print("上行站点：")
for name in response[0].info.routes[0].names:
    print(name)

print()

print("下行站点：")
for name in response[0].info.routes[1].names:
    print(name)

print("首末班车如下：")
print(response2[0].info.routes[0].names[0] + " -> " + response2[0].info.routes[0].names[len(response2[0].info.routes[0].names) - 1] + ": " + response2[0].info.routes[0].time.early + " " + response2[0].info.routes[0].time.late)
print(response2[0].info.routes[1].names[0] + " -> " + response2[0].info.routes[1].names[len(response2[0].info.routes[1].names) - 1] + ": " + response2[0].info.routes[1].time.early + " " + response2[0].info.routes[1].time.late)
```

### 实时公交

```
>>> from shbus import realtime, LineSequence
>>> response = realtime.get_realtime_bus([LineSequence(line='虹桥枢纽4路', sequence=30)])[0].items[0]
>>> print('%s: 还剩%s站, 距离%s米, 还剩%s秒' % (response.vehicle, response.stops, response.distance, response.time))
沪D-D6957: 还剩2站, 距离1354米, 还剩153秒
```

具体参见 example 目录下的示例代码。

## Principle

### 数据结构

「上海公交」APP 有两种数据结构，分别用于~~普通的线路查询~~ (已废弃)以及实时公交查询。

1. ~~线路查询所返回的数据结构~~ (已废弃):

    貌似是项目组自己实现的二进制格式，需要逐比特位分析。
    
    - 字节: 1字节，通常转为10进制使用
    - 整型：4字节
    - 字符串：前4字节表示字符串的长度 n，后 2 * n 个字节是字符串的 UTF-16LE 编码表示
    - 浮点数：4字节。先转换为整型，再调用 Java 自带的 intBitsToFloat 转换为浮点数

2. 实时公交所返回的数据结构:
    
    是经过 AES CBC 加密的 Protobuf 结构化消息。解析起来有两个难点：如何找到 AES 的加密秘钥和初始向量；以及如何构造 Protobuf 消息的结构。这里不多赘述。

### 线路查询

「上海公交」的线路查询功能共有3个接口：

1. 搜索接口 （需要进行修改，原理是采用AES，和实时公交查询使用的URL一致）:
    `[http://lbs.jt.sh.cn:8082/app/rls/monitor](http://lbs.jt.sh.cn:8082/app/rls/monitor)`

2. 线路信息，查询首末班车，起点站和终点站:
    使用实时公交查询链接，实时公交查询连接如果能进行post并返还成功后，定义resposnse如下：
    ```
    response = realtime.get_realtime_bus([LineSequence(line='83路', info=True)])
    ```
    
    用于查询线路的始末站以及首末班车时间。
    上行起点站（83路为例，是三林世博家园）：
   ```
   print(response[0].info.routes[0].names[0])
   ```
    上行终点站（83路为例，是长清北路枢纽站）：
   ```
   print(response[0].info.routes[0].names[len(response[0].info.routes[0].names) - 1])
   ```
    下行起点站（83路为例，是长清北路枢纽站）：
   ```
   print(response[0].info.routes[1].names[0])
   ```
    下行终点站（83路为例，是三林世博家园）：
   ```
   print(response[0].info.routes[1].names[len(response[0].info.routes[1].names) - 1)
   ```
    
    上行（83路为例，三林世博家园发往长清北路枢纽站），所有站点代码如下：
    ```
    for name in response[0].info.routes[0].names:
        print(name)
    ```
    下行（83路为例，长清北路枢纽站发往三林世博家园），所有站点代码如下：
    ```
    for name in response[0].info.routes[1].names:
        print(name)
    ```
   
    上行首班车，末班车，下行首班车，末班车代码如下：
    ```
    print(response[0].info.routes[0].time.early)
    print(response[0].info.routes[0].time.late)
    print(response[0].info.routes[1].time.early)
    print(response[0].info.routes[1].time.late)
    6:00
    20:30
    6:30
    21:00
    ```

### 实时公交查询

`http://lbs.jt.sh.cn:8082/app/rls/monitor`

需要先用 Protobuf 构造一条请求消息，将其使用 AES 加密后 POST 到上述 URL，接收到信息后再使用 AES 解密后用 Protobuf 解包。具体详见代码。

## The MIT License

Copyright 2019

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

