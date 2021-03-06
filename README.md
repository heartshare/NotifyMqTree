# NotifyMqTree
Notify消息中间件

![](https://i.imgur.com/arAfaRA.png)

<pre>
      Notify设计的核心思想是为了维护大量的消息堆积。

      市面上大部分的消息中间件都是点对点的消息传输通道，然后非常激进的使用内存来提升整体的
  系统性能，这样做虽然能够达到很高的tps，但是这种设计的思路很难符合大规模分布式场景的实际需要。

      在实际的分布式场景中，这样的系统会存在着较大的应用场景瓶颈，在后端有大量消费者的前提
  下，消费者出现问题是个很常见的情况，而消息系统则必须能够在后端消费不稳定的情况下，仍然能够
   保证用户写入的正常。

      正因为如此，Notify在设计中，最优先考虑的就是消息的堆积问题，在目前的方案中，使用了持
  久化磁盘的方式，在每次用户发送消息到Notify的时候都将消息先落盘，然后再异步的进行消息投递。

      虽然这种方式导致系统的性能比目前市面的MQ要差一些，但是稳定，安全可靠是系统的核心诉求。

</pre>

<pre>
集群架构
      配置服务器集群(Config Server):
             这个集群的主要目的是动态的感知应用集群，消息集群机器上下线的过程，并及时广播
         给其他集群。如当业务接收消息的机器下线时，config server会感知到机器下线，从而将
         机器从目标用户组内踢出，并通知给notify server, notify server在获取通知后，就可
         以将已经下线的机器从自己的投递目标列表中删除，这样就可以实现机器的自动上下线扩容。

      消息服务器（Notify Server）
             消息服务器，也就是真正承载消息发送与消息接收的服务器，也是一个集群，应用发送
      消息时可以随机选择一台机器进行消息发送，任意一台server挂掉，系统都可以正常运行。当需
      要增加处理能力时，只要简单的增加Notify server就行。

      存储服务器(Storage Server)
             Notify存储集群有多种不同的实现方式，以实现不同应用的实际存储需求。针对消息安
      全性要求较高的应用，我们会选择使用多份落盘的方式存储消息数据，而对于吞吐量而不要求消
      息安全的场景，可以使用内存存储模型，

             消息接收集群业务方用于处理消息的服务器组，上下线机器时也能够动态的由
      Config Server感知机器上下线的时机，从而可以实现机器自动扩展。
</pre>