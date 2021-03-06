# Core
FlowerIp定义了主机协议地址信息（HPAI）结构，该结构包含Ip地址和端口号。HPAI应该被数据请求，然后发送一个flowerIp帧到另一个设备。flowerIp规范使用术语“flowerip端点”作为设备对外的逻辑接口   
每个flowerip设备应支持唯一的设备相关的，双向的无连接的端点，用于主机协议请求发现服务。Flowerip应支持至少一个双向的无连接的端点用于控制和至少一个双向的，面向链接的端点用于服务相关的数据传送。   

控制端点在flowerip设备中应该有唯一的地址来找到一个入口，该设备有能力提供至少一个flowerip服务类型    

这个入口叫做服务容器，该容器可能链接到一个flower网络。如果flowerip服务器设备支持更多的flower链接，则每个网络都有不同的控制端点   

这些flowerip端点应该展现一个逻辑接口给flowerip通讯的设备。


# 发现和自描述
## 简介   
在一些支持热插拔或ip地址会动态变化的场合，非标准或手动的建立连接的方式和不合适的。自描述机制是非常有必要的

## 发现
一个flowerip服务器应该应具备发现功能，如果可行，请尽可能使用自动发现功能来代替手动配    

发现的过程应把SEARCH_REQUEST数据包，通过发现功能的端点的多播来发送，该数据包应该包含flowerip客户端发现端点的HPAI。HPAI可能包含一个单播IP地址来接收不同flowerip服务器之间的点对点操作。典型地，它应包含flowerip系统安装多播地址来保证来自各个不同支线的flowerip服务器的接收。    

在发送请求后，客户端应等待来自服务器的SEARCH_RESPONSE帧SEARCH_TIMEOUT时间。过了这个时间后，任何接收到的响应帧都应该被客户端忽略，直到下一个发现请求。来自其他客户端的搜索请求帧应被客户端忽略。   

任何服务器接收到搜索请求帧后，应立即响应一个搜索响应帧到给定的HPAI中，通过自身的发现端点。该响应帧应只包含服务器控制端点的HPAI用于后续所有通讯   

任何服务器应支持发现功能，通过处理搜索请求和发送正确的响应。服务器可以支持连接多于1个的支线，这样的话每个支线对应的容器的控制端点应发送一个搜索响应。即使它一次只能支持一个数据连接。    

## 自描述   
典型地，在发现一个服务器之后，客户端会发送一个描述请求，通过一个点对点连接的单播到所有服务器的控制端点。这需要每个flowerip设备支持描述请求。所以在客户端与服务器通讯之前，应当检查服务器是否支持自发现服务。   

如果一个服务器收到一个合法的描述请求，服务器将回复一个描述响应帧来提供协议版本信息，自身的能力，状态信息和可选的服务器连接的名称。作为一个服务器，可能支持连接到多于1个支线，它应支持对每个支线都有响应。

# 通讯通道
## 简介    
一个通讯通道应该是在服务器和客户端之间的数据端点连接。数据端点连接应当被建立用于需要点对点通讯的服务，如flowerip隧道或设备管理。任何在客户端和服务器之间的点对点连接都应通过客户端来初始化。任何服务器应支持至少一个客户端同时连接。它可能支持同时多于1个客户端连接，但必须保证现有的连接不会被新连接所影响。例如一个服务器应不接受隧道连接在相同的物理接入到支线通过两种不同的模式（正常模式，监视模式）

## 建立连接
为了建立客户端与服务器之间的连接，客户端引导发送一个连接请求帧到服务器的控制端点。该请求应提供连接类型的信息（数据隧道或远程登录）、通用的或指定的连接类型选项（如数据链路层或总线监视模式）和数据端点的HPAI（客户端使用该HPAI来连接通讯通道）。   

在发送一个指定类型的连接请求之前（带有指定选项），客户端应检查其自描述

服务器应随后发送一个连接响应帧到客户端。如果服务器接收请求，则连接响应帧应额外包含一个标识符和数据端点的HPAI    

在发送连接请求之后，客户端应等待10s，让服务器有响应。在10s后的收到的帧将被忽略，直至下一次的连接请求。    

目前的协议规范假设一个连接不能被多个客户端使用。所以一个服务器不应该接收多个连接请求。

## 连接头
### 描述    
在一个已建立的通讯通道上的flowerip帧的主体的开头应该包含额外的数据连接的通用信息。这些数据的数量和信息的类型应在连接阶段通过多种选项来确定。这些数据就被称作连接头。    

### 结构长度   
结构长度是整个连接头的总长度。

### 通讯通道ID
服务器应该分配一个通讯通道ID给每个已建立的通讯通道。   

### 序列计数器
每个通讯通道的序列计数器应该得到维护，每产生一个通过数据连接发送的flowerip帧时，计数器加一。    

当一个连接建立时，对应的计数器清零。所以在新建的通信通道上的第一个帧的序列号为0。

### 连接类型指定头项
。。。

## 心跳监测
客户端应周期发送连接状态请求帧（例如每隔60秒）到服务器的控制端点，该端点应立即回应一个连接状态响应帧（这个就是心跳响应）

如果客户端在10s后没有收到收到心跳响应，或收到的响应包含错误信息，客户端应重发三次连接状态请求，均未果后发送断链请求到服务器的控制断链来终止连接。    

如果服务器在120秒内没有收到心跳请求，服务器将终止连接，方法同上。服务器在收到错误序列号的报文后不应重触发计时。

## 断开连接
一般由客户端断开连接。在正常的操作中，客户端应发送一个断链请求帧给服务器控制端点来请求终止数据通道连接。
如果可行的话，客户端应优雅地尝试断开连接，即使在错误的情况下。服务器可以在发生错误的情况下发送断链请求。但还是建议让客户端来中断连接。   

flowerip设备接收到一个断链请求时，应响应一个断链响应帧。这个数据包应指示链接的结束。


# 通用实现指导
## 简介
该章节定义了编码指导，用于指导服务器和客户端的开发，认证参考这部分要求。

## 服务器
- 如果服务器接收到一个数据包含有不支持的版本信息，将回复一个否定应答帧来指示错误状态。
- 如果收到一个非法信息，服务器应忽略该数据包并不做任何进一步处理。   
- 如果连接已经建立，所有的数据包应该按照相同的版本信息发送。   
- 如果链接已经建立，但接收到的数据包中的版本信息发生了改变，服务器将关闭连接。

## flowerip路由设置
### flowerip路由出厂默认设置
路由器的一个重要特点是在用户不干预的情况下提供合适的路由功能。这种即插即用的路由行为需要一个标准的出厂默认配置：
- 路由的默认物理地址应该为FF00H
- 路由的多播地址应该等于系统设置的多播地址
- flower广播报文应该从一条支线上被路由到其他支线上，即使其他支线的路由器还是出厂默认配置。   
- 路由器应能正常工作，即使合法的单播IP地址没有获取   
- 路由器应能正常工作，隧道应能配置路由器（路由器已经分配了独立地址）。

### flowerip路由器 IP地址分配
路由器应支持即插即用，即使DHCP或手动分配或ABS进行配还没有给出IP地址。这需要ip协议栈可以发送和接收多播IP信息，尽管没有一个合法的单播IP地址。




# Routing
## 简介
### 范围  
- 本章节描述了一个点对多点的标准协议用于F设备之间的路由   
- 本文件定义了一个标注的协议，该协议与F设备和F工具配合来交换F数据通过非F网络

# F报文的FIP路由
## 简介
- FIP路由被定义为一个FIP路由器多播通讯，在这个路由器中F数据从一个设备同时转移到一个或多个设备通过IP网络。一个路由器可以替代支线耦合器或干线耦合器，提高通讯速率。
