该文档里面所包含的案例仅供参考
# 获取设备数据程序（GrabFakeDataOneByOne）使用简介
## 1 获取设备数据程序功能概括
仿生眼系列设备的标定数据储存在配置文件中，该例程展示了如何 ***获取从设备中获取数据***。

## 2 获取设备数据程序使用方法
根据 *path_to_workspace/BionicEyes/README.md* 中的流程成功编译仿生眼 *workspace* 后，所有 *sample* 程序的可执行文件保存在 *path_to_workspace/BionicEyes/bin/* 目录中，根据 *ubuntu* 操作系统的版本，程序的可执行文件被命名为：

>evo_be_Sample_GrabFakeDataOneByOne_2004  
>evo_be_Sample_GrabFakeDataOneByOne_1804  
>...

### 2.1 开启远端服务
远程控制例程中创建的是远程连接实例 (创建直连实例同样可以获得标定数据)，故在使用时请确保已经有一台 *PC* 直连仿生眼设备并开启了仿生眼服务。具体方法如下:

在直连PC的 *path_to_workspace/BionicEyes/bin/* 目录下，根据对应设备不同，运行相应程序：
|设备        |可执行文件                     | 设备依赖库    |
|:--:       |:--:                         | :--:    |
|仿生眼III   |evo_be_Device_*               | libevo_be_Device_shared_*.so|
|仿生眼V     |evo_be_Device_5_*             | libevo_be_Device_5_shared_*.so|
|仿生鹰眼I   |evo_be_EEDevice_*             | libevo_be_EEDevice_shared_*.so|
|三轴平台    |evo_be_Device_ThreeAxis_*     | libevo_be_Device_ThreeAxis_shared_*.so|

**注：直连 PC 除运行上述设备程序外，还需要在另一个终端运行 *evo_be_Device_Service_\** 用于启动仿生眼服务后，才能供远端 PC 检测到仿生眼系列设备的存在。**
***\*代表对应系统平台，例如 1804 代表 ubuntu18.04***

### 2.2 运行远程程序
直连 PC 开启仿生眼服务后，***与直连 PC 处于同一个局域网下的***远程 *PC* 设备即可检测到仿生眼存在并进行连接，在远程 *PC* 设备中进行如下操作：
#### 2.2.1 进入bin目录
 `cd  path_to_workspace/BionicEyes/bin/`
 
#### 2.2.2 运行可执行文件
 `./evo_be_Sample_GrabFakeDataOneByOne_2004`
运行可执行文件指令后，首先会进行当前场景中设备的检测，**检测到多台设备时, 终端输出为：**
```
[2023-08-03 10:41:23.483] [EthernetSpec_Console] [info] EthComSpec: <<requestBeIpAddress>> Request to get be_device ip address...
[2023-08-03 10:41:26.319] [BE_Console] [info] BionicEyes: BionicEye device detected!!!
[2023-08-03 10:41:26.319] [BE_Console] [critical] BionicEyes: 0: BinoSense_BE150A28210000  192.168.31.88
[2023-08-03 10:41:26.319] [BE_Console] [critical] BionicEyes: 1: BinoSense_BE150A28210010  192.168.31.35
[2023-08-03 10:41:26.319] [BE_Console] [critical] BionicEyes: Please input the device Id(0,1...):
```
上述终端输出中，显示了远程连接例程检测到了两台仿生眼设备, 并将设备信息进行了输出，输出格式为：
|         输出信息            |  含义   |
|:-------------------------:|:-------:|
|             0/1/2...      | 可选设备编号 |
| BinoSense_BE150A28210000  | 设备编号    |
|        192.168.31.88      | 设备IP地址  |

根据输出的可选设备编号，以及设备编号信息，在终端输入0或1、2...即可选择想要连接的设备。

例如选择0号设备，在终端中输入0即可成功连接，如下所示：
```
[2023-08-03 10:41:23.483] [EthernetSpec_Console] [info] EthComSpec: <<requestBeIpAddress>> Request to get be_device ip address...
[2023-08-03 10:41:26.319] [BE_Console] [info] BionicEyes: BionicEye device detected!!!
[2023-08-03 10:41:26.319] [BE_Console] [critical] BionicEyes: 0: BinoSense_BE150A28210000  192.168.31.88
[2023-08-03 10:41:26.319] [BE_Console] [critical] BionicEyes: 1: BinoSense_BE150A28210010  192.168.31.35
[2023-08-03 10:41:26.319] [BE_Console] [critical] BionicEyes: Please input the device Id(0,1...):
0 
[2023-08-03 11:00:05.724] [BE_Console] [info] BionicEyes: Ready to connect be device server(Data)....
[2023-08-03 11:00:05.726] [BE_Console] [info] BionicEyes: Ready to connect be device server(Control)...
[2023-08-03 11:00:05.726] [BE_Console] [info] BionicEyes: Ready to get remote config file....
[2023-08-03 11:00:05.727] [BEService_Console] [info] BEService: Request to download config file...
[2023-08-03 11:00:05.776] [BEService_Console] [info] BEService: BionicEyes config File have got!
```
成功连接后会有图像的输出，并在终端中显示当前接收到的图像的 ID 信息，但由于是远程传输且代码中设定的设备接收数据方式为逐个帧接收，所以显示的图像会较为卡顿且存在延迟。终端显示结果案例如下：
```
Receive new image(ID is: 46682)
Receive new image(ID is: 46683)
Receive new image(ID is: 46684)
Receive new image(ID is: 46685)
```


## 3 BE_Sample_GrabFakeDataOneByOne
***evo_be*** 为本案例所使用的函数变量所在的命名空间。  
***CBionicEyes*** 类为仿生眼的设备接口类，包含设备使用的所有方法例如控制设备的运动、获取设备数据或者设置设备运动参数。并设定一个类指针 * *device*。*device->create( )* 的功能是创建设备实例以供连接使用。
```C++
	CBionicEyes *device = device->create(enumConnect_Image, enumLocalServer_First, enumDataTransmission_OneByOne);
```
*create( )* 函数的重载函数模板为：
```C++
	static CBionicEyes *create(BE_Connect_Type type,  
                               BE_Connect_DataServerType dataServerType = enumDeviceServer_Only,  
                               BE_Data_TransmissionType dataTransmissionType = enumDataTransmission_ASPAP,  
                               void *logger_ptr = NULL);
```
重载函数中涉及的参数列表如下。

`参数一：BE_Connect_Type 参数为设备的连接类型，下表为该参数可选类型。`
| 	参数 	  	| 	含义 	|
| 	---- 	  	|	 ---- 	|
| enumdisConnect  	|  不与仿生眼设备连接 |
| enumConnect_Control  |  与仿生眼设备连接但只运行控制功能 |
| enumConnect_Image  	|  与仿生眼设备连接但只运行图传功能 |
| enumConnect_ImageControl  |  与仿生眼设备连接并同时运行控制和图传功能 |  

该参数只有仿生眼设备远程传输时才可以使用。

`参数二：BE_Connect_DataServerType 参数为设置设备的数据服务器类型，下表为该参数可选类型。`
| 	参数 	  	| 	含义 	|
| 	---- 	  	|	 ---- 	|
| enumLocalServer_First |  对于远程使用，请首先连接本地服务器 |
| enumDeviceServer_First|  对于远程使用，请首先连接设备服务器 |
| enumDeviceServer_Only |  对于远程使用，仅连接设备服务器  |
| enumLocalServer_Only	 |  对于远程使用，仅连接本地服务器  |  

该参数只有仿生眼设备远程传输时才可以使用。通过网络直接连接仿生眼设备，可选择 *enumDeviceServer_First* 或 *enumDeviceServer _Only*，此时运行 *N* 个独立进程时，这将占用 *N* 个带宽。启动本地服务器才可以选择 *enumLocalServer_First* 或 *enumLocal server_Only*，此时在一台PC中将仅占用一个带宽。

`参数三：BE_Data_TransmissionType 参数为设置数据传输模式类型，下表为该参数可选类型。`
| 	参数 	  	| 	含义 	|
| 	---- 	  	|	 ---- 	|
| enumDataTransmission_ASPAP 	|  速度优先，但是会丢失帧 |
| enumDeviceServer_First	|  质量优先，逐个帧接收   |  

该参数只有仿生眼设备远程传输时才可以使用。选择 *enumDataTransmission_ASPAP* 将获取最新数据但可能会丢失一些数据。选择 *enumDataTransmission_OneByOne* 不会丢失数据，但是可能延迟最新的数据接收时间。如果重新激活与设备传输数据相比太慢，刚开始的数据仍可能丢失。

`参数四默认为空。`

**注意：一个 *create( )* 实例只能连接一个仿生眼设备。如果要与更多设备连接，那么应该调用更多的 *create( )* 函数。**
定义一个参数为 *lastFrameId* 来判断接受的数据是否为真，如果为 -1 表示接收的数据为假。
```C++
	int lastFrameId = -1;
```
接下来的 *while(1)* 循环作用是从设备逐帧读取图像数据，但是没有循环终止条件，需要手动终止。
循环中首先判断是否已经准备好接收数据。只有判断条件为真，才可以接收数据，即将仿生眼设备的数据进行传输。
```C++
	if (device->isBeDataReady())
```
***BE_GeneralData*** 是定义在头文件 ***evo_be.h*** 的结构体。该结构体中包含了所有仿生眼基础同步数据以及一些读写的内联函数。
`结构体中的数据成员如下表所示。`
|类型	| 	成员 	| 	含义 	|
|----	| 	---- 	  |	 ---- 	|
|cv::Mat	| image[*MAXCAMERASInDEVICE*] |  Mat类数组，用于存放仿生眼设备获取的图像  |
|uint32_t	| 	id		|  		唯一全局id 			|
|uint32_t	|	timeStamp 	|  		时间戳（100 us） |
|BE_IpInfo	|be_device_info 	|  		仿生眼设备信息，详细请参见结构体 *BE_IpInfo*  |
|bool		| imageFlag[*MAXCAMERASInDEVICE*] |  	仿生眼图像存在标志  |
|BE_ImageInfo	|imageInfo[*MAXCAMERASInDEVICE*] |  仿生眼图像信息，详细请参见结构体 *BE_ImageInfo*|
|float		|	motorData[6] 	|  	电机编码器数据 		|
|float		|imuData[*MAXIMUBUFFER*][4]	| 	 IMU数据	|
|double	| 	gpsData[4] 	|  GPS数据（经度、纬度、高度、GMT） 	|
|bool		|	isMovingFastly	|  获取这些数据时眼睛是否快速移动  	|

`表格中定义的常量 *MAXCAMERASInDEVICE* = 4，*MAXIMUBUFFER* = 16。`  
**注：Mat 类的数组 *image* 定义为包含四个元素的数组，在连接双目仿生眼时，从仿生眼设备中获取的两张图像会左右拼接为一张图像并存储在 *image[2]* 中。 如果连接的是仿生鹰眼，则从设备中获取的四个图像会分别存储在*image* 数组的四个元素中。同理，*imageFlag* 和 *imageInfo* 的存储逻辑相似。**  
根据该结构体定义一个结构体变量 *data*。并使用 *CBionicEyes* 类的 *getBeData( )* 函数尝试从仿生眼设备获取数据。
```C++
	BE_GeneralData data = device->getBeData();
```
然后对接收的数据进行判断，当前帧接收的图像 *ID* 是否与上一帧的图像 *ID* 相同。如果相同则说明仿生眼设备没有接收到下一帧图像，则等待一段时间后再获取数据进行判断。如果不相同，则说明仿生眼设备接收到下一帧图像。然后进行下一步。
```C++
	if(data.id != lastFrameId)
```
向仿生眼服务器发送触发信号以发送下一个数据。
**注意 *triggerDataTransmission( )* 函数只能用于远程连接，即 *create( )* 函数的参数 *dataTransmissionType* 为 *enumDataTransmission_OneByOne* 时，才可以使用。然后打印新接受到的图片的 *ID* 信息。**
```C++
	device->triggerDataTransmission();
	cout<<"Receive new image(ID is: "<<data.id<<")"<<endl;
```
最后判断从仿生眼设备左右两个相机中获得的图像是否为空，如果不为空，则将图像进行可视化。前文中，从仿生眼设备中获取的两张图像会左右拼接为一张图像并存储在 *image[2]* 中，因此从 *data.image[enumBoth]* 中读取设备的图像。
```C++
    if(!data.image[enumBoth].empty())
    {
        cv::Mat temp;
        cv::resize(data.image[enumBoth], temp, cv::Size(960, 540));
        cv::imshow("BE_Image", temp);
        cv::waitKey(1);
    }
```
其中数据 *data* 的图像 *image* 对应的相机索引类型参数如下表。
| 参数 | 含义 	|
| ---- | ---- 	|
| enumLeft |  左相机 |
| enumRight|  右相机   |
| enumBoth |  双相机   |

再将上一帧的数据 *ID* 替换为当前帧的数据 *ID*，进行下一帧数据的输入和判断。
```
    lastFrameId = data.id;
```
实例结束。
