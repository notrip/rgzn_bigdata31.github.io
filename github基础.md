---
title: Github基础
date: 2022-03-10 21:58:54
tags:  暂时停更
---

#                        **Git 基础操作 （4-1）**  

* ### git init --------初始化仓库

在使用Git进行版本管理，必须先出实话仓库。出实话之前先创建一个目录（git-tutorial）

```
$ mkdir git-tutorial
$ cd git-tutorial
$ git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint: 	git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint: 	git branch -m <name> 
```

代码执行成后，目录会生成.git目录（储存管理当前目录内容所需的数据仓库）



* ### git status -------查看仓库状态

##### **git status命令用于显示Git仓库的状态。**

因工作树和仓库被操作的过程中，状态会不断发生变化。所以需要git status 命令查看当前状态。

```
$ git ststua
On branch master        #当前处于master分支

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

所谓提交（commit）：是指“ 记录工作数中所有文件的当前状态 ”

因为我们第一次操作没有可提交内容（没有记录任何文件的任何状态），所以我们建立README.md文件作为对象

```
$ touch README.md 
$ git status 
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README.md

nothing added to commit but untracked files present (use "git add" to track)
```

可以看到在Untracked files中显示了README.md文件。以此类推只要对Git进行操作 **git status**显示的结果就会发变化。



* ### git add ---------先暂时区中添加文件

 如果仅仅用Git仓库的工作树创建了文件，那么该文件并不会被记入git仓库的版本管理对象中。所以当使用**git status**时会显示在Untracked files中。

若想将其成为Git仓库的管理对象，解决办法是使用 **git add**命令将其加入暂存区（stage or index） ==暂存区是提交之前的一个临时区域==

```
$ git add README.md 
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README.md
```

将README.md加入暂存区后，**git status**可看出README.md文件在changes to be committed 中。



* ### git commit ---------保存仓库的历史记录   

  **git commit**命令可将当前出存在暂存区的文件保存到仓库的历史记录中。通过这些，我们可以在工作树中福源文件。

  * 记述一行提交信息

    ```  
    $ git commit -m "First commit"
    [master (root-commit) 335488e] First commit
     1 file changed, 0 insertions(+), 0 deletions(-)
     create mode 100644 README.md
    ```

    -m后的“First commit” 是对提交的概述.

  * 记述详细提交信息

    若想记录更详细，则不加 -m,执行后直接编辑即可

    ● 第一行：用一行文字简述提交的更改内容

    ● 第二行：空行

    ● 第三行以后：记述更改的原因和详细内容

  * 中止提交

    若在编译器启动后想终止，则将提交信息留空并关闭编译器即可

    

* ### git log --------查看提交日志

  **git log**可以查看以往仓库中提交的日志。如：什么人在什么时候进行了提交或合并，以及操作前后有怎样的差别。

  ```
  $ git log 
  commit 335488e5478c13f9cf0259a9a66bafae8de13b9d (HEAD -> master)
  Author: notrip <“1837357465@qq.com”>
  Date:   Tue May 17 09:47:09 2022 -0400
  
      First commit
  ```

  * **git log --pretty=short**不显示Date，方便把握多个提交。

    ```
    $ git log 
    commit 335488e5478c13f9cf0259a9a66bafae8de13b9d (HEAD -> master)
    Author: notrip <“1837357465@qq.com”>
    
        First commit
    ```

  * 特定查找摸个文件

    `$ git log READMR.md`

  * 显示文件的改动

    若想看提交所带来的改动怎加上 -p 

    ```
    $ git log -p
    $ git log -p README.md
    commit 335488e5478c13f9cf0259a9a66bafae8de13b9d (HEAD -> master)
    Author: notrip <“1837357465@qq.com”>
    Date:   Tue May 17 09:47:09 2022 -0400
    
        First commit
    
    diff --git a/README.md b/README.md
    new file mode 100644
    index 0000000..e69de29
    ```



* ### git diff --------查看更改前后的差别

  先vim README.md

  `# Git教程`

  * 查看工作树和暂存区的差别

    ```
    $ git diff
    diff --git a/README.md b/README.md
    index e69de29..981aef2 100644
    --- a/README.md
    +++ b/README.md
    @@ -0,0 +1 @@
    +#Git教程
    ```

    在最后一行出现的+（-）表示先添加的行（被删除）

    用**git add**将md文件加入暂存区

    `$ git add READMD.md`

    * #### 查看工作树和最新提交的差别

      ```
      $ git diff HEAD
      diff --git a/README.md b/README.md
      index e69de29..981aef2 100644
      --- a/README.md
      +++ b/README.md
      @@ -0,0 +1 @@
      +#Git教程
      ```

      **养成好习惯：执行git commit 前先执行 git diff HEAD**

      对比后进行提交 

      ```
      $ git commit -m "Add index"
      [master fe4c8d9] Add index
       1 file changed, 1 insertion(+)
      ```

      保险起见再查看下一觉日志

      ```
      $ git log
      commit fe4c8d999e632799390fc35ca8d9529e90940014 (HEAD -> master)
      Author: notrip <“1837357465@qq.com”>
      Date:   Tue May 17 10:02:02 2022 -0400
      
          Add index
      
      commit 335488e5478c13f9cf0259a9a66bafae8de13b9d
      Author: notrip <“1837357465@qq.com”>
      Date:   Tue May 17 09:47:09 2022 -0400
      
          First commit
      ```

      看到两个commit表示成功    

      

#                                  分支的操作（4-2）

* ### git branch --------显示分支----览表

  **git branch**显示当前所在分支

  ```
  $ git branch
  * master
  ```

* ### git checkout -b ----------创建、切换分支

  * 切换到feature -A分支并进行提交(两种方法)

    ```
    $ git branch feature-A
    $ git checkout -b feature-A
    Switched to a new branch 'feature-A'
    $ git branch
    * feature-A
      master
    ```

    **不断对一个分支进行提交的操作被称为“培育分支”**

    œ添加到feature-A分支中

    > $ vim README.md
    >
    > > #Git教程
    >
    > ```
    > $ git commit -m "Add feature-A"
    > [feature-A e5033d6] Add feature-A
    >  1 file changed, 2 insertions(+)
    > ```

  * 切换到master分支

    ```
    git checkout master
    Switched to branch 'master'
    ```

    切换到master分支中看到README.md中的内容不变。因为feature-A不会影响master。

  * 切换回上一个分支

    ```
    $ git checkout -
    Switched to branch 'feature-A'
    ```

    

最近只在学习github，一直没有整理，没有一直更新，不久就会更新 ！😭
