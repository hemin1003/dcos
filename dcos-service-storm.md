## Storm on DC/OS

[Apache Storm](http://storm.apache.org)是Twitter开源的分布式实时计算平台。


### 在DC/OS上部署Storm

Storm有自己一套独立的计算作业调度管理机制。Storm通过Nimbus管理Supervisor节点，计算作业（Topology）通过Nimbus部署到Supervisor节点上，然后根据拓扑配置启动相应的Worker（JVM）进程处理作业任务。在一个Supervisor节点上可以启动多个Worker，每个Worker对应一个WorkerSlot，它是由Host和Port组成。
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