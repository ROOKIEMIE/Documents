# Git

## 常用命令

1. 常用的git“一键三连”

```shell
git add . && git commit -m "<commit message>" && git push
```

2. 检查当前分支情况，这将列出本地的所有分支。**当前活动的分支（也就是你处于的分_branch_) 会标有一个星号 (*)。**

```shell
git branch
```

3. 切换分支，``` [branch-name] ```为已有的分支名字

```shell
git checkout [branch-name]
```

4. 在本地创建分支，这个命令会创建一个新的分支，并自动切换到这个新分支。 ``` [branch-name] ```为你想要的分支名称。

```shell
git checkout -b [branch-name]
```

5. 推送新创建的本地分支到远程仓库，``` [branch-name] ```为分支名字。

```shell
git push origin [branch-name]
```

通过上述第4点和第5点中的命令，即可完成在**远程仓库中创建一个分支**。

6. 下面给出带分支的“一键四连”

```shell
git add . && git commit -m "<commit message>" && git pull origin <branchname> && git push origin <branchname>
```

上面给出的命令中，比常规的“一键三连”多了一个origin，这里解释一下origin是什么意思：

> 在Git中，“origin” 是默认的名称，用来引用你克隆源（即，你克隆代码库的远程服务器的URL）。当你使用 `git clone` 命令来复制一个项目时，这个远程源会自动被命名为 "origin"。
>
> 所以，在运行命令 `git push origin [branch-name]` 或 `git pull origin [branch-name]` 时，你是在告诉Git将你的更改推送或拉取到命名为 "origin" 的远程源。
>
> 你可以使用 `git remote -v` 命令来查看你所有的远程源以及它们的URL。
>
> 另外，虽然 "origin" 是默认的远程名，但你也可以更改它或者添加其他的远程源。例如，你可以运行 `git remote add [source-name] [source-url]` 来添加一个新的远程源，又或者你可以运行 `git remote rename origin [new-name]` 来更改 "origin" 的名称。

7. 将远程仓库中的某个分支（假设这个分支名为dev）合并到主分支中

   1. 首先，需要确保本地仓库是最新的。使用以下命令来获取远程仓库中的所有新提交

   ```shell
   git pull origin dev
   ```

   这将会获取远程仓库中 dev 分支的所有新提交。

   2. 接着，切换到要合并到的主分支上

   ```shell
   git checkout master
   ```

   3. 然后，将 dev 分支合并到当前所在的主分支

   ```shell
   git merge dev
   ```

   4. 如果没有冲突，或者已经解决了所有冲突，便可以将合并后的主分支推送回远程仓库

   ```shell
   git push origin master
   ```

   关于合并时若产生了冲突，以下为解决方法的示例：

   > 合并时的文件冲突表示在两个分支上都修改了同一文件的同一个部分。Git不知道应该保留哪个版本，所以需要你手动解决冲突。
   >
   > 要解决合并冲突，你首先需要找出这些冲突。`git status`命令可以列出所有存在冲突的文件。在文件中，Git用`<<<<<<<`, `=======`,和`>>>>>>>`标记出冲突的地方，你可以打开这些文件来查看。
   >
   > 例如，如果文件A的冲突如下所示：
   >
   > ```bash
   > <<<<<<< HEAD
   > // 这是master分支的版本
   > =======
   > // 这是dev分支的版本
   > >>>>>>> dev
   > ```
   >
   > 在这种情况下，你想保留dev分支的改动。因此，你需要手动编辑文件，**删除Git添加的`<<<<<<<`, `=======`,和`>>>>>>>`标记，并删除你不想保留的部分（在这个例子里，删除master分支的版本）**，最终文件应该看起来像这样：
   >
   > ```bash
   > // 这是dev分支的版本
   > ```
   >
   > **然后，你可以使用`git add .`将解决冲突后的文件添加到暂存区，并用`git commit -m "解决冲突"`来提交这些解决冲突的文件。**
   >
   > 最后，你可以使用`git push origin master`将这些更改推送到远程的master分支。

关于``` merge ```和``` rebase ```参数：

> Git中的`merge`和`rebase`都是用于整合代码更改的两种方式，但它们的方式和目的有所不同。
>
> 假设你有两个分支，一个是主分支`master`，另一个是你在工作的特性分支`feature`。将`feature`并入`master`的时候，你可以选择`merge`或`rebase`。
>
> **Merge**
>
> 当你执行`merge`操作的时候，Git会找到这两个分支（`master`和`feature`）的最新提交和它们共同的祖先，然后进行“三方合并”。三方合并涉及到的三方是`master`分支的祖先，`master`分支的最新状态和`feature`分支的最新状态。合并可能会产生一个新的“合并提交”。
>
> 合并后的`master`分支会包含`feature`分支的所有提交，它们按照实际发生的顺序排列，合并的操作也在提交历史中有所反映。
>
> **Rebase**
>
> 执行`rebase`操作的时候，Git首先找到`master`和`feature`分支的共同祖先和`feature`分支的每一个提交，然后把每一个`feature`分支的提交在`master`分支上重新应用。你可以把`rebase`想象成“把我的更改移到一个新的基线上”。
>
> 这种方式得到的提交历史是线性的，因为`feature`分支的每一个更改都基于`master`分支的最新状态。
>
> **适用场合**
>
> - `merge`更适合在团队合作中使用。因为`merge`保留了所有的提交和提交的时间顺序，所以对于一个团队来说，有助于理解项目发展的真实情况。但是，合并可能导致提交历史变得复杂。
> - `rebase`则非常适合在维护简洁、线性的提交历史的时候使用，特别是在代码审查和单人项目中。`rebase`提供了一个清晰的更改历史，这样可以更容易地理解项目的发展过程。然而，`rebase`可能会复写提交历史，所以在和他人一起工作的时候使用`rebase`需要谨慎。

