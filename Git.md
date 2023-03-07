 Git :分布式 版本 控制工具

###### 个人信息的配置:

用户名:

```shell
git config --global user.email "邮箱"
```

邮箱:

```shell
git config --global user.name "用户名"
```



###### 版本控制的步骤:

​		1.进入要管理的文件夹

​		2.初始化   

```shell
git init
```

​		3.管理

```shell
git add 文件名    .(代表将该目录下所有未管理的文件管理起来)
```

​		4.生成版本 

```shell
git commit -m '版本信息'
```

​		5.查看版本记录

```shell
git log  / git reflog 查看回滚之前的版本号
```

###### git三大区域

​      工作区,暂存区,版本库

###### 版本的回滚

```
git reset --hard 版本号
```

![1677668451058](C:\Users\2541843624\AppData\Roaming\Typora\typora-user-images\1677668451058.png)

```
git checkout -- 文件名    （对修改的文件有效）
```

```
git reset HEAD 文件名   由暂存区回到工作区 
```

```
git branch 查看目前所处的分支
```

```
git branch 分支名      创建一个新的分支
```

```
git checkout 分支名      切换到该分支
```

```
git branch -d 分支名    删除掉分支
```

```
git merge 分支名     将该分支合并到目前所处的分支
```

```
git remote add origin  远程仓库的url,将远程仓库的名字定义为origin
```

```
git push -u origin master    将master分支推送到远程仓库
```

```
git clone 仓库地址   将远程仓库克隆到本地	
```

```
git pull 仓库地址 想要拉取的分支

git fetch 仓库地址 想要拉取的分支
git merge 仓库地址/想要拉取的分支
```

```
git rebase   将多个提交记录整合成一个提交记录
```

