### MQTT 核心概念[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#mqtt-core-concepts)

> MQTT（消息队列遥测传输）是物联网（IoT）最常用的**轻量级消息传递**协议。
> 协议基于用于消息通信的**发布/订阅 （pub/sub）** 模式。它允许设备和应用程序使用简单高效的消息格式实时交换数据，从而最大限度地减少==网络开销==并降低功耗。

### EMQX

EMQX Enterprise 作为 MQTT 消息平台，全面支持一整套 MQTT 消息功能。本节简要介绍MQTT的核心概念。您可以通过 MQTT 博客系列的链接进一步了解每个概念以及有关 [MQTT](https://www.emqx.com/en/blog/category/mqtt) 的更多信息。

## 发布/订阅模式[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#publish-subscribe-pattern)

> 该协议是==事件驱动==的，并使用发布/订阅模式连接设备。
- 与传统的客户端/服务器模式不同，它是一种消息传递模式，在这种模式中，发送方（发布者）不直接向特定的接收方（订阅者）发送消息。
- 相反，发布者将消息分类为主题，订阅者订阅他们感兴趣的特定主题。当发布者向主题发送消息时，MQTT 代理会路由并过滤所有传入消息，然后将消息传递给对该主题表示感兴趣的所有订阅者。

发布者和订阅者彼此分离，不需要知道彼此的存在。它们的唯一联系是基于有关消息的预定协议。发布/订阅模式可实现灵活的消息通信，因为可以根据需要动态添加或删除订阅者和发布者。它还使消息**广播、多播和单播**的实现更加容易。

有关发布/订阅模式的更多信息，请参阅 [MQTT 发布-订阅模式简介](https://www.emqx.com/en/blog/mqtt-5-introduction-to-publish-subscribe-model)。

## MQTT服务器[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#mqtt-server)

MQTT 服务器充当发布客户端和订阅客户端之间的代理，将所有接收到的消息转发到匹配的订阅客户端。因此，有时服务器直接称为 MQTT Broker。

## MQTT 客户端[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#mqtt-client)

客户端是指可以使用MQTT协议连接到MQTT服务器的设备或应用程序。他们既可以充当发布者，也可以同时充当订阅者，也可以单独担任其中任一角色。

## 主题和通配符[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#topic-and-wildcards)

主题用于识别和区分不同的消息，构成了 MQTT 消息路由的基础。发布者可以在发布时指定消息的主题，而订阅者可以选择订阅感兴趣的主题以接收相关消息。

订阅者可以在订阅的主题中使用通配符，实现一次订阅多个主题的目标。MQTT提供单级通配符和多级通配符两种主题通配符，满足不同的订阅需求。

有关主题和通配符的更多信息，请参阅[按大小写了解 MQTT 主题和通配符](https://www.emqx.com/en/blog/advanced-features-of-mqtt-topics)。

## [[服务质量 （QoS）​]]

MQTT 定义了三个级别的 QoS，以提供不同级别的消息可靠性。每条消息在发布时都可以独立设置自己的 QoS。

- QoS 0：最多下发一条消息一次，可能会丢失;
- QoS 1：至少传递一次消息并保证到达，但可能会重复;
- QoS 2：只传递一次消息，保证到达时不会重复。

随着QoS等级的提高，消息传输的复杂度也随之增加。您需要根据实际情况选择合适的QoS级别。

有关 QoS 的更多信息，请参阅 [MQTT QoS 0、1、2 简介](https://www.emqx.com/en/blog/introduction-to-mqtt-qos)。

## 会期[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#session)

QoS 是一种理论机制，旨在确保可靠的消息传递，而会话则确保 QoS 1 和 2 协议过程的正确实现。

会话是指客户端和服务器之间的有状态交互，可以持续与网络连接相同的持续时间，也可以跨越多个网络连接，通常称为持久会话。连接可以从现有会话恢复，也可以从新会话开始。

有关会话的更多信息，请参阅 [MQTT 持久会话和清理会话说明](https://www.emqx.com/en/blog/mqtt-session)。

## 保留留言[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#retained-message)

与常规消息不同，保留的消息可以存储在 MQTT 服务器上。当任何新订阅者订阅的主题与保留消息的主题匹配时，他们会立即收到该消息，即使该消息是在他们订阅该主题之前发布的。

保留消息功能允许订阅者在连接后立即接收数据更新，而无需等待发布者重新发布消息。保留的消息在某些方面可以看作是消息“云盘”：随时将消息上传到“云盘”，随时从“云盘”中检索消息。但是，此“云驱动器”仅限于每个主题仅存储一条最新保留的消息。

您可以按照[保留消息](https://www.emqx.io/docs/en/latest/messaging/mqtt-retained-message.html)中的指示信息，尝试使用 MQTTX 客户端发布保留消息。

要了解有关保留消息技术的更多信息，请参阅 [MQTT 保留消息初学者指南](https://www.emqx.com/en/blog/mqtt5-features-retain-message)。

## 遗嘱留言[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#will-message)

Pub/Sub 模式的功能决定了除服务器之外，没有客户端知道客户端离开通信网络。但是，遗嘱消息使断开连接的客户端能够通知其他客户端。

客户端在建立连接时可以与服务器设置自己的遗嘱消息，如果客户端意外断开连接，服务器会立即发布此消息或在指定的延迟后发布此消息。订阅了相应消息主题的客户端将收到此消息并采取适当的操作，例如更新该客户端的在线状态等。

您可以按照[遗嘱消息](https://www.emqx.io/docs/en/latest/messaging/mqtt-will-message.html)中的说明，尝试使用 MQTTX 客户端发布遗嘱消息。

要了解有关遗嘱消息技术的更多信息，请参阅 [MQTT 遗嘱消息的使用](https://www.emqx.com/en/blog/use-of-mqtt-will-message)。

## 共享订阅[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#shared-subscription)

在一般情况下，消息会转发给所有匹配的订阅者。但是，在某些情况下，您可能希望协调多个客户端以水平可扩展的方式处理收到的消息，以增加负载容量。或者，用户可能希望添加一个备份客户端，以便客户端在主客户端脱机时无缝切换到，从而确保高可用性。

共享订阅功能提供了此类功能。客户端可以划分为多个订阅组，消息仍会转发到所有订阅组，但每个订阅组中一次只有一个客户端接收消息。

您可以按照[共享订阅](https://www.emqx.io/docs/en/latest/messaging/mqtt-shared-subscription.html)中的说明，尝试使用 MQTTX 客户端创建共享订阅。

要了解有关共享订阅技术的更多信息，请参阅[共享订阅 - MQTT 5.0 新功能](https://www.emqx.com/en/blog/introduction-to-mqtt5-protocol-shared-subscription)。

## 系统主题[​](https://www.emqx.io/docs/en/latest/messaging/mqtt-concepts.html#system-topic)

前缀的主题是为服务器保留的，用于发布特定消息，例如服务器正常运行时间、客户端联机/脱机事件通知和当前连接的客户端数。这些主题通常称为系统主题，客户端可以订阅这些系统主题以获取有关服务器的信息。`$SYS/`