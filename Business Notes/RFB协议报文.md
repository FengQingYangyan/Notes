## RFB协议报文

在报文剖析前，首先我们需要知道，前后端发送的报文存储在哪些文件中。报文在服务器中以”.rfb“、".rfb.con"、".rfb.idx"、”.[rfb.st](http://rfb.st/)“为后缀的文件中存储，以上文件如何产生、产生的内容以及各文件的含义，在[图形会话回放文件记录](http://wiki.shterm.com/pages/viewpage.action?pageId=23396519)中，有详细解释（必读）；".rfb"文件存储内容的格式详见[RFB文件转为MP4文件流程](http://wiki.shterm.com/pages/viewpage.action?pageId=26485887)（必读）。在阅读以上的两个Wiki文档后，我们可以了解到在审计模块中，会话是如何产生的以及会话的文件的内容是以怎样的格式存储的。

在已知存储格式后，报文即根据该格式进行交互，其中”.rfb.idx“文件代表”.rfb“与".rfb.con"文件的映射关系，并不进行数据交互

### “.rfb”文件报文

![开发经验文档 > RFB协议报文剖析 > image-20210129112921501.png](http://wiki.shterm.com/download/thumbnails/76056324/image-20210129112921501.png?version=1&modificationDate=1612352441966&api=v2)

如图为一个”.rfb“的16进制存储形式，以及每字节对应的值，下面就对该”.rfb“文件格式进行解析

#### 1.”.rfb“文件header

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-32-2.png](http://wiki.shterm.com/download/thumbnails/76056324/image2021-2-1_17-32-2.png?version=1&modificationDate=1612352441991&api=v2)

| 字节      | 字节数（byte） | 值   | 含义                         |
| --------- | -------------- | ---- | ---------------------------- |
| 5352 4642 | 4              | SRFB | 自定义的".rfb"文件的开始标志 |
| 01        | 1              | 1    | 该文件对应的大版本号         |
| 00        | 1              | 0    | 该文件对应的小版本号         |

该部分为整个”.rfb“文件的header，每个文件中只有1个，占6个字节

注：以上部分为公司自定义的数据，与RFB协议无关。

#### 2.报文（自定义）

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-35-52.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-1_17-35-52.png?version=1&modificationDate=1612352442011&api=v2)

| 字节     | 字节数（byte） | 值   |              含义              |
| -------- | -------------- | ---- | :----------------------------: |
| 01       | 1              | 1    |        type：数据包类型        |
| 000081   | 3              | 125  |      len：消息部分的长度       |
| 00000000 | 4              | 无   |           stamp：无            |
| 略       | 125            | 略   | 消息：产生该回放的用户，时间， |

#### 3.报文（协议）

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-37-1.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-1_17-37-1.png?version=1&modificationDate=1612352442029&api=v2)

| 字节                          | 字节数（byte） | 值            |                             含义                             |
| ----------------------------- | -------------- | ------------- | :----------------------------------------------------------: |
| 04                            | 1              | 4             |                       type:数据包类型                        |
| 00000c                        | 3              | 12            |                     len：消息部分的长度                      |
| 00000000                      | 4              | 无            |                        stamp：时间戳                         |
| 5246 4220 3030 332e 3030 330a | 12             | RFB 003.003\n | vnc服务器所能够支持的最高RFB协议版本号为3.3，不足部分补零，且该部分字节数为固定值12 |

该数据包表示RFB协议的握手信息

#### 4.报文（协议）

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-39-38.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-1_17-39-38.png?version=1&modificationDate=1612352442043&api=v2)

| 字节     | 字节数（byte） | 值   |                             含义                             |             |
| -------- | -------------- | ---- | :----------------------------------------------------------: | ----------- |
| 04       | 1              | 4    |                       type:数据包类型                        |             |
| 000036   | 3              | 54   |                     len：消息部分的长度                      |             |
| 00000000 | 4              | 无   |                        stamp：时间戳                         |             |
| 0a00     | 2              | 2560 | message部分：字节数为：4+16+4+30=54 服务器发送初始化消息，主要告知客户端服务器的帧缓存（桌面屏幕）的高、宽、象素格式和桌面相关的名称。 | 屏幕的width |
| 03f9     | 2              | 1017 |                         屏幕的height                         |             |
| 10       | 1              | 16   |                             bpp                              |             |
| 10       | 1              | 16   |                            depth                             |             |
| 00       | 1              | 0    |                       big-endian-flag                        |             |
| 01       | 1              | 1    |                       true-color-flag                        |             |
| 001f     | 2              | 31   |                           red-max                            |             |
| 003f     | 2              | 63   |                          green-max                           |             |
| 001f     | 2              | 31   |                           blue-max                           |             |
| 0b       | 1              | 11   |                          red-shift                           |             |
| 05       | 1              | 5    |                         green-shift                          |             |
| 00       | 1              | 0    |                          blue-shift                          |             |
| 000000   | 3              | 0    |                           padding                            |             |
| 0000001e | 4              | 30   |                         name-length                          |             |
|          | 30             |      |                         name-string                          |             |

其中每个字节的含义在下方会有详细解释

#### 5.报文（协议）

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-40-57.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-1_17-40-57.png?version=1&modificationDate=1612352442056&api=v2)

| 字节     | 字节数（byte） | 值   |          含义          |
| -------- | -------------- | ---- | :--------------------: |
| 03       | 1              | 3    |    type:数据包类型     |
| 000014   | 3              | 20   |  len：消息部分的长度   |
| 00000000 | 4              | 无   |     stamp：时间戳      |
| 略       | 20             | 略   | 客户端发往服务端的数据 |

#### 6.报文（协议）

| 字节     | 字节数（byte） | 值    |          含义          |
| -------- | -------------- | ----- | :--------------------: |
| 02       | 1              | 2     |    type:数据包类型     |
| 0029a4   | 3              | 10660 |  len：消息部分的长度   |
| 0000002b | 4              |       |     stamp：时间戳      |
| 略       | 10660          |       | 服务端发往客户端的数据 |

#### 7.报文格式

由以上六个报文可知，可以将".rfb"文件按照这样类似的报文划分开来，同时也可以看出报文的统一格式：由8个字节的header与len长度的消息内容组成，总字节数为：8+len；

| 报文    | name            | 字节数（byte）   |    含义    |
| ------- | --------------- | ---------------- | :--------: |
| header  | type            | 1                | 报文的类型 |
| len     | 3               | 报文中消息的长度 |            |
| stamp   | 4               | 时间戳           |            |
| message | message（消息） | len所对应的值    |            |
|         | 总长            | 8+len            |            |

在报文中，如何确定是那种格式的报文，是通过type来确定的，以下将会对文件的type进行介绍；

#### 8.type类型

在知道报文的统一格式后，通过type来区分不同的报文：

| type | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| -2   | RFB_EOB：rfb报文结束                                         |
| -1   | RFB_EOF：rfb文件结束                                         |
| 1    | RFB_SESSION_INFO：会话信息，包括产生回放的用户，IP，时间。   |
| 2    | RFB_SERVER_PACKET：服务端发往客户端的数据                    |
| 3    | RFB_CLIENT_PACKET：客户端发往服务端的数据                    |
| 4    | RFB_AUTH_PACKET：协议握手信息：协议所使用的版本，协商版本安全类型，服务器初始化信息 |
| 5    | RFB_CONTEXT_PACKET：”.rfb.con“文件的报文                     |
| 0xf  | RFB_PACKET_MASK：mask                                        |
| 0    | 该形式的报文是在桌面审计模块下，写在”.rfb“文件的最后一个报文，用来标志该文件的结束，在其他模块不会有该type的报文，注意区分。 |

由以上分析可知，一个“.rfb”文件的构成

在对报文类型进行分析后，我们可以知道，不同的type其message不同，但是在type值为2，3，4时，其同一type中message中也有不同的标志位来区分message，下面对type=2,3时的message部分进行分析。

#### 9.message部分

type=2，3

