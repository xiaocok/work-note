[TOC]



## kubebuilder-gen标记(注解)



### 更新CR的Status

> 设计subresource风格的status

k8s中`status subresource`的使用规范：

1. 用户只能指定一个CR实例的spec部分。

2. CR实例的status部分由控制器进行变更。



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



### kubectl get 命令行显示的字段

> 允许在`kubectl get`命令中显示crd的更多字段

```go
//+kubebuilder:printcolumn:JSONPath=<string>
//+kubebuilder:printcolumn:JSONPath=".status.replicas",name=Replicas,type=string
```

可选参数：

1. `JSONPath`显示的字段
2. `name`当前列标题
3. `format`:当前列格式
4. `priority=<int>`当前列的优先级
5. `type=<string>`当前列的类型



kubectl get 时显示crd的status.replicas：

```shell
// +kubebuilder:printcolumn:JSONPath=".status.replicas",name=Replicas,type=string
```

添加后，使用kubectl get，即可以显示出对应的Replicas列的数据

```go
type CronjonStatus struct {
   State    string `json:"state,omitempty"`
   // 副本数：该字段即为上面配置的需要显示的字段
   Replicas string `json:"replicas,omitempty"`
}

// +kubebuilder:object:root=true
// +kubebuilder:subresource:status
// +kubebuilder:printcolumn:JSONPath=".status.replicas",name=Replicas,type=string
// Cronjob is the Schema for the Cronjobs API
type Cronjob struct {
   metav1.TypeMeta   `json:",inline"`
   metav1.ObjectMeta `json:"metadata,omitempty"`

   Spec   CronjobSpec   `json:"spec,omitempty"`
   Status CronjobStatus `json:"status,omitempty"`
}
```





### kubectl get 使用别名SHORTNAMES

```go
// +kubebuilder:resource:scope="Cluster"/"Namespaced",shortName=shortName1;shortName2,categories=categories1;categories2
```

常用参数：

1. scope:资源范围

   - Namespaced：命令空间的资源，默认值。

   - Cluster：非命名空间资源（于node类似，不需要指定命名空间）。若不指定，则默认为命名空间资源。k8s中node、pv等资源是集群级别的，它们没有namespace字段，因此查询node资源时也无需规定要从哪个namespace查。

2. shortName：定义CRD的别名

   可以定义多个(使用分号隔开)，例如k8s的pod则为shortName=pod;pods

3. categories：资源组的别名

   配置与shortName类似。

4. 每个配置项使用逗号隔开。

5. 可以配置多个项，也可以只配置一项

   ```go
   配置三项，同时shortName配置了多个
   // +kubebuilder:resource:scope="Namespaced",shortName=sub;subs,categories=clusternet
   
   配置三项，shortName配置了一个
   // +kubebuilder:resource:scope="Namespaced",shortName=desc,categories=clusternet
   
   只配置一项
   // +kubebuilder:resource:shortName=sub;subs
   ```

   



### 参考

- [Kubebuilder中的kubebuilder-gen标记（注解）](https://blog.csdn.net/XiaoYunKuaiFei/article/details/121057499)
- [kubebuilder2.0学习笔记——进阶使用](https://segmentfault.com/a/1190000020359577)
- [github.com/clusternet/apis/helm.go](https://github.com/clusternet/apis/blob/c4d0c0cb194cb3f75765ae0b6c484a46adcd3774/apps/v1alpha1/helm.go)
- [github.com/clusternet/apis/subscription.go](https://github.com/clusternet/apis/blob/c4d0c0cb194cb3f75765ae0b6c484a46adcd3774/apps/v1alpha1/subscription.go)



