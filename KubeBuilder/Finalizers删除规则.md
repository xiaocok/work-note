## Finalizer资源删除

`finalizer`即终结器，存在于每一个k8s内的资源实例中，即`**.metadata.finalizers`,它是一个字符串数组，每一个成员表示一个`finalizer`。控制器在删除某个资源时，会根据该资源的`finalizers`配置，进行异步预删除处理，所有的`finalizer`都执行完毕后，该资源会被真正删除。

这里的预删除处理，一般指对该资源的关联资源进行增删改操作。比如：一个A资源被删除时，其finalizer规定必须将A资源的Selector指向的所有service都删除。

当我们需要设计这类finalizer时，就可以自定义一个controller来实现。



### 添加*Finalizer*规则

*Finalizer* 能够让控制器实现异步的删除前（Pre-delete）回调。 与内置对象类似，定制对象也支持 Finalizer。

添加 Finalizer：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  finalizers:
  - stable.example.com/finalizer
```

> 自定义 Finalizer 的标识符包含一个域名、一个正向斜线和 finalizer 的名称。 任何控制器都可以在任何对象的 finalizer 列表中添加新的 finalizer。



### 删除流程

1. 删除一个对象时，如果该对象带有 Finalizer 的属性。则**无法强制删除对应的对象**。第一个执行删除操作时，会为其 `metadata.deletionTimestamp` 设置一个值，但不会真的删除对象。

   一旦此值被设置，`finalizers` 列表中的表项只能被移除，不能新增加。

   如果finalizers列表中的数据未被完全移除时，无法强制删除对应的对象。

2. 当 `metadata.deletionTimestamp` 字段被设置时，监视该对象的各个控制器需要执行它们所能处理的 finalizer，并在完成处理之后将其从列表中移除。 每个控制器负责将其 finalizer 从列表中删除。

   ```yaml
   apiVersion: "stable.example.com/v1"
   kind: CronTab
   metadata:
     finalizers:
     - stable.example.com/service
     - stable.example.com/daemonset
   ```

   比如：上面负责添加finalizers的两个控制器都能触发协调。当他们检测到metadata.deletionTimestamp有值时，表示要删除CR。

   则它们都处理自己对应的业务路基，完成CR删除时需要处理的逻辑，然后再删除CR中对应的finalizer。

3. `metadata.deletionGracePeriodSeconds` 的取值控制对更新的轮询周期。
4. 一旦 finalizers 列表为空时，就意味着所有 finalizer 都被执行过， Kubernetes 会最终删除该资源。



### K8s 的垃圾回收策略

在 K8s 中，有两大类 GC：

- 级联（Cascading）：在级联删除中，所有者被删除，那集群中的从属对象也会被删除。

  - 前台级联删除（Foreground Cascading Deletion）

    在这种删除策略中，所有者对象的删除将会持续到其所有从属对象都被删除为止。

  - 后台级联删除（Background Cascading Deletion）

    这种删除策略会简单很多，它会立即删除所有者的对象，并由垃圾回收器在后台删除其从属对象。这种方式比前台级联删除快的多，因为不用等待时间来删除从属对象。

- 孤儿（Orphan）：这种情况下，对所有者的进行删除只会将其从集群中删除，并使所有对象处于“孤儿”状态。



### 参考

- [使用 CustomResourceDefinition 扩展 Kubernetes API](https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#advanced-topics)
- [Kubernetes之Garbage Collection](https://blog.csdn.net/dkfajsldfsdfsd/article/details/81130786)
- [Kubernetes 中的垃圾回收](https://zhuanlan.zhihu.com/p/160816303)