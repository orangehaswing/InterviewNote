# Git内部原理-Git引用

首先来搞清楚什么是Git引用，前文讲了Git提交对象的哈希、存储原理，理论上我们只要知道该对象的hash值，就能往前推出整个提交历史，例如：

```
$ git log --pretty=oneline 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31 third commit
d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c second commit
db1d6f137952f2b24e3c85724ebd7528587a067a first commit

```

现在问题来了，提交对象的这40位hash值不好记忆，Git引用相当于给40位hash值取一个别名，便于识别和读取。Git引用对象都存储在`.git/refs`目录下，该目录下有3个子文件夹`heads`、`tags`和`remotes`，分别对应于HEAD引用、标签引用和远程引用，下面分别讲一讲每种引用的原理。

# HEAD引用

HEAD引用是用来指向每个分支的最后一次提交对象，这样切换到一个分支之后，才能知道分支的“尾巴”在哪里。HEAD引用存储在`.git/refs/heads`目录下，有多少个分支，就有相应的同名HEAD引用对象。例如代码库里面有`master`和`test`两个分支，那么`.git/refs/heads`目录下就存在`master`和`test`两个文件，分别记录了分支的最后一次提交。

HEAD引用的内容就是提交对象的hash值，理论上我们可以手动地构造一个HEAD引用：

```
$ echo "3ac728ac62f0a7b5ac201fd3ed1f69165df8be31" > .git/refs/heads/master

```

Git提供了一个专有命令`update-ref`，用来查看和修改Git引用对象，当然也包括HEAD引用：

```
$ git update-ref refs/heads/master 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31
$ git update-ref refs/heads/master
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31

```

上面的命令我们将`master`分支的HEAD指向了`3ac728ac62f0a7b5ac201fd3ed1f69165df8be31`，现在用`git log`查看下`master`的提交历史，可以发现最后一次提交就是所更新的hash值：

```
$ git log --pretty=oneline master
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31 (HEAD -> master) third commit
d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c second commit
db1d6f137952f2b24e3c85724ebd7528587a067a first commit

```

同理，可以使用同样的方法更新`test`分支的HEAD：

```
$ git update-ref refs/heads/test d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c
$ git log --pretty=oneline test
d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c (test) second commit
db1d6f137952f2b24e3c85724ebd7528587a067a first commit

```

`.git/refs/heads`目录下存储了每个分支的HEAD，那怎么知道代码库当前处于哪个分支呢？这就需要一个代码库级别的HEAD引用。`.git/HEAD`这个文件就是整个代码库级别的HEAD引用。我们先查看一下`.git/HEAD`文件的内容：

```
$ cat .git/HEAD
ref: refs/heads/master

```

我们发现`.git/HEAD`文件的内容不是40位hash值，而像是指向`.git/refs/heads/master`。尝试切换到`test`：

```
$ git checkout test
$ cat .git/HEAD
ref: refs/heads/test

```

切换分支后，`.git/HEAD`文件的内容也跟着指向`.git/refs/heads/test`。`.git/HEAD`也是HEAD引用对象，与一般引用不同的是，它是“符号引用”。符号引用类似于文件的快捷方式，链接到要引用的对象上。

Git提供专门的命令`git symbolic-ref`，用来查看和更新符号引用：

```
$ git symbolic-ref HEAD refs/heads/master
$ git symbolic-ref HEAD refs/heads/test

```

至此，我们分析了两种HEAD引用，一种是分支级别的HEAD引用，用来记录各分支的最后一次提交，存储在`.git/refs/heads`目录下，使用`git update-ref`来维护；一种是代码库级别的HEAD引用，用来记录代码库所处的分支，存储在`.git/HEAD`文件，使用`git symbolic-ref`来维护。

# 标签引用

标签引用，顾名思义就是给Git对象打标签，便于记忆。例如，我们可以将某个提交对象打v1.0标签，表示是1.0版本。标签引用都存储在`.git/refs/tags`里面。

标签引用和HEAD引用本质是Git引用对象，同样使用`git update-ref`来查看和修改：

```
$ git update-ref refs/tags/v1.0 d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c
$ cat .git/refs/tags/v1.0
d4d2c6cffb408d978cb6f1eb6cfc70e977378a5c

```

还有一种标签引用称为“附注引用”，可以为标签添加说明信息。上面的标签引用打了一个`v1.0`的标签表示发布1.0版本，有时候发布软件的时候除了版本号信息，还要写更新说明。附注引用就是用来实现打标签的同时，也可以附带说明信息。

附注引用是怎么实现的呢？与常规标签引用不同的是，它不直接指向提交对象，而是新建一个Git对象存储到`.git/objects`中，用来记录附注信息，然后附注标签指向这个Git对象。

使用`git tag`建立一个附注标签：

```
$ git tag -a v1.1 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31 -m "test tag"
$ cat .git/refs/tags/v1.1
8be4d8e4e8e80711dd7bae304ccfa63b35a6eb8c

```

使用`git cat-file`来查看附注标签所指向的Git对象：

```
$ git cat-file -p 8be4d8e4e8e80711dd7bae304ccfa63b35a6eb8c
object 3ac728ac62f0a7b5ac201fd3ed1f69165df8be31
type commit
tag v1.1
tagger jingsam <jing-sam@qq.com> 1529481368 +0800

test tag

```

可以看到，上面的Git对象存储了我们填写的附注信息。

总之，普通的标签引用和附注引用同样都是存储的是40位hash值，指向一个Git对象，所不同的是普通的标签引用是直接指向提交对象，而附注标签是指向一个附注对象，附注对象再指向具体的提交对象。

另外，本质上标签引用并不是只可以指向提交对象，实际上可以指向任何Git对象，即可以给任何Git对象打标签。

# 远程引用

远程引用，类似于`.git/refs/heads`中存储的本地仓库各分支的最后一次提交，在`.git/refs/remotes`是用来记录多个远程仓库各分支的最后一次提交。

我们可以使用`git remote`来管理远程分支：

```
$ git remote add origin git@github.com:jingsam/git-test.git

```

上面添加了一个`origin`远程分支，接下来我们把本地仓库的`master`推送到远程仓库上：

```
$ git push origin master
Counting objects: 9, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (9/9), 720 bytes | 360.00 KiB/s, done.
Total 9 (delta 0), reused 0 (delta 0)
To github.com:jingsam/git-test.git
 * [new branch]      master -> master

```

这时候在`.git/refs/remotes`中的远程引用就会更新：

```
$ cat .git/refs/remotes/origin/master
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31

```

和本地仓库的`master`比较一下，发现是一模一样的，表示远程分支和本地分支是同步的：

```
$ cat .git/refs/heads/master
3ac728ac62f0a7b5ac201fd3ed1f69165df8be31

```

由于远程引用也是Git引用对象，所以理论上也可以使用`git update-ref`来手动维护。但是，我们需要先把代码与远程仓库进行同步，在远程仓库中找到对应分支的HEAD，然后使用`git update-ref`进行更新，过程比较麻烦。而我们在执行`git pull`或`git push`这样的高层命令的时候，远程引用会自动更新。

# 总结

到这里，三种Git引用都已分析完毕。总的来说，三种Git引用都统一存储到`.git/refs`目录下，Git引用中的内容都是40位的hash值，指向某个Git对象，这个对象可以是任意的Git对象，可以是数据对象、树对象、提交对象。三种Git引用都可以使用`git update-ref`来手动维护。

三种Git引用对象所不同的是，分别存储于`.git/refs/heads`、`.git/refs/tags`、`.git/refs/remotes`,存储的文件夹不同，赋予了引用对象不同的功能。HEAD引用用来记录本地分支的最后一次提交，标签引用用来给任意Git对象打标签，远程引用正式用来记录远程分支的最后一次提交。









