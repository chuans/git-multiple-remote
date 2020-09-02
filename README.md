# git-multiple-remote

### 简介：
为了解决一个本地仓库对应多个远程地址的问题，比如此时我在做一个伟大的发明，在本地一个仓库开发，想每次提交都把代码同步到github、码云、coding、igit等远程仓库中，那么此方法用起来特别香。

-------

### 免输入密码操作远程仓库
执行远程仓库操作需要输入密码是件比较麻烦的事情，以下方法可以免密配置，注意如果你的账号密码中存在@符号需要转义%40
```
http://${你的账号}:${你的密码}@igit.58corp.com/58finance_baseService/report-system.git
```

-------

### Git Config
当我们只有一个remote的时候，config配置文件一般是这个样子（以报表平台项目为例http://igit.58corp.com/58finance_baseService/report-system, 文件地址：.git/config）

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[remote "origin"]
	url = http://${username}:${password}@github.com/test.git
	fetch = +refs/heads/*:refs/remotes/origin/*
	pushurl = http://${username}:${password}@github.com/test.git
[branch "master"]
	remote = origin
	merge = refs/heads/master
[branch "prod.1.0.0"]
	remote = origin
	merge = refs/heads/prod.1.0.0
[branch "dev"]
	remote = origin
	merge = refs/heads/dev
[branch "prod.2.0.0"]
	remote = origin
	merge = refs/heads/prod.2.0.0
[branch "prod.3.0.0_2019.10.8"]
	remote = origin
	merge = refs/heads/prod.3.0.0_2019.10.8
[branch "dev.4.0.0"]
	remote = origin
	merge = refs/heads/dev.4.0.0
```
那么我们先分析下上面配置文件

* [ core ] 内的配置信息详解请移步官网说明https://git-scm.com/docs/git-config

* [ remote "origin" ] 这个表示目前所使用的远程源地址，名为：origin
* [ remote.url ] 表示当前远程名为origin的源所使用的地址（俗话说，仓库地址，为了避免账户密码问题，最简单的方式是使用git自己在url中去查找账号密码）
* [ branch "xxx" ] 表示当前仓库里面所有的分支状态

###### 假如这个时候 我们额外添加一个远程仓库

```
git remote add github https://${username}:${password}@github.com/chuans/git-multiple-remote.git
```

这个时候配置文件在最底部会新增几行（账号密码已经打码）

```
[remote "github"]
	url = https://username:password@github.com/chuans/git-multiple-remote.git
	fetch = +refs/heads/*:refs/remotes/github/*
```

**到此我们已经新增了一个额外的源，现在可以使用命令将代码上传到新的仓库了**

```
// 使用新源
git pull github master  // 拉去github源master分支代码
git push github master  // 上传github源master分支代码

// 使用旧源
git pull origin master  // 拉去origin源master分支代码
git push origin master  // 上传origin源master分支代码
```

-------

### 每次都要输入2次命令 是不是很麻烦？
*有木有只输入一次命令就可以同时操作多个源的方法呢？*

*当然有*

可以执行以下命令，给主源（origin）添加额外的url
```
git remote set-url --add origin https://username:password@github.com/chuans/git-multiple-remote.git
```

那么现在配置文件发生改变的部分 如下所示

```
[remote "origin"]
	url = http://username:password@github.com/test.git
	url = https://username:password@github.com/chuans/git-multiple-remote.git
	fetch = +refs/heads/*:refs/remotes/origin/*
	pushurl = http://username:password@github.com/test.git
```
*我们看到在url = xxx后面有新增了一行url = xxx ，xxx的值就是我们输入的remote地址，那么这个时候再执行命令的时候，命令会去依次执行提交到url挂钩的remote源地址*
```
    /*
        这个时候执行的push操作 会分别向origin源内的两个url地址去git push本地代码
    */
    git push
```
***需要注意一点的是，remote.x.pushurl这个属性 如果配置了单命令多源的话 需要删掉，否则可能会出现push不成功的问题***


-------

### 我记不住命令啊
***办法总比困难多😁，刚刚我们执行了以下几个操作，其实最终改变的就是.git/config配置文件***

```
git remote add github https://${username}:${password}@github.com/chuans/git-multiple-remote.git
git remote set-url --add origin https://username:password@github.com/chuans/git-multiple-remote.git
```

so 我们其实只需要修改.git/config文件就可以啦

这个时候在执行git push的时候 就是提交到多个远程了


### 遇到的问题
**配置的时候出现了一个问题，就是我本地仓库和A仓库能对应上（就是一直在喝这个仓库进行上传或者拉代码操作），但是和B仓库对应不上（和B仓库从未进行一次操作）,这个时候会提示B仓库没有历史记录（好像是这个提示）**

*解决办法就是：除非能遇见后果：git push -f*

*否则需要新建远程仓库来上传*



