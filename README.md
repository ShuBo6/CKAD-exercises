# 练习[CKAD-exercises](https://github.com/dgkanatsios/CKAD-exercises)的笔记

系统的练习一遍。。。否则直接裸考会因为两个小时不够用
本训练题中缺少一些具体的pod,container,命名的要求。实际考试中是有具体的命名要求的。要仔细读题
## 章节考点梳理
|序号|章节|知识点|
|--|--|--|
| 1 | Core Concepts (13%) 核心概念|TODO: | 
| 2 | Multi-container Pods (10%) 多容器pods|TODO: |
| 3 | Pod design (20%) pod设计| label,anotation部分:增删改查,标签选择表达式。<br /> deployment 部分： 创建,replicas设置,deployment版本管理（查看revision,更新,回滚,暂停,恢复）,横向扩容,自动横向扩容（hpa）,金丝雀发布。<br />  job 部分：增删改查，配置并行数，完成数量  <br />TODO: CronJob 部分:|
| 4 | Configuration (18%) 配置文件| TODO: |


## 官方文档参考

1. [cheatsheet起手](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)



## 写给自己的一些考前小抄

1. --dry-run=client -o yaml 这个要打熟练

2. annotate label操作要熟练。标签选择表达式 ,
    例如 :

    ```shell
    k get nodes --show-labels

     k label po -l "app in (v1,v2)" app=v3
     k annotate po -l "app in (v1,v2)" tier=web

     k annotate po nginx1 --list //冷门容易忘
    ```

3. 删除label或annotate 用减号。
    例如：

```shell
     k label po nginx1 app-
     k annotate po nginx1 app-
```

4. 善用帮助,例如我要为一个pod设置node的label选择器,我可以通过`kubectl explain pods.spec| grep node`来查询nodeSelector这个tag是如何拼写。当然这样很慢。为了提升速度还是记住。

5. deployment 篇会考deployment创建,replicas设置,deployment版本管理（查看revision,更新,回滚,暂停,恢复）,扩容,自动扩容,金丝雀发布
```shell
    # deployment 创建较简单
     k rollout [status|history|undo] deploy deployname
     # 回滚到特定的版本,带 --to-revision参数指定版本号
     kubectl rollout undo daemonset/abc --to-revision=3
     # 暂停
     kubectl rollout pause daemonset/abc 
     # 恢复
     kubectl rollout resume daemonset/abc 
    # 查看某一个版本的详情,需要指定--revision=参数
     k rollout history deploy nginx-test-deploy --revision=4
# 扩容replicas
     kubectl scale --replicas=5 deploy nginx-test-deploy
     # 自动扩容
    kubectl autoscale deployment nginx-test-deploy --min=5 --max=10 --cpu-percent=80
    # 查看横向自动扩容的配置 horizontalpodautoscalers缩写（hpa）
    kubectl get hpa nginx
```
6. 金丝雀发布
    这个难以避免要手动写一些deployment.yaml的配置,尤其volume挂载。这里简单描述下思路。
    
    创建v1版本的deploy,replicas=3,挂载Mount,initContainers写入v1版本的html,为pod模板添加version：v1的label；

    复制一份yaml创建v2版本修改html为v2,replicas=3,修改version：v2的label。以他们公共的app:nginx 为label创建svc.

    访问svc,验证金丝雀发布是否成功。业务上当金丝雀stable之后,将v2的replicas扩到4,v1缩掉即可
7. job 
    关于job的使用，要熟练job.yaml的快速编辑。

    尽可能使用`-- sh -c '命令'`避免语义歧议

  

    要搞明白`.spec.completions`和`spec.parallelism`的 含义，参考官方文档：https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/#parallel-jobs

```shell

# 查看pod的日志,-f流动
k logs xxxx -f
# automatically terminated by kubernetes，单位为秒
k explain job.spec |grep activeDeadlineSeconds -A 10

```


## 关于vim
以下是我目前对vim的掌握,需要注意的是,vim编辑yaml的时候统一使用空格,tab会带来意料以外的错误。

### 正常模式
进入vim默认为正常模式,或按esc从其他模式回到正常模式
常用快捷键：
```
hjkl移动光标,ZZ保存退出,ZQ不保存退出
y 复制
x 剪贴
dd 剪贴行
p 粘贴
0 回到行首
$ 到行尾
gg 到文首
GG 到文末
ctrl-d 光标跳下一页
i 当前光标进入插入模式
a 下一个光标进入插入模式
shift-A 到行尾进入插入模式
```

### 输入冒号进入命令模式：
常用命令：
:set number/nonumber 显示/隐藏行号
:set paste 粘贴模式
:%s/foo/bar/g 全文替换
### visual模式（按v进入）
可以使用快捷键进行行选择,复制粘贴
### 列visual模式（按ctrl-v进入）
常用于列空格填充,顶格子
搭配shift+i,双击esc自动填充
可以使用快捷键进行行选择,复制粘贴
