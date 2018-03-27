# Namespace

kubernetes支持由同一个物理集群支持的多个虚拟集群。
这些虚拟集群被称为命名空间。

在一个Kubernetes集群中可以使用namespace创建多个“虚拟集群”，这些namespace之间可以完全隔离，也可以通过某种方式，让一个namespace中的service可以访问到其他的namespace中的服务，我们[在CentOS中部署kubernetes1.6集群](../practice/install-kubernetes-on-centos.md)的时候就用到了好几个跨越namespace的服务，比如Traefik ingress和`kube-system`namespace下的service就可以为整个集群提供服务，这些都需要通过RBAC定义集群级别的角色来实现。

## 哪些情况下适合使用多个namespace

因为namespace可以提供独立的命名空间，并且namespace是一种在多个用户之间划分群集资源的方法（通过资源配额）,不同的用户属于不同的namespace,namespace中有独立的集群资源，与其他资源相互不干扰。

当你的项目和人员众多的时候可以考虑根据项目属性，例如生产、测试、开发划分不同的namespace。

## Namespace使用

**获取集群中有哪些namespace**

	#kubectl get namespaces
	NAME          STATUS    AGE
	default       Active    11d
	kube-public   Active    11d
	kube-system   Active    11d



集群启动有三个初始的namesapce:

- `default`  默认的命名空间
- `kube-system` 由k8s集群创建资源对象所处的namespace，如kube-dns
- `kube-public` k8s创建的对所有用户可读namespace。（包括认证和未认证的用户）

在执行`kubectl`命令时可以使用`-n`指定操作的namespace。

用户的普通应用默认是在`default`下，与集群管理相关的为整个集群提供服务的应用一般部署在`kube-system`的namespace下，例如我们在安装kubernetes集群时部署的`kubedns`、`heapseter`、`EFK`等都是在这个namespace下面。

### namespace和dns
当你在k8s创建一个Service后，你的服务会分配到一个dns记录(FQDN)：
	<service-name>.<namespace-name>.svc.cluster.local

同一个namespace中的容器通过通过service-name相互访问，不同的namespace通过FQDN访问。

### 不是所有的object都在namespace中

大多数的k8s资源对象都是处于某个具体的namespace中。但是对于偏底层的资源，如`node`和`persistentVolume`就不属于任何namespace。

