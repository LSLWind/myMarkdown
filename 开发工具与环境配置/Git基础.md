### git初始化配置

$ git config --global user.name "用户名"

$ git config --global user.email "邮箱"

上述命令会在家目录下建立一个叫 .gitconfig 的配置文件（该文件为隐藏文件）.

配置文件就是 Git 全局配置的文件，一般配置方法是 git config --global <配置名称> <配置的值>。

如果不带 --global 选项来设置. 会在你当前的项目目录下创建 .git/config，从而使用针对当前项目的配置。







获得仓库：从已有的Git 仓库中 clone ；新建仓库，把未进行版本控制的文件进行版本控制

1.clone:git clone 网址(存放仓库位置)

2.新建仓库:  进入项目文件

                     git init
    
                   初始化后会有一个名叫 .git 的目录被创建，这意味着一个仓库被初始化了。

 **绑定远程origin master**

```csharp
git remote add origin https://gitee.com/LSLWIND/aladdin-business.git
```


git操作:

       Git 的基本流程如下：

 创建或修改文件
使用 git add 命令添加新创建或修改的文件到本地的缓存区（Index）
使用 git commit 命令提交到本地代码库
（可选，有的时候并没有可以同步的远端代码库）使用git push命令将本地代码库同步到远端代码库
进入项目文件（即进入版本代码仓库）在命令行中操作文件

常用命令:

       git status    查看仓库状态
    
       git add 文件名  将修改后的文件加入缓冲区
    
                    （可选参数 *）将所有修改或新建的文件加入缓冲区
    
                    当所有新建，修改的文件都被添加到了缓存区，要使用 git commit 提交到本地仓库
    
       git commit -m "注释"      将缓冲区的所有文件提交,完成后就会记录一个新的项目版本，缓冲区清空
    
                     (可选参数) -a 将所有没有加到缓存区的修改也一起提交，但 不会添加新建的文件。
    
       git rm 文件名
    
            删除文件，删除后会自动将已删除文件的信息添加到缓存区，git commit 提交后就会将本地仓库中的对应文件删除。
    
       git remote add 主机名  网址     与远程仓库相关联
    
            如果本地的仓库连接到了远程Git服务器（即相互关联），可以使用git push将本地仓库同步到远端服务器：

 




git分支

Git 的分支可以让你在主线（master 分支）之外进行代码提交，同时又不会影响代码库主线。分支的作用体现在多人协作开发中，比如一个团队开发软件，你负责独立的一个功能需要一个月的时间来完成，你就可以创建一个分支，只把该功能的代码提交到这个分支，而其他同事仍然可以继续使用主线开发，你每天的提交不会对他们造成任何影响。当你完成功能后，测试通过再把你的功能分支合并到主线。

       git branch 分支名      创建分支
    
       git branch                  查看分支
    
       git checkout 分支名    切换分支

分支的代码是分支的代码，主线的代码是主线的代码，分支的任何更改不影响主线和其它分支

       git merge -m '注释' 分支名      当前分支合并分支
    
              分支冲突：合并的两个分支修改了同一文件，两个分支修改的内容会合并，但会在文件中产生标志冲突的 <<<<<< 等符号
    
       git branch -d 分支名      删除那些已经被当前分支合并的分支
    
       git branch -D 分支名     强制删除分支
    
       git reset --hard HEAD    撤销上一次合并

 




git 日志

       git log     显示日志
    
            如果提交的历史纪录很长，回车会逐步显示，输入 q 可以退出。git log 有很多选项，可以使用 git help log 查看
    
      git log --stat     打印详细的日志提交记录
    
     --pretty 参数可以使用若干表现格式(即格式化日志)，如git log --pretty=oneline
    
                      除oneline外，还有medium，full，fuller，email ， raw等。
    
      --graph 选项可以可视化你的提交图（commit graph），例：git log --graph --pretty=oneline 

 




git比较查看修改的文件

      git diff    查看所有比较修改的或提交的文件内容
    
               (可选参数) --cached   查看缓冲区当中比较修改的或提交的文件内容
    
     git diff  分支名1  分支名2   比较两个分支的异同
    
      git diff 分支名        比较当前目录（分支）与分支的异同
    
      git diff [分支名]  文件名   查看（可选）分支的文件与当前目录（分支）的异同

 




