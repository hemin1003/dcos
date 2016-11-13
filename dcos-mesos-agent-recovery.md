## Agent节点恢复

如果主机上的mesos-agent进程退出（可能是由于Mesos错误或者由于运维人员在升级Mesos时杀死了进程），那么由mesos-agent进程管理的任何执行器（executors）\/任务（tasks）将继续运行。当重新启动mesos-agent时，运维员可以控制如何处理那些旧的执行器\/任务：

默认情况下，所有由旧的mesos-agent进程管理的执行器\/任务将被终止。

如果框架在向Master注册时启用了检查点（checkpointing）操作，则属于该框架的任何执行程序都可以重新连接到新的mesos-agent进程，并继续不间断运行。

因此，启用框架的检查点使任务能够容忍Mesos的Agent升级和意外的mesos-agent进程崩溃，而不会遇到任何停机时间。

Agent节点恢复通过将Agent的任务和执行器的检查点信息（例如，任务信息，执行器信息，状态更新）保存到本地磁盘来实现。如果框架启用检查点操作，任何后续Agent节点的重新启动都将恢复检查点信息，并与仍在运行的任何执行程序重新连接。

注意，如果Agent节点上的操作系统重新启动，则节点上运行的所有执行程序和任务将被终止，并且在Agent节点重启后不会自动重新启动。

框架配置

框架可以在向Master注册时通过在其FrameworkInfo中设置checkpoint标志来控制是否恢复其执行程序。启用此功能会导致运行框架启动的任务的每个Agent节点上的I \/ O开销增加。默认情况下，框架不检查它们的状态。

Agent节点配置

三个配置参数用来控制Mesos的Agent节点的恢复行为：

strict： 是否在严格模式下执行代理恢复，默认值：true。

如果strict=true，所有恢复错误被认为是致命的。

如果strict=false，恢复期间的任何错误（例如，检查点数据中的损坏）被忽略，并且恢复尽可能多的状态。

recover：是否恢复状态更新并与旧的执行程序重新连接，默认值：reconnect。

如果recover=reconnect，只要执行者的框架启用检查点，重新连接任何旧的活的执行者。

如果recover=cleanup，杀死任何旧的活的执行者并退出。在执行不兼容的Agent节点或执行程序升级时使用此选项！注意：如果不存在检查点信息，则不执行恢复，并且Agent节点作为新节点向Master注册。

revovery\_timeout：分配给Agent节点恢复的时间量，默认值：15分钟。

如果Agent节点花费的恢复时间大于recovery\_timeout，则任何等待重新连接到Agent节点的执行程序都将自行终止。

注意：如果没有一个框架启用了检查点设置，则Agent节点上运行的执行程序和任务随着Agent节点死亡而死亡，并且不会再恢复。

重新启动的Agent节点应在超时间隔（默认情况下为75秒）内向Master重新注册：请参阅--max\_agent\_ping\_timeouts和--agent\_ping\_timeout配置参数。如果Agent节点重新注册花费的时间超过此超时设置，Master将关闭Agent节点，随后，Agent节点会关闭任何活动的执行程序\/任务。因此，强烈建议将重新启动Agent节点的过程自动化（例如，使用诸如monit或systemd的进程监视器）。

存在的问题

对systemd进程使用默认的**KillMode**，这是control-group，当Agent节点停止时会杀死所有子进程。这确保了“辅助”过程（例如，提取程序和perf）与Agent节点进程一起终止。这确保执行程序在Agent节点重新启动后继续存在。

```
[Service] 
ExecStart=/usr/bin/mesos-agent 
KillMode=control-cgroup
```

