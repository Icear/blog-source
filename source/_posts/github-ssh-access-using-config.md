---
title: Linux 下配置通过 SSH KEY 访问 Github 以及自定义 key 名后的配置
date: 2020-01-15 20:40:31
updated: 2020-01-15 20:40:31
tags: [Linux, Github, SSH key]
toc: true
---

最近将开发环境集体转移到了 WSL 环境，既然是 Linux 的环境就用 Linux 的机制。在 Windows 下常用储存敏感凭据的方式是 Windows 凭据管理，git 对此有较好的支持，可以直接储存账号密码；而在 linux 下 ssh key 是个更加方便的方式，可以把各种凭据收容在一起管理。于是切换环境后远端 git 仓库的访问就改成使用 ssh key 了。下面把过程列出：

## 生成 ssh key
生成 key 的话可以使用 ssh-keygen，在 shell 里使用这个命令就可以进入创建流程，根据提示流程走完命令后它会在 ~/.ssh/ 目录下生成 id_rsa 与 id_rsa.pub 两个文件，文件名称可以在生成过程中修改，但是后缀是相同的：一个无后缀，保存私钥；另一个后缀为 .pub，保存公钥

## 提交公钥部分至 GitHub
进入 https://github.com/settings/keys 页面，选择 New SSH key 选项。里面 title 项是用来备注的，自己填一填，下面的 key 一栏填刚才的生成的 key 的公钥部分，即 id_rsa.pub，然后保存即可

## 验证密钥
尝试 ssh git@github.com，如果配置正确的话 Github 的反馈应该与此类似
> Hi *** ! You've successfully authenticated, but GitHub does not provide shell access.


## 更新 git remote url
使用 SSH key 方式之后访问自己 GitHub repo 的方式会有变化，访问 repo 时使用的 url 格式为 <pre>git@github.com:&lt;username&gt;/&lt;repo-name&gt;.git</pre>

补充更新本地 git 仓库的 remote url 的方式：
<pre>git remote set-url origin git@github.com:&lt;username&gt;/&lt;repo-name&gt;.git</pre>

## （可选）自定义 key 名称后的额外配置
如果你的 key 名称保持为 id_rsa 的话，到此 git 已经可以正常运作了。

如果你的 key 名称有变化的话，可以通过设置 ~/.ssh/config 来让 git 找到正确的密钥，因为 git 通过 SSH key 方式访问远程仓库实际上是通过调用 ssh 来操作的，只要让 ssh 能够找到密钥，git 就可以正确运作。

具体操作是，在 ~/.ssh/config 文件中添加以下几行:
```
Host github.com
    HostName github.com
    port 22
    User git
    IdentityFile <your-private-key-path>
```
其中 IdentityFile 指向私钥路径。

至此配置结束