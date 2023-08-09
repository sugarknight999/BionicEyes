该文档里面所包含的案例仅供参考
# 仿生眼三轴移动程序（ThreeAxis）使用简介
## 1 获取设备列表程序功能概括
仿生眼系列设备的标定数据储存在配置文件中，该例程展示了如何 ***控制仿生眼设备的三轴移动***。

## 2 获取设备列表程序使用方法
根据 *path_to_workspace/BionicEyes/README.md* 中的流程成功编译仿生眼 *workspace* 后，所有 *sample* 程序的可执行文件保存在 *path_to_workspace/BionicEyes/bin/* 目录中，根据 *ubuntu* 操作系统的版本，程序的可执行文件被命名为：

>evo_be_Sample_ThreeAxis_2004  
>evo_be_Sample_ThreeAxis_1804  
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

**注：直连PC除运行上述设备程序外，还需要在另一个终端运行 *evo_be_Device_Service_\** 用于启动仿生眼服务后，才能供远端PC检测到仿生眼系列设备的存在。**  
***\*代表对应系统平台，例如1804代表ubuntu18.04***

### 2.2 获取设备列表程序
直连PC开启仿生眼服务后，***与直连PC处于同一个局域网下的***远程 *PC* 设备即可检测到仿生眼存在并进行连接，在远程 *PC* 设备中进行如下操作：
#### 2.2.1 进入bin目录
 `cd  path_to_workspace/BionicEyes/bin/`
 
#### 2.2.2 运行可执行文件
 `./evo_be_Sample_ThreeAxis_2004`
 

## 3 BE_Sample_ThreeAxis
***evo_be*** 为本案例所使用的函数变量所在的命名空间。
定义了一个结构体 *BeCmdOptions*，其中包含了两个参数，分别是 *bool* 类型的 *openNetworkService*，用于开启网络服务了；*int* 类型的 *dataDefaultFrameRate*，用于设置数据默认帧获取速率。
```C++
	struct BeCmdOptions
	{
	    bool openNetworkService;
	    int dataDefaultFrameRate;/
	};
```
然后回归主函数。
首先是按照已经定义的结构体，定义一个结构体变量 *opts*，并设定该变量里对应的 *dataDefaultFrameRate* 为 10，*openNetworkService* 为 *false*。具体如何设置见下文。
```C++
	BeCmdOptions opts;  
	opts.dataDefaultFrameRate = 100;     // max 100fps  
	opts.openNetworkService = false;    // whether use remote control function
```
***CBionicEyes*** 类为仿生眼的设备接口类，包含设备使用的所有方法例如控制设备的运动、获取设备数据或者设置设备运动参数。并设定一个类指针 * *device*。*device->create( )* 的功能是创建设备实例以供连接使用。
```C++
	CBionicEyes *device = CBionicEyes::create(opts.openNetworkService, opts.dataDefaultFrameRate);
```
*create( )* 函数的 ***重载函数*** 模板为：
```C++
	static CBionicEyes *create(bool openNetworkService = true, int dataDefaultFrameRate = 25, void *logger_ptr = NULL);
```
重载函数对应的主要两个参数：
参数一：*openNetworkService* 设置为 *true* 以使用网络服务并支持远程连接，或设置为 *false* 则不使用网络服务可以减少 *CPU* 使用。
参数二：*dataDefaultFrameRate* 设置默认帧（包括图像、电机编码器、IMU数据等）的获取率。可以更改，且最大可接受值为200。
设置 *openNetworkService* 为 *true* 时，仍需要启动 *evo_be_Device_Services_* *程序来支持远程用户

**注意：一个 *create( )* 实例只能连接一个仿生眼设备。如果要与更多设备连接，那么应该调用更多的 *create( )* 函数。**

关闭设备的 *VOR* 功能。
```C++
	device->onoff_VOR(false);
```
 *VOR* 为仿生眼设备的左右眼 ***协同对齐*** 功能。关闭之后才可以让左右两个设备的其中一个单独运动
设置一些设备中电机的运动参数
```C++
	float ampli = 38;  
	int MaxStep = 380;  
	int countID = MaxStep / 2;  
	bool moveFlag[6]={0,0,0,1,1,1};  // 表示左右一共6个电动机中只有左侧的3个电动机可以进行运动
	float moveAngle[6] = {0};  
```
*while(1)* 循环的功能就是设置电动机的其中一侧的电动机运动到指定的 ***绝对位置***。
*if* 语句判断条件里面的 *isBeDataReady( )* 函数的类型为 *bool* 型，作用是判断设备是否准备好接收数据。且在调用 *getBeData( )* 函数获取数据之前，应确保此函数返回值为 *true*。
```C++
	if(device->isBeDataReady())
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

根据该结构体定义一个结构体变量 *dataT*。并使用 *CBionicEyes* 类的 *getBeData( )* 函数尝试从仿生眼设备获取数据。
如果满足数据获取的条件，进行数据的获取。定义一个包含所有仿生眼基础同步数据的结构体 *dataT*。并使用 *CBionicEyes* 类的 *getBeData( )* 函数尝试从仿生眼设备获取数据。然后从数据 *dataT* 中读取电机编码器数据和 *IMU* 数据。
```C++  
	BE_GeneralData dataT;  
	device->getBeData(dataT);  
	// 将编码器数据显示出来
	cout<<dataT.motorData[3]<<"  "<<dataT.motorData[4]<<"  "<<dataT.motorData[5]<<"  "
	<<dataT.imuData[enumQuat][0]<<"  "<<dataT.imuData[enumQuat][1]<<"  "<<dataT.imuData[enumQuat][2]<<"  "<<dataT.imuData[enumQuat][3]<<"  "
	<<dataT.imuData[enumMag][0]<<"  "<<dataT.imuData[enumMag][1]<<"  "<<dataT.imuData[enumMag][2]<<"  "<<dataT.imuData[enumMag][3]<<endl;  
```
以代码中的 *motorData[3]、motorData[4]、motorData[5]* 为例指代的是仿生眼设备左眼部分的电机编码器数据。而 *IMU* 数据的参数及其所指代的意义，如下表所示。
| 	参数 	  | 	含义 	|
| 	---- 	  |	 ---- 	|
| enumAcc |  加速度  |
| enumGyo  |  陀螺仪  |
| enumMag  |  磁强计  |
| enumEuler  |  Euler角度  |
| enumQuat  |  四元数   |
| enumLaser   |  激光数据  |

然后按照之前设定的设备中电机的运动参数，计算左侧的三个电动机运动的角度。
```C++
	float amp_now = ampli * std::sin(2.f * M_PI / (float)MaxStep * (float)(countID++ % MaxStep));  
	float moveAngle[6] = {0, 0, 0, amp_now, amp_now, amp_now};  
```
最后设置仿生眼电动机的 ***绝对位置***。前面设置的 *bool* 型参数 *moveFlag* 已经确定了要选择进行运动的电动机，经过上一步的计算的到了 *float* 型参数 *moveAngle* 的结果，即电动机运动到的 ***绝对位置***。然后通过 *setAbsolutePosition( )* 函数来将设定仿生眼设备上的电动机运动到的 ***绝对位置***。
```C++
	device->setAbsolutePosition(moveFlag, moveAngle);  
```
实例结束。
