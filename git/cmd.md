# 生成新SSH key

1. 在Git Bash中输入
```
$ ssh-keygen -t ed25519 -C "best5721@sina.com"
```

2. 可以按默认提示一直回车, 成功后会在`C:\Users\you\.ssh`这个目录下面会生成2个文件

```
id_ed25519
id_ed25519.pub
```

# 把生成的公钥添加到github账号

1. 登录**gibhub**, **"Settings"** -> **"SSH and GPG keys"** -> **"New SSH key"**
2. 输入**Title**, 可以任意输入
3. 在**Key**中复制粘贴`id_ed25519.pub`文件内容


# 校验SSH Key

在Git Bash中输入

```
ssh -T git@github.com
```

如果显示如下信息，则校验通过

```
Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.
```

# 多个github账号管理

修改`C:\Users\you\.ssh\config`文件, 添加如下示例配置

```
Host            github.com
HostName        github.com
User            git
IdentityFile    C:\Users\you\.ssh\id_ed25519
```

参考上面**校验SSH Key**

