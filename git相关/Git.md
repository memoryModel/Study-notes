#Git使用

## 命令行指令

##### Git global setup（Git全球设置）

> ```java
> git config --global user.name "你的名称"
> git config --global user.email "你的邮箱"
> ```

##### Create a new repository（创建一个新的存储库）

> ```java
> git clone “git地址（git@github.com:memoryModel/springboot-model.git）”
> cd “项目地址（springboot-model）”
> touch README.md
> git add README.md
> git commit -m "add README"
> git push -u origin master
> ```

##### Existing folder（现有文件夹）

>```java
>cd "本地项目地址"
>git init # 将项目初始化为仓库
>git remote add origin http://36.110.59.88:9000/live/WxMobileSite.git # 关联GitHub仓库
>git add . // 将本目录下的所有文件添加到仓库
>git commit -m "Initial commit"  // 提交文件
>git push -u origin master // 向GitHub仓库推送
>```

##### Existing Git repository（现有的Git存储库）

> ```java
> cd existing_repo
> git remote rename origin old-origin
> git remote add origin http://36.110.59.88:9000/live/WxMobileSite.git
> git push -u origin --all
> git push -u origin --tags
> ```

## 上传实例

1. git add .
2. git commit -m "注释"
3. git push

