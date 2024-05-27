远端版本库:------------------------------------------------------------
               /\            ||
           git push       git pull
               ||            \/
本地版本库:------------------------------------------------------------
               /\            ||                 ||          ||
          git commit  git reset --soft          ||          ||
               ||            \/                 ||          ||
索引缓存区:------------------------------------------------------------
               /\            ||                 ||          ||    
            git add  git restore --stage  git reset --mixed ||    
               ||            \/                 \/          ||    
已修改文件:------------------------------------------------------------
                             ||                             ||         
                        git restore                  git reset --hard 
                             \/                             \/         
未修改文件:------------------------------------------------------------


-------------------------------------------------------------------------------------
删除所有提交
git checkout --orphan latest_branch
git add -A
git commit -am "init"
git branch -D main
git branch -m main
git push -f origin main
-------------------------------------------------------------------------------------
.git/目录内容
hooks/*.sample
info/exclude
logs/refs/heads/master       ->0000000000000000000000000000000000000000 b5886646cf79fba2108afe49f55d392aa5c4d1d2 xt9852 <xt9852@github.com> 1628853974 +0800	clone: from https://github.com/xt9852/cpu.git
logs/refs/remotes/origin/HEAD->0000000000000000000000000000000000000000 b5886646cf79fba2108afe49f55d392aa5c4d1d2 xt9852 <xt9852@github.com> 1628853974 +0800	clone: from https://github.com/xt9852/cpu.git
logs/HEAD                    ->0000000000000000000000000000000000000000 b5886646cf79fba2108afe49f55d392aa5c4d1d2 xt9852 <xt9852@github.com> 1628853974 +0800	clone: from https://github.com/xt9852/cpu.git
objects/info/
objects/pack/pack-08e60f9c9c60c73dbfc08e7f515cf24f0b0d5127.idx
objects/pack/pack-08e60f9c9c60c73dbfc08e7f515cf24f0b0d5127.pack
refs/heads/master            ->b5886646cf79fba2108afe49f55d392aa5c4d1d2
refs/remotes/origin/HEAD     ->ref: refs/remotes/origin/master
refs/tags/
config
description
FETCH_HEAD ->b5886646cf79fba2108afe49f55d392aa5c4d1d2		branch 'master' of https://github.com/xt9852/cpu
HEAD       ->ref: refs/heads/master
index      ->DIRC
packed-refs->b5886646cf79fba2108afe49f55d392aa5c4d1d2 refs/remotes/origin/master

-------------------------------------------------------------------------------------
git的四种对象:数据对象,树对象,提交对象,标签对象

       tree
     /  |    \
 README file lib
   /    |      \
blob   blob    tree
               /  \
             fileA fileB
             /      \
           blob     blob

提交对象指向tree
标签对象指向tree,也可指向其它文件

文件结构:zlib("blob {content.length}\u0000{content}")

可以通过 irb 命令启动 Ruby 的交互模式：
$ irb
>> content = "what is up, doc?"
=> "what is up, doc?"

>> header = "blob #{content.length}\0"
=> "blob 16\u0000"

>> store = header + content
=> "blob 16\u0000what is up, doc?"
>> require 'digest/sha1'
=> true
>> sha1 = Digest::SHA1.hexdigest(store)
=> "bd9dbf5aae1a3862dd1526723246b20206e5fc37"

>> require 'zlib'
=> true
>> zlib_content = Zlib::Deflate.deflate(store)

$ git cat-file -p bd9dbf5aae1a3862dd1526723246b20206e5fc37
what is up, doc?

-------------------------------------------------------------------------------------
// 添加对象,无名字
echo 'test content' | git hash-object -w --stdin
4894e536120417b9897821d3301120c4114ad522

// 添加对象,有名字
echo 'version 1' > test.txt
git hash-object -w test.txt
d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566

// 添加对象,有名字
echo 'version 2' > test2.txt
git hash-object -w test2.txt
4001e264a0d1dc40170a99cd402803d9e4640fba

// 显示内容
git cat-file -p 4894e536120417b9897821d3301120c4114ad522
'test content'
git cat-file -p d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566
'version 1'
git cat-file -p 4001e264a0d1dc40170a99cd402803d9e4640fba
'version 2'

// 显示类型:blob数据对象（blob object）
git cat-file -t 4894e536120417b9897821d3301120c4114ad522
blob
git cat-file -t d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566
blob
git cat-file -t 4001e264a0d1dc40170a99cd402803d9e4640fba
blob

// 添加4894e536120417b9897821d3301120c4114ad522(无文件名)进索引中
git update-index --add --cacheinfo 100644 4894e536120417b9897821d3301120c4114ad522 content.txt
git update-index --add --cacheinfo 100644 d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566 test.txt
git update-index --add --cacheinfo 100644 4001e264a0d1dc40170a99cd402803d9e4640fba test2.txt

-------------------------------------------------------------------------------------
// 新建树(目录)对象
git write-tree
5a1442f3af0b27dbf5feceaa813d8627a95bdcd9

git cat-file -t 5a1442f3af0b27dbf5feceaa813d8627a95bdcd9
tree

git cat-file -p 5a1442f3af0b27dbf5feceaa813d8627a95bdcd9
100644 blob 4894e536120417b9897821d3301120c4114ad522    content.txt
100644 blob d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566    test.txt
100644 blob 4001e264a0d1dc40170a99cd402803d9e4640fba    test2.txt

git cat-file -p f8bebff99b4386f5a69affc91307be4c575c85b6
100644 blob d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566    test.txt
100644 blob 4001e264a0d1dc40170a99cd402803d9e4640fba    test2.txt

// 添加进当前目录
git read-tree --prefix=bak f8bebff99b4386f5a69affc91307be4c575c85b6

git write-tree
02b09eb674e2b53f57cdbc70c550d24160e46117

git cat-file -p 02b09eb674e2b53f57cdbc70c550d24160e46117
040000 tree f8bebff99b4386f5a69affc91307be4c575c85b6    bak
100644 blob 4894e536120417b9897821d3301120c4114ad522    content.txt
100644 blob d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566    test.txt
100644 blob 4001e264a0d1dc40170a99cd402803d9e4640fba    test2.txt

// 当前状态
D:\2.code\git-test>git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   bak/test.txt
        new file:   bak/test2.txt
        new file:   content.txt
        new file:   test.txt
        new file:   test2.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    bak/test.txt
        deleted:    bak/test2.txt
        deleted:    content.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        git

-------------------------------------------------------------------------------------
// 创建提交对象,指向树对象
D:\2.code\git-test>echo 'first commit' | git commit-tree 02b09eb674e2b53f57cdbc70c550d24160e46117
81cd8925bfb46312fcf8ab9478384b69d9800129

D:\2.code\git-test>git cat-file -p 81cd8925bfb46312fcf8ab9478384b69d9800129
tree 02b09eb674e2b53f57cdbc70c550d24160e46117
author xt9852 <xt9852@github.com> 1628860074 +0800
committer xt9852 <xt9852@github.com> 1628860074 +0800

'first commit'

D:\2.code\git-test>git cat-file -t 81cd8925bfb46312fcf8ab9478384b69d9800129
commit

git log --stat 81cd8925bfb46312fcf8ab9478384b69d9800129
error: cannot spawn less: No such file or directory
commit 81cd8925bfb46312fcf8ab9478384b69d9800129
Author: xt9852 <xt9852@github.com>
Date:   Fri Aug 13 21:07:54 2021 +0800

    'first commit'

 bak/test.txt  | 1 +
 bak/test2.txt | 1 +
 content.txt   | 1 +
 test.txt      | 1 +
 test2.txt     | 1 +
 5 files changed, 5 insertions(+)

-------------------------------------------------------------------------------------
// 添加引用文件
echo 81cd8925bfb46312fcf8ab9478384b69d9800129 > .git/refs/heads/master
或
git update-ref refs/heads/master 81cd8925bfb46312fcf8ab9478384b69d9800129

D:\2.code\git-test>git log
error: cannot spawn less: No such file or directory
commit 81cd8925bfb46312fcf8ab9478384b69d9800129 (HEAD -> master)
Author: xt9852 <xt9852@github.com>
Date:   Fri Aug 13 21:07:54 2021 +0800

    'first commit'

添加分支引用,多了个文件.git/refs/heads/test
git update-ref refs/heads/test 81cd8925bfb46312fcf8ab9478384b69d9800129

D:\2.code\git-test>git log
error: cannot spawn less: No such file or directory
commit 81cd8925bfb46312fcf8ab9478384b69d9800129 (HEAD -> master, test)
Author: xt9852 <xt9852@github.com>
Date:   Fri Aug 13 21:07:54 2021 +0800

    'first commit'

// 当前分支,就是.git/HEAD的内容ref: refs/heads/master,是一个引用指向当前分支
git symbolic-ref HEAD
refs/heads/master

D:\2.code\git-test>git branch
error: cannot spawn less: No such file or directory
* master
  test

// 切换分支,HEAD文件内容为ref: refs/heads/test
D:\2.code\git-test>git symbolic-ref HEAD refs/heads/test

D:\2.code\git-test>git branch
error: cannot spawn less: No such file or directory
  master
* test

// 切换分支
D:\2.code\git-test>git checkout master
Switched to branch 'master'
D       bak/test.txt
D       bak/test2.txt
D       content.txt

D:\2.code\git-test>git branch
error: cannot spawn less: No such file or directory
* master
  test

-------------------------------------------------------------------------------------
// 添加标签v1.0,在.git/refs/v1.0中,指向一个 提交对象
git update-ref refs/tags/v1.0 81cd8925bfb46312fcf8ab9478384b69d9800129

git tag -a v1.1 81cd8925bfb46312fcf8ab9478384b69d9800129 -m 'test tag'

// 有两个tag
D:\2.code\git-test>git tag
error: cannot spawn less: No such file or directory
v1.0
v1.1

D:\2.code\git-test>git cat-file -t 83aac346c96a6c058ca96dd2d00c8e298ede3bca
tag

D:\2.code\git-test>git cat-file -p 83aac346c96a6c058ca96dd2d00c8e298ede3bca
object 81cd8925bfb46312fcf8ab9478384b69d9800129
type commit
tag v1.1
tagger xt9852 <xt9852@github.com> 1628862604 +0800

test tag

-------------------------------------------------------------------------------------
// 添加一个远端版本库,名称为origin
D:\2.code\git-test>git remote add origin https://github.com/xt9852/git-test.git

// 配置文件config中多了
[remote "origin"]
	url = git@github.com:xt9852/git-test.git
	fetch = +refs/heads/*:refs/remotes/origin/*

// 推送到远端版本库,会新增文件.git/refs/remotes/origin/master
git push origin master

-------------------------------------------------------------------------------------
// 包文件（packfile）,将多个对象打包成一个称为“包文件（packfile）”的二进制文件

// gc,垃圾整理,将多个小文件,整合成一个大文件,多了个文件.git\info\refs
D:\2.code\git-test>git gc
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (7/7), done.
Total 7 (delta 0), reused 0 (delta 0), pack-reused 0

D:\2.code\git-test>git verify-pack -v .git\objects\pack\pack-d15eafbb82083d27f528ebfa2f08cfae3217e411.idx
81cd8925bfb46312fcf8ab9478384b69d9800129 commit 169 121 12
83aac346c96a6c058ca96dd2d00c8e298ede3bca tag    130 121 133
02b09eb674e2b53f57cdbc70c550d24160e46117 tree   142 134 254
f8bebff99b4386f5a69affc91307be4c575c85b6 tree   73 75 388
d2a4cbf60df98ddfaaea1facf3ce68f66ef7e566 blob   13 22 463
4001e264a0d1dc40170a99cd402803d9e4640fba blob   13 22 485
4894e536120417b9897821d3301120c4114ad522 blob   17 27 507
non delta: 7 objects
.git\objects\pack\pack-d15eafbb82083d27f528ebfa2f08cfae3217e411.pack: ok
