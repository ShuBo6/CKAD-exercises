# 练习[CKAD-exercises](https://github.com/dgkanatsios/CKAD-exercises)的笔记

系统的练习一遍。。。否则直接裸考会因为两个小时不够用
本训练题中缺少一些具体的pod,container,命名的要求。实际考试中是有具体的命名要求的。要仔细读题
## 章节考点梳理
|序号|章节|知识点|
|--|--|--|
| 1 | Core Concepts (13%) 核心概念|TODO: | 
| 2 | Multi-container Pods (10%) 多容器pods|TODO: |
| 3 | Pod design (20%) pod设计| label,anotation部分:增删改查,标签选择表达式。<br /> deployment 部分： 创建,replicas设置,deployment版本管理（查看revision,更新,回滚,暂停,恢复）,横向扩容,自动横向扩容（hpa）,金丝雀发布。<br />  job 部分：增删改查，配置并行数，完成数量，设置job的deadline  <br /> CronJob 部分:设置schedule，设置job的两种deadline|
| 4 | Configuration (18%) 配置文件| ConfigMap && Secret 部分： configMap的[字面量｜从文件]创建,挂载configMap为[环境变量｜Volume(文件)] <br />  SecurityContext 部分: 设置容器的用户组，capabilities（权能），seccompProfile<br />Requests And limits 部分: 现用现查 <br />serviceAccounts 部分:以serviceAccountName 的方式声明在spec中 |
| 5 | Observability (18%)可观察性配置 | Probe 部分: livenessProbe,readinessProbe,startupProbe 的创建及观察，pod 日志观察，资源使用情况观察|
| 6 | Services and Networking (13%) | TODO:|


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
8. cronjob
    定时任务 使用命令创建时必须要指定schedule
    需要注意的是有以下这样几条表达式规则

    读题仔细区分`startingDeadlineSeconds`和`activeDeadlineSeconds`该用哪一个

    表达式
        在一个区域里填写多个数值的方法：

        逗号（,）表示列举，例如： 1,3,4,7 * * * * echo hello world 表示，在每小时的1、3、4、7分时，打印"hello world"。

        连词符（-）表示范围，例如：1-6 * * * * echo hello world ，表示，每小时的1到6分钟内，每分钟都会打印"hello world"。

        星号（*）代表任何可能的值。例如：在“小时域”里的星号等于是“每一个小时”。

        百分号(%) 表示“每"。例如：*%10 * * * * echo hello world 表示，每10分钟打印一回"hello world"。
    非标准字符
        某些cron程序的扩展版本（如：Quartz Java scheduler）也支持斜线（'/'）操作符，用于表示跳过某些给定的数。例如，
```
        “*/3”在小时域中等于“0,3,6,9,12,15,18,21”等被3整除的数；
        “*/1 * * * *” 表示每分钟
```

```shell

# 文件格式說明，参考https://zh.wikipedia.org/wiki/Cron
# ┌──分鐘（0 - 59）
# │ ┌──小時（0 - 23）
# │ │ ┌──日（1 - 31）
# │ │ │ ┌─月（1 - 12）
# │ │ │ │ ┌─星期（0 - 6，表示从周日到周六）
# │ │ │ │ │
# *  *  *  *  * 被執行的命令
k explain cronjob.spec | grep schedule
```
9. ConfigMap && Secrets
```shell
#   创建config的语法
kubectl create configmap NAME [--from-file=[key=]source] [--from-literal=key1=value1] [--dry-run=server|client|none] [options]


# 这样可以查一下pod中挂载configmap的用法
# 方法1 指定configmap中的某一个kv作为环境变量
k explain pod.spec.containers.env.valueFrom.configMapKeyRef
# 方法2 指定configmap中的所有kv作为环境变量

k explain pods.spec.containers.envFrom.configMapRef

# 挂载configmap到volume作为文件
# 思路： 挂载cofigmap为volumes,首先要声明volumes,volumes是声明在spec下的
k explain pods.spec.volumes.configMap.items
# 思路：显然，volumeMounts是containers下的属性
k explain pods.spec.containers.volumeMounts
# 同理，挂载secret为volumes
k explain po.spec.volumes.secret.items

```
10. SecurityContext（是设置给容器的安全上下文）
```shell
# runAsGroup，runAsUser，capabilities，seccompProfile这几个考的时候查一下文档吧，不多做记忆
# 见名知义
# k explain po.spec.containers.securityContext
```

11. Requests and limits
```shell
# 同样的思路，这个属性是属于容器的resources下的。
k explain po.spec.containers.resources
```
12. ServiceAccounts
```shell
# 服务账号，用户api资源访问控制，默认情况下，每一个namespace下都会有一个名为default的serviceAccount。pod资源创建时如果不指定serviceAccounts他会自动选择default,填入spec.serviceAccountName。

k explain po.spec.serviceAccountName

```
13. Observability 
```shell
kubectl explain pod.spec.containers | grep Probe
# 存活探针livenessProbe（检测应用程序是否被死锁，是否存活）、就绪探针readinessProbe（通过就绪探针控制service会不会将其选择为endpoint）、启动探针startupProbe（解决慢启动程序的探活问题）
k explain po.spec.containers.

# 获取pod的事件，可以用来查probe的报错信息。
k get events 

k get events -A | grep -i "probe failed"|awk '{print $1,$5}'

# 查看pod日志 -f参数是flow的意思
k logs podname [-f]

# 强制立即删除pod。grace-period 是平滑的删除一个资源时的等待时间，只有--force为true的时候--grace-period才能等于0，否则可以设置为1或负数，负数是默认值，即忽略这个option
k delete po poname --force --grace-period=0
# 观察资源使用情况(需要安装metrics-service)
k top resourceName
```
14.  service && network
```shell
# 加一个--expose参数会在创建pod的同时为其创建一个svc
k run nginx --image=nginx --port=80 --expose
# 启动临时容器的来测试svc的连通性，一定要记得加上--restart=Never参数！！！
k run test --rm --image=nginx  -it --restart=Never -- bash -c 'curl nginx'

# 为deploy 暴露6262端口并指向8080
k expose deploy foo --port=6262 --target-port=8080
# TODO: networkPolicy
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

## 关于tmux
tmux在单窗口的shell终端中可以复用打开多个会话。

通常是以ctrl + b 开头然后跟一个指令进行会话操作

例如：
```shell
tmux detach 挂起客户端，回到shell
tmux attach 恢复客户端

tmux split-window [-h (垂直方向切分为两个窗口)]

# c-b 简写代表ctrl+b
c-b ? 查看帮助
c-b U/D/L/R 或者方向键上下左右 切换窗口
c-b w 切换以及管理会话
c-b x 关闭当前窗格
c-b [ 开始屏幕翻页（mac下fn+方向键是翻页）


```