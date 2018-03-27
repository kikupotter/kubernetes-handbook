# Node

Node是kubernetes集群的工作节点，提供pod的运行环境，可以是异构/同构物理机也可以是虚拟机。


## Node的状态

Node包括如下状态信息：

- Address
  - HostName：kebelet主机的名字，可以被kubelet中的`--hostname-override`参数替代。
  - ExternalIP：可以被集群外部路由到的IP地址。
  - InternalIP：集群内部使用的IP，集群外部无法访问。

- node Condition:
	- OutOfDisk
		- True(磁盘空间不足，没空间启动新pod)
		- False
	- MemoryPressure
		- True(快没内存了)
		- False
	- Diskpressure
		- True(快没磁盘空间了）
		- Flase
	- NetworkUnavailable
		- True（网络不可达）
		- False
	- ConfigOk
		- True（kubelet配置正确）
		- False
	- Ready
		- True（node状态健康可以跑pod）
		- False
		- Unknown（管理节点超过40S，没有收到node的消息。)



> 当节点Ready处于Unknown或者False超过kube-controller-manager设置的pod-eviction-timeout值时，node controller 会将node上所有的pod删除。默认的pod-eviction-timeout时间为5分钟。 同时被删除的节点会在其他的node上根据副本策略重新运行。

>删除的指令是apiserver发送给kubelet的，所有有一种特殊情况，就是kubelet网络不可达，这时候需要等node节点重新连上集群才能接受删除。。		

condition按照如下格式表达：

		"conditions": [
	  {
	    "type": "Ready",
	    "status": "True"
	  }
	]
	
		
### node Capacity

描述节点上可用的资源：

- cpu
- 内存
- 可以调度到节点上的最大Pod数量。


### node info 

kubelet收集：

- os kernel
- OS name
- kubelet
- docker
- kube-proxy version
	 
	 
## Node管理

禁止pod调度到该节点上

```bash
kubectl cordon <node>
```

驱逐该节点上的所有pod

```bash
kubectl drain <node>
```

该命令会删除该节点上的所有Pod（DaemonSet除外），在其他node上重新启动它们，通常该节点需要维护时使用该命令。直接使用该命令会自动调用`kubectl cordon <node>`命令。当该节点维护完成，启动了kubelet后，再使用`kubectl uncordon <node>`即可将该节点添加到kubernetes集群中。
