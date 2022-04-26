# git ssh配置

### 1. 生成ssh钥匙对:
```shell
$ ssh-keygen -t ed25519 -C "your_email@example.com"
```
不支持Ed25519的旧机器:
```shell
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### 2. 添加私钥到ssh-agent:
- 方式一，手动添加到ssh-agent和钥匙串
```shell
$ ssh-add --apple-use-keychain ~/.ssh/id_ed25519 
$ #--apple-use-keychain等同于废弃的-K
```
- 方式二，自动添加到ssh-agent和钥匙串（macOS Sierra 10.12.2 or later）
 配置～/.ssh/config
    > Host *
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519

    可同时配置多个，其他配置项https://zhuanlan.zhihu.com/p/35922004

### 3. 添加公钥到github等:
```shell
# 拷贝公钥
cat ~/.ssh/id_ed25519.pub |pbcopy
```

---
### 关于多账户多ssh配置
- 分别生成ssh钥匙对，并正确添加好公钥私钥；
- ☀️☀️☀️***同一组ssh钥匙对，不能用于多个git账号***(同一个共钥不能出现在多个git账户中);
- 如出现💣ERROR: Permission to XXX.git denied to user XXX，检查错误中user所属的账户中是否存在相同的公钥，删除，重新生成配置新的钥匙对；
- 理论上正确配置过ssh后，不需要再设置用户名和邮箱，如需修改，只修改仓库配置即可:
```shell
$ git config --local user.name "xxx" 
$ git config --local user.email "xxx" 
# 或直接在仓库配置文件中修改，.git/config
```