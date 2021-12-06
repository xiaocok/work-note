# KebeBuilder

## 环境准备

[环境准备](环境准备.md)



##  初始化准备

在初始化之前，首先想好CRD资源的名称，名称不要与现有的资源名称冲突，api groupVersion，建议使用自定义的api groupVersion与内置的区别开。



**查看现有的所有resource**：

```bash
kubectl api-resources -o wide
```

**查看现有的api groupVersion**：

```bash
kubectl api-versions
```

我的CRD 名称定为: **Unit**, api groupVersion定为 **custom/v1**



## 项目创建

**官方的CronJob例子**

> 这里只是参考

```bash
# 创建工程
# 后续api的group即为：<group>.tutorial.kubebuilder.io
kubebuilder init --domain tutorial.kubebuilder.io

# 创建一个Api
# 指定GVK
kubebuilder create api --group batch --version v1 --kind CronJob
```



**生成Unit项目**

```bash
# domian可自定义
kubebuilder init --domain unit.kubebuilder.io

# 为CRD生成API groupVersion
kubebuilder create api --group custom --version v1 --kind Unit
```



## 生成的代码

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

   

3. 







### 参考

- [Kubebuilder 使用简介](https://www.bilibili.com/video/BV1zq4y1o7Lg)
- [《 Kubebuilder v2 使用指南 》](https://wqyin.cn/gitbooks/kubebuilder)