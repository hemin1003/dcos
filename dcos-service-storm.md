## Storm on DC/OS

[Apache Storm](http://storm.apache.org)是Twitter开源的分布式实时计算平台。[Mesos Storm](https://github.com/mesos/storm)让Storm框架能够在DCOS集群中部署并使用Mesos为计算拓扑调度资源。

### Storm框架回顾


### 在DC/OS上运行Storm

Storm有自己一套独立的计算作业调度管理机制。Storm通过Nimbus管理Supervisor节点，计算作业（Topology）通过Nimbus部署到Supervisor节点上，然后根据拓扑配置启动相应的Worker（JVM）进程处理作业任务。在一个Supervisor节点上可以启动多个Worker，每个Worker对应一个WorkerSlot，它是由Host和Port及对应的资源配置组成。

Storm的上述拓扑调度机制与Mesos的资源调度非常相似，Mesos向框架提供计算资源，框架的Scheduler接收资源并在分配的资源上通过Executor启动任务。

因此，为实现Storm框架使用Mesos提供的计算资源，Mesos Storm重写了Storm提供的INimbus和IScheduler接口。

**INimbus接口：**

```java
public interface INimbus {
    void prepare(Map stormConf, String schedulerLocalDir);
    /**
     * Returns all slots that are available for the next round of scheduling. A slot is available for scheduling
     * if it is free and can be assigned to, or if it is used and can be reassigned.
     */
    Collection<WorkerSlot> allSlotsAvailableForScheduling(Collection<SupervisorDetails> existingSupervisors, Topologies topologies, Set<String> topologiesMissingAssignments);


    /**
     * this is called after the assignment is changed in ZK
     * @param topologies
     * @param newSlotsByTopologyId
     */
    void assignSlots(Topologies topologies, Map<String, Collection<WorkerSlot>> newSlotsByTopologyId);

    /**
     * map from node id to supervisor details
     * @param existingSupervisors
     * @param nodeId
     * @return
     */
    String getHostName(Map<String, SupervisorDetails> existingSupervisors, String nodeId);
    
    IScheduler getForcedScheduler(); 
}
```

**IScheduler接口：**

```java
public interface IScheduler {
    
    void prepare(Map conf);
    
    /**
     * Set assignments for the topologies which needs scheduling. The new assignments is available 
     * through `cluster.getAssignments()`
     *
     *@param topologies all the topologies in the cluster, some of them need schedule. Topologies object here 
     *       only contain static information about topologies. Information like assignments, slots are all in
     *       the `cluster` object.
     *@param cluster the cluster these topologies are running in. `cluster` contains everything user
     *       need to develop a new scheduling logic. e.g. supervisors information, available slots, current 
     *       assignments for all the topologies etc. User can set the new assignment for topologies using
     *       cluster.setAssignmentById()`
     */
    void schedule(Topologies topologies, Cluster cluster);
}
```

在代码中，MesosNimbus实现INimbus接口，向Mesos Master注册并接收Mesos提供的资源。当接收到拓扑部署请求时，它向IMesosStormScheduler接口的实现获取所有可用于处理该拓扑的WorkerSlot。

**IMesosStormScheduler接口：**

```java
/**
 * A scheduler needs to implement the following interface for it to be MesosNimbus compatible.
 */
public interface IMesosStormScheduler {

  /**
   * This method is invoked by Nimbus when it wants to get a list of worker slots that are available for assigning the
   * topology workers. In Nimbus's view, a "WorkerSlot" is a host and port that it can use to assign a worker.
   * <p/>
   * TODO: Rotating Map itself needs to be refactored. Perhaps make RotatingMap inherit Map so users can pass in a
   * map? or Make the IMesosStormScheduler itself generic so _offers could be of any type?
   */
  public List<WorkerSlot> allSlotsAvailableForScheduling(RotatingMap<Protos.OfferID, Protos.Offer> offers,
                                                         Collection<SupervisorDetails> existingSupervisors,
                                                         Topologies topologies, Set<String> topologiesMissingAssignments);
}

```

IMesosStormScheduler接口的实现同时实现了IScheduler接口，它负责根据拓扑的请求信息以及Mesos提供的资源，生成特定于拓扑的MesosWorkerSlot。
