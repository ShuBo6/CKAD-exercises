# 练习[CKAD-exercises](https://github.com/dgkanatsios/CKAD-exercises)的笔记

系统的练习一遍。。。否则直接裸考会因为两个小时不够用


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

4. 善用帮助，例如我要为一个pod设置node的label选择器，我可以通过`kubectl explain pods.spec| grep node`来查询nodeSelector这个tag是如何拼写。当然这样很慢。为了提升速度还是记住。

5. deployment 篇会考deployment创建，replicas设置，deployment版本管理（查看revision，更新，回滚,暂停，恢复）,扩容,自动扩容，金丝雀发布
```shell
    # deployment 创建较简单
     k rollout [status|history|undo] deploy deployname
     # 回滚到特定的版本，带 --to-revision参数指定版本号
     kubectl rollout undo daemonset/abc --to-revision=3
     # 暂停
     kubectl rollout pause daemonset/abc 
     # 恢复
     kubectl rollout resume daemonset/abc 
    # 查看某一个版本的详情，需要指定--revision=参数
     k rollout history deploy nginx-test-deploy --revision=4
# 扩容replicas
     kubectl scale --replicas=5 deploy nginx-test-deploy
     # 自动扩容
    kubectl autoscale deployment nginx-test-deploy --min=5 --max=10 --cpu-percent=80
    # 查看横向自动扩容的配置 horizontalpodautoscalers缩写（hpa）
    kubectl get hpa nginx
    # 金丝雀发布，这个难以避免要手动写一些deployment.yaml的配置，尤其volume挂

```


## 关于vim
以下是我目前对vim的掌握,需要注意的是，vim编辑yaml的时候统一使用空格，tab会带来意料以外的错误。

### 正常模式
进入vim默认为正常模式，或按esc从其他模式回到正常模式
常用快捷键：
```
hjkl移动光标，ZZ保存退出，ZQ不保存退出
y 复制
x 剪贴
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
set number/nonumber 显示/隐藏行号
set paste 粘贴模式
### visual模式（按v进入）
可以使用快捷键进行行选择，复制粘贴
### 列visual模式（按ctrl-v进入）
常用于列空格填充，顶格子
搭配shift+i,双击esc自动填充
可以使用快捷键进行行选择，复制粘贴
