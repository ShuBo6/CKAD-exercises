# 练习中的一些tips

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

