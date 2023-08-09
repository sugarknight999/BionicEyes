该文档里面所包含的案例仅供参考
# 远程控制程序（Remote control）使用简介
## 1 远程控制程序功能概括
仿生眼系列设备的标定数据储存在配置文件中，该例程展示了如何 ***获得电动机初始位置*** 、***设定仿生眼设备的绝对位置和相对位置***。

## 2 远程控制程序使用方法
根据 *path_to_workspace/BionicEyes/README.md* 中的流程成功编译仿生眼 *workspace* 后，所有 *sample* 程序的可执行文件保存在 *path_to_workspace/BionicEyes/bin/* 目录中，根据 *ubuntu* 操作系统的版本，程序的可执行文件被命名为：

>evo_be_Sample_Control_Remote_2004
>evo_be_Sample_Control_Remote_1804
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

### 2.2 运行远程程序
直连PC开启仿生眼服务后，***与直连PC处于同一个局域网下的***远程 *PC* 设备即可检测到仿生眼存在并进行连接，在远程 *PC* 设备中进行如下操作：
#### 2.2.1 进入bin目录
 `cd  path_to_workspace/BionicEyes/bin/`
#### 2.2.2 运行可执行文件
 `./evo_be_Sample_Control_Remote_2004` 

运行可执行文件指令后，会显示左右电动机的序号以及的 *Pitch、Roll、Yaw* 的向上和向下限制位置，并且当设备回到初始化位置时输出 *go to init positions(0, 0, 0, 0, 0, 0)!* ，结果如下所示。
例如：
>Motor 0(up, down)(34.093128, -35.662502)  
>Motor 1(up, down)(29.773127, -30.566252)  
>Motor 2(up, down)(34.756878, -35.921253)  
>Motor 3(up, down)(35.055000, -35.893127)  
>Motor 4(up, down)(29.598751, -30.639376)  
>Motor 5(up, down)(33.114376, -34.869377)  
>[2023-08-02 10:41:12.840] [BE_Console] [info] BionicEyes: <<goInitPosition>> Motor(No.6) go to init positions(0, 0, 0, 0, 0, 0)!  
>[2023-08-02 10:41:18.841] [BE_Console] [info] BionicEyes: <<goInitPosition>> Motor(No.6) go to init positions(0, 0, 0, 0, 0, 0)!  
>[2023-08-02 10:41:24.841] [BE_Console] [info] BionicEyes: <<goInitPosition>> Motor(No.6) go to init positions(0, 0, 0, 0, 0, 0)!  

## 3 远程控制程序详解（BE_Sample_Control_Remote）
***evo_be*** 为本案例所使用的函数变量所在的命名空间。
***CBionicEyes*** 类为仿生眼的设备接口类，包含设备使用的所有方法例如控制设备的运动、获取设备数据或者设置设备运动参数。并设定一个类指针 *\*device*。*device->create( )* 的功能是创建设备实例以供连接使用。
```C++
	CBionicEyes *device = device->create(enumConnect_Control);
```
*create( )* 函数的 ***重载函数*** 模板为：
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

分别关闭 *VOR* 和 *SV* 功能。
```C++
	device->onoff_VOR(false);
	device->onoff_SV(false);
```
 *VOR* 和 *SV* 分别为仿生眼设备的左右眼 ***协同对齐*** 和 ***稳定防抖*** 功能。然后等待下一步的操作。
```C++
	msleep(1000);
```
分别定义三个数组 *up_bd[ ]、low_bd[ ]、init_pos[ ]*。分别为电动机三轴向上运动限制位置、向下运动限制位置和点击的初始位姿。
获得选择的电动机初始位置值并存于 *init_pos[ ]* 数组中。
```C++
	device->getInitPosition(enumAllMotor, init_pos);
```
其中函数的第一个参数 *MotorType* 代表的是想要获取初始位置值的电动机。下表为该参数可选类型。
| 	参数 	  | 	含义 	|
| 	---- 	  |	 ---- 	|
| enumRightPitch |  右侧 Pitch 轴电动机  |
| enumRightRoll  |  右侧 Roll 轴电动机   |
| enumRightYaw   |  右侧 Yaw 轴电动机    |
| enumLeftPitch  |  左侧 Pitch 轴电动机  |
| enumLeftRoll   |  左侧 Roll 轴电动机   |
| enumLeftYaw    |  左侧 Yaw 轴电动机    |
| enumAllMotor   |  全部的电动机  |
| enumNoMotor    |  无电动机 	|

在 *for* 循环中获得左右电动机的 ***Pitch、Roll、Yaw*** 向上和向下限制位置并存于 *up_bd[ ]、low_bd[ ]* 数组中，然后进行可视化。
```C++
	device->getUpDownLimit((MotorType)i, up_bd[i], low_bd[i]);
```
接下来是控制仿生眼电动机进行运动的环节，分别为通过设置仿生眼的 ***绝对位置*** 进行运动和设置仿生眼的 ***相对位置*** 进行运动。

设置选定的电动机进入初始位置，然后等待进一步操作。
```C++
	device->goInitPosition(enumAllMotor);
```
设置仿生眼电动机的 ***绝对位置*** 。设置 *bool* 型参数 *motorWorkFlag* 来选择想要进行运动的电动机，*float* 型参数 *motor_value* 来表示电动机运动到的 ***绝对位置***。然后通过 *setAbsolutePosition( )* 函数来将设定仿生眼设备的 ***绝对位置***。
```C++
	device->setAbsolutePosition(motorWorkFlag, motor_value);
```
再一次设置选定的电动机进入初始位置，然后等待进一步操作。
与设置绝对位置参数相似的步骤，再设置 *bool* 型参数 *motorWorkFlag2* 和 *float* 型参数 *motor_value2* ，其中 *motor_value2* 表示的仿生眼运动的 ***相对位置***。然后通过 *setRelativePosition( )* 函数来将设定仿生眼设备的 ***相对位置***。
```C++
	device->setRelativePosition(motorWorkFlag2, motor_value2);
```
最后将仿生眼归位到初始位置。
实例结束。
