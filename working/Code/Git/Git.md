ctrl A \[ 复制
ctrl A \] 粘贴

---

```git
git add .
git commit om " "
git push origin master
```

---

# Git 基本概念

- 工作区：存放仓库的目录。它是独立于各分支的。
- 暂存区：用于临时存放数据的区域，在将其写入本地版本库之前。它也是独立于各分支的。
- 版本库：包含已提交至本地仓库的所有代码版本。
- 版本结构：以**树**状结构显示，其中每个节点代表一个代码版本。

# Git 常用命令

- `git config --global user.name XXX` 设置全局用户名。该信息存储在 `~/.gitconfig` 文件中。
- `git config --global user.email XXX@XXX.com` 设置全局电子邮件地址。该信息也存储在 `~/.gitconfig` 文件中。
- `git init` 将当前目录配置为 Git 仓库。相关信息存储在一个名为 `.git` 的隐藏文件夹中。
- `git add XX` 向暂存区添加名为 XX 的文件。
- `git add .` 向暂存区添加所有待添加文件。
- `git rm --cached XX` 删除 XX 文件在仓库索引目录中的记录。
- `git commit -m "备注信息"` 将暂存区的内容提交到当前分支部署。
- `git status` 查看仓库的状态。
- `git diff XX` 显示 XX 文件相对于暂存区的更改内容。
- `git log` 显示当前分支部署的所有版本。
- `git log --pretty=oneline` 简化显示，每个版本显示一行
- `git reflog` 显示 HEAD 指针的历史记录（包括已回滚的版本）。
- `git reset --hard HEAD^` 或 `git reset --hard HEAD~` 将代码库回滚至上一个版本。
- `git reset --hard HEAD^^` 将代码库回滚至两个版本之前，依此类推。
- `git reset --hard HEAD~100` 将代码库回滚至前一百个版本。
- `git reset --hard 版本号` 将代码库回滚至指定版本。
- `git checkout -- XX` 或 `git restore XX` 取消对 XX 文件未暂存的更改。
- `git push -f origin master`强制推送更改。覆盖远程仓库的更改，使远程仓库和本地仓库完全一样。

### 远程操作

- `git remote add origin git@git.acwing.com:XXX/XXX.git` 关联本地仓库与远程仓库。
- `git push -u` （首次使用需加 `-u`，之后不再需要）推送当前分支部署至远程仓库。
- `git push origin 分支部署名` 将本地仓库中的特定分支部署推送至远程仓库。
- `git clone git@git.acwing.com:XXX/XXX.git` 下载远程仓库 XXX 到当前目录。
- `git checkout -b 分支部署名` 创建并转至名为分支部署名的新分支部署。
- `git branch` 查看所有分支部署以及当前所在分支部署。
- `git checkout 分支部署名` 转至名为分支部署名的分支部署。
- `git merge 分支部署名` 将名为分支部署名的分支部署与当前分支部署合并。
- `git branch -d 分支部署名` 删除本地仓库中的名为分支部署名的分支部署。
- `git branch 分支部署名` 创建新的分支部署。
- `git push --set-upstream origin 分支部署名` 将本地名为分支部署名的分支部署与对应的远程分支部署相关联。
- `git push -d origin 分支部署名` 删除远程仓库中的名为分支部署名的分支部署。
- `git pull` 合并远程仓库中的当前分支部署与本地仓库的当前分支部署。
- git pull origin branch_name：将远程仓库的 branch_name 分支与本地仓库的当前分支合并
- git branch --set-upstream-to=origin/branch_name1 branch_name2：将远程的 branch_name1 分支与本地的 branch_name2 分支对应
- git checkout -t origin/branch_name 将远程的 branch_name 分支拉取到本地
- git stash：将工作区和暂存区中尚未提交的修改存入栈中
- git stash apply：将栈顶存储的修改恢复到当前分支，但不删除栈顶元素
- git stash drop：删除栈顶存储的修改
- git stash pop：将栈顶存储的修改恢复到当前分支，同时删除栈顶元素
- git stash list：查看栈中所有元素
- `git config --list` 查看git配置信息
## Git 常用设置

## Git 安装

- 选中 add windows terminal
- 安装完成后打开终端进入设置修改 Git Bash 的启动目录，修改为使用父进程目录

## 行尾序列

- 使用`git core.autocrlf`配置
- 当我们用 windows 电脑 git clone 代码的时候，若 autocrlf(在 windows 下安装 git，该选项默认为 true)为 true，那么 文件每行会被自动转成以 CRLF 结尾，若对文件不做任何修改，pre-commit 执行 eslint 的时候就会提示你删除 CR。
- 修改全局设置

