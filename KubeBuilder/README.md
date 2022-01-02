# KebeBuilder

## 环境准备

[环境准备](环境准备.md)



##  初始化准备

[初始化项目](初始化项目.md)





### 生成的代码

**生成的的结构体定义**

```go
type CronJob struct {
	metav1.TypeMeta   `json:",inline"`					// 基本信息：类型的信息，kind/apiversion
	metav1.ObjectMeta `json:"metadata,omitempty"`		// 基本信息：名字、命名空间

	Spec   CronJobSpec   `json:"spec,omitempty"`		// 期望达到的目标的状态
	Status CronJobStatus `json:"status,omitempty"`		// 当前目标的状态
}
```

**注册结构体、底层才能识别**

```go
func init() {
   SchemeBuilder.Register(&CronJob{}, &CronJobList{})
}
```



1. 定义CRC

   **根据业务指定的属性参数**

   ```go
   // 根据业务，期望达到哪些目标状态，而定义的属性字段
   // CronJobSpec defines the desired state of CronJob
   type CronJobSpec struct {
   	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
   	// Important: Run "make" to regenerate code after modifying this file
   
   	// Foo is an example field of CronJob. Edit cronjob_types.go to remove/update
   	Foo string `json:"foo,omitempty"`
   
   	// Template describes the pods that will be created.
   	Template         corev1.PodTemplateSpec   `json:"template"`
   }
   
   // 期望记录哪些当前的状态
   // CronJobStatus defines the observed state of CronJob
   type CronJobStatus struct {
   	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
   	// Important: Run "make" to regenerate code after modifying this file
       
       // 增加的字段：最后调度时间
       LastScheduleTime *metav1.Time
   }
   ```

   

2. 协调的处理

   ```go
   // 协调的逻辑
   func (r *CronJobReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
   	_ = log.FromContext(ctx)
   
   	// 获取指定Namespace下的Cronjob
   	var cronJob batchv1.CronJob
   	if err := r.Get(ctx, req.NamespacedName, &cronJob); err != nil {
   		log.Log.Error(err, "unable to fetch CronJob")
   		return ctrl.Result{}, err
   	}
       
       // 逻辑操作
       
       // 设置调度时间
       cronJob.Status.LastScheduleTime = &metav1.Time{Time:time.Now()}
       
       // 更新资源
       if err := r.Status().Update(ctx, &cronJob); err != nil {
   		log.Log.Error(err, "unable to update CronJob status")
   		return ctrl.Result{}, err
   	}
       
       return ctrl.Result{RequeueAfter: 5 * time.Minute}, nil	// 下次协调的时间
   	return ctrl.Result{}, nil
   }
   
   // 监控什么资源
   // 当前CronJob的Reconciler加入Ctrl的Manager管理。
   // 指定当前Controller监控batchv1.CronJob的资源变化，触发Reconcile协调。
   func (r *CronJobReconciler) SetupWithManager(mgr ctrl.Manager) error {
   	return ctrl.NewControllerManagedBy(mgr).
   		For(&batchv1.CronJob{}).
   		Complete(r)
   }
   ```

   

3. 创建WebHook

   ```shell
   kubebuilder create webhook --group custom --version v1 --kind Unit --defaulting --programmatic-validation
   ```

   





## 开发设计

### 设计subresource风格的status

k8s中`status subresource`的使用规范：

1. 用户只能指定一个CRD实例的spec部分。

2. CRD实例的status部分由控制器进行变更。



- 需要在Bucket的注释中添加一行`// +kubebuilder:subresource:status`，变成如下：

  ```go
  // +kubebuilder:subresource:status
  // +kubebuilder:object:root=true
  
  // Bucket is the Schema for the buckets API
  type Bucket struct {
      metav1.TypeMeta   `json:",inline"`
      metav1.ObjectMeta `json:"metadata,omitempty"`
  
      Spec   BucketSpec   `json:"spec,omitempty"`
      Status BucketStatus `json:"status,omitempty"`
  }
  ```

  如果没有添加上面的注释。则我们在controller的代码中调用到`Status().Update()`,会触发panic，并报错：`the server could not find the requested resource`

- 创建Bucket资源时，即便我们填入了非空的`status`结构，也不会更新到apiserver中。Status只能通过对应的client进行更新。比如在controller中：

  ```go
  if bucket.Status.Progress == 0 {
        bucket.Status.Progress = 1
        err := r.Status().Update(ctx, &bucket)
        if err != nil {
            return ctrl.Result{}, err
        }
  }
  ```

  这样，只要bucket实例的`status.Progress`为0时（比如我们创建一个bucket实例时，由于`status.Progress`无法配置，故初始化为默认值，即0），controller就会帮我们将它变更为1.

  

  注意：

  - kubebuilder 2.0开发生成的crd模板，无法通过apiserver的crd校验。社区有相关的记录和修复

  - 所以1.11.*版本的k8s，要使用kubebuilder 2.0 必须给apiserver配置一个featuregate： - --feature-gates=CustomResourceValidation=false，关闭对crd的校验。



### finalizer

`finalizer`即终结器，存在于每一个k8s内的资源实例中，即`**.metadata.finalizers`,它是一个字符串数组，每一个成员表示一个`finalizer`。控制器在删除某个资源时，会根据该资源的`finalizers`配置，进行异步预删除处理，所有的`finalizer`都执行完毕后，该资源会被真正删除。

这里的预删除处理，一般指对该资源的关联资源进行增删改操作。比如：一个A资源被删除时，其finalizer规定必须将A资源的Selector指向的所有service都删除。

当我们需要设计这类finalizer时，就可以自定义一个controller来实现。



### kubebuilder 注释标记







## 最佳实践



### 模式

1.使用 OwnerRefrence 来做资源关联，有两个特性：

- Owner 资源被删除，被 Own 的资源会被级联删除，这利用了 K8s 的 GC；
- 被 Own 的资源对象的事件变更可以触发 Owner 对象的 Reconcile 方法；

2.使用 Finalizer 来做资源的清理。



### 注意点

- 不使用 Finalizer 时，资源被删除无法获取任何信息；
- 对象的 Status 字段变化也会触发 Reconcile 方法；
- Reconcile 逻辑需要幂等；

 

### 优化

使用 IndexFunc 来优化资源查询的效率



## 参考

- [Kubebuilder 使用简介](https://www.bilibili.com/video/BV1zq4y1o7Lg)
- [《 Kubebuilder v2 使用指南 》](https://wqyin.cn/gitbooks/kubebuilder)

- [kubebuilder2.0学习笔记——进阶使用](https://segmentfault.com/a/1190000020359577)
- [k8s crd operator踩坑点](https://blog.csdn.net/hahachenchen789/article/details/116274973)
- [深入解析 Kubebuilder：让编写 CRD 变得更简单](https://developer.aliyun.com/article/719215)

