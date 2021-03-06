---
title: "Git指南-实战篇"
date: 2021-09-18 14:14:43 +/-TTT
categories: [Tech, Git]
tags: [git]
---

# 准备阶段

## 创建Github仓库

首先，我们打开Github，新建一个仓库Test，用来学习团队Git合作开发：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitNewRepository.png)

## 创建本地仓库

然后在本地电脑上新建一个同名的文件夹Test📂，这里因为是示例项目，全程只用一个README.md文件，所以我们只用在Test文件夹里创建一个`README.md`文件：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitCreateREADME.png)

现在我们先在`README.md`文件写入我们第一次要提交的内容：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitMain1thCommit.png)

其中[main]表示的是当前的分支名称，其后的内容表示第一次修改的内容信息。

## 提交本地仓库至远程仓库

接着我们回到Test文件夹里，单击鼠标右键，选择`Git Bash Here`，进入Git命令行界面，按照创建Github仓库时给出的提示（上面第一张图中的内容）依次输入命令初始化仓库：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitInit.png)

接下来返回Github仓库刷新一下界面就可以看到我们刚刚推上去的README.md文件了。

让我们回到电脑本地的`README.md`文件里，再提交一次新的commit：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitMain2thCOmmit.png)

注意，这里作为示例，commit提交说明写得不太规范，commit提交信息规范请参考另一篇Git指南文章。

那么此时我们的分支状态如下，其中AB分别表示我们在Main分支上的两次提交：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitFeature1.png)

这个Main分支作为项目开发的主分支（这里是示例项目，正常的项目一般使用develop分支作为开发分支，main分支作为线上发布版本的分支），此时假如我们需要为项目开发一个新的功能，**先从Main分支上切出一个新的分支`feature`**，基于此分支进行开发。首先我们现在Git Bash内输入下面这行代码，表示新建并切换到`feature`分支上：

```
git checkout -b feature
```

此时我们已经创建了一个分支feature，并切换到该分支上进行工作了：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitCheckoutBFeature.png)

接下来我们分别在此分支上进行三次commits：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitFeature3Commits.png)

并使用`git log --oneline`查看log记录：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitLog1.png)

此时我们的分支状态如下，其中CDE分别表示在Feature分支上的三次提交：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitFeature2.png)

# 多人协作开发

这里我们先模仿此时有另外一位同事向远端提交了代码，**首先checkout回main分支**，并新加一次commit，push到远端仓库：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitAnotherCommit.png)

此时的分支状态如下，F表示我们模仿的同事的提交：

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitFeature3.png)

那么这时如果已经开发完新功能的你**想要push代码到远程仓库上去**的话，有两种选择。

## Git Merge

**第一种选择是merge**，即将`feature`分支与`main`分支内容合并，并产生一个新的提交记录，对应的步骤如下：

1. 在`feature`分支输入`git pull origin main`拉去远程仓库main分支的最新代码，如果产生冲突则打开指定文件解决冲突。
2. 解决完冲突后分别git add、git commit
3. 然后输入`git push --set-upstream feature`，表示**提交当前分支内容到远程一个名为`feature`的分支上**，如果没有则自动创建该远程分支。
4. 此时打开Github仓库，你会发现多出了一个新的分支，进入该分支根据提示提交PR，选择`Create a merge commit`即可合并`feature`代码到`main`分支上，别忘了**合并后选择`Delete Branch`删除远程仓库的该分支**哦。

OK，这就是第一种合并的方法，也是最简单、最普遍的提交方法，你不需要关心无法提交、代码内容覆盖等问题，但是，这种方法也有很大的问题暴露出来了：

- 在Github每次使用`Create a merge commit`时都会按照提交时间先后合并所有的commits记录，并在合并完成后创建一次新的commit提交，这样会导致commits历史提交记录繁杂冗余，**一是不方便他人review代码，二是不方便之后如果代码出了问题需要回退版本**，因为commits太多太乱，很难找到自己要回退的版本。

