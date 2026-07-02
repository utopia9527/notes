- [参考链接](https://blog.csdn.net/qq_37140632/article/details/85786089?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-85786089-blog-110071195.pc_relevant_antiscanv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-85786089-blog-110071195.pc_relevant_antiscanv3&utm_relevant_index=2)

- 初始化仓库

  - git init
  - git add 
  - git commit 
  - git remote add origin `git地址`

- 想要分支之间互不干扰，有两种方式

  - git add、git commit 然后进行切换
  - `git stash`
    - 隐藏当前工作现场，这个时候用 git status 查看工作区是干净的，所以就可以放心地去其他分支了。用 `git stash list`可以查看隐藏起来的工作现场
  - 恢复
    - 用 `git stash apply`恢复，但是恢复后，stash 内容并不删除，需要用 `git stash drop`来删除；
    - 用 `git stash pop`，恢复的同时把stash内容也删了，这时候用 `git stash list`就看不到任何 stash 内容了

- 合并冲突时，取消合并
  - `git merge --abort`

- 删除分支
  - `git branch --delete dev`

- 版本回退

  - ```
    git reset --hard HEAD^
    ```

- 只克隆最近一次的commit，而不克隆所有的提交记录
  - git  clone --depyh=1

- 查看修改记录

  - 尚未commit: `git diff`
  - 已经commit：`git log -p`

- 查看commit历史版本记录 `git log`

- 将本地分支同步到远端
  - git push --set-upstream origin dev      // 将本地新建的dev分支推送到远程，并且建立跟踪关系 

- 取消合并

  - git merge --abort

- 创建并且切换分支

  - git checkout -b 

- [git rebase](https://www.cnblogs.com/yxhblogs/p/10527271.html)

  - 合并 commit 记录，保持分支整洁
  - 相比 merge 来说会减少分支合并的记录。但是如果你的分支不只有你在进行开发，那么git-rebase可能会导致提交记录的丢失
  - [详细链接](https://blog.csdn.net/u012140251/article/details/120960025?ops_request_misc=&request_id=&biz_id=102&utm_term=git%20rebase%20%E5%90%88%E5%B9%B6commit&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-120960025.142^v88^control_2,239^v2^insert_chatgpt&spm=1018.2226.3001.4187)

- 同步本地master和远程master

  - Git 强制覆盖本地命令(单条执行):

  - git fetch --all &&  git reset --hard origin/master && git pull 

    - 第一个是：拉取所有更新，不同步
    - 第二个是：本地代码同步线上最新版本(会覆盖本地所有与远程仓库上同名的文件)
    - 第三个是：再更新一次

- 查看当前的git地址

  - git remote -v

- 统计代码行数

  -  git ls-files | xargs cat | wc -l
  
- 比较分支的不同

  - ###  git diff branch1 branch2 --stat

- 拉不到远程分支
  - git fetch origin release:refs/remotes/origin/release 
  - git checkout -b origin/release
