[TOC]



# beego配置pprof

## Graphviz环境准备

### 下载Graphviz

> 下载地址：https://graphviz.org/download/

> 例如：下载windows版本

> [stable_windows_10_cmake_Release_x64_graphviz-install-2.47.2-win64.exe](https://gitlab.com/api/v4/projects/4207231/packages/generic/graphviz-releases/2.47.2/stable_windows_10_cmake_Release_x64_graphviz-install-2.47.2-win64.exe) 

### 安装Graphviz

> 执行exe程序安装，选择自动加入环境变量。否则需要手动将安装目录/bin 添加至系统环境变量。

> 否则golang生成图片时，无法找到对应的Graphviz工具。

参考：[Graphviz安装及简单使用](https://www.cnblogs.com/shuodehaoa/p/8667045.html)



## beego项目

### 配置pprof

配置路径：/conf/app.conf

```ini
#启动管理模式
EnableAdmin=true
#监听IP
AdminAddr = "localhost"
#监听端口
AdminPort = 8888
```

参考：[beego 参数配置](https://www.cnblogs.com/cz-xjw/p/10960658.html)

###　启动beego

```go
func main() {
	beego.Run()
}
```

```shell
go run main.go
```

### 访问web页面，生成pprof文件

* 地址为上面管理模式Admin的配置地址

  ```shell
  http://localhost:8888
  ```

* 点击按钮，触发生成pprof文件

  ```shell
  Performance profiling -> get cpuprof
  
  Performance profiling -> get memprof
  ```



### 执行调试pprof文件

* 命令行，控制台查看

  ```shell
  go tool pprof main.exe cpu-13792.pprof
  ```

  ```shell
  go tool pprof cpu-13792.pprof
  ```

  **交互命令：**

  * top 10				消耗最高的10个函数

  * list 10	

  * pdf					生成pdf文件

  * png					生成png图片

  * svg		 			生成svg文件，浏览器可以访问

  * tree					函数调用链，包含上下文

  * web					web浏览器方式的打开svg文件

  * weblist main	函数以及源码，参数为函数名

  * peek main		

  

  **CPU profile 的 top 命令**

  ```shell
  (pprof) top
  Showing nodes accounting for 29.92s, 94.56% of 31.64s total
  Dropped 117 nodes (cum <= 0.16s)
  Showing top 10 nodes out of 33
        flat  flat%   sum%        cum   cum%
      28.52s 90.14% 90.14%     28.58s 90.33%  runtime.cgocall
       0.81s  2.56% 92.70%      0.82s  2.59%  runtime.stdcall1
       0.24s  0.76% 93.46%      0.25s  0.79%  runtime.stdcall3
       0.16s  0.51% 93.96%     29.10s 91.97%  internal/poll.(*FD).writeConsole
       0.05s  0.16% 94.12%     29.28s 92.54%  internal/poll.(*FD).Write
       0.04s  0.13% 94.25%      0.18s  0.57%  runtime.findrunnable
       0.03s 0.095% 94.34%      0.18s  0.57%  runtime.mallocgc
       0.03s 0.095% 94.44%      0.25s  0.79%  runtime.mcall
       0.02s 0.063% 94.50%     29.49s 93.20%  log.(*Logger).Output
       0.02s 0.063% 94.56%     29.71s 93.90%  log.Println
  (pprof)
  ```

  信息：

  - 显示的节点在总共 31.64s 的抽样中，占 29.92s，比例 94.56%
  - 在 33 个样本中显示了 top 10
  - 列表解释（cum: cumulative 堆积的）
    - flat: 在给定函数上运行耗时
    - flat%: 在给定函数上运行耗时总比例
    - sum%: 给定函数 **累计** 使用 CPU 总比例
    - cum: 当前函数 **以及包含子函数** 的调用运行总耗时
    - cum%: 同上的 CPU 运行耗时总比例
    - 最后一列为函数名称

  

  **heap profile 的 top 命令**

  ```she
  (pprof) top
  Showing nodes accounting for 10.04GB, 100% of 10.04GB total
        flat  flat%   sum%        cum   cum%
     10.04GB   100%   100%    10.04GB   100%  main.Add
           0     0%   100%    10.04GB   100%  main.main.func1
  (pprof)
  ```

  - inuse_space: 常驻内存占用情况

  - alloc_objects： 内存临时分配情况

    

* web页面查看

  ```shell
  go tool pprof -http=:8080 cpu-13792.pprof
  ```

  打开浏览器http://localhost:8080查看

  * VIEW->Top				   函数消耗CPU列表
  * VIEW->Graph               函数调用图，线框越大越粗表明占用时间越多
  * VIEW->Flame Graph   火焰图，调用顺序是从上到下（有些是从下到上，如火焰一般），每一块代表一个函数，越大代表占用 CPU 时间越长。
  * VIEW->Peek                 在 Top 的基础上显示了调用的上下文文件
  * VIEW->Source             看到了源码，及源码处执行时间
  * VIEW->Dissasemble



## 参考

* [pprof 的使用](https://www.jianshu.com/p/f4690622930d)
* [Graphviz安装及简单使用](https://www.cnblogs.com/shuodehaoa/p/8667045.html)

* [Graphviz官网](https://graphviz.org/download/)

