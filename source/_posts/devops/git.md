## git解决了什么问题

1. **分布式**: 解决以前**集中式**单点文件服务器宕机的痛点,;由于git是分布式的vcs系统,`无需联网`就可以在本地提交保存各自的操作记录,最后与其他人合并推送到远程服务器保存即可.

2. **轻量健全的分支系统**:有想法就可以开分支,分支都在自己仓库内,不会影响其他人,便于切换和协作开发.

3. **快**:所有操作在本地,肯定会快.

   **tips** : git的学习曲线真的不算平滑,当然如果只是当个下载器那没啥可说.

## 建仓

> `git init `
>
> git config --global user.name "sui"
>
> git config --global user.email "a@b.com"
>
> `git config --list` 查看现有配置
>
> 初始化仓库,配置自己的`名字`和`邮箱`

**tips:** 如果文件是代码的话,一般要配置上.gitignore和README.md文件.

**tips:**  *作者* 和 *提交者* 之间究竟有何差别， 其实作者指的是实际作出修改的人，提交者指的是最后将此工作成果提交到仓库的人。当你为某个项目发布补丁，然后某个核心成员将你的补丁并入项目时，你就是作者，而那个核心成员就是提交者。



## 分区

> **工作区**--->**暂存区**--->**本地仓库**
>
> `git add 文件` 开始追踪该文件,并加入缓存区.已经追踪但是修改了的文件也要重新add才能加入暂存区.
>
> `git commit -m` 把暂存区的所有变更提到本地仓库,工作区直接提交加-a
>
> `git diff` 查看未暂存的修改了哪些地方,不好用.
>
> `git diff --cached` 查看已暂存的变化.
>
> `git rm 文件` 从工作区删除该文件,并且不再追踪.
>
> `git rm --cached 文件` 不会从工作区删除该文件,不再追踪该文件. 
>
> `git mv source target` 重命名,并且依旧追踪.
>
> `git log` 查看提交记录

## 撤销

> `git commit --amend` 修正提交,把暂存区的文件提交,第二次提交将代替第一次提交的结果.
>
> 用于修正提交信息和提交遗漏文件
>
> `git checkout 文件` 把该文件还原成暂存区中的状态 
>
> `git reset HEAD`  重置暂存区为上一次提交,工作区不变,暂存区改文件未提交.(放弃提交)
>
>  	1. 默认是 git reset --mixed HEAD 同上
>  	2. git reset --hard HEAD 重置所有没有提交入库的,工作区和暂存区都会修改,谨慎使用!
>  	3. git reset --soft HEAD  重置HEAD指针,工作区和暂存区不会改变
>  	4. git reset 文件 可以指定回退文件
>
> `git checkout HEAD .` 把工作区的文件都会退到HEAD指向的提交
>
> `git checkout comId 文件  `把工作区改文件退回到comId

**tips:** 对于git,几乎所有的已提交的东西总能恢复.

## 远程仓库

> `git remote -v` 查看要读写的远程仓库简称以及URL 
>
> `git remote add <为远程仓库起的别名> <远程仓库地址>` 添加一个远程仓库
>
> `git fetch <别名>` 从远程仓库下载本地缺失的提交,更新远程分支,并没有修改本地文件,需要自己手动合并.
>
> `git pull <别名>` 拉取并合并
>
> `git push <别名> <branch>` 把本地仓库提交推送到远程仓库,如果已经有其他人推送,必须先拉取.
>
> `git remote show <别名>` 查看远程仓库所有分支与URL
>
> `git remote rename <旧名字> <新名字>` 重新命名简称
>
> `git remote remove <别名>` 移除与远程仓库的关联

## 打标签

**标签是为了标记重要版本**

git支持两种标签:轻量标签,附注标签.其中轻量标签只是简单打个标签不含其他额外信息,而附注标签会带有打标签人的姓名,邮箱,日期,和标签独有信息.

