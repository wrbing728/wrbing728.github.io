title: 解决 hexo d 的问题之思路整理
date: 2017-02-06 00:39:41
tags:
---

## gitlab & github 多账号 ssh 配置

1. 配置 ssh： 使用 ssh-keygen -t rsa -C "email" 设置两个 ssh key 分别给 github 和 gitlab 使用，注意命名区分（id_rsa_github、id_rsa）。（自行将公钥粘贴到 gitlab 和 github）
2. 切换账号： 在 github 仓库下，使用 git config --local user.name 'name' && git config --local user.email 'email' 为 github 仓库设置独立的账号。
3. 至此，使用 ssh -T git@github 和 ssh -T git@gitlab 看是否完成 ssh 权限配置。


## hexo d 时无权限

完成以上配置后， hexo d 时仍提示 Permission denied (publickey)
再次检查是否可通过 ssh 链接 github， 使用 ssh -T git@github**.com**，发现提示无权限(两种命令表现不一致 why？）

使用 ssh -T -v git@github.com 列出错误明细：

``` bash
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /Users/wangrubing/.ssh/id_rsa
debug1: Authentications that can continue: publickey
debug1: Trying private key: /Users/wangrubing/.ssh/id_dsa
debug1: Trying private key: /Users/wangrubing/.ssh/id_ecdsa
debug1: Trying private key: /Users/wangrubing/.ssh/id_ed25519
debug1: No more authentication methods to try.
Permission denied (publickey).
```

其中最后几行 `Trying private key ...` 看出，没有使用第一步中生成的 `id_rsa_github` 配置

在~/.ssh/目录下，增加 config 文件，加上一下内容搞定

``` bash
# gitlab
Host gitlab
  HostName gitlab.${gitlab domain}.com
  IdentityFile ~/.ssh/id_rsa

# github.*
Host github
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_github
```

** ssh: Could not resolve hostname github: nodename nor servname provided, or not known ：查看 git remote url 是否一致（ssh or https）**