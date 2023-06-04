## tag

Git tag是用来标记Git历史上重要的特定时间点，通常用于跟踪版本或标记生产准备就绪的状态

tag与Git分支类似，但不同于分支，标签不会更改，并且提供了一种方便的方法来访问代码的特定版本

总而言之， tag是一个对于历史提交记录的快照

一般情况下，每上一次生产环境 就需要打上一个新的tag，也就是需要发布一个新的版本

```shell
# 以下创建的标签都是本地标签，在推送的时候要和修改一起推送到远程分支上

# 创建一个轻量级标签
# ps: tag名虽然可以任意起 但是一般以v开头，遵循semver version 不需要使用引号进行包裹
git tag v1.0.1

# 创建一个附注标签
git tag [-a] v1.0.1 -m 'release message'

# 查看所有的标签 --- 此时是看不到描述信息的
git tag

# 查看某一个具体的tag的详细信息 --- 此时可以看到修改内容和tag的描述信息
git show v1.0.1

# 查看某一个具体的tag
git tag -l 'v1.0.1'

# 在查看tag的时候使用通配符
# ps: tag是使用引号进行包裹的
git tag -l 'v1.*'

# 移除某一个具体的标签
git tag -d v1.0.1
```

![image.png](https://s2.loli.net/2023/01/30/mX7hUoNSJu8VOlp.png) 



## diff

```shell
# 查看某一个文件的修改记录
git blame <file_name>

# 类unix系统中，原生diff命令的使用
# 查看两个文件之间的内容差异
diff foo.js baz.js
# 输出两个文件之间的差异
2c2 # 2c2 表示两个文件之间的修改都是位于第二行
< console.log('Hello Git') 
---
> console.log('Hello Vue')

# 查看两个文件之间的详细差异 需要加上参数 -u
diff -u  foo.js baz.js
# --- 表示的是源文件 在这里就是foo.js
--- foo.js      2023-01-30 22:01:47 
# +++ 表示的是目标文件 在这里就是baz.js
+++ baz.js      2023-01-30 22:02:49
# -1,2 源文件表示的内容是从第一行开始的连续两行
# +1,2 目标文件表示的内容是从第一行开始的连续两行
@@ -1,2 +1,2 @@
 # 假设以目标文件为准
 # 源文件移除Hello Git，加上Hello Vue就是目标文件
 console.log('Hello World')
-console.log('Hello Git') 
+console.log('Hello Vue')

# git diff的使用
# 查看工作区和暂存区中文件的区别 
git diff 

# 查看工作区和某一个具体提交之间文件的差异
# HEAD  表示最新一次的提交记录
# 参数也可以是某一个具体的提交
git diff <HEAD|commit_id>

# 查看暂存区和某一个具体的提交之间文件的差异
# 参数commit_id 可以省略 
# 如果省略了commit_id 表示和最新的一次提交记录进行比对
# 如果不省略commit_id 表示和某一个具体的提交记录进行比对
git diff --cached [<commit_id>]
```

