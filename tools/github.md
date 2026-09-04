# github

- [github](#github)
  - [一、为git配置github账户授权](#一为git配置github账户授权)
  - [二、github仓库的基本操作](#二github仓库的基本操作)
  - [](#)
  - [](#)
  - [](#)
  - [](#)
  - [](#)
  - [](#)


## 一、为git配置github账户授权

### 1.使用ssh协议
~~~bash
ssh-keygen
cd ~/.ssh
cat id_ed25519.pub
~~~
- ssh-keygen会生成一个公钥和一个私钥到home目录，有.pub后缀的是公钥，没有的则是私钥。我们要先把公钥给github，我们再用私钥去与github请求连接。
- 原理：ssh 采用非对称加密，连接服务器时，服务器发回Challenge，客户端用私钥加密后，把密文发回服务端，服务端用公钥解密，匹配则连接成功。

### 2.使用http协议
- 传统方式：
    - 进入 GitHub Developer Settings
    - 创建 PAT (Personal access tokens)
    - git push 时使用 username 和 token 登录
    - 原理：原理：git 访问 GitHub 时携带 HTTP 认证头：Authorization: Basic <base64>
- 现代方式：
    - 本地add remote后，首次push弹出github登录页面，直接授权登录。
  

## 二、github仓库的基本操作

### 1.仓库的基本设置
- 1.team工作流
    - 可以在仓库的setting中的Collaborators来add people添加合作者
- IM-PR工作流
    - 在setting中的rulesets设置参与者的权限。  

### 2.本地连接远程仓库
~~~bash
git remote add origin 仓库url
~~~
- origin为我们为仓库起的名字
~~~bash
git push -u origin master
~~~
- 把master分支推送到origin远程仓库
- 可在项目的.git目录下的config文件查看远程仓库信息
- "-u"的作用是每次提交都同步提交到远程仓库。  