# ffmpeg 视频过滤器scale官方简介

tags:[ffmpeg]

---

### 11.173 scale 过滤器
Scale 过滤器通过使用libswscale 库能重新调整输入的视频的大小.

scale过滤器 通过改变输出样本的宽高比来强制输出展示的宽高比与输入相同.

如果输入的图片格式与下个过滤器请求的格式不同，scale过滤器将会将输入转换为要求的格式.

### 11.173.1 选项

该过滤器接受以下选项，或者任何在libswscale scaler库支持的选项.

完全手册参考[(ffmpeg-scaler)the ffmpeg-scaler manual](http://www.ffmpeg.org/ffmpeg-scaler.html#scaler_005foptions).

**width,w**

**height,h**

设置输出视频的宽高.默认值与输入视频的宽高相同.

如果**width**或者**w**的值为0,那么输出视频的宽度将采用输入视频的宽度作为默认宽度.高度同之.

如果当且仅有其中一个值为 **-n**,并且 n >= 1,那么scale过滤器将使用一个值来维护输入视频的宽高比,这个值将由另一个指定的维度计算获得.另外，这种情况下，需要确保另一个指定的维度需要能够被n整除，否则需要重新调整该值的大小.(比如，宽度为-2,高度为480，则整除计算获得宽度为240)

如果两个值都为 **-n** 并且 n >= 1,这种情况等同于将两个值都设置为0，也就是说输出视频将与输入视频的宽高等同.

**以下是宽高比表达式所支持的常量列表:**

**eval**

用来指定宽高等同的表达式.它接受以下值:


‘init’

仅在过滤器初始化期间或处理命令时对表达式求值一次.

‘frame’

为每个传入帧计算表达式的值.

默认值为**‘init’**.


**interl**

设置隔行扫描的模式.它接受以下值:

‘1’
开启隔行扫描.

‘0’
不应用隔行扫描.

‘-1’
是否使用隔行扫描取决于源帧是否被标记为隔行扫描.

默认值为 ‘0’.

**falgs**

设置libswscale 扩展的一些标志.详细查看 [(ffmpeg-scaler)the ffmpeg-scaler manual](http://www.ffmpeg.org/ffmpeg-scaler.html#sws_005fflags) . 在未为过滤器明确指定的情况下都将采用默认参数.

**param0, param1**

为libswscale 缩放算法设置输入参数.完整文档参考[(ffmpeg-scaler)the ffmpeg-scaler manual](http://www.ffmpeg.org/ffmpeg-scaler.html#sws_005fparams) 未明确指定的情况下将置为空.

**size, s**
设置视频的大小.这个选项的具体语法参考[(ffmpeg-utils)"Video size" section](http://www.ffmpeg.org/ffmpeg-scaler.html#sws_005fparams) .

**in_color_matrix**
**out_color_matrix**
设置输入输出的色彩空间类型.

这允许你强制指定一个值用于输出和编码器.

如果未特别指定.色彩空间类型取决于像素类型.

可能的值如下:

**‘auto’**

自动选择.

**‘bt709’**

Format conforming to International Telecommunication Union (ITU) Recommendation BT.709.


**‘fcc’**

Set color space conforming to the United States Federal Communications Commission (FCC) Code of Federal Regulations (CFR) Title 47 (2003) 73.682 (a).

**‘bt601’**
**‘bt470’**
**‘smpte170m’**

Set color space conforming to:

ITU Radiocommunication Sector (ITU-R) Recommendation BT.601
ITU-R Rec. BT.470-6 (1998) Systems B, B1, and G
Society of Motion Picture and Television Engineers (SMPTE) ST 170:2004
**‘smpte240m’**

Set color space conforming to SMPTE ST 240:1999.

**‘bt2020’**

Set color space conforming to ITU-R BT.2020 non-constant luminance system.

**in_range**
**out_range**
设置输入输出色彩空间样本的范围.

这允许你强制指定一个值用于输出和编码器.

如果未特别指定.色彩空间类型取决于像素类型.

可能的值如下:

**‘auto/unknown’**

自动选择.

**‘jpeg/full/pc’**

Set full range (0-255 in case of 8-bit luma).

**‘mpeg/limited/tv’**

Set "MPEG" range (16-235 in case of 8-bit luma).

**force_original_aspect_ratio**

在需要时自动减少或者增加输出的视频的宽度或者高度来保证源视频的宽高比.

可能的值如下:


**‘disable’**
按设定缩放视频并且关闭此特性.

**‘decrease’**
输出视频宽高在需要时将自动减少.

**‘increase’**
输出视频宽高在需要时将自动增加.

比如，设备A最多只允许播放1280x720的视频，并且你的视频宽高是1920 x 800.使用这个选项(设置为decrease)并且指定输出1280x720的视频，最后将输出1280 x 533的视频(与原视频宽高比相同).

另外需要注意的是这个选项与将**w**或者**h**指定为**-1**不同，你依然需要设置输出的分辨率才能让这个设置生效.

**force_divisible_by**

确保输出的参数，宽度和高度能够被给予的参数整除.这个设置与在**w**和**h**选项中设置**-n**的效果等同.

这个设置同样受**force_original_aspect_ratio**选项影响，会按照该选项设置增加或者减少相应的分辨率.

当你有一个视频既受编码器宽度或者高度整除的限制，又需要微调视频的输出分辨率，可以采用这个设置.

**w**和**h**选项的值是包含以下常量的表达式：

in_w
in_h
The input width and height

iw
ih
These are the same as in_w and in_h.

out_w
out_h
The output (scaled) width and height

ow
oh
These are the same as out_w and out_h

a
The same as iw / ih

sar
input sample aspect ratio

dar
The input display aspect ratio. Calculated from (iw / ih) * sar.

hsub
vsub
horizontal and vertical input chroma subsample values. For example for the pixel format "yuv422p" hsub is 2 and vsub is 1.

ohsub
ovsub
horizontal and vertical output chroma subsample values. For example for the pixel format "yuv422p" hsub is 2 and vsub is 1.

### 11.173.2 Examples


**Scale the input video to a size of 200x100**

	scale=w=200:h=100

This is equivalent to:

	scale=200:100

or:

	scale=200x100

**Specify a size abbreviation for the output size:**

	scale=qcif

which can also be written as:

	scale=size=qcif

**Scale the input to 2x:**

	scale=w=2*iw:h=2*ih

**The above is the same as:**

	scale=2*in_w:2*in_h
**Scale the input to 2x with forced interlaced scaling:**

	scale=2*iw:2*ih:interl=1

**Scale the input to half size:**

	scale=w=iw/2:h=ih/2
**Increase the width, and set the height to the same size:**

	scale=3/2*iw:ow

**Seek Greek harmony:**
```
scale=iw:1/PHI*iw
scale=ih*PHI:ih
```
**Increase the height, and set the width to 3/2 of the height:**

	scale=w=3/2*oh:h=3/5*ih

**Increase the size, making the size a multiple of the chroma subsample values:**

	scale="trunc(3/2*iw/hsub)*hsub:trunc(3/2*ih/vsub)*vsub"

**Increase the width to a maximum of 500 pixels, keeping the same aspect ratio as the input:**

	scale=w='min(500\, iw*3/2):h=-1'

**Make pixels square by combining scale and setsar:**

	scale='trunc(ih*dar):ih',setsar=1/1

**Make pixels square by combining scale and setsar, making sure the resulting resolution is even (required by some codecs):**

	scale='trunc(ih*dar/2)*2:trunc(ih/2)*2',setsar=1/1


[官方引用](http://www.ffmpeg.org/ffmpeg-filters.html#scale-1)