# 08\_Git 图形工具

\[TOC]

## 8.1 GitHub Desktop

[GitHub Desktop](https://desktop.github.com/) 可能是所有 Git 可视化应用中最著名的方案。几乎所有开发人员都熟悉 GitHub，而 Github Desktop 正是 Github 推出的开源 Git GUI 图形客户端。可以在Windows和Mac平台上进行使用，目前暂不支持Linux平台。

### 8.1.1 登录

完成下载后，第一次打开软件会直接要求登录个人 Github 账户进行授权，并配置用户名和邮箱（识别个人创建的commits提交）。

如果没有找到让你登录 GitHub 账号的地方，你需要在 File -> Options -> Accounts -> Sign in 登录。

### 8.1.2 建立首个仓库

初次登陆会看到三个选项，也就是建立自己的第一个repository，可以通过三个方式：

* clone a repository：克隆一个repo
* create new repository：建立一个新的repo
* add a local repository：添加一个本地的repo

### 8.1.3 提交Pull Request

#### fork

由于在实际开源项目贡献的过程中，开发者往往并没有直接修改仓库内容的权限，因此需要先对目标仓库进行fork操作，再通过提交PR的方式进行代码的贡献。在下图中，可以通过左下角的warning标志⚠，判断用户是否有目标仓库的权限。如果没有写入权限，点击create a fork，将目标仓库复刻为自己的仓库，进行随意的修改。在 Github Desktop 中完成fork后，登录 Github 网页就可以在个人仓库中看到目标仓库的复刻版

![](../Git/images/github\_desktop\_4.png)

#### commit & push

在完成了fork后，当前仓库就会索引到用户个人的复刻仓库，对应于本地指定目录下的文件。此时，用户拥有复刻仓库的所有权限，包括修改，删除，更改可视状态等等。接下来，就可以对本地分支中的代码进行修改，更新而当操作，再push到用户个人的复刻仓库中。

![](../Git/images/github\_desktop\_6.png) ![](../Git/images/github\_desktop\_7.png)

此时，登录 Github 网页版就会发现本地修改的代码已经上传到云端，个人复刻仓库进行了本地同步。

![](../Git/images/github\_desktop\_8.png)

#### pr

在完成个人仓库的代码更新后，还要注意个人仓库的分支和目标分支的先后情况，如果目标分支领先于fork分支，需要先通过fetch upstream操作进行更新后，再提交PR。

> **upstream**分支指向上游地址即目标分支，这里的**upstream**名字可以任意指定，只是一般都把上游地址都叫**upstream**。

点击Contribute，并Open pull request，向目标仓库提交上传申请。

![](../Git/images/github\_desktop\_9.png) ![](../Git/images/github\_desktop\_10.png)

![](../Git/images/github\_desktop\_11.png)

在完成PR后，会自动跳转到目标仓库，可以看到在Pull requests一栏中，上标增加了1，1就是贡献者所提交PR。之后就需要目标仓库的拥有者对贡献的代码进行审阅，如果代码合规可利用，就会将fork分支的commits合并到主分支中。这样一来，就完成了一次贡献！

![](../Git/images/github\_desktop\_12.png)

#### 代码比较与冲突处理

与团队协作相伴的往往就是修改冲突（Conflit）的问题了，可以使用代码比较的软件[Beyond Compare](https://www.scootersoftware.com/download.php)

将bc用作代码比较工具可以较为方便地在git中进行配置，且拥有较成熟的图形化界面（对不同系统的换行符CR、lF，也能较为合理地自动处理），相比与手动解决冲突的效率还是会好许多。

在bc完成安装之后，直接用git命令配置，这里是直接做了git的全局配置，如果只想让它在某个代码仓库生效可以将下面这段中的global都改为local。

```shell
git config --global diff.tool bc4
git config --global difftool.bc4.cmd "\"D:\Beyond Compare 4\BComp.exe\" \"\$LOCAL\" \"\$REMOTE\" \"\$BASE\" \"\$MERGED\""
git config --global difftool.bc4.trustExitCode true

git config --global merge.tool bc4
git config --global mergetool.bc4.cmd "\"D:\Beyond Compare 4\BComp.exe\" \"\$LOCAL\" \"\$REMOTE\" \"\$BASE\" \"\$MERGED\""
git config --global mergetool.bc4.trustExitCode true
```

## 8.2 TortoiseGit

TortoiseGit 简称 tgit， 中文名海龟 Git ，是一个开放的 Windows 系统下的 Git 版本控制系统的源客户端，提供有中文版支持。由于它不是针对特定IDE(如Visual Studio、Eclipse或其他)的集成，所以可以与任何开发工具和任何类型的文件一起使用。与 Github Desktop 一类的传统图形化交互不同，与 TortoiseGit 的交互主要利用 Windows 资源管理器的上下文菜单，因此不需要打开任何软件，十分轻量、便捷。

* TortoiseGit 通过鼠标右键菜单栏的方式进行 git 命令的交互
* 在完成项目的代码更新后，可以右键选择 Git 提交进行add、commit以及push操作
* 当需要更新本地代码时，可以右键选择TortoiseGit，再选择拉取进行fetch

## 8.3 Vscode Git

在实际项目开发过程中，往往遇到的场景是项目开发者直接通过代码编辑器进行Git操作。

* 暂存所有更改，再在消息栏中输入message并点击勾进行提交，或者使用快捷键Ctrl+Enter进行提交
* 完成add和commit操作后，点击同步，即可以push到远端
* 使用vscode解决冲突，在提交代码的时候，一定要先拉取代码；不然就会造成冲突
