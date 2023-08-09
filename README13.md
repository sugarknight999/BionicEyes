该文档里面所包含的案例仅供参考
# 设备声音处理程序（Voice Process）使用简介
## 1 获取设备列表程序功能概括
仿生眼系列设备的标定数据储存在配置文件中，该例程展示了设备如何 ***对外部声源进行处理*** 和 ***发出声音***。

## 2 获取设备列表程序使用方法
根据 *path_to_workspace/B0ionicEyes/README.md* 中的流程成功编译仿生眼 *workspace* 后，所有 *sample* 程序的可执行文件保存在 *path_to_workspace/BionicEyes/bin/* 目录中，根据 *ubuntu* 操作系统的版本，程序的可执行文件被命名为：

>evo_be_Sample_VoiceProcess_2004  
>evo_be_Sample_VoiceProcess_1804  
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

### 2.2 获取设备列表程序
直连PC开启仿生眼服务后，***与直连 PC 处于同一个局域网下的***远程 *PC* 设备即可检测到仿生眼存在并进行连接，在远程 *PC* 设备中进行如下操作：
#### 2.2.1 进入bin目录
 `cd  path_to_workspace/BionicEyes/bin/`
 
#### 2.2.2 运行可执行文件
 `./evo_be_Sample_VoiceProcess_2004`
 
运行可执行文件指令后，什么都不会显示。由于仿生眼设备不具备声卡部件，因此单独的仿生眼设备无法接收及发出声音，但是如果外接声卡等设备则可以实现代码中对声音的处理和发出声音的功能。


## 3 BE_Sample_VoiceProcess_Remote
***evo_be*** 为本案例所使用的函数变量所在的命名空间。  
***CBionicEyes*** 类为仿生眼的设备接口类，包含设备使用的所有方法例如控制设备的运动、获取设备数据或者设置设备运动参数。并设定一个类指针 *device*。*device->create( )* 的功能是创建设备实例以供连接使用。
```C++
	CBionicEyes *device = device->create(enumdisConnect);
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
然后通过 *device->getBeDevice_Ip_str( )* 函数获取连接的仿生眼设备 *IP* 地址（字符串类型，如192.168.1.1）并赋值给定义的字符串 *deviceIP*。
```C++
	string deviceIP = device->getBeDevice_Ip_str();
```
***BE_Voice*** 类为仿生眼设备针对声音处理的类，包含例如获取声音的来源方向、获取声音的结果、支持设备发出声音等函数。并设定一个类指针 * *voice*。*voice->create( )* 的功能是创建设备实例以供使用。
```C++
	BE_Voice *voice = voice->create(deviceIP.c_str(), enumRemote);  
```
*create( )* 函数的函数模板为：
```C++
	static BE_Voice* create(const char *str_IP, BE_RunSite_Type type = enumRemote);
```
其中参数 *str_IP* 指的就是上面通过 *getBeDevice_Ip_str( )* 获取的连接的仿生眼设备 *IP* 地址。参数 *BE_RunSite_Type* 表示运行站点的类型，参数列表如下。
| 	参数 	  	| 	含义 	|
| 	---- 	  	|	 ---- 	|
| enumLocal	|  本地站点 |
| enumRemote |  远程连接   |

定义了三个变量 *angle*、*res*、*count* 分别对应下面 *while(1)* 循环中的获取声源角度、获取声音的结果以及让设备发生三个部分。
```C++
	int angle = 0;  
	string res;  
	int count = 0;
```
*getVoiceResult( )* 函数可以获取声音的结果。
```C++
	if(voice->getVoiceResult(res))
	{
		printf("%s\n", res.c_str());
	}
```
*getVoiceAngle( )* 函数可以获取声音来源的角度，在0到359度之间。
```C++
	if(voice->getVoiceAngle(angle))
	{
		printf("%d\n", angle);
	}
```
*sendTTSMessage( )* 函数可以支持设备说一些语句，但是仅限中文和英文。例如“你好”、“hello”。
```C++
	if((count++)%500  == 0)  
	{
		voice->sendTTSMessage("我是小白");  
	}
```
实例结束。
