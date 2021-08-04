# 生成新SSH key

1. 在Git Bash中输入
```
$ ssh-keygen -t ed25519 -C "xxx@xxx.com"
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
Host            github
HostName        github.com
User            git
IdentityFile    C:\Users\you\.ssh\id_ed25519
```

参考上面**校验SSH Key**

# git配置文件说明

- 全局配置文件路径: `~/.gitconfig`, 通过`git config --global`设置的内容都在这个文件
- git服务SSH Key配置: `~/config`
- 如果全局配置已经指定了一个用户名和邮箱，但又需要对别的工程在`git commit`的时候显示不同的用户名和邮箱, 可以在`工程目录/.git/config`中添加如下内容

```
[user]
	email = xxx@xxx.com
	name = xxx
```