```git
git config --global core.autocrlf false
```

- **true**: 提交时转换为 LF，拉取时转换为 CRLF
- **false**: 提交拉取均不转换
- **input**: 提交时转换为 LF，拉取时不转换

**5157**

## 多项目合并

如果你的项目需要使用另一个项目中的一个组件，并且希望在那个组件更新时能够自动同步到你的项目中，你可以使用Git子模块（Git Submodules）或者Git子树（Git Subtrees）。这两种方法都可以帮助你管理和同步多个项目之间的代码。

### 使用Git子模块

Git子模块允许你将一个Git存储库作为另一个Git存储库的子目录。这样，你可以轻松地跟踪和更新外部存储库的特定版本。

1. **添加子模块**：使用以下命令添加GitHub存储库作为子模块。请将`<repository_url>`替换为你想要添加的GitHub存储库的URL，将`<path>`替换为你希望存储库被放置的路径，通常是`components/<repository_name>`。

   ```bash
   git submodule add <repository_url> <path>
   ```

2. **初始化和更新子模块**：添加子模块后，你需要初始化并更新它。这可以通过以下命令完成：

   ```bash
   git submodule update --init --recursive
   ```

3. **提交更改**：添加子模块后，你需要将这些更改提交到你的项目中。

   ```bash
   git add .gitmodules <path>
   git commit -m "Added submodule"
   ```

4. **同步子模块**：当GitHub存储库更新时，你可以通过以下命令更新你的项目中的子模块：

   ```bash
   git submodule update --remote --merge
   ```

### 使用Git子树

要将一个Git项目作为子树（subtree）添加到另一个项目中，并将其放在特定的文件夹下（在这个例子中是`components`文件夹），你可以按照以下步骤操作。这里假设你已经在你的项目中初始化了Git，并且你想要将`git@github.com:missing-shell/task_list.git`这个项目添加到你的`components`文件夹下。

1. **添加远程仓库**：首先，你需要将`task_list`项目的远程仓库添加到你的本地仓库。这可以通过`git remote add`命令完成。

```shell
git remote add task_list git@github.com:missing-shell/task_list.git
```

2. **拉取远程仓库的内容**：使用`git fetch`命令从远程仓库拉取内容。

```shell
git fetch task_list
```

3. **将远程仓库的内容添加为子树**：使用`git subtree add`命令将远程仓库的内容添加到你的项目中，并指定要添加到的目录。在这个例子中，目录是`components/task_list`。

```shell
git subtree add --prefix=components/task_list task_list master --squash
````

这里的`--prefix=components/task_list`参数指定了子树应该添加到的目录，`task_list`是你之前添加的远程仓库的名称，`master`是你想要添加的分支（如果你想要添加的是主分支），`--squash`参数将所有的提交压缩成一个单一的提交，这有助于保持你的项目的提交历史更加清晰。

4. **验证子树的添加**：你可以通过查看`components/task_list`目录的内容来验证子树是否已经成功添加。

```shell
ls components/task_list
```

5. **提交更改**：最后，你需要提交这些更改到你的项目中。

```shell
git add . git commit -m "Add task_list as a subtree"
```
### 两者区别
`git submodule`和`git subtree`是两种不同的方法，用于将一个Git仓库的内容嵌入到另一个Git仓库中。它们的主要区别在于它们如何管理和跟踪代码的更改。
### Git Submodule

- **子模块**：子模块是一个独立的Git仓库，它被嵌入到另一个Git仓库中。子模块有自己的历史记录，可以独立于主仓库进行提交和更新。
- **优点**：子模块允许你维护独立的项目，同时允许你在主项目中使用这些项目。子模块可以独立更新，这意味着你可以在主项目中使用不同版本的子模块。
- **缺点**：子模块的管理相对复杂，因为它们有自己的历史记录和提交。这可能会导致更复杂的版本控制流程。

### Git Subtree

- **子树**：子树是一个Git仓库的一部分，它被合并到另一个Git仓库中。子树的提交会被合并到主仓库的历史记录中，这意味着主仓库会跟踪子树的更改。
- **优点**：子树的管理相对简单，因为它们的提交会被合并到主仓库的历史记录中。这使得版本控制流程更加直观和简单。
- **缺点**：子树不能独立于主仓库进行提交和更新，这意味着你不能在主项目中使用不同版本的子树。

总的来说，子模块和子树都可以用于将一个Git仓库的内容嵌入到另一个Git仓库中，但它们在管理和跟踪代码更改方面有所不同。选择哪种方法取决于你的项目需求和偏好。

```SHELL
git clone --recurse-submodules
```