|                                            |                         | 值                                                           | 字节数（byte）                                               |            含义             |
| ------------------------------------------ | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | :-------------------------: |
| type=3客户端到服务器的6种message           | message                 | 0                                                            | 1                                                            |  SetPixelFormat像素块格式   |
|                                            | 3                       | 填充                                                         |                                                              |                             |
|                                            | 16                      | pixel-format，该部分的16个字节：该部分16字节的含义表示![开发经验文档 > RFB协议报文剖析 > image2021-2-1_14-42-42.png](http://wiki.shterm.com/download/thumbnails/76056324/image2021-2-1_14-42-42.png?version=1&modificationDate=1612352442071&api=v2) |                                                              |                             |
| message                                    | 2                       | 1                                                            | SetEncoding                                                  |                             |
|                                            | 1                       | 填充                                                         |                                                              |                             |
|                                            | 2                       | number-of-encodings                                          |                                                              |                             |
|                                            | (number-of-encodings)*4 | 该部分为编码数量个encoding-type的重复                        |                                                              |                             |
| message                                    | 3                       | 1                                                            | FramebufferUpdateRequest请求帧缓存更新                       |                             |
|                                            | 1                       | increamental（当该值为0时，表示必须发送完整内容过来）        |                                                              |                             |
|                                            | 2                       | x-坐标                                                       |                                                              |                             |
|                                            | 2                       | y-坐标                                                       |                                                              |                             |
|                                            | 2                       | width                                                        |                                                              |                             |
|                                            | 2                       | height                                                       |                                                              |                             |
| message                                    | 4                       | 1                                                            | KeyBoardEvent键盘事件                                        |                             |
| 当某一个键被按下时，该值非零，释放时变为零 | 1                       | down-flag                                                    |                                                              |                             |
|                                            | 2                       | 填充                                                         |                                                              |                             |
|                                            | 4                       | key                                                          |                                                              |                             |
| message                                    | 5                       | 1                                                            | PointerEvent                                                 |                             |
|                                            | 1                       | button-mask                                                  |                                                              |                             |
|                                            | 2                       | x-position                                                   |                                                              |                             |
|                                            | 2                       | y-position                                                   |                                                              |                             |
| message                                    | 6                       | 1                                                            | ClientCutText客户端文本剪切                                  |                             |
|                                            | 3                       | 填充                                                         |                                                              |                             |
|                                            | 4                       | length                                                       |                                                              |                             |
|                                            | length                  | text                                                         |                                                              |                             |
| message                                    | 101                     | 1                                                            | DualAuthClientCmd                                            |                             |
|                                            | 3                       |                                                              |                                                              |                             |
| type=2服务器到客户端的4种message           | message                 | 0                                                            | 1                                                            | FramebufferUpdate帧缓存更新 |
|                                            | 1                       | 填充                                                         |                                                              |                             |
|                                            | 2                       | number-of-rectangles：矩形像素数据个数                       |                                                              |                             |
|                                            | 12                      | 像素数据的内容：![开发经验文档 > RFB协议报文剖析 > image2021-2-1_15-44-29.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-1_15-44-29.png?version=1&modificationDate=1612352442085&api=v2)在确定编码格式后，紧接着即为特定编码格式的数据 |                                                              |                             |
| message                                    | 1                       | 1                                                            | SetColorMapEntries颜色面板映射                               |                             |
|                                            |                         | 1                                                            | 填充                                                         |                             |
|                                            |                         | 2                                                            | first-color                                                  |                             |
|                                            |                         | 2                                                            | number-of-colors                                             |                             |
|                                            |                         |                                                              | 后续为number-of-colors个RGB值，每个RGB的格式为：![开发经验文档 > RFB协议报文剖析 > image2021-2-1_16-14-35.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-1_16-14-35.png?version=1&modificationDate=1612352442097&api=v2) |                             |
|                                            | 2                       | 1                                                            | Bell响铃事件                                                 |                             |
| message                                    | 3                       | 1                                                            | ServerCutText服务器剪切文本                                  |                             |
|                                            | 3                       | 填充                                                         |                                                              |                             |
|                                            | 4                       | length                                                       |                                                              |                             |
|                                            | length                  | text：该部分的消息为length个1字节的text                      |                                                              |                             |
| message                                    | 250                     | 1                                                            | Colin Dean xvp                                               |                             |
|                                            | 1                       | 填充                                                         |                                                              |                             |
|                                            | 1                       | version                                                      |                                                              |                             |
|                                            | 1                       | message：其中message对应的值有0，1                           |                                                              |                             |
| message                                    | 102                     | 1                                                            | DualAuthServeerCmd                                           |                             |
|                                            | 3                       | 内容为3个字节的数组                                          |                                                              |                             |

尽管message部分的type有很多种，但是，我们需要着重关注的仅有上图中绿色部分；

相信通过以上部分的讲解，对“.rfb”文件已经有了了解，下面对整个“.rfb“的组成做一下说明：

| type     | number | 字节数（byte） | 含义                                                         |
| -------- | ------ | -------------- | ------------------------------------------------------------ |
|          | 1      | 6              | ".rfb"文件的Header                                           |
| 1        | 1      | 133            | 产生该文件的用户，时间                                       |
| 4        | 1      | 20             | RFB协议所使用的版本                                          |
| 4        | 1      | 62             | 协商协议的安全类型                                           |
| 4        | 1      | 24             | 初始化信息（该部分的初始化信息并非在每个文件中都会存在），服务器定义服务器本来的像素格式，这种象素格式会被一直使用，除非客户端使用设置象素格式消息来请求另一种象素格式。 |
| 2 \|\| 3 | 未知   |                | 这2种类型的报文，组成了".rfb"文件的大部分，数量不固定        |
| -1       | 1      |                | ".rfb"文件的结束                                             |

其中type = 5的报文均存储在".rfb.con"文件中；

### “.rfb.con”文件

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-45-49.png](http://wiki.shterm.com/download/thumbnails/76056324/image2021-2-1_17-45-49.png?version=1&modificationDate=1612352442111&api=v2)

该文件与“.rfb”文件的关系在上方的Wiki链接中已做详细解释，此处不再做说明，下面对该文件的格式做详细说明：

#### 1.“.rfb.con”文件header

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-47-4.png](http://wiki.shterm.com/download/thumbnails/76056324/image2021-2-1_17-47-4.png?version=1&modificationDate=1612352442123&api=v2)

| 字节                | 字节数（byte） | 值       | 含义                   |
| ------------------- | -------------- | -------- | ---------------------- |
| 5352 4642 2e43 4f4e | 8              | SRFB.CON | ".rfb.con"文件的标识   |
| 01                  | 1              | 1        | “.rfb.con”文件的大版本 |
| 02                  | 1              | 1        | “.rfb.con”文件的小版本 |

#### 2.报文

![开发经验文档 > RFB协议报文剖析 > image2021-2-1_17-48-24.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-1_17-48-24.png?version=1&modificationDate=1612352442136&api=v2)

|                      |                             | 字节 |                        字节数（byte）                        |                              值                              |                      含义                      |
| :------------------: | :-------------------------: | :--: | :----------------------------------------------------------: | :----------------------------------------------------------: | :--------------------------------------------: |
|     header8字节      |        header 8字节         |  05  |                              1                               |                              5                               | type：RFB_CONTEXT_PACKET，“.rfb.con”文件的报文 |
|        000016        |              3              |  22  |                    len：message部分的长度                    |                                                              |                                                |
|       00000000       |              4              |  无  |                        stamp：时间戳                         |                                                              |                                                |
|    message22字节     | 服务器桌面屏幕的宽和高4字节 | 0a00 |                              2                               |                             2560                             |             framebuffer-width：宽              |
|         03f9         |              2              | 1017 |                    framebuffer-height：高                    |                                                              |                                                |
| 服务器像素格式13字节 |             10              |  1   |                              16                              |   bits-pre-pixel：每个像素值需要的位数，该值必须大于depth    |                                                |
|          10          |              1              |  13  |                  depth：像素值中有用的位数                   |                                                              |                                                |
|          00          |              1              |  0   |                big-endiaan-flag：高位编址标志                |                                                              |                                                |
|          01          |              1              |  1   | true-color-flag：真彩标志，如果非零采用下边6项渲染颜色；如果为零则采用颜色面板映射 |                                                              |                                                |
|         001f         |              2              |  31  |                    red-max：红色的最大值                     |                                                              |                                                |
|         003f         |              2              |  63  |                   green-max：绿色的最大值                    |                                                              |                                                |
|         001f         |              2              |  31  |                    blue-max：蓝色的最大值                    |                                                              |                                                |
|          0b          |              1              |  11  |         red-shift：要得到最低明显位所需要的替换个数          |                                                              |                                                |
|          05          |              1              |  5   |                      green-shift：同上                       |                                                              |                                                |
|          00          |              1              |  0   |                       blue-shift：同上                       |                                                              |                                                |
|   inflateSync1字节   |             00              |  1   |                              0                               | inflateSync：该部分含义已不明确，在老版本中该值会有改动，在当前及以后版本不会做修改，为固定值0 |                                                |
|     imgLen4字节      |             00              |  1   |                              0                               |               具体图像数据的长度：该值为len-22               |                                                |
|          00          |              1              |  0   |                                                              |                                                              |                                                |
|          00          |              1              |  0   |                                                              |                                                              |                                                |
|          00          |              1              |  0   |                                                              |                                                              |                                                |
| img数据(len-22)字节  |             无              |  无  |                              无                              |       由于本报文中没有img数据，因此，此处没有对其解析        |                                                |

“.rfb.con”文件为公司自定义文件，因此其中的报文格式也是由公司自定义产生，因此该部分的数据与SetPixelFormat的格式类似；在“.rfb.con”文件中，就是由文件header与一个个的报文组成。

### “.rfb.idx”文件

![开发经验文档 > RFB协议报文剖析 > image2021-2-5_16-5-46.png](http://wiki.shterm.com/download/thumbnails/76056324/image2021-2-5_16-5-46.png?version=1&modificationDate=1612512346348&api=v2)

#### ".rfb.idx"文件的header

![开发经验文档 > RFB协议报文剖析 > image2021-2-5_16-7-47.png](http://wiki.shterm.com/download/thumbnails/76056324/image2021-2-5_16-7-47.png?version=1&modificationDate=1612512468044&api=v2)

| 字节                | 字节数（byte） | 值       | 含义                   |
| ------------------- | -------------- | -------- | ---------------------- |
| 5352 4642 2e49 4458 | 8              | SRFB.IDX | ".rfb.idx"文件的标识   |
| 01                  | 1              | 1        | “.rfb.idx”文件的大版本 |
| 02                  | 1              | 1        | “.rfb.idx”文件的小版本 |

#### 时间戳

![开发经验文档 > RFB协议报文剖析 > image2021-2-5_16-8-4.png](http://wiki.shterm.com/download/thumbnails/76056324/image2021-2-5_16-8-4.png?version=1&modificationDate=1612512484644&api=v2)

| 字节      | 字节数 | 值     | 含义                                                 |
| --------- | ------ | ------ | ---------------------------------------------------- |
| 0001 d344 | 4      | 119620 | 记录了“.rfb”文件的最后一帧的时间戳，因此为最大时间戳 |

#### message

![开发经验文档 > RFB协议报文剖析 > image2021-2-5_16-12-51.png](http://wiki.shterm.com/download/attachments/76056324/image2021-2-5_16-12-51.png?version=1&modificationDate=1612512771601&api=v2)



| 字节                | 字节数 | 值   | 含义                                            |
| ------------------- | ------ | ---- | ----------------------------------------------- |
| 0000 0000           | 4      | 0    | 某一帧的时间戳                                  |
| 0000 0000 0000 0000 | 8      | 0    | rfb_pos：时间戳对应的".rfb"文件的游标位置       |
| fd00 0000 0000 0000 | 8      | 223  | con_pos：时间戳对应的“.rfb.con”文件的游标的位置 |

需要注意的是，以上3部分的数据均为高位在右的方式进行的存储，因此在换算成具体的值时，需要从右至左来换算；



其中的message部分即为“.rfb.idx”文件的主要部分；