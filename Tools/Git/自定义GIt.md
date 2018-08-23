## 忽略特殊文件

.gitignore文件里面存放的是一些需要忽略的文件的配置信息，如果被.gitignore文件忽略的文件需要强制添加可以使用以下命令：

```
git add -f App.class
```

或者说，如果发现.gitignore文件配置有问题，可以使用如下了命令查看哪个规则写错了：

```
git check-ignore -v App.class
```



## 设置别名

如果有经常使用的操作不好记或者是容易出错，我们可以使用别名的方法，使用自定义的命令来完成：

```
git config --global alias. co checkout // 就可以用co替换checkout了
```

--global命令表示对当前用户起作用，如果不加的话，表示的是只对当前仓库起作用。其实别名的配置都是放在了.git/config文件里面的，[alias]下面的就是别名的而配置。

```
$ cat .git/config 
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
    ignorecase = true
    precomposeunicode = true
[remote "origin"]
    url = git@github.com:michaelliao/learngit.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = origin
    merge = refs/heads/master
[alias]
    last = log -1
```

当前库的别名配置文件是在当前库.git/config文件里，而当前用户的Git别名配置文件是在C盘的.git/config文件里。