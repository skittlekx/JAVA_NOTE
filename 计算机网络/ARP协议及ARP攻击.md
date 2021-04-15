# ARP协议

ARP（Address Resolution Protocol）地址解析协议，目的是实现IP地址到MAC地址的转换。

在计算机间通信的时候，计算机要知道目的计算机是谁（就像我们人交流一样，要知道对方是谁），这中间需要涉及到MAC地址，而MAC是真正的电脑的唯一标识符。

为什么需要ARP协议呢？因为在OSI七层模型中，对数据从上到下进行封装发送出去，然后对数据从下到上解包接收，但是上层（网络层）关心的IP地址，下层关心的是MAC地址，这个时候就需要映射IP和MAC。

## ARP简单请求应答

![](https://github.com/skittlekx/JAVA_NOTE/blob/22268d6e48427d832c4fa5242aea69b31c99be9e/img/Arp_Ping%E7%AE%80%E5%8D%95%E8%BF%87%E7%A8%8B.png?raw=true)

PC1依据OSI模型:
1. 依次从上至下对数据进行封装，包括对ICMP Date加IP包头的封装。
2. 但是到了封装MAC地址的时候,PC1首先查询自己的ARP缓存表，发现没有IP2和他的MAC地址的映射，这个时候MAC数据帧封装失败。我们使用ping命令的时候，是指定PC2的IP2的，计算机是知道目的主机的IP地址，能够完成网络层的数据封装，因为设备通信还需要对方的MAC地址，但是PC1的缓存表里没有，所以在MAC封装的时候填入不了目的MAC地址。
3. 那么PC1为了获取PC2的MAC地址，PC1要发送询问信息，询问PC2的MAC地址，询问信息包括PC1的IP和MAC地址、PC2的IP地址，这里我们想到一个问题，即使是询问信息，也是需要进行MAC数据帧的封装，那这个询问信息的目的MAC地址填什么呢，规定当目的MAC地址为ff-ff-ff-ff-ff-ff时，就代表这是一个询问信息，也即使后面我要说的广播。
4. PC2收到这个询问信息后，将这里面的IP1和MAC1（PC1的IP和MAC）添加到本地的ARP缓存表中，然后PC2发送应答信息，对数据进行IP和MAC的封装，发送给PC1，因为缓存表里已经有PC1的IP和MAC的映射了呢。这个应答信息包含PC2的IP2和MAC2。PC1收到这个应答信息，理所应当的就获取了PC2的MAC地址，并添加到自己的缓存表中。

经过这样交互式的一问一答，PC1和PC2都获得了对方的MAC地址，值得注意的是，目的主机先完成ARP缓存，然后才是源主机完成ARP缓存。之后PC1和PC2就可以真正交流了。

![](https://github.com/skittlekx/JAVA_NOTE/blob/22268d6e48427d832c4fa5242aea69b31c99be9e/img/ARP%E5%B9%BF%E6%92%AD%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%8D%95%E6%92%AD%E5%9B%9E%E5%BA%94.png?raw=true)

**广播：** 首先PC1广播发送询问信息（**ff-ff-ff-ff-ff-ff**），在这个普通交换机上连接的设备都会受到这个PC1发送的询问信息。

**回复：** 所有在这个交换机上的设备需要判断询问信息，如果各自的IP和要询问的IP不一致，则丢弃，如图PC3、Route均丢弃该询问信息，而对于PC2判断该询问信息发现满足一致的要求，则接受，同样的写入PC1的IP和MAC到自己的ARP映射表中。最后，PC2单播发送应答信息给PC1，告诉PC1自己的IP和MAC地址。

# ARP数据包信息

|||
|:--:|:--:|
|Hardware type|硬件类型，标识链路层协议|
|Protocol type|协议类型，标识网络层协议|
|Hardware size|硬件地址大小，标识MAC地址长度，这里是6个字节（48bit）|
|Protocol size|协议地址大小，标识IP地址长度，这里是4个字节（32bit）|
|Opcode|操作代码，标识ARP数据包类型，1表示请求，2表示回应|
|Sender MAC address|发送者MAC|
|Sender IP address|发送者IP|
|Target MAC address|目标MAC，此处全0表示在请求|
|Target IP address|目标IP|

# ARP攻击

![](https://github.com/skittlekx/JAVA_NOTE/blob/22268d6e48427d832c4fa5242aea69b31c99be9e/img/ARP%E6%94%BB%E5%87%BB.png?raw=true)

当PC1对PC2正常通信的时候（先别管攻击者PC3），PC2、PC1会先后建立对方的IP和MAC地址的映射（即建立ARP缓存表），同时对于交换机而言，它也具有记忆功能，会基于源MAC地址建立一个CAM缓存表（记录MAC对应接口的信息），理解为当PC1发送消息至交换机的Port1时，交换机会把源MAC（也就是MAC1）记录下来，添加一条MAC1和Port1的映射，之后交换机可以根据MAC帧的目的MAC进行端口转发，这个时候PC3只是处于监听状态，会把PC1的广播丢弃。

 

正常的PC3会把广播包丢弃，同样的PC3可以抓住这一环节的漏洞，把不属于自己的广播包接收，同时回应一个虚假的回应包，告诉PC1我就是PC2（IP2-MAC3），这样PC1会收到两个回应包（一个正确的IP2-MAC2，一个虚假的IP2-MAC3），但是PC1并不知道到底哪个是真的，所以PC1会做出判断，并且判断后到达的为真，那么怎么让虚假的回应包后到达呢，PC3可以连续不断的发送这样的回应包，总会把哪个正确的回应包覆盖掉。


而后PC1会建立IP2-MAC3这样一条ARP缓存条目，以后当PC1给PC2发送信息的时候，PC1依据OSI模型从上至下在网络层给数据封装目的IP为IP2的包头，在链路层通过查询ARP缓存表封装目的MAC为MAC3的数据帧，送至交换机，根据查询CAM表，发现MAC3对应的接口为Port3，就这样把信息交付到了PC3，完成了一次ARP攻击。


# ARP协议攻击防御

**网关防御**

1. 合法ARP绑定，防御网关被欺骗
1. ARP数量限制，防御ARP泛洪攻击  

**接入设备防御**

1. 网关IP/MAC绑定，过滤掉仿冒网关的报文
1. 合法用户IP/MAC绑定，过滤掉终端仿冒报文
1. ARP限速

**客户端防御**
1. 绑定网关信息

常见技术有DHCP snooping、IP Source Guard、交换机端口安全技术、MAC访问控制列表、ARP报文限速、ARP mac地址手工绑定