#Git使用总结#
----------
###Git简单流程###
```java
1、在github上创建项目

2、使用git clone https://github.com/xxxxxxx/xxxxx.git克隆到本地

3、编辑项目

4、git add . （将改动添加到暂存区）

5、git commit -m "提交说明"

6、git push origin master 将本地更改推送到远程master分支。
```

这样你就完成了向远程仓库的推送。
###Git使用代理加速命令###
```java
git config --global http.proxy 'socks5://127.0.0.1:1080'   
git config --global https.proxy 'socks5://127.0.0.1:1080'

```