> `git tag` 列出所有标签
>
> `git tag -l "v1.0*"` 匹配所有v1.0版本的
>
> `git tag -a v1.0 -m "my version 1.0"` 打上`附注标签`标签v1.0,并填上信息
>
> `git show v1.0` 看标签信息
>
> `git tag tagName`  直接打上轻量标签
>
> `git tag -a <tagName> <commId> ` 给历史提交打标签
>
> `git push origin <tagName>` 把标签推送到远程仓库(不会自动推送到远程)
>
> `git push origin tags` 把所有标签推送到远程仓库
>
> `git tag -d <tagName>` 删除标签,并不会删除远程标签
>
> `git push <remote>:refs/tags/<tagName>`删除远程关联的标签
>
> `git push origin --delete <tagName>` 效果同上,删除远程标签
>
> `git checkout <tagName>`  检出标签所指文件版本,会使仓库处于头指针分离状态,不建议用

## 起别名

> `git config --global alias.co checkout` 把checkout改名为co

## 分支

轻量的,无需副本的,支持频繁合并

> HEAD 是一个指向当前所在的本地分支,可以把HEAD想象成当前分支的别名.
>
> 切换分支会重置工作区,而且工作目录和暂存区没有提交的内容如果和checkout的分支有冲突,那么将会切换失败,此时要先解决冲突.
>
> 冲突是指某个文件某一部分进行了不同的修改,这个是git无法处理的,合并完需要用`git add`来标记为已解决.

> `git branch <分支名字>`  创建一个分支,也就是创建了一个可移动的指向提交对象的指针,执行命令后HEAD并不会指向新建的分支.
>
> `git checkout <分支名称>` 切换分支,即把HEAD指向当前分支,`工作区可能被改变`,git发现有冲突时将切不过来.
>
> `git merge hotfix` 把hotfix合并到当前分支,喊过来一起干大事
>
> `git log --oneline --decorate --graph --all` 项目分叉历史.
>
> `git checkout -b <分支名称>` 创建并切换到新分支
>
> `git branch -d <分支名字>` 删除一个分支,一般分支要解决的问题完成后,就删除该分支	
>
> `git branch -f main HEAD~3` 让分支指向另一个提交,此处让main往回走三次.

### 如何管理分支

> `git branch` 查看所有分支
>
> `git branch -v` 查看每个分支最后一次提交
>
> `git branch --merged` 查看哪些分支已经合并到当前分支
>
> `git branch --no-merged` 查看还没有合并到当前的分支,对这些分支进行删除会失败.

### 关于远程分支

> `git ls-remote origin` 获取远程仓库名为origin的所有分支引用.
>
> `git remote show origin` 获取远程分支的跟踪,合并,推送等信息
>
> `git fetch origin` 拉取origin上本地没有数据
>
> `git push origin master` 把本地仓库推送到远程仓库
>
> `git checkout -b release origin/release` 建立一个指向远程release的本地分支
>
> `git push origin --delete hotFix` 删除服务器上的hotFix分支

### 变基

> 给另一个分支A当儿子,找到当前分支和A的最近公共祖先结点,把最近祖先结点后到当前的提交并入A下.
>
> 相比于merge的优点:使提交历史更加简洁,把并行开发弄得像穿行开发.
>
> 相同点:结果快照和merge都是一样的,只是变更了历史.
>
> tips: git merge 有直接移动HEAD指针的功能
>
> **wanrning**: **如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。**

> `git rebase master` 当前分支接着master
>
> `git rebase --onto master server client` 这里server拉取于master,client拉取于server,把属于client但是不属于server的提交变基到master下.**git可以把树的当前节点合并到任意节点下**
>
> `git rebase <basebranch> <topicbranch>` 直接将topic合入master,无需切换分支
>
> `git pull --rebase` 正确拉取已经被编辑的提交

## 服务器上的Git

> 基于不同协议传输数据
>
> 1. 本地协议,基于文件共享系统的.
> 2. HTTP协议
> 3. SSH协议
> 4. git协议

> `ssh-keygen -o` 生成 ssh公钥到home路径的.ssh下的.pub文件,用于免密登录.
>
> `git cherry-pick commId1 commId2 ..`  摘樱桃,摘取有用提交.

## Git工具

> `git revert release HEAD~1` 生成新提交,并撤销已存在提交的所有修改. 

常见Git服务器: GitLab

第三方托管:GitHub,Gitee

[参考文档](https://git-scm.com/book/zh/v2)

## 本地代码与远程仓库关联：

1. git init
2. git remote add <别名> <clone地址>
3. 提交所有本地代码到本地仓库
4. git push <别名> <分支> 这里的分支要和本地一致
5. 完成