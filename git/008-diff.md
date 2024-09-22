## blame

`git blame <file>` 用于标识`<file>`中每一行是由谁，在什么时候，在那个CommitID 中 进行了修改

```shell
git blame foo.txt
```

输出结构类似如下:

```shell
59749d8b (klaus 2024-09-21 08:48:49 +0800 1) Hello World
9258c188 (klaus 2024-09-21 08:48:56 +0800 2) Hello Vue
67882475 (klaus 2024-09-21 08:49:15 +0800 3) Hello React
67882475 (klaus 2024-09-21 08:49:15 +0800 4) Hello Node
5fcdcfd2 (klaus 2024-09-21 08:49:26 +0800 5) Hello Nuxt
5fcdcfd2 (klaus 2024-09-21 08:49:26 +0800 6) Hello Next
```



## diff

```shell
diff <source_file> <target_file>

diff -u <source_file> <target_file>
```

```shell
2,3c2,3
< bbbb
< cccc
\ No newline at end of file
---
> cccc
> dddd
\ No newline at end of file
```



```shell
--- a.txt       2024-09-21 12:43:00
+++ b.txt       2024-09-21 12:43:13
@@ -1,3 +1,3 @@
 aaaa  # 相同部分 
-bbbb  # 存在于a.txt中的部分
-cccc
\ No newline at end of file
+cccc  # 存在于b.txt中的部分
+dddd
\ No newline at end of file

# 表示a.txt 删除bbbb 和 cccc后，再加上cccc 和 dddd 就能得到 b.txt
```

其中 

1. `-` 表示 源文件，`+`表示目标文件
2. `@@ xxxx @@`是固定格式

3. `1,3` => 表示从第一个行开始的连续3行 => 其实就是整个文件





diff a b？？？？