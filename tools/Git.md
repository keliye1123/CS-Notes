# Git

- [Git](#Git)
  - [一、git设计理念->解决问题](#一git设计理念解决问题)
  - [二、git安装](#二git安装)
  - [三、git配置](#三git配置)
  - [四、git仓库的基本使用（本地）](#四git仓库的基本使用本地)
  - [五、git中文件的四种状态](#五git中文件的四种状态)
  - [六、分支操作](#六分支操作)
  - [七、语义化版本号与工作流](#七语义化版本号与工作流)
  - [八、git仓库其他常用操作](#八git仓库其他常用操作)


## 一、git设计理念解决问题
- 1.版本控制，快照存档。
- 2.分支，并行开发。
- 3.remote,协作开发。

## 二、git安装
在官网找到对应的安装包  
https://git-scm.com/install/windows  
https://git-scm.com/instal1/mac  
https://git-scm.com/install/linux

## 三、git配置

### 1.配置姓名和邮箱
~~~bash
git config --global user.name "名字"
git config --global user.email 邮箱
~~~  
- name: 出现在 commit 元数据中，协作时别人认识你   
- email: 个人联系方式，与 Github 无关  

### 2.查看git配置信息
~~~bash
git config --list
~~~

### 3.git三份配置文件
~~~  
    层级           作用范围          优先级
    system(系统级) 整台机器所有用户    低
    global(用户级) 当前用户           中
    local(项目级)  当前仓库           高
~~~  
- 查找三份配置文件的位置
    - windows:
        - system:C:\Program Files\Git\etc\gitconfig或C:\ProgramData\Git\config（取决于安装方式）
        - global:C:\Users\<用户名>\.gitconfig
        - local:<你的项目目录>\.git\config
    - Linux：
        - system:/etc/gitconfig
        - global:/home/<用户名>/.gitconfig
        - local:<项目目录>/.git


## 四、git仓库的基本使用（本地）

### 1.创建一个仓库 
在项目目录下执行
~~~bash
git init
~~~
- 会在当前目录下生成.git目录=git仓库

### 2.提交
~~~bash
git add README.md
git commit -m 'first commit'
git log
~~~
- git log 可以查看提交记录
- git log --pretty=oneline --graph，高级显示

## 五、git中文件的四种状态
- Untracked:未被git跟踪状态（在git中是不可见的，git不会对其进行任何操作）
- Staged:该文件的在add时状态会进入到下一次commit提交中。
- Unmodified:该文件相对于上一次commit没有被更改，通过.gitignore文件可变为Untracked状态。
- Modified:该文件相对于上一次commit更改了。Modified状态要通过add变成Staged状态再commit才能变成Unmodified状态。
- 除了Untracked,其余的都是tracked状态，git是可见的。
- git status 可以查看文件状态
![Git_image1.png](Image/tools/Git/Git_image1.png)

## 六、分支操作

### 1.创建分支
~~~bash  
git branch 1.0-dev
~~~

### 2.切换分支
~~~bash  
git checkout 1.0-dev
~~~
![Git_image2.png](Image/tools/Git/Git_image2.png)

### 3.分支合并,将1.0-dev分支拉去到master分支
~~~
git checkout master
git merge 1.0-dev
~~~

### 4.取消fast forward
第一条为单次使用取消，第二条为全局取消
~~~bash
git merge --no-ff 1.0-dev
git config --global merge.ff false
~~~

## 七、语义化版本号与工作流
![Git_image3.png](Image/tools/Git/Git_image3.png)
![Git_image4.png](Image/tools/Git/Git_image4.png)
![Git_image5.png](Image/tools/Git/Git_image5.png)
![Git_image6.png](Image/tools/Git/Git_image6.png)

## 八、git仓库其他常用操作

### 1.对上次提交进行修改，覆盖上一次提交
~~~bash
git commit --amend
~~~

### 2..gitignore文件的使用
在.gitignore中的文件会被Untrack。

### 3.删除文件操作 
.gitignore对已经track的文件没用，必须用以下命名  
~~~bash  
git rm --cached xxx
~~~

### 4.Revert和Reset
revert -> 创建一次 commit，反转上一次 commit 的更改
~~~bash
git revert HEAD
~~~
- reset:回退上一次commit
~~~bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
~~~
- soft:把更改退回staged
- mixed:把更改退回 modified
- hard:把更改彻底删除  

### 5.更改分支名
~~~bash
git checkout 旧分支名
git branch -m 新分支名
~~~

