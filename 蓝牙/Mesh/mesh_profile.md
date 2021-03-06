Mesh 系统架构

#### 2.1 架构

模型层（Model Layer）：定义了用于标准化典型用户场景的操作模型，在蓝牙 mesh model spec 中或其他更高层规范（例如照明和传感器模型）中定义。

基础模型层（Foundation Model Layer）：定义配置和管理 mesh 网络所需的状态（states）、消息（messages）和模型（models）。

访问层（Access Layer）：定义了应用程序的数据格式，定义和控制在上层传输层中应用数据的加密和解密。

上层传输层（Upper Transport Layer）：对应用程序数据进行加密、解密和身份验证，并为访问消息提供机密性，还定义了如何使用传输控制消息来管理节点之间的上层传输层（包括朋友特性何时使用）。

下层传输层（Lower Transport Layer）：定义了如何将上层传输层消息分段并重新组装成多个下层传输PDU，以便将大型的上层传输层消息传递给其他节点，还定义了一个控制消息来管理分段和重新组装。

网络层（Network Layer）：定义了如何将传输消息定位到一个或多个元素，定义了承载层传输PDU的网络消息格式。决定了是否中继/转发消息、接受消息并进行处理、或者拒绝消息。还定义了如何对网络消息进行加密和身份验证。

承载层（Bearer Layer）：定义了网络消息如何在节点之间传输。定义了两个载体（广播载体和 GATT 载体）。



#### 2.2 mesh 操作

mesh 网络操作有以下定义：

- 消息能从一个元素发送到一个或多个元素。
- 允许消息通过其他节点转发，以扩展通信范围。
- 针对已知的安全攻击保护消息。
- 以及时的方式传递消息。
- 当一个或多个设备被移除或停止工作时，可以继续工作。

mesh 网络使用广播信道传输消息，故其他节点能够接收并转发这些消息，从而扩展了原始消息的范围。处于 mesh 网络中的任何设备随时都可以发送消息（前提是侦听和转发消息的设备具有足够的密度）。

网络消息缓存（cache）通过将所有消息添加到一个缓冲列表来防止设备转发以前收到的消息。当收到消息时，根据列表对其进行检查，若消息已存在，则忽略该消息，否则将其添加到缓存中。缓存的消息数量是有限制的。

每条消息都包含一个生存时间（TTL）值，该值限制消息可以转发的次数。每当一个设备接收一条消息并将其转发时，该消息的 TTL 值减 1。

2.2.1 网络和子网

mesh 网络由共享下列四种公共资源的节点组成：

- 用于标识消息源/目的地的网络地址（network address）。
- 用于在网络层保护和验证消息的网络密钥（network key）。
- 用于在访问层保护和验证消息的应用密钥（application key）。
- 用于延长网络生命周期的 IV 索引（IV index）。

一个网络可以有一个或多个子网（subnet）来实现区域隔离。子网是一组节点，他们可以在网络层相互通信，因为它们共享网络密钥。通过网络密钥的数量，一个节点可以属于一个或多个子网。

2.2.2 设备和节点

非 mesh 网络成员的设备称为非供应设备（unprovisioned device），作为 mesh 网络成员的设备称为节点（node）。供应器（Provisioner）用于管理非供应设备和节点间的转换。

非供应设备不能发送或接收 mesh 消息，然而，它可以告诉供应器它的存在。供应器会在非供应设备通过身份验证后邀请该设备进入 mesh 网络，从而将该设备转换为节点。

节点可以发送或接收 mesh 消息，并由配置客户端（configuration client）进行管理，以配置节点如何与其他节点通信。配置客户端可以从 mesh 网络中删除节点。

一个设备可以支持多个实例。mesh 网络的每个实例由地址和设备在供应器获得的设备密钥确定。

2.2.3 添加设备到 mesh 网络

设备由供应者添加到 mesh 网络中，这与蓝牙技术中使用的点对点绑定和配对不同，设备供应可以使用简单的广播载体或基于GATT的点对点载体。基于广播载体的供应器由所有设备实现，基于GATT载体的供应器可由手机等实现。

为了协助多个设备的配置，设备有一个由供应者设置的注意定时器，当设置为非 0 值时，设备可以使用任何方法标识自己，当定时器到期时，设备停止识别自己，这允许供应器发送一个单一的消息到设备。

在两个承载间运行的协议是SM协议的派生类，引入了对用户界面非常有限的设备进行身份验证的能力。

#### 2.3 架构概念

**状态（States）**：表示元素条件的值。公开状态的元素称为服务器，访问状态的元素称为客户端。由两个以上的值组成的状态称为复合状态（composite states）。

**约束状态（Bound states）**：当一个状态绑定到另一个状态时，其中一个状态的更改会导致另一个状态的更改。

**消息（Messages）**：mesh 网络中的所有通信都是通过发送消息来完成的，消息对状态进行操作。对于每个状态，服务器都支持一组已定义的消息，客户端可以使用这些消息获取状态值或更改状态。服务器也可以传输未经请求的消息，其中带有状态或状态编号的信息。

消息具有操作码、相关参数和行为。操作码可以为 1/2/3 字节，

**元素（Elements）**：元素是节点中的可寻址实体。每个节点至少有一个元素（主元素），且可能有一个或多个附加元素。

地址（Addresses）：地址可以是单播（unicast）地址、虚拟（virtual）地址或组（group）地址。

单播地址被分配给一个元素，一个 mesh 网络可以有 32767 个单播地址。

虚拟地址是一个多播地址，可以表示一个或多个节点上的多个元素。每个虚拟地址在逻辑上代表一个标签UUID（128位）。

模型（Models）：定义了节点的基本功能。

3 网状网络