git分布式工作流程

      1.两个仓库的合并，相当于版本的合并,使用pull操作拉取代码
    
       git pull 项目路径 分支名    将指定路径的项目的指定分支与当前主机的分支合并(可能会造成文件的冲突，需要手                                    工修改)
    
       git pull 命令等同于执行两个操作: 先使用 git fetch从远程分支抓取最新的分支修改信息，然后使用 git merge 把修改合并                      进当前的分支。
    
        git pull: 如果当前仓库是克隆的，就会从克隆的位置拉取代码并更新本地仓库(从配置文件中读取克隆地址)

开发过程中，通常大家都会使用一个公共的仓库，并 clone 到自己的开发环境中，完成一个阶段的代码后可以告诉目标仓库的维护者来 pull 自己的代码

       2.提交本地仓库到公共仓库使用push操作推送代码
    
                  git push     公共仓库地址

如果报错error: remote 'refs/heads/master' is not an ancestor of local 'refs/heads/master'. Maybe you are not up-to-date and need to pull first? error: failed to push to 'ssh://yourserver.com/~you/proj.git'

这种情况通常是因为没有使用 git pull 获取远端仓库的最新更新，在本地修改的同时，远端仓库已经变化了（其他协作者提交了代码），此时应该先使用 git pull合并最新的修改后再执行 git push

     git pull
    
     git push 公共仓库地址

git标签

    每一次提交(commit)都会更新仓库并生成一个版本号，通过git log 查看，为此，可以为每一个版本自定义标签，标签包含其仓库版本的信息，内容等，仓库号即标签名称 
    
      1.轻量级标签   git tag 标签名 版本号      将版本号的别名设置为标签名
    
                              git tag   查看所有标签
    
        2.标签对象
    
    git tag 中使用 -a， -s 或是 -u三个参数中任意一个，都会创建一个标签对象，并且需要一个标签消息（tag              message）来为 tag 添加注释。 如果没有 -m 或是 -F 这些参数，命令执行时会启动一个编辑器来让用户输入标签消息。
    
             当这样的一条命令执行后，一个新的对象被添加到 Git 对象库中，并且标签引用就指向了一个标签对象，而不是指向一个提交，这就是与轻量级标签的区别。
    
            git tag -a 标签名 版本号 -m "注释"
## 常见问题

### 配置

#### Git-remote Incorrect username or password ( access token )

要修改计算机凭据

打开电脑的控制面板（win+R，输入control.exe）–>用户账户–>管理Windows凭据，修改凭据即可

### 上传

####  [rejected] master -> master (non-fast-forward)”

```bash
 git push origin master
To https://gitee.com/XXXXX.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://gitee.com/XXXXX.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

从提示语中可以看出是，问题（Non-fast-forward）的出现原因在于：git仓库中已经有一部分代码，所以它不允许你直接把你的代码覆盖上去。于是你有2个选择方式：

1、强推，即利用强覆盖方式用你本地的代码替代git仓库内的内容，如果远程仓库是刚建的，没有代码，可以这样操作，尽量避免这种操作方法。

```bash
git push -f origin master
```

2、先把git的东西fetch到你本地然后merge后再push

```bash
$ git fetch
$ git merge
```

3、在使用的时候，git merge，又出现了以下的问题

```bash
xu:QProj xiaokai
$ git merge
fatal: refusing to merge unrelated histories
```


对于这个问题。使用git pull origin master --allow-unrelated-histories来处理一下。

4、然后继续git merge,依然有问题

```
fatal: You have not concluded your merge (MERGE_HEAD exists).
Please, commit your changes before you merge.
```


这个就好处理了，是我们没有提交当前的变化， git add .,git commit -am "提交信息"

然后再来一次git merge,然后ok.

#### fatal: refusing to merge unrelated histories

强制git接收不相关版本历史

```bash
git pull origin master --allow-unrelated-histories
```