所以这里我个人更加推荐第二种提交方式：**git rebase**。

## Git Rebase

Rebase翻译过来是变基的意思，而`git rebase`就是改变当前分支的基（这里我也描述不清楚）。

### 📖理解

举个现实中的例子，假如一名大叔年过三十仍未娶妻，于是他打算收养一名孩子，虽然收养来的孩子待遇和亲身孩子一样，但血脉却和大叔的不一样。但是`git base`这个黑魔法却可以让大叔收养来的这个孩子血脉也变得和大叔一样，也就是说`rebase`让这个收养来的孩子变成了大叔的亲生孩子。（这个例子举的或许有些离谱，但只是我个人的理解哈😅）

那么上面提到的大叔就是在我们创建Feature分支前的Main分支，领养孩子的行为相当于新建一个分支，并将其命名为`Feature`，当然，这时新建的分支的`base`其实和Main还是一模一样的，所以没必要`rebase`，但是当我们模拟一名同事提交一个新的commit到Main上后，Main分支的`base`就变了，这里我们就需要用到**黑魔法`git rebase`**来把`feature`分支重新拼接到`Main`分支上，即将feature分支的base同步为最新的。

其实在你push代码之前，还可以使用`git rebase -i`指令来完成对commits的一系列操作，所以这里我会先介绍其的使用。

### 🎯实战

#### 1. push之前 - 对commits进行的骚操作

##### 🌠squash - 压缩多条提交记录

在`Git Merge`中我们已经提到过直接Merge代码会导致commits冗余繁杂，那么其实我们可以使用`git rebase -i`来压缩合并多次commit记录：

1. 首先切换回`feature`分支，这里我们以合并feature分支上的最后三次提交为例来介绍该指令。首先在命令行中输入下面的指令：

   ```
   git rebase -i HEAD~3
   ```

   这里我们在最后加了个**参数`HEAD~3`**，表示对最后三次提交记录进行操作。按下`Enter`，你会进入下图的界面：

   ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitRebase1.png)
   
   别看上面一大串英文，其实有效代码只有前三行，剩下的都是注释说明，介绍`git rebase -i`指令的使用说明，这里我们即将要用到的`s，squash <commit>`表示压缩提交记录，**它会自动压缩当前commit与上一条commit的提交**，所以下面我们只需要将后两个`pick`指令改为`s`或者`squash`即可。

2. 按下`i`键进入编辑模式，并修改后两行的`pick`为`s`，修改完后按下`Esc`键，并输入`:wq`（这属于vim编辑器的操作）：

   ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitRebase2.png)

3. 第二步完成后按下`Enter`，会进入下面的界面：

   ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitRebase3.png)

   在上图中的第一行和第二行之间新建一行，并在其中写下此次合并Commit的主要信息，剩下的几行是对压缩的每次提交的额外描述（别忘了点击`i`进入编辑模式哦）：

   ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitRebase4.png)

   编辑完后按`Esc`，并输入`:wq`然后Enter表示保存并退出。

   因为在这里我们可以对压缩的每个commit进行详细说明，所以不会产生别人看不到你的commit的情况，因此我个人更推荐这样的提交方式。

4. 之后你会看到压缩成功的提示。输入`git log --oneline`查看历史记录，你会发现最后三次提交已经被压缩成了一次提交：

   ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitLog2.png)

   PR之后的效果如下图所示：

   ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitPR1.png)

##### 🌠reword - 修改提交信息

`r/reword`：修改commit的信息，注意`reword`只会修改commit的提交说明信息，而不会修改本次commit的具体内容。注意，**reword之后会修改commit的SHA值**。

##### 🌠edit - 插入新的commit或者修改commit具体内容

`e/edit`：它可以帮助你**在过去的某个commit间插入新的commit**或者**修改某次commit的具体内容**（注意这里要和`reword`区分开）。修改成edit操作后，git会先回退到edit指向的commit记录，之后你可以对当前版本的commit进行修改，修改完后使用`git add .`将文件添加到暂存区，此时你有两个选择，如果你不想新增一个commit，直接使用`git commit --amend`，然后再`git rebase --continue`；如果你想在当前commit之后再新增一条commit，直接使用`git commit -m"提交信息"`，然后再`git rebase --continue`来新增commit。注意：**edit以及之后的commits的SHA值都会被修改**。

##### 🌠fixup - 压缩提交但丢弃较晚的提交信息

`f/fixup`：它类似于`squash`，但我们使用`squash`的时候会被要求添加压缩提交的具体信息，但是使用`fixup`时Git会自动以较早的提交信息作为此次压缩的info。举个例子，假如现在你想要对下面这两条提交进行fixup操作，那么首先使用`git rebase -i HEAD~2`进行编辑指令界面，然后修改第二行的`pick`为`f`，按下Enter后会直接提示你变基成功。

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitLog3.png)

之后我们查看log信息如下图所示，可以发现：第四次commit和第五次commit被压缩生成了一次新的commit，并新commit的提交信息和第四次的commit的信息一样，而且要注意的是，新生成的commit的SHA值也会重新生成。

![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitLog4.png)

##### 🌠exec - 执行shell语句

`e/exec`：这个我很少用到，就在这里简单记录一下叭。就是在进入编辑指令界面后任意一行插入`e`+shell指令，在变基时会自动执行。例如`e echo "This is a echo..."`，Enter后就可以看到命令行里多出一句`Executing: echo "This is a echo......"`。

##### 🌠drop - 删除提交

`d/drop`：这个使用起来比较简单，直接将`pick`指令换成`d`即可。但是需要注意的是建议**只对最后一次的commit进行删除**，如果你删除了前面的commit，那么git会从你删除的commit后最早的一次commit开始，重新合并解决冲突，直到最晚的commit为止。

举个例子，假如现在这里有五次commit，你对第三次的commit执行了`drop`命令，那么此时git的HEAD会首先指向第二次的commit，接着它会依次与第四次commit、第五次commit合并，如果遇到冲突则解决冲突，对于每一次的合并操作，都会改变其SHA值并提醒你重新输入commit提交信息。所以如果你突然删除了中间的某个commit，但是(你要删除的commit)之后的commits都是基于该commit来完成的，这样就会为你的开发带来一定的麻烦。

参考博客：

