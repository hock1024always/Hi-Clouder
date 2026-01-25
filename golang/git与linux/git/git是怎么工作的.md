# Git是怎么工作的

git是通过三个对象的引用来工作的：

- Commit：每次改动代码之后的提交
- Tree：表示commit发生时的目录
- Blob：存储了文件的具体样子

## 引用关系

一句话来说：Commit对象引用一个Tree对象，这个Tree对象又引用了一个Blob对象

### 具体情况

我们先commit一个text1文件，输入`cat-file -p 哈希值`命令和提交可以发现：

- commit对象里面很明显是有tree对象的哈希值

  ![image-20260125182937756](D:\github\Hi-Clouder\golang\git与linux\git\图片\image-20260125182937756.png)

- Tree对象引用的是一个Blob对象

  ![image-20260125184246066](D:\github\Hi-Clouder\golang\git与linux\git\图片\image-20260125184246066.png)

- Blob对象存储的是文件的内容

  ![image-20260125184327738](D:\github\Hi-Clouder\golang\git与linux\git\图片\image-20260125184327738.png)

## 好处

![image-20260125184718273](D:\github\Hi-Clouder\golang\git与linux\git\图片\image-20260125184718273.png)

- 每次提交本质上是要存所有变化的，但是这样操作的话，那些没有发生变化的内容我们引用原来的Blob就可以了

### 具体情况

我们先进行第二次提交，哈希值是957c...

- 然后我们查看，发现有一个parent字段，这个字段就是第一次Commit的哈希值（这个行表示的是这次commit是从哪一次衍生而来的）

![image-20260125185447149](D:\github\Hi-Clouder\image-20260125185447149.png)

- 然后我们看一下这次Tree的内容

  ![image-20260125185811048](D:\github\Hi-Clouder\image-20260125185811048.png)

  - 可以发现它引用了两个Blob，一个上次就存在的代表的text1，一个新增的代表的是text2

结构是这样的 ：

![image-20260125185922313](D:\github\Hi-Clouder\image-20260125185922313.png)

## 说明

1. Blob对象一旦被创建就**不会再被修改或者删除**

   1. 也就是说就算我删除或者修改了text2，这个Blob对象依旧不变的存在
      1. 为什么呢？我的理解是为了支持回滚
   2. 那我们怎么才能给这个blob删除掉呢？
      1. `git resit`: 先删除提交text2的这次commit，让模型的Blob对象成为没有被引用的悬空对象（有点三色标记法的感觉了）
      2. `git gc --prune=now`再删除没有被引用的对象

2. branch是什么？

   1. branch本质上并不是一个对象，我们每次切到一个branch本质上是跳到某次commit上面

   2. 但是我们可以在`.git/heads/`文件夹下面找到关于分支的内容

      1. 可以看到我们这个main分支的文件里面就是记录的就是最新的commit的哈希值

      ![image-20260125190510890](D:\github\Hi-Clouder\image-20260125190510890.png)****

# 下一步

太晚要润了，下周末要学习的有趣内容：

- 本次内容的补充
  - diff或者git status是怎么做的
  - reset是怎么做的
  - 上游 下游 分支究竟和commit的联系是什么
- 关于本地 远程仓库的结构
  - merge rebase变基是什么样子的，啥时候用哪个
  - fetch pull push是咋回事，为啥推的时候就直接退了拉的时候要先fetch
  - fork出来的仓库 本地远程仓库 缓冲区的关系
