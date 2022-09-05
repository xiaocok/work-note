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

   





## 开发设计规范

[开发设计规范](开发设计规范.md)

- 协调规则
- kubebuilder-gen标记（注解）
  - 更新CR的Status
  - kubectl get 命令行显示的字段
  - 别名SHORTNAMES
- 





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

- [Kubebuilder中文文档](https://cloudnative.to/kubebuilder/introduction.html)
- [Kubebuilder 使用简介](https://www.bilibili.com/video/BV1zq4y1o7Lg)
- [《 Kubebuilder v2 使用指南 》](https://wqyin.cn/gitbooks/kubebuilder)
- [kubebuilder2.0学习笔记——进阶使用](https://segmentfault.com/a/1190000020359577)
- [k8s crd operator踩坑点](https://blog.csdn.net/hahachenchen789/article/details/116274973)
- [深入解析 Kubebuilder：让编写 CRD 变得更简单](https://developer.aliyun.com/article/719215)