[【git 整理提交】git rebase -i 命令详解_the_power的博客-CSDN博客](https://blog.csdn.net/the_power/article/details/104651772/)

[git rebase使用场景 - 看风景就 - 博客园 (cnblogs.com)](https://www.cnblogs.com/mengff/p/11608864.html)

[彻底搞懂 Git-Rebase - Jartto's blog](http://jartto.wang/2018/12/11/git-rebase/)

#### 2. push代码到Github

- 第一步：在提交你的代码之前，请先使用`git pull origin main --rebase`来拉取远程仓库中**main分支**的代码，`--rebase`表示在拉取代码后执行`git rebase`操作。**👻注意**，这里你需要**分别对每个**commit执行解决冲突操作，并在每次解决完冲突后可以选择修改此次解决冲突后commit的提交信息。所以为了避免麻烦，可以选择第一个步里的`squash`指令，先将几次commit合并后再提交。

- 第二步：使用`git push`提交你的代码到远程仓库

- 第三步：打开Github仓库页面，切换到`feature`分支上会有提示合并的消息，选择`Open pull request`：

  ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitPR2.png)

  按照提示一直点下去，最后选择`Rebase and merge`选项，并在合并完成之后删除该分支：

  ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitPR3.png)

  ![](https://gitee.com/aefottt/pictures/raw/master/ARTS-1/GitPR4.png)

  关于三种PR方式的不同我会在另一篇Git指南文章中讲到。

这样你就成功将代码安全的提交到Github远程仓库上去了。

### 🚨注意

#### rebase的实质

`Git Rebase`虽然强大，但也有一些它的缺陷。其实在上面的内容中相信你也已经发现了，每个commit都有属于自己的专属hash值（相当于身份证），但是`git rebase`的操作会**改变目标commit的hash值**，因为它是通过将你现有的commits复制一份到新的commits上，并在新的commits上进行改进，之后将原来的commits再**删除**。

> When you rebase stuff, you’re abandoning existing commits and creating new ones that are similar but different.
>
> 变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。
>
> (摘自[Pro Git 变基 (git-scm.com)](https://git-scm.com/book/zh/v2/Git-分支-变基))

#### rebase带来的风险

举个栗子：

这里有一个远程分支A，A上有两次提交B和C，现在你和同事张三一起把分支A拉取到本地进行开发。假如你对提交B或C进行了`git rebase`操作，并将其推送到了远程分支上，那么张三在完成开发后打算拉取远程最新代码时，就不得不重新整合他的代码，因为在张三那里，提交B和C的hash值和最初A仓库中的是一样的，而由于你的`rebase`操作改变了这两个提交的hash值，那么很尴尬的处境就在张三那里发生了，两个一模一样的提交同时出现在了他那里，这让张三感到很混乱且头大。

再举个不恰当的例子方便理解，假如你的妻子生了一个孩子，你俩共同抚养其长大，结果突然有一天你发现这个孩子的血脉和你不一样，一问才知道，你的妻子使用了黑魔法`git rebase`让这个孩子的血脉变成别人的了，你说你气不气。👻

> **Do not rebase commits that exist outside your repository and that people may have based work on.**
>
> If you follow that guideline, you’ll be fine. If you don’t, people will hate you, and you’ll be scorned by friends and family.
>
> **如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。**
>
> 如果你遵循这条金科玉律，就不会出差错。 否则，人民群众会仇恨你，你的朋友和家人也会嘲笑你，唾弃你。
>
> (摘自[Pro Git - 变基 (git-scm.com)](https://git-scm.com/book/zh/v2/Git-分支-变基))

即**不要对已经推送到远程分支的commits进行变基操作**。

#### 如何避免风险

- 第一种方法就是当你推送分支时使用`git push --force-with-lease`，如果远端有其他人推送了新的提交，那么你此次的提交会被自动拒绝。可以参考博客[Git 更安全的强制推送，--force-with-lease_walterlv - 吕毅-CSDN博客](https://blog.csdn.net/wpwalter/article/details/80371264)

- 第二种方法就是在你每次提交前都要使用`git pull --rebase`操作，它会对拉取到的代码自动执行变基操作，即[Pro Git](https://git-scm.com/book/zh/v2/Git-分支-变基)中提到的**用变基解决变基**。如果你不想每次pull时都多写一遍`--rebase`的话，可以使用`git config --global pull.rebase true`语句更改pull的配置，每次pull时会自动rebase。
- 个人推荐做法：每当你打算基于某个分支进行开发时，首先基于它创建一个新的只属于你自己的分支，并在新的分支进行开发，开发完成后使用`git pull origin main --rebase`拉取远程仓库分支的代码，然后直接使用`git push`提交，最后在Github的PR里选择`Rebase and merge`进行合并操作。这也是上面关于Git Rebase实战的流程。

## Rebase vs. Merge

有人认为项目仓库的每次提交历史都在**记录实际发生过什么**，不能对它们随意修改；也有人认为提交历史是**项目过程中发生的事**，需要反复修改才能得到较好的效果。

实际上你应当视具体项目来选择用哪一种方式，抑或是两者结合着用，但无论如何，请始终记住一个原则：

**只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作，这样，你才能享受到两种方式带来的便利。**

参考文章：

[git - Why should we never use rebase with commits that have been pushed - Stack Overflow](https://stackoverflow.com/questions/48708239/why-should-we-never-use-rebase-with-commits-that-have-been-pushed)

[Git - 变基 (git-scm.com)](https://git-scm.com/book/zh/v2/Git-分支-变基)

[Git - Rebasing (git-scm.com)](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
