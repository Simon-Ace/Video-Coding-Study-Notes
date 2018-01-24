

## H.264视频编解码器——参考软件JM的下载与编解码

### 一、下载JM工程：
JM是H.264标准制定团队所认可的官方参考软件。网址如下
> http://iphome.hhi.de/suehring/tml/

从页面中可找到相应的工程源码，本次选择JM 8.6版本，此版本为经典版本：
> http://iphome.hhi.de/suehring/tml/download/old_jm/

### 二、配置编码环境：
下载后打开工程目录中**tml.sln**文件，VS中会有三个工程，其中rtpdump没用，删掉。另外两个ldecod和lencod分别为解码和编码工程。

首先将lencod右键-设为启动项目，并将其**“属性->常规->输出目录”**修改为`$(ProjectDir)bin`，**“属性->调试->工作目录”**也修改为`$(ProjectDir)bin`。
![1 lencod属性配置](http://ouei1rgxt.bkt.clouddn.com/18-1-24/22995529.jpg?imageMogr2/thumbnail/700x)
编译lencod工程 —— 右键lencod -> 仅用于项目 -> 仅重新生成lencod
之后在bin目录下可找到编译生成的文件。

在工作目录bin中，可以找到三个config配置文件，表示三个profile的配置，本次使用最简单的baseline配置文件进行修改。复制一份，并将文件名改为`encoder.cfg`，文件名必须是这个才能作为工程中默认参数，否则还要修改相关配置。
 - encoder_main.cfg
 - encoder_baseline.cfg
 - encoder_extended.cfg

其中配置文件部分内容如下，修改的地方为INputFile（编码文件）、FramesToBeEncoded（编码帧数），IntraPeriod（所有帧都设为I针）
```
##########################################################################################
# Files
##########################################################################################
InputFile             = "akiyo_qcif.yuv"       # Input sequence, YUV 4:2:0
InputHeaderLength     = 0      # If the inputfile has a header, state it's length in byte here 
StartFrame            = 0      # Start frame for encoding. (0-N)
FramesToBeEncoded     = 10      # Number of frames to be coded
FrameRate             = 30	   # Frame Rate per second (1-100)
SourceWidth           = 176    # Image width in Pels, must be multiple of 16
SourceHeight          = 144    # Image height in Pels, must be multiple of 16
TraceFile             = "trace_enc.txt"
ReconFile             = "test_rec.yuv"
OutputFile            = "test.264"


##########################################################################################
# Encoder Control
##########################################################################################
ProfileIDC            = 66  # Profile IDC (66=baseline, 77=main, 88=extended)
LevelIDC              = 30  # Level IDC   (e.g. 20 = level 2.0)

IntraPeriod           =  1  # Period of I-Frames (0=only first)				### if 1 -> make all the frames are I_frames
IDRIntraEnable	      =  0  # Force IDR Intra  (0=disable 1=enable)         ### if 1 -> make all I frames to IDR关键帧
.... ...
```
配置好后，运行工程，运行过程中cmd页面如下所示：
![2 cmd页面](http://ouei1rgxt.bkt.clouddn.com/18-1-24/37862327.jpg?imageMogr2/thumbnail/450x)

之后去看工作目录bin中生成的文件：
`test.264`为生成的H.264码流文件，`trace_enc.txt`是生成的日志，由于是关闭的，所以没有内容，`test_rec.yuv`为编码过程中重建的视频图像，可将此文件与原始视频作比较，即可看出失真所在。

### 三、配置解码环境：
首先将ldecod设为启动项目，编译ldecod项目（右键->仅用于项目->仅重新生成ldecod），同样修改工程配置文件：
**“属性->常规->输出目录”**修改为`$(ProjectDir)bin`，**“属性->调试->工作目录”**也修改为`$(ProjectDir)bin`。
![3](http://ouei1rgxt.bkt.clouddn.com/18-1-24/62016279.jpg?imageMogr2/thumbnail/650x)

工作目录中 `decoder.cfg`为解码配置文件，参数如下：
```
test.264                 ........H.264 coded bitstream    需要解码的码流文件
test_dec.yuv             ........Output file, YUV 4:2:0 format    输出的文件
test_rec.yuv             ........Ref sequence (for SNR)           参考帧
10                       ........Decoded Picture Buffer size
0                        ........NAL mode (0=Annex B, 1: RTP packets)
0                        ........SNR computation offset
1                        ........Poc Scale (1 or 2)
500000                   ........Rate_Decoder
104000                   ........B_decoder
73000                    ........F_decoder
leakybucketparam.cfg     ........LeakyBucket Params
```

需要将此配置文件填写到，ldecod属性->调试->命令参数中：decoder.cfg。
之后直接运行程序，即可得到解码的文件，运行过程cmd页面：
![4](http://ouei1rgxt.bkt.clouddn.com/18-1-24/18459401.jpg?imageMogr2/thumbnail/450x)
工作目录中`test_dec.yuv`即为解码后输出文件，此文件应与源文件相同。