通过上述第3，第4点所提的命令合并并推送了其他分支改动到主分支后，实际上进行的是**增量**改动，即，主分支上原有的未被改动的文件依旧会存在。

合并完成后，源分支依旧存在，可以继续使用。

> 若需要删除分支，则可以使用以下两个命令：

1. 删除本地分支

```shell
git branch -d <branch name>
```

2. 删除远程分支

```shell
git push origin --delete <branch name>
```

---

> 当从远程仓库拉取一个本地不存在的分支

1. 在本地创建并切换到一个名为 \<branch name> 的新分支，这个分支将跟踪远程仓库的 \<branch name> 分支。

```shell
git checkout -b <branch name> origin/<branch name>
```

2. 在本地创建一个名为 \<branch name> 的新分支，且这个新分支的内容等同于远程的 \<branch name> 分支

```shell
git fetch origin <branch name>:<branch name>
```

最后记得使用checkout命令切换分支

```shell
git checkout <branch name>
```

通过上述的方式，在本地创建并跟踪远程仓库的 `<branch name>` 分支后，实际上已经自动地拉取了 `<branch name>` 分支上的最新文件和提交历史。因此，可以不再需要再次使用```git pull origin <branch name>```进行分支的同步。

---

8. 远程仓库相关的命令

查看当前本地Git仓库所配置的远程仓库地址

```shell
git remote -v
```

设置（重设）远程仓库地址

```shell
git remote set-url origin <protocol>://<IP:Port/Domain>/<git address>.git
```



## git方便提交脚本

1. 普通提交

```shell
timestamp="$(date +%Y-%m-%dT%H:%M:%S)"
git add . && git commit -m "$1 $timestamp" && git pull && git push
```

使用方法

```shell
./git_push "<commit message>"
# 输入用户名和密码
```

2. 提交到分支

```shell
timestamp="$(date +%Y-%m-%dT%H:%M:%S)"
git add . && git commit -m "$2 $timestamp" && git pull origin $1 && git push origin $1
```

使用方法

```shell
./git_push <branch name> "<commit message>"
```



## 配置使用SSH拉取/推送代码至Github仓库

### Linux下的配置

#### 0x01 创建SSH公私钥

```shell
ssh-keygen -t rsa -b 4096 -C "<github email>"
```

说明：

1. ``` -t ```参数后跟着的是生成公司密钥对时所使用的**非对称加密算法**，目前（截止2024/12/31）Github支持的**非对称加密算法**有：ssh-rsa、ecdsa-sha-nistp256、ecdsa-sha-nistp386、ecdsa-sha-nistp521、ssh-ed25519、```sk-ecdsa-sha2-nistp256@openssh.com```。
2. ``` -b ```参数后跟着的是所生成的私钥的位数。
3. ``` -C ```参数后所跟着的是用以生成密钥所使用的电子邮箱地址，这里需要配置为**Github账号所使用的邮箱地址**。 

#### 0x02 拷贝并配置公钥至Github

查看刚才生成的私钥的公钥

```shell
cat ~/.ssh/"<your generated key file name>.pub"
```

将上述命令的**所有输出**（~~公钥~~）复制到剪贴板。然后，在 GitHub 页面中：

1. 登录到你的 GitHub 账户。
2. 点击右上角的头像，选择 **Settings**。
3. 在侧边栏中找到 **SSH and GPG keys**，点击进入。
4. 点击 **New SSH key** 或 **Add SSH key**。
5. 给你的密钥命名，然后将你复制的公钥粘贴到相应的区域。
6. 点击 **Add SSH key**。

#### 0x03 测试SSH连接

```shell
ssh -T git@github.com
```

若出现了错误``` git@github.com: Permission denied (publickey). ```

则可以使用下述命令输出其中的详细过程（~~虽然可能也没啥用~~ ）

```shell
ssh -vT git@github.com
```

如何上述命令运行成功，则会返回

```shell
Hi XXX! You've successfully authenticated, but GitHub does not provide shell access.
```

#### 0x03.1 配置ssh config文件（非必须）

``` config ```文件位于``` ~/.ssh ```目录下，配置格式如下：

```txt
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/<your generated key file name>
  IdentitiesOnly yes
```

保存后重试``` 0x03 ```中所示的命令。

#### 0x04 使用

初始拉取仓库

```shell
git clone git@github.com:xxx/xxx.git
```

查看远程仓库

```shell
git remote -v
```

设置远程仓库url

```shell
git remote set-url origin git@github.com:xxx/xxx.git
```

推送更改至远程仓库

```shell
git push
git push origin <branch name>
```

### Windows下的配置

与Linux中的操作基本相同。

