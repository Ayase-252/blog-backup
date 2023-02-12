---
title: 2023 年了，怎么用 Github Action 与 hexo 发布在 Github Pages 上发布文章
date: 2023-02-13 00:46:14
category: Notes
tags:
  - hexo
---

每一年，每当我想发一些文章的时候，我都会和这个文章发布流程搏斗一番。
要么主题出了个 Breaking Change 导致构建错误，要么哪里莫名其妙配置变了。
今年也不例外，今年的主题是，如何用 Github Action 来自动地构建与发布文章。

这本来是一个已经解决的问题，至少在一年之前，这个流程是完全能够跑通的。但是不知道从什么时候，
在 repo URL 中通过 `username@` 的方式携带 Personal Access Token （PAT）的方式已经行不通了，
会报错。

```plain
fatal: could not read Password for 'https://***@github.com': No such device or address
FATAL {
  err: Error: Spawn failed
      at ChildProcess.<anonymous> (/home/runner/work/blog-backup/blog-backup/node_modules/hexo-deployer-git/node_modules/hexo-util/lib/spawn.js:51:21)
      at ChildProcess.emit (node:events:513:28)
      at Process.ChildProcess._handle.onexit (node:internal/child_process:293:12) {
    code: 128
  }
} Something's wrong. Maybe you can find the solution here: %s https://hexo.io/docs/troubleshooting.html
```

这是可能是因为 Github 不认携带在 URL 上的 PAT。转而向用户请求密码，
但是在 CI 环境里没有键盘这样的输入设备，然后就报错了。

hexo 官方的 hexo-deploy-git 在支持 token 方面也采用了[上述方法](https://github.com/hexojs/hexo-deployer-git/blob/f125535fb2477ab71702b9ca819223f715d7e628/lib/parse_config.js#L33)。
即使正确配置 token，也会部署失败。hexo-deploy-git 在 token 配置上还有一些坑，
例如当 `repo` 字段是字符串时，`token` 字段是不生效的。只有在 `repo` 字段里面配置 `token` 字段才有效。

```yaml
deploy:
  type: git
  repo: https://github.com/aaaa/bbb.git
  token: '' # ❌

deploy:
  type: git
  repo:
    github:
      url: https://github.com/aaaa/bbb.git
      token: '' # ✅
```

由于 PAT 现在只能够人工输入，在 CI 环境下做不到。所以，我们似乎只能转向使用 SSH 的方式来操作 git。

## 为 Github Action 创建一个 SSH

在本机创建一个 SSH Key，用 `-C` 参数注明要操作的 repo 的 git 链接。

```bash
ssh-keygen -C git@github.com:xxxx/yyy.git
```

执行生成一个新的 key，将公钥上传到 GitHub 账号的 SSH and GPG keys 中。务必注意，
SSH key 的权限比 PAT 要高，
能够读写账户里所有的 repo。所以**请务必像密码一样的保管私钥。**

把私钥通过 repo 的 Secrets and variables 设置，在 action 下配置一个 Repository secrets，
假设 secrets 的名字是 `SSH_PRIVATE_KEY`。我们可以在 `action` 的 yaml 配置中用
`${{ secrets.SSH_PRIVATE_KEY }}` 获得这个私钥。

## 在部署前增加一个配置 SSH 的 action

在部署前增加一个可以配置 ssh 的 action。
例如 [`webfactory/ssh-agent`](https://github.com/webfactory/ssh-agent)，yaml 配置如下：

```
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # ...
    - uses: webfactory/ssh-agent@v0.7.0
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
	# ... other build
```

## 修改 `_config.yaml` 中的 `deploy` 配置

将 `deploy` 中的 url 配置为 repo 的 SSH 链接，
例如 `git@github.com:a/a.github.io.git`。

做完之后，hexo deploy 会切换为通过 SSH 的方式去发布文章。
绕过了 PAT 输入困难的问题。但是，前面提到过 SSH Key 的权限很高，
会丧失 PAT 可以限制权限的好处。谨慎使用。
