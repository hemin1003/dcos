## 资源分配-DRF算法

Mesos采用两级调度来控制集群中的资源配给。不同于集中式的存储集群中的资源信息，它通过资源提供（resource offers）的概念，让运行的框架通过Master获取Agent节点所提供的资源，从而保持整个系统架构的简洁和可扩展性。

Master的分配模块负责确定哪个应用程序应该接收下一次资源提供，并且依赖主导资源公平（DRF）算法来做出这些决策。本文针对不同的用例使用简单的Mesos框架实现来观察和分析DRF行为。

架构回顾

框架

Mesos框架包括两个主要组件：调度器（Scheduler）和执行器（Executor）。

调度器是一个与Master协商的单实例/进程，负责处理资源提供，任务提交，任务状态更新，框架消息和异常情况，例如从丢失，断开和各种错误。有时高可用性是调度器的一个要求，因此需要某种类型的领导者选举（leader election）或特定逻辑来避免任务提交中的冲突。

执行器是在Agent节点上执行的进程，在Agent节点上接收任务并运行它们。执行器生命周期与调度器绑定，所以当Scheduler完成它的工作时，它也关闭执行器（实际上Mesos Master在框架终止时执行这个例程）。如果执行器是一个JVM进程，它通常有一个线程池用于执行接收的任务。

任务信息使用protobuf进行序列化，它包含关于所需的资源，待运行的执行程序，一些二进制序列化的有效负载（例如Spark任务被序列化并作为Mesos任务中的有效载荷传送）等所有的信息。以下是创建任务的示例代码：

```scala
def buildTask(offer: Offer, cpus: Double, memory: Int, executorInfo: ExecutorInfo) = { 
    val cpuResource = Resource.newBuilder 
                        .setType(SCALAR) 
                        .setName("cpus") 
                        .setScalar(Scalar.newBuilder.setValue(cpus)) 
                        .setRole("*") 
                        .build 
    val memResource = Resource.newBuilder 
                        .setType(SCALAR) 
                        .setName("mem") 
                        .setScalar(Scalar.newBuilder.setValue(memory)) 
                        .setRole("*") 
                        .build 
    TaskInfo.newBuilder() 
            .setSlaveId(SlaveID.newBuilder().setValue(offer.getSlaveId.getValue).build()) 
            .setTaskId(TaskID.newBuilder().setValue(s"$uuid")) 
            .setExecutor(executorInfo) 
            .setName(UUID.randomUUID().toString) 
            .addResources(cpuResource) 
            .addResources(memResource) 
            .build() 
} 
def buildExecutorInfo(d: SchedulerDriver, prefix: String): ExecutorInfo = { 
           val scriptPath = System.getProperty("executor.path","/throttle/throttle-executor.sh") 
           ExecutorInfo.newBuilder() 
                    .setCommand(CommandInfo.newBuilder().setValue("/bin/sh "+scriptPath)) 
                    .setExecutorId(ExecutorID.newBuilder().setValue(s"${prefix}_$uuid")) 
                    .build() 
}
```

源代码可参考[Github Repo](https://github.com/datastrophic/mesos-workshop/tree/master/src/main/scala/io/datastrophic/mesos)。

资源提供（Resource Offers）

Agent节点定期向Master汇报它们所能提供的资源。

分配模块开始向框架提供资源。

### 参考

http:\/\/datastrophic.io\/resource-allocation-in-mesos-dominant-resource-fairness-explained

