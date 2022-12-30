# git ssh配置

### 一、单用户常规配置：
#### 1. 生成ssh钥匙对:
```shell
$ ssh-keygen -t ed25519 -C "your_email@example.com"
```
不支持Ed25519的旧机器:
```shell
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

#### 2. 添加私钥证书到ssh-agent:
- 方式一，手动添加到ssh-agent和钥匙串
```shell
$ ssh-add --apple-use-keychain ~/.ssh/id_ed25519 
$ #--apple-use-keychain等同于废弃的-K
```
- 方式二，自动添加到ssh-agent和钥匙串（macOS Sierra 10.12.2 or later）
 配置～/.ssh/config
```shell
Host github.com             #host别名，使用ssh时按此匹配配置
    HostName github.com     #真实主机地址（域名｜IP）
    AddKeysToAgent yes      #使用ssh时自动添加私钥到ssh-agent（不启用的话需要按方式一手动添加）
    UseKeychain yes         #使用钥匙串填充密码（等同于--apple-use-keychain）
    PreferredAuthentications publickey  #使用公钥认证而不是账号密码
    IdentityFile ~/.ssh/id_ed25519  #证书文件（私钥）
```
测试是否成功连接：
```shell
$ ssh -T git@github.com
```

#### 3. 添加公钥到github等:
```shell
$ #拷贝公钥
$ cat ~/.ssh/id_ed25519.pub |pbcopy
```

#### 4. 修改本地仓库配置
```shell
$ git config user.email "xxx@xxx.com"
$ git config user.name "xxname"
$ #或直接在仓库配置文件中修改，.git/config
```

---
### 二、多账户配置

- **情形一**，为不同平台的账号添加不同证书和配置(执行步骤1，2);
- **情形二**，为同一个平台的不同账号配置不同的证书，
其关键在于：利用Host别名为同一个服务器地址（域名）添加不同的ssh证书和配置;
相应就需要修改本地仓库的远程地址中的域名为不同的Host别名，以匹配不同的ssh证书和配置;(执行步骤1，2，3);
1. 分别生成ssh钥匙对，并正确添加好公钥私钥；
2. 添加～/.ssh/config配置:
```
# github账号
Host github.com
  HostName github.com
  User git
  AddKeysToAgent yes
  UseKeychain yes
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_ed25519

# 其他github账号
Host xxx.github.com
  HostName github.com
  User git
  AddKeysToAgent yes
  UseKeychain yes
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_ed25519_xxx

# gitee账号
Host gitee.com
  HostName gitee.com
  User git
  AddKeysToAgent yes
  UseKeychain yes
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_ed25519_gitee
```
确认各个配置是否能成功连接：
![](https://raw.githubusercontent.com/fegnze/pic/main/20220427164128.png)

3. 修改本地仓空中远程URL的域名（☀️☀️☀️XXX.github.com☀️☀️☀️需同～/.ssh/config中的对应Host匹配）
```shell
$ git remote rm origin
$ git remote -v
$ git remote add origin git@XXX.github.com:xxx/xxx.git
```
---
#### 如完成上述过程,重启后失效,是因为没要触发ssh-add操作,
可先执行:
```shell 
$ ssh -T git@github.com # 触发(AddKeysToAgent)
``` 

或在.bash_profile中添加:
```shell
$ ssh-add ~/.ssh/id_ed25519 > /dev/null 2>&1
$ ssh-add ~/.ssh/id_ed25519_xxx > /dev/null 2>&1
$ ssh-add ~/.ssh/id_ed25519_yyy > /dev/null 2>&1
```
---
#### 如出现💣ERROR: Permission to XXX.git denied to user XXX，说明相同平台的不同账户中是否存在相同证书,平台无法自行做出选择;此时,需删除重复的证书,重新按上述步骤配置多套证书;