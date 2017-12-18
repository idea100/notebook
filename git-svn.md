最近在研究了一下如果使用git客户端来管理svn

## 关于在svn项目中使用git

svn作为一个优秀源码版本的管理工具，可以适合绝大多数项目。但是因为它的采用中心化管理，不可避免的存在本地代码的备份和版本管理问题。也就是说对于尚未或暂无法提交到Subversion服务器的本地代码来说，存在着被误删除和版本更新无法回退两大情形。
git作为一个分布式版本管理工具，可以很好的解决这个问题。因为它的大多数操作是在本地进行的。这里要说的是git是如何做到既可以管理好本地代码又可以与已有的SVN中心库进行同步的。

#### 第一步 将svn上的仓库clone到本地
假设clone一个工程名为vendor-web并且有2600多个commit，如果全部clone下来会很慢，我们只取其中最后的100个commit，
  ```
    git svn clone -r2500:HEAD https://develop.example.com/trunk/vendor-web/
  ```
从第2500个commit开始clone仓库，这里clone下来的仓库就是master分支。

#### 第二步 拉取vendor-web的分支
毕竟是svn，拉分支的方式比较繁琐，远没有git好用。假设我们拉取branches/dev这个分支

  ```
    cd vendor-web
    git config --add svn-remote.dev.url https://develop.example.com/branches/dev/vendor-web/
    git config --add svn-remote.dev.fetch :refs/remotes/git-svn-dev
    git svn fetch dev
    git checkout -b dev refs/remotes/git-svn-dev
  ```
这样我们就将svn上面的dev这个分支拉取下来了

#### 第三步 既然主干和分支都拉取下来了，我们就可以为所欲为的使用git啦
首先来试试git checkout
  ```
    git checkout master
    git checkout dev
  ```
这种切分支的方式真是切的不要太爽。

再来试试提交代码到svn,假设我们需要在dev上面提交代码，首先切到dev分支
```
  git checkout dev
```
再假设我们修改了index.html这个文件，我们可以用git add index.html 或者git add .将所有的改动的代码添加到git仓库，然后提交到本地分支
```
  git add index.html
  // 或者 git add .
  git commit -m "提交index.html到dev分支"
```
代码提交到本地分支之后就应该push到远程仓库了，因为我们是使用svn，所以push的方式跟git还是有很大的区别
```
  // 要将本地内容提交到远程svn中，可以先让当前分支和远程svn同步
  git rebase
  // 如果在git svn rebase时发生代码冲突，需要先手动解决冲突，然后用git add将修改加入索引，然后继续rebase
  git svn rebase --continue
  // 然后将所有已经提交到1.4.1分支的本地修改提交到svn
  git svn dcommit
```

基本的git提交代码流程走通了，接下来就可以随心所欲的使用git来开发了。

```
  git svn show-ignore > .gitignore
```
