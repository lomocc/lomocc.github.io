---
layout: post
title: Secure Enclave SSH 密钥保护指南
tags:
  - devops
  - security
  - macos
  - ssh
---

# Secure Enclave SSH 密钥保护指南

在 Mac Secure Enclave 中保存不可导出的 SSH 私钥，并通过 Touch ID 批准使用。适用于 GitHub、GitLab、自建 Git 服务和远程 SSH 服务器；下文以 GitHub 为例。

## 1. 准备

- 使用 macOS 26 Tahoe 或更高版本；`-t bio` 还需要 Touch ID。
- 建议开启 FileVault，防止设备丢失后本机数据被离线读取；这不是使用 Secure Enclave SSH 密钥的必要条件。
- 确认可登录 GitHub，并已保存恢复码。
- 只在自己刚执行 SSH 或 Git 命令后批准 Touch ID。

```shell
command -v sc_auth # 确保 sc_auth 存在
command -v ssh-keygen # 确保 ssh-keygen 存在
ls /usr/lib/ssh-keychain.dylib # 确保 SSH provider 存在
```

三条命令都有输出即可，否则不要继续。

## 2. 创建密钥

```shell
sc_auth create-ctk-identity -l "my-ssh-se" -k p-256-ne -t bio # 创建不可导出且需要 Touch ID 的身份
```

### 参数解说

- `-l`：设置身份名称；创建多个身份时使用不同名称，例如 `github-ssh-se`、`server-ssh-se`。
- `-k`：设置密钥类型；`p-256-ne` 不可导出，`p-256` 可以导出。
- `-t`：设置私钥保护方式；`bio` 要求 Touch ID，`none` 不要求生物识别。

```shell
sc_auth list-ctk-identities # 确认 my-ssh-se 已创建
```

新身份应显示 `p-256-ne`、`bio`、`my-ssh-se` 和 `YES`。

- `D1E9...11ED`：默认显示的 SHA-1 十六进制 Hash，用于删除、导出等 `sc_auth` 操作。
- `SHA256:upo7...`：`-t ssh` 显示的 SSH 指纹，用于与 `ssh-keygen -lf` 的结果核对。

两者来自同一公钥，但格式和用途不同，不能互换。

## 3. 生成 SSH 引用文件

每创建一个身份就立即完成本节，再创建下一个身份。当前 provider 无法按标签或 Public Key Hash 筛选 `ssh-keygen -K` 返回的身份。

```shell
sc_auth list-ctk-identities -t ssh # 记录 my-ssh-se 的目标 SSH 指纹
mkdir -p ~/.ssh/ssh-se-setup && cd ~/.ssh/ssh-se-setup # 创建并进入操作目录
ssh-keygen -v -w /usr/lib/ssh-keychain.dylib -K -N "" # 生成引用文件并显示每个身份的 SSH 指纹
```

如果提示 `Enter PIN for authenticator:`，直接按回车，再使用 Touch ID。应生成 `id_ecdsa_sk_rk` 和 `id_ecdsa_sk_rk.pub`。

如果已经存在多个身份，查看每个 `do_download_sk` 后面的 `SHA256:` 指纹：`y` 表示用当前身份覆盖本地文件并继续遍历，`n` 表示立即停止遍历并保留前一个身份。到达目标身份前必须持续回答 `y`；目标身份出现时也回答 `y`，下一个身份出现时回答 `n`。如果第一个身份就是目标，在第二个身份出现时回答 `n`；如果目标是最后一个身份，最后一次回答 `y` 后命令会直接结束。

```shell
ssh-keygen -lf id_ecdsa_sk_rk.pub # 查看生成公钥的指纹
```

两个 `SHA256:` 指纹必须完全一致，否则停止操作。

```shell
mv -i id_ecdsa_sk_rk ~/.ssh/my-ssh-se # 移动并命名引用文件，禁止静默覆盖
mv -i id_ecdsa_sk_rk.pub ~/.ssh/my-ssh-se.pub # 移动并命名公钥
```

## 4. 添加公钥

GitHub：

```shell
pbcopy < ~/.ssh/my-ssh-se.pub # 复制公钥到剪贴板
```

打开 <https://github.com/settings/keys>，选择 **New SSH key**，Title 填写便于识别的设备名称，Key type 选择 **Authentication Key**，粘贴并保存。

远程 SSH 服务器：把 `my-ssh-se.pub` 的完整内容添加到目标账户的 `~/.ssh/authorized_keys`。服务器需要接受 `sk-ecdsa-sha2-nistp256@openssh.com` 公钥。

## 5. 配置 Git

```shell
nano ~/.gitconfig # 编辑 Git 全局配置
```

```gitconfig
[core]
    sshCommand = ssh -i ~/.ssh/my-ssh-se -o IdentitiesOnly=yes -o SecurityKeyProvider=/usr/lib/ssh-keychain.dylib
```

该配置默认应用于所有 Git 仓库。

### 不同仓库使用不同密钥

在默认配置后添加按目录加载规则：

```gitconfig
# ~/.gitconfig
[includeIf "gitdir:~/Projects/project-a/"]
    path = ~/.gitconfig-project-a
```

在独立配置中指定密钥：

```gitconfig
# ~/.gitconfig-project-a
[core]
    sshCommand = ssh -i ~/.ssh/project-a-ssh-se -o IdentitiesOnly=yes -o SecurityKeyProvider=/usr/lib/ssh-keychain.dylib
```

路径末尾的 `/` 表示匹配该目录下的所有仓库。其他目录使用另一份配置和密钥引用文件；remote 仍可保持 `git@github.com:OWNER/REPOSITORY.git`。

## 6. 测试

在目标 Git 仓库目录中执行：

```shell
git config --show-origin --get core.sshCommand # 确认该仓库使用的 SSH 配置
git ls-remote origin HEAD # 测试远程认证，不修改仓库
```

首次连接时，先通过可信渠道核对服务器指纹，再输入 `yes`。GitHub 指纹见[官方说明](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)。成功时会显示一行 commit hash。

## 7. 日常使用

Git 可正常执行 `git pull`、`git fetch` 和 `git push`。`.gitconfig` 只影响 Git；直接使用 `ssh`、`scp`、`sftp` 或 `rsync` 时，需要在命令中指定相同的密钥和 provider 参数。

## 8. 撤销密钥

设备丢失或怀疑泄漏时，立即从 GitHub、GitLab 或远程服务器的 `authorized_keys` 中删除公钥。

```shell
sc_auth list-ctk-identities # 查找目标身份的 Public Key Hash
```

```shell
sc_auth delete-ctk-identity -h "PUBLIC_KEY_HASH" # 删除指定 Secure Enclave 身份
```

此操作不可恢复，不要照抄占位符。

## 9. 排错

出现 `Permission denied (publickey)` 时，检查：

1. 服务端是否添加了正确的 `my-ssh-se.pub`。
2. `git config --show-origin --get core.sshCommand` 是否显示预期配置。
3. `-i` 指定的引用文件和 `SecurityKeyProvider` 是否正确。

```shell
ssh -Tv -i ~/.ssh/my-ssh-se -o IdentitiesOnly=yes -o SecurityKeyProvider=/usr/lib/ssh-keychain.dylib git@github.com # 输出详细认证过程
```

调试输出可能包含用户名、主机名和文件路径，公开前先检查。

## 10. Exportable keys 警告

- `p-256-ne` 和 `p-384-ne` 中的 `ne` 表示不可导出，本指南使用 `p-256-ne`。
- `p-256`、`p-384` 和 `p-521` 是可导出密钥，可以导出为 PKCS#12 文件并复制到其他设备。
- 导出的私钥可能因文件、备份或密码泄漏而被盗。除非明确需要迁移或备份，否则不要使用可导出密钥。

仅在确实需要迁移或备份时使用：

```shell
sc_auth create-ctk-identity -l "my-ssh-exportable" -k p-256 -t bio # 创建可导出身份
sc_auth list-ctk-identities # 获取 my-ssh-exportable 的 Public Key Hash
sc_auth export-ctk-identity -h "PUBLIC_KEY_HASH" -f my-ssh-exportable # 导出并按提示设置密码
```

```shell
sc_auth import-ctk-identities -f my-ssh-exportable.p12 -t bio # 之后可在另一台设备重新导入
```

把 `PUBLIC_KEY_HASH` 替换为实际值。使用高强度独立密码，不要通过 `-p` 把密码写进命令。安全保存 `my-ssh-exportable.p12`；该文件可以恢复私钥，泄漏后必须视为密钥泄漏。

## 参考资料

- [OpenSSH 客户端配置](https://man.openbsd.org/ssh_config)
- [GitHub 添加 SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- [macOS Secure Enclave SSH provider 技术记录](https://gist.github.com/arianvp/5f59f1783e3eaf1a2d4cd8e952bb4acf)
