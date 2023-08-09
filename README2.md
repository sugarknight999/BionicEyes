该文档里面所包含的案例仅供参考
# 获取设备列表程序（Get Device List）使用简介
## 1 获取设备列表程序功能概括 
仿生眼系列设备的标定数据储存在配置文件中，该例程展示了如何 ***获取设备列表信息*** 。

## 2 获取设备列表程序使用方法
根据 *path_to_workspace/BionicEyes/README.md* 中的流程成功编译仿生眼 *workspace* 后，所有 *sample* 程序的可执行文件保存在 *path_to_workspace/BionicEyes/bin/* 目录中，根据 *ubuntu* 操作系统的版本，程序的可执行文件被命名为：

>evo_be_Sample_GetDeviceList_2004  
>evo_be_Sample_GetDeviceList_1804  
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
 `./evo_be_Sample_GetDeviceList_2004`
 
运行可执行文件指令后，会显示左右电动机的序号以及的 *Pitch、Roll、Yaw* 的向上和向下限制位置，并且当设备回到初始化位置时输出 *go to init positions(0, 0, 0, 0, 0, 0)!* ，结果如下所示。
例如：
>BionicEye device detected(count == 1)!!!  
>0: BinoSense_BE150A28210000 192.168.31.88  

## 3 获取设备列表程序详解（BE_Sample_GetDeviceList）
***evo_be*** 为本案例所使用的函数变量所在的命名空间。
***CBE_Communication_Ethernet_Special*** 设定是一个用于网络通信的接口 ***API*** 的类，定义了一个类指针 *device_ipAddr_Service*，使用 *create( )* 创建一个设备服务器来设定以太网连接类型。
```C++
	CBE_Communication_Ethernet_Special *device_ipAddr_Service;
	device_ipAddr_Service = device_ipAddr_Service->create(enumEth_Req);
```
*create( )* 函数的函数模板为：
```C++
	static CBE_Communication_Ethernet_Special *create(Ethernet_Connect_Type connectType, void *logger_ptr = NULL);
```
`Ethernet_Connect_Type 参数为以太网连接类型，下表为该参数可选类型。`
| 参数 	  | 	含义 	|
| ---- 	  |	 ---- 	|
| enumEth_Pair  |  P2P通信    |
| enumEth_Pub   |  发布模式   |
| enumEth_Sub   |  订阅模式   |
| enumEth_Req   |  请求模式   |
| enumEth_Rep   |  应答模式   |
| enumEth_Daler |  经销商模式（多需求）|
| enumEth_Router|  路由器模式（多应答）|
| enumEth_Pull  |  牵引模式    |
| enumEth_Push  |  推送模式    |

然后定义一个 *vector* 用来存放仿生眼 *IP* 信息，其中 *BE_IpInfo* 表示的是存放仿生眼 *IP* 信息的结构体。
```C++
	std::vector<BE_IpInfo> deviceList;
```
发送对设备 *IP* 地址的请求指令，直到获得了设备的 *IP* 地址信息。然后打印“检测到BionicEye设备”。
```C++
	while (!device_ipAddr_Service->requestBeIpAddress(deviceList)) {
   		printf("No BionicEye device detected. Retrying.....\n");
    	msleep(2000);
	}
	printf("BionicEye device detected(count == %d)!!!\n", deviceList.size());
```
*for* 循环的作用是输出设备的名字和对应的 *IP* 地址，其中 *deviceList.size( )* 函数表示在容器 *deviceList* 中存放了多少个类型为 *BE_IpInfo* 的结构体。
```C++
	for (int i = 0; i < deviceList.size(); i++)
    	{
        	printf("%d: %s %s\n", i, deviceList[i].deviceName, deviceList[i].ipAddrStr);
    	}
```
实例结束。
