## git 使用相关经验

### git status 时，汉字显示为数字乱码

- 情形显示如下

  ![]()

- 解决方式，加入如下配置：git config --global core.quotepath false

  ![]()

---

### .gitignore 新增忽略规则未生效

把某些目录或文件加入忽略规则，按照上述方法定义后发现并未生效，原因是.gitignore只能忽略那些原来没有被追踪的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未被追踪状态），然后再提交：

- git rm -r --cached .
- git add .
- git commit . -m "update .gitignore"

---

### 使用git mergetool合并代码分支时，禁止产生.orig的备份文件

添加如下配置：

git config --global mergetool.keepBackup false

---

### gitk显示中文乱码问题

添加如下配置：

- git config --global gui.encoding UTF-8
- git config --global [i18n.commitencoding](http://i18n.commitencoding/) UTF-8
- git config --global i18n.logoutputencoding UTF-8

---

### 文件名太长无法提交

添加如下配置：

git config --global core.longpaths true

---

### 设置git log的日志打印时间格式

添加如下配置：

git config --global log.date format:%Y-%m-%d\ %H:%M

---

### git 常用别名配置

git lg 别名配置：

> git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr %cd) %C(bold blue)<%an>%Creset' --abbrev-commit"

git last 别名配置：

> git config --global alias.last "log -1 HEAD"

---

### 查看 git 相关配置信息

- 查看 git 版本

  > git --version

- 查看 git 所有配置项

  > git config -l  或者  git config --list

- 查看当前系统范围内 git 所有配置项

  > git config --system -l  或者 git config --system --list

- 查看当前用户范围内 git 所有配置项

  > git config --global -l  或者  git config --global --list

- 查看当前仓库范围内 git 所有配置项

  > git config --local -l  或者  git config --local --list

- 查看 git 某个配置项，如查看用户名

  > git config user.name

- 查看 git 远程地址

  > git remote -v

- 查看remote地址，远程分支，还有本地分支与之相对应关系等一系列信息

  > git remote show origin

### 配置第三方比对工具

git查看有哪些对比工具可以设置

> git difftool --tool-help
>
> git mergetool --tool-help

设置对比工具:

在 ~/.gitconfig 中添加如下代码：

>[diff]
>	tool = bc3
>[difftool]
>	prompt = false
>[difftool "bc3"]
>	path = E:\\software\\29-beyond-compare4\\BeyondCompare4\\Beyond Compare 4\\BComp.exe
>[merge]
>	tool = bc3
>[mergetool]
>	prompt = false
>	keepBackup = false
>[mergetool "bc3"]
>	path = E:\\software\\29-beyond-compare4\\BeyondCompare4\\Beyond Compare 4\\BComp.exe
>	trustexitcode=true

### 合并分支相关

合并过程中，发现冲突很多，想中断这一次合并

> git merge --abort
>
> 该命令会尝试恢复到你运行合并前的状态
>
> 注意：在工作目录中有未储藏、未提交的修改时，该命令不能完美处理，除此之外该命令都工作良好

如果因为某些原因你发现自己处在一个混乱的状态中，只是想重来一次

> git reset --hard HEAD
>
> 该命令会将仓库回到之前的状态
>
> 注意：该命令会清除工作目录中的所有内容，所以运行该命令之前确保不需要保存工作空间中的任何改动

创建合并节点并忽略空格引起的冲突

> git merge -Xignore-space-change --no-ff branchName  或者  git merge -Xignore-all-space --no-ff branchName
>
> 注意：
>
> -Xignore-all-space 选项是忽略任意数量的已有空白的修改
>
> -Xignore-space-change 选项是忽略所有空白修改

### git 查看日志相关

- 查看全部提交日志

  > git log

- 以一行的形式显示 log 的 title

  > git log --oneline

- 查看提交修改了哪些内容

  > git log -p

- 查看某次提交修改了哪些内容

  > git log -p commitId<git提交版本号>

- 查看指定用户的提交

  > git log --author=str        str可以是git中的用户名，也可以是正则表达式

- 根据log中的一些关键词进行过滤

  > git log --grep=str        str可以是提交信息中的某些关键词，也可以是正则表达式

- git查看某个文件的提交历史

  > git log --prety=oneline fileName<文件名>
  >
  > git show commitId<git提交版本号> fileName<文件名>

- git查看某个日期之后的提交

  > git log --after={2018.07.01}

- git查看某个日期之间的提交

  > git log --before={2018.07.01}

- 只查找merge提交信息

  > git log --grep=Merge\ branch