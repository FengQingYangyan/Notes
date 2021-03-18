### TIGHT编码

Tight编码为像素数据提供了有效的压缩。为了减少实现的复杂性，任何Tight编码的宽度不能超过2048像素，如果需要一个更宽的矩形，它必须被分割为几个矩形，每个矩形都应该被单独编码。



TIGHT编码的矩形像素块的数据，首先都是1个字节的compression-control：

| No. of bytes | Type |     Description     |
| ------------ | ---- | :-----------------: |
| 1            | U8   | compression-control |

compression-control 字节的最低4位通知客户端在解码矩形之前应该重置哪个zlib压缩流，每个位都是独立的，对应一个单独的应该被重置的zlib流；

| Bit  |  Description   |
| ---- | :------------: |
| 0    | Reset stream 0 |
| 1    | Reset stream 1 |
| 2    | Reset stream 2 |
| 3    | Reset stream 3 |



在TIGHT编码中支持的压缩方式有3种，分别为BasicCompression，FillCompression和JpegCompression方式，当bit 7为0时，将使用BasicComprssion方式；此时，4-6位被解释为：

| Bits | Binary Values |     Description      |
| :--: | :-----------: | :------------------: |
| 5-4  |      00       |     Use stream 0     |
|      |      01       |     Use stream 1     |
|      |      10       |     Use stream 2     |
|      |      11       |     Use stream 3     |
|  6   |       0       |         ...          |
|      |       1       |    read-filter-id    |
|  7   |       0       | **BasicCompression** |



如果bit 7被设置为1，将使用FillCompression或JpegCompression方式，这依赖于该字节的其他几位：

| Bits | Binary value |     Description     |
| ---- | ------------ | :-----------------: |
| 7-4  | 1000         | **FillCompression** |
|      | 1001         | **JpegCompression** |
|      | any other    |       Invalid       |

Note：**JpegCompression**方式只能在 bits-per-pixel 为16或32，并且客户端已经使用JPEG质量级别伪编码发布了一个质量级别时使用。

TIGHT编码使用了一种新的像素格式TPIXEL（Tight pixel），该格式与约定的像素格式（setPixelFormat中的格式）是相同的，除了true-color-flag是非零、 bits-per-pixel为32，depth为24，并且构成红、绿和蓝色的位恰好都为8位宽；在本例中，TPIXEL只有3个字节长，这三个字节是像素颜色值，其中第一个字节为红色组成部分，第二个字节为绿色组成部分，第三个字节为蓝色组成部分；

在compression-control字节之后的数据取决于压缩方式：



**FillCompression**

如果压缩方式是FillCompression，那么只有像素值，以TPIXEL格式。此值适用于矩形的所有像素。



**Jpegcompression**

当压缩方式是JpegCompression时，数据将采用以下格式：

| No. of bytes |   Type   |           Description            |
| :----------: | :------: | :------------------------------: |
|     1-3      |          | length in compact representation |
|    length    | U8 array |            jpeg-data             |

当使用1个或2个或3个字节表示长度时，按照以下方案：

|           Value            |    Description    |
| :------------------------: | :---------------: |
|          0xxxxxxx          |     值0...127     |
|     1xxxxxxx 0yyyyyyy      |   值128...16383   |
| 1xxxxxxx 1yyyyyyy zzzzzzzz | 值16384...4194303 |

这里每个字符代表一个位，xxxxxxx是这个值的最低有效位(0-6位)，yyyyyyy是第7-13位，zzzzzz是最高有效位(14-21位)。例如，十进制值10000应该表示为两个字节:二进制数10010000 01001110，或十六进制数90 4E。

**BasicCompression**

如果压缩方式为BasicCompression类型并且compression-control的bit 6的值为1时，下1个字节告诉解码器将使用什么过滤器类型，filter-id表明什么过滤器被编码器使用来将压缩前的数据进行预处理。filter-id字节可以是以下之一：

| No. of bytes | Type | Value |         Description         |
| :----------: | :--: | :---: | :-------------------------: |
|      1       |  U8  |       |          filer-id           |
|              |      |   0   | **CopyFilter**（no filter） |
|              |      |   1   |      **PaletteFilter**      |
|              |      |   2   |     **GradientFilter**      |

如果compression-control 的bit 6被设置为0，将使用CopyFilter。

**CopyFilter**

当CopyFilter被压缩时，TPIXEL格式的原始像素值将被压缩。关于压缩的详细信息见下面。



**PaletteFilter**

PaletteFilter 将true-color数据转换为索引颜色数据和包含2...256种颜色的调色板，如果颜色的数量为2，接下来的像素用1位来编码，否则用8位来编码一个像素。1位编码的执行方式是：最有效位对应于最左边的像素，每一行像素都与字节边界对齐。当使用PaletteFilter时，调色板将被发送在像素数据之前，调色板以一个无符号位字节开始，该字节的值是调色板中的颜色数减去1（即1表示2种颜色，255表示调色板中的256种颜色）。然后跟随调色板本身，它由TPIXEL格式的像素值组成。



**GradientFilter**

GradientFilter使用一个简单的算法预处理像素数据，将每个颜色分量转换成“预测”强度和实际强度之间的差值。这种技术不会影响未压缩的数据大小，但有助于更好地压缩类似照片的图像。将强度转换为差异的伪代码如下:

```
P[i,j] := V[i-1,j] + V[i,j-1] - V[i-1,j-1]; if (P[i,j] < 0) then P[i,j] := 0; if (P[i,j] > MAX) then P[i,j] := MAX; D[i,j] := V[i,j] - P[i,j];
```

这里V[i,j]是坐标(i,j)处像素的颜色分量的强度。对于当前矩形以外的像素，假设V[i,j]为零(这与P[i,0]和P[0,j]有关)。MAX是颜色组件的最大强度值。

Note：GradientFilter只有当bits-per-pixel是16或32的时候被使用。

在使用上述三个过滤器之一过滤像素数据后，使用zlib库对其进行压缩。但是如果在应用过滤器之后但在压缩之前的数据大小小于12，那么数据将按原样发送，不压缩。可以使用四个单独的zlib流(0..3)，解码器应该从压缩控制字节中读取实际的流id。

如果没有使用压缩，则像素数据按原样发送，并且数据流是以下格式:

| No. of bytes | Type     |            Description             |
| ------------ | -------- | :--------------------------------: |
| 1-3          |          | *length* in compact representation |
| length       | U8 array |              zlibData              |

长度用一个、两个或三个字节来表示，就像JpegCompression方式。

**NOTE：**如果compression-control字节中的bits 0、1、2和3设置为1，解码器必须在解码矩形之前重置zlib流。请注意，解码器必须重置指定的zlib流，即使压缩类型是FillCompression或JpegCompression。

以上的信息源文档：https://github.com/rfbproto/rfbproto/blob/master/rfbproto.rst