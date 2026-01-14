

git学习资料

学习网站：https://learngitbranching.js.org/?locale=zh_CN

不会的话去查一下指令：https://www.runoob.com/git/git-tutorial.html

b站上还有一个视频收藏了，之后去看看



# 使用git同步idea和远程开发机会遇到的问题

## 拉取远程仓库之后切换版本

1. 查询（在拉取的仓库的文件夹下面）

   ```bash
   # 1.1 查询所有分支
   # 拉取远程所有标签（关键：标签默认不会自动同步）
   git fetch origin --tags
   # 拉取远程所有分支（确保分支列表最新）
   git fetch origin
   
   # 1.2 查询需要切换的分支存不存在
   # 查找含 5.4.3 的标签（最常用，版本一般打标签）
   git tag | grep "5.4.3"
   # 查找含 5.4.3 的分支（备用，若版本以分支形式存在）
   git branch -r | grep "5.4.3"
   ```

2. 切换分支（这部分刚创建的时候可以不看，直接走3）

   ```bash
   # tag切换
   # 切换到 5.4.3 标签（根据实际标签名调整，如 v5.4.3 则写 v5.4.3）
   git checkout 5.4.3
   # （可选）若需要基于该标签开发，创建并切换到新分支
   git checkout -b my-5.4.3-branch 5.4.3
   
   # 分支切换
   # 切换到远程 5.4.3 分支（本地自动创建对应分支）
   git checkout 5.4.3
   # 或显式指定远程分支
   git checkout -b 5.4.3 origin/5.4.3
   ```

3. 切换分支

   ```bash
   # 1. 基于远程 5.4.3 分支创建并切换到本地 5.4.3 分支（自动关联远程）
   git checkout 5.4.3
   # （若上述命令报错，执行下面的显式命令）
   git checkout -b 5.4.3 origin/5.4.3
   
   # 2. 拉取远程 5.4.3 分支的最新代码（确保本地代码与远程一致）
   git pull origin 5.4.3
   
   
   # 3.检查版本切换
   # 查看当前分支（应显示 * 5.4.3）
   git branch -v
   # 查看分支关联的远程分支（应显示 origin/5.4.3）
   git branch -vv
   ```

## 将公司代码配置为上游

1. 没有远程（`upstream`），但是公司代码是origin

```bash
# 1. 查看当前本地仓库关联的所有远程仓库地址（-v 是 verbose 缩写，显示详细信息）
# 作用：确认当前 origin 指向的是公司主仓库 zstackio/zstack
[root@172-20-19-106 zstack]# git remote -v
origin  http://dev.zstack.io:9080/zstackio/zstack (fetch)
origin  http://dev.zstack.io:9080/zstackio/zstack (push)

# 2. 删除本地仓库中名为 "origin" 的远程仓库关联
# 作用：清除原来指向公司主仓库的 origin 别名，为后续关联个人仓库做准备
[root@172-20-19-106 zstack]# git remote remove origin

# 3. 添加名为 "upstream" 的远程仓库关联，指向公司主仓库
# 作用：保留公司主仓库的关联（命名为 upstream 是行业惯例，代表上游主仓库），方便后续从主仓库拉取最新代码
[root@172-20-19-106 zstack]# git remote add upstream http://dev.zstack.io:9080/zstackio/zstack

# 4. 添加名为 "origin" 的远程仓库关联，指向你的个人仓库
# 作用：将 origin（默认推送/拉取的远程仓库）改为你的个人仓库 haoqian.li/zstack.git
[root@172-20-19-106 zstack]# git remote add origin http://dev.zstack.io:9080/haoqian.li/zstack.git

# 5. 再次查看远程仓库关联，验证修改是否生效
# 作用：确认 origin 已指向个人仓库，upstream 指向公司主仓库，符合预期
[root@172-20-19-106 zstack]# git remote -v
origin  http://dev.zstack.io:9080/haoqian.li/zstack.git (fetch)
origin  http://dev.zstack.io:9080/haoqian.li/zstack.git (push)
upstream        http://dev.zstack.io:9080/zstackio/zstack (fetch)
upstream        http://dev.zstack.io:9080/zstackio/zstack (push)
```

2. 没有远程，本地是自己的分支

```bash
# 1. 先查看当前远程仓库关联（确认 origin 是你的个人分支，无 upstream）
git remote -v

# 2. 直接添加 upstream 远程，指向公司主分支（无需动 origin，保留你的分支）
# 作用：upstream 是行业惯例的「上游主仓库」别名，专门指向公司公共主分支
git remote add upstream http://dev.zstack.io:9080/zstackio/zstack

# 3. 验证结果（确认 origin=你的分支，upstream=公司主分支）
git remote -v
```

3. 有上游，之前整错了

```bash
# 1. 查看当前远程（确认旧的 upstream 地址）
git remote -v

# 2. 直接修改 upstream 的地址为公司主分支（无需删除，一步到位）
git remote set-url upstream http://dev.zstack.io:9080/zstackio/zstack

# 3. 验证修改结果
git remote -v
```

## 创建修改分支到服务器上面跑

### 本地开发机操作

在zstack文件夹下面打开git bash

1. 查看本地 / 远程分支（确认当前分支状态）

   ```bash
   # 查看本地所有分支（*标注当前所在分支）
   git branch
   # 查看本地+远程所有分支（含远程仓库分支）
   git branch -a
   ```

   **作用**：确认当前分支（如 main/master），以及远程仓库已存在的分支，避免创建重复分支

2. 创建并切换到新分支（这一步也可以在idea里面实现）

   ```bash
   git checkout -b helloworlddemo
   ```

   **拆解说明**：

   - `checkout -b`：等价于 `git branch helloworld-test`（创建分支） + `git checkout helloworld-test`（切换分支）；

   - `helloworld-test`：指定要创建的分支名称；

   作用：创建名为 helloworld-test 的分支并立即切换到该分支；

3. 提交修改

   ```bash
   # 如果有未提交的修改，先提交
   git add .
   git commit -m "添加helloworld插件"
   ```

4. 推送新分支到远程仓库

   ```bash
   git push -u origin helloworlddemo
   ```

   **拆解说明**：

   - `push`：将本地分支推送到远程仓库；
   - `-u`：关联本地分支与远程分支（后续推送 / 拉取可直接用`git push/pull`，无需指定分支）；
   - `origin`：远程仓库的默认别名（若仓库别名非 origin，需替换）；
   - `helloworld-test`：要推送的分支名称；

   **作用**：将本地 helloworld-test 分支推送到远程仓库，并建立本地与远程分支的关联

### 云服务器操作

1. 拉取远程最新分支列表（同步远程分支信息）

   ```bash
   git fetch origin
   ```

   **作用**：从远程仓库（origin）拉取最新的分支、提交记录等信息（仅同步信息，不合并代码），确保云服务器能识别新创建的 helloworld-test 分支；

2. 查看所有分支（确认远程分支已同步）

   ```bash
   git branch -a
   ```

   **作用**：验证远程分支`remotes/origin/helloworld-test`是否存在；

3. 切换到 helloworld-test 分支

   ```bash
   git checkout helloworlddemo
   ```

   **作用**：将云服务器的本地代码目录切换到 helloworld-test 分支（若为首次切换，会基于远程分支创建本地同名分支并切换）；

   **补充**：若提示 “找不到分支”，可执行 `git checkout -b helloworld-test origin/helloworld-test`（手动关联远程分支创建本地分支）；

4. `git branch -v` 可以用这个看一下是不是当前分支

4. 拉取分支最新代码（可选，确保代码最新）

   ```bash
   git pull origin helloworlddemo
   
   # 如果冲突的话
   # 1. 暂存当前修改（不会丢！）
   git stash
   # 2. 拉取远程最新代码
   git pull origin helloworlddemo
   # 3. 恢复你的修改（可能会有冲突，需手动解决）
   git stash pop
   ```
   
   **作用**：将远程 helloworld-test 分支的最新代码拉取到云服务器本地分支，避免代码过期；

## 将公司代码同步到本机

步骤 1：先确认 upstream 有哪些分支（关键！避免分支名错误）

首先查看 upstream 远程仓库的所有分支，确认 5.4.3 分支是否存在、名称是否准确：

```bash
git branch -r | grep upstream  # 列出所有upstream的远程分支
```

输出会类似：`upstream/5.4.3`、`upstream/master` 等，先确认能看到 `upstream/5.4.3`。

步骤 2：切换到 upstream 的 5.4.3 分支（分 2 种场景）

场景 A：本地已有 5.4.3 分支（你的日志显示本地有 5.4.3 分支）

直接切换到本地 5.4.3 分支，然后**拉取 upstream/5.4.3 的最新代码**并关联：

```bash
# 1. 切换到本地5.4.3分支
git checkout 5.4.3

# 2. 关联 upstream/5.4.3（让本地5.4.3跟踪远程upstream的5.4.3）
git branch --set-upstream-to=upstream/5.4.3 5.4.3

# 3. 拉取 upstream/5.4.3 的最新代码（覆盖本地5.4.3，确保和upstream一致）
git pull upstream 5.4.3
```

场景 B：若本地 5.4.3 分支想重建（可选，确保纯净）

如果想放弃本地 5.4.3 分支的修改，完全基于 upstream/5.4.3 重建：

```bash
# 1. 删除本地旧的5.4.3分支（先切换到其他分支，比如helloworlddemo）
git checkout helloworlddemo
git branch -D 5.4.3  # -D是强制删除，确保删除干净

# 2. 基于upstream/5.4.3创建并切换新的本地5.4.3分支
git checkout -b 5.4.3 upstream/5.4.3
```

步骤 3：验证是否成功切换到 upstream 的 5.4.3

执行以下命令，确认当前分支、关联的远程分支、代码版本是否匹配：

```bash
# 1. 查看当前分支及关联的远程分支
git branch -vv

# 2. 查看当前代码的提交记录（确认是upstream/5.4.3的最新提交）
git log --oneline -5  # 显示最近5条提交
```

## 将代码回滚到历史版本

**在已经 push 到远程仓库的分支上，想要回滚到某个之前的提交（比如从 `test3_1` 回滚到 `test2_compiled_success`）**。

- 你当前在 `helloworlddemo` 分支上，但是更新的代码不理想，想要回滚回去
- 现在想**将当前分支回滚到 `test2_compiled_success` 这个提交**

1. 查看提交历史（确认目标提交）：`git log --oneline -10`

   - 找到 `test2_compiled_success` 对应的 commit hash（例如：`abc1234`）
2. 将当前分支重置到目标提交 **注意使用ID** 不是commit信息 ：`git reset --hard comit_id`

   - `--hard` 会丢弃之后的所有更改（包括未提交的），只保留该提交的状态。
