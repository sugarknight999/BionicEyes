该文档里面所包含的案例仅供参考
# 仿生眼协同对齐和稳定防抖功能检测程序（VOR & SV）使用简介
## 1 获取设备列表程序功能概括
仿生眼系列设备的标定数据储存在配置文件中，该例程展示了如何 ***通过键盘控制仿生眼设备的协同对齐和稳定防抖功能*** 和 ***仿生眼设备的位置初始化***。*VOR* 和 *SV* 分别为仿生眼设备的左右眼 ***协同对齐*** 和 ***稳定防抖*** 功能。

## 2 获取设备列表程序使用方法
根据 *path_to_workspace/BionicEyes/README.md* 中的流程成功编译仿生眼 *workspace* 后，所有 *sample* 程序的可执行文件保存在 *path_to_workspace/BionicEyes/bin/* 目录中，根据 *ubuntu* 操作系统的版本，程序的可执行文件被命名为：

>evo_be_Sample_SV_VOR_2004  
>evo_be_Sample_SV_VOR_1804  
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
直连 PC 开启仿生眼服务后，***与直连 PC 处于同一个局域网下的***远程 *PC* 设备即可检测到仿生眼存在并进行连接，在远程 *PC* 设备中进行如下操作：
#### 2.2.1 进入bin目录
 `cd  path_to_workspace/BionicEyes/bin/`
 
#### 2.2.2 运行可执行文件
 `./evo_be_Sample_SV_VOR_2004`
 
运行可执行文件指令后，首先会显示电机进入初始化位置 *go to init positions(0, 0, 0, 0, 0, 0)* ，然后会打印代码中 *cout* 显示输出的部分。结果如下所示。
例如：
> [2023-08-02 13:29:19.680] [BE_Console] [info] BionicEyes: <<goInitPosition>> Motor(No.6) go to init positions(0, 0, 0, 0, 0, 0)!  
>Bionic Eyes SV_VOR Sample:  
>Press S to switch ON/OFF of SV  
>Press V to Switch ON/OFF of VOR  
>Press I to move to init position

然后可以通过键盘的 *S*、*V* 和 *I* 键分别控制仿生眼设备 *SV* 、*VOR* 功能的开关和位置的初始化。


## 3 BE_Sample_SV_VOR
***evo_be*** 为本案例所使用的函数变量所在的命名空间。  
首先定义一个静态全局变量的结构体指针 * *be_keypress_dpy*，其中 *Display* 是通过 *typedef struct _XDisplay Display* 定义的结构体 *_XDisplay* 的别名。*XOpenDisplay* 函数是 *Xlib* 库（*X*库）中的一个函数，它允许应用程序与*X*服务器建立连接，并返回一个指向 *Display* 结构的指针。
```C++
	static Display *be_keypress_dpy = XOpenDisplay(NULL);
```
在 *bool* 类型的 *KEY_DOWN( )* 函数中检测键盘的按压。其中函数的形参 *keyValue_Linux* 可以在路径*/usr/include/X11/keysymdef.h* 下找到。例如：*XK_Shift_L*，*XK_A*，*XK_a*，*XK_0*。不用详细探究该函数的原理，只需要知道函数的功能即可。
```C+
	static bool KEY_DOWN(KeySym keyValue_Linux)
	{
    		char keys_return[32];
    		XQueryKeymap(be_keypress_dpy, keys_return);
    		KeyCode kc2 = XKeysymToKeycode(be_keypress_dpy, keyValue_Linux);
    		return !!(keys_return[kc2 >> 3] & (1 << (kc2 & 7)));
	}
```
然后回归主函数。
***CBionicEyes*** 类为仿生眼的设备接口类，包含设备使用的所有方法例如控制设备的运动、获取设备数据或者设置设备运动参数。并设定一个类指针 * *device*。*device->create( )* 的功能是创建设备实例以供连接使用。
```C++
	CBionicEyes *device = device->create(enumConnect_ImageControl);
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
| 		---- 	  	|	 ---- 	|
| enumdisConnect  			|  不与仿生眼设备连接 |
| enumConnect_Control  	|  与仿生眼设备连接但只运行控制功能 |
| enumConnect_Image  	|  与仿生眼设备连接但只运行图传功能 |
| enumConnect_ImageControl  |  与仿生眼设备连接并同时运行控制和图传功能 |

该参数只有仿生眼设备远程传输时才可以使用。

`参数二：BE_Connect_DataServerType 参数为设置设备的数据服务器类型，下表为该参数可选类型。`
| 	参数 	  	| 	含义 	|
| 	---- 	  	|	 ---- 	|
| enumLocalServer_First 		|  对于远程使用，请首先连接本地服务器 |
| enumDeviceServer_First	|  对于远程使用，请首先连接设备服务器 |
| enumDeviceServer_Only 	|  对于远程使用，仅连接设备服务器  |
| enumLocalServer_Only		|  对于远程使用，仅连接本地服务器  |

该参数只有仿生眼设备远程传输时才可以使用。通过网络直接连接仿生眼设备，可选择 *enumDeviceServer_First* 或 *enumDeviceServer _Only*，此时运行 *N* 个独立进程时，这将占用 *N* 个带宽。启动本地服务器才可以选择 *enumLocalServer_First* 或 *enumLocal server_Only*，此时在一台PC中将仅占用一个带宽。

`参数三：BE_Data_TransmissionType 参数为设置数据传输模式类型，下表为该参数可选类型。`
| 	参数 	  	| 	含义 	|
| 	---- 	  		|	 ---- 	|
| enumDataTransmission_ASPAP 	|  速度优先，但是会丢失帧 |
| enumDeviceServer_First				|  质量优先，逐个帧接收   |

该参数只有仿生眼设备远程传输时才可以使用。选择 *enumDataTransmission_ASPAP* 将获取最新数据但可能会丢失一些数据。选择 *enumDataTransmission_OneByOne* 不会丢失数据，但是可能延迟最新的数据接收时间。如果重新激活与设备传输数据相比太慢，刚开始的数据仍可能丢失。

`参数四默认为空。`

**注意：一个 *create( )* 实例只能连接一个仿生眼设备。如果要与更多设备连接，那么应该调用更多的 *create( )* 函数。**  
然后进行设备的初始化和一些基本功能的设置。
分别关闭 *VOR* 和 *SV* 功能。并让仿生眼设备运动到初始化位置。等待下一步。
```C++
	device->onoff_VOR(false);
	device->goInitPosition(enumAllMotor);  
	device->onoff_SV(false);
```
 *goInitPosition( )* 函数的参数 *MotorType* 代表的是想要获取初始位置值的电动机。下表为该参数可选类型。
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

等设备运动到初始化位置后，重新打开仿生眼设备的左右眼 ***协同对齐*** 功能。
```C++
	msleep(500);  
	device->onoff_VOR(true);
```
设置 *bool* 类型的参数 *flag_VOR* 和 *flag_SV* 作为后面控制设备左右眼协同对齐和稳定防抖功能开和关的标志。
```C++
	bool flag_SV = false, flag_VOR = true;
```
*setImageResolution_Transfer( )* 函数设置网络传输图像分辨率（*SideBySide*大小！），如果网络不理想，可以对传输图像进行下采样。
```C++
	device->setImageResolution_Transfer(cv::Size(1920,540));
```
*setImageResolution( )* 函数设置图像分辨率（*SideBySide*大小！）（*BE_ImageGeneralInfo.image*，仅限远程使用）。
```C++
	device->setImageResolution(cv::Size(1920,540));
```
*setImageColor_Transfer( ) *函数设置网络传输图像的颜色。这不会影响返回图像颜色类型的函数“getBeData”。
```C++
	device->setImageColor_Transfer(enumColor);
```
*setImageCompressionQuality( )* 函数设置图像无损压缩或有损压缩（***仅适用于网络传输***）。参数为 *imgQual* 表示压缩时的损失。无损失压缩：*imgQual=100*；有损压缩：*imgQual*（*0*=最差，*99*=最佳）。
```C++
	device->setImageCompressionQuality(95);
```
setDataRate_Transfer() 函数设置网络传输帧速率。
```C++
	device->setDataRate_Transfer(25)
```
输出显示的内容，说明按下 *S* 键，控制 *SV* 协同的开和关。按下 *V* 键，控制 *VOR* 稳定防抖的开和关。按下 *I* 键，控制电动机位置的初始化。
```C++
	cout<< "Bionic Eyes SV_VOR Sample:" <<endl
	<<"    Press S to switch ON/OFF of SV"<<endl  
	<<"    Press V to Switch ON/OFF of VOR"<<endl  
	<<"    Press I to move to init position"<<endl;
```
下面的 *while(1)* 循环中表达的就是上面的显示内容的功能。*XK_S*、*XK_V*、*XK_I* 所在的 *if* 语句的功能类似，只对 *KEY_DOWN(XK_S)* 所在段进行详细说明。
首先 *if* 的判断条件中先判断是否在键盘按压 *S* 键，如果是 *S* 键，则将 *flag_SV* 置反，然后调用 *device->onoff_SV( )* 控制协同对齐功能的开和关。
```C++
if(KEY_DOWN(XK_S))  
{  
    flag_SV = !flag_SV;  
    device->onoff_SV(flag_SV);  
}
```  
*XK_V*、*XK_I* 基本同理，但是判断键盘按压 *I* 键的模块，是 *goInitPosition( )* 函数的功能是选择电动机并控制对应的电动机进行位置初始化。
函数的参数 *MotorType* 代表的是想要获取初始位置值的电动机。下表为该参数可选类型。
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

*if* 语句判断条件里面的 *isBeDataReady( )* 函数的类型为 *bool* 型，作用是判断设备是否准备好接收数据。且在调用 *getBeData( )* 函数获取数据之前，应确保此函数返回值为 *True*。
```C++
	device->isBeDataReady()
```
如果满足数据获取的条件，进行图像的获取并显示。定义一个包含所有仿生眼基础同步数据的结构体 *data*。并使用 *CBionicEyes* 类的 *getBeData( )* 函数尝试从仿生眼设备获取数据。然后使用 *OpenCV* 的函数 *imshow* 进行图像的显示。
```C++
	if (device->isBeDataReady())  
	{  
	    BE_GeneralData data = device->getBeData();  
	    //cv::Mat temp;  
	    //cv::resize(data.image, temp, cv::Size(1920, 540)); 可选择是否调节图像的尺寸   
	    cv::imshow("VOR_Sample", data.image[enumBoth]);  
	}
```
实例结束。
