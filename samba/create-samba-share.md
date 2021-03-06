# Samba 文件共享

如果有人问：“哪种文件共享支持所有主流类型的操作系统？”，我会毫不犹豫的回答 —— Samba！

Samba 是微软 CIFS	协议的开源实现，几乎所有智能路由器上的 USB 共享功能都是通过 Samba 实现的。

## 安装 Samba

在《[通过 SSH 远程管理服务器](initialization/use-ssh.md)》中的 `用主机名访问 NAS 主机` 部分已经安装了 Samba 软件包，如果没有请安装：

```
getnas@getnas:~$ sudo apt install samba
```

## 创建共享

### 第一步 创建共享目录

我们前面配置的数据盘挂载在 `/mnt/storage` 目录，虽然可以直接共享这个目录，但还是建议创建专门的文件夹进行共享，我们在该目录中创建一个 `share` 文件夹：

```
getnas@getnas:~$ mkdir /mnt/storage/share
```

### 第二步 创建 samba 用户

直接使用 Debian 系统中的用户是无法访问 Samba 共享的，必须在 samba 中创建与系统用户同名的用户，使用 `smbpasswd` 命令附带 `-a` 参数添加新 samba 用户：

```
getnas@getnas:/mnt/storage$ sudo smbpasswd -a getnas
New SMB password:
Retype new SMB password:
Added user getnas.
```

上述命令在 samba 中创建了 `getnas` 用户，在执行命令的过程中，程序会要求输入并确认用户密码。

> 注意：如果系统中不存在 `getnas` 用户，在创建 samba 用户时会报错。

### 第三步 配置共享目录

Samba 的共享需要在 `/etc/samba/smb.conf` 配置文件中进行配置，使用 `nano` 编辑器打开：

```
getnas@getnas:/mnt/storage$ sudo nano /etc/samba/smb.conf
```

在配置文件的最后新增以下共享配置信息：

```
[Share]
  comment = Test samba share
  path = /mnt/storage/share
  guest ok = no
  read only = no
  browseable = yes
```

上述配置在我们指定的目录创建了名为 `Share` 的共享，该共享不允许匿名访问，但任何拥有 samba 账号的用户都可以登录访问。

编辑完成后使用组合键 `CTRL + o` 保存，然后用 `CTRL + x` 退出编辑器。

要让新的配置生效，需要重启 samba 服务：

```
getnas@getnas:/mnt/storage$ sudo systemctl restart smbd.service
```

## 多用户访问共享

### 第一步 新建普通用户

使用 `adduser` 命令在系统中创建名为 `tom` 的普通用户，创建过程中程序会交互式的提出一系列问题，除了 `密码` 外都是可选的，可以设置或直接按 `Enter` 回车键跳过：

```
getnas@getnas:/mnt/storage$ sudo adduser tom
正在添加用户"tom"...
正在添加新组"tom" (1001)...
正在添加新用户"tom" (1001) 到组"tom"...
创建主目录"/home/tom"...
正在从"/etc/skel"复制文件...
输入新的 UNIX 密码：
重新输入新的 UNIX 密码：
passwd：已成功更新密码
正在改变 tom 的用户信息
请输入新值，或直接敲回车键以使用默认值
	全名 []: Tom
	房间号码 []:
	工作电话 []:
	家庭电话 []:
	其它 []:
这些信息是否正确？ [Y/n]
```

### 第二步 创建相应的 samba 用户

前面已经介绍过，想要访问 samba 共享，必须创建与系统同名的 samba 用户：

```
getnas@getnas:/mnt/storage$ sudo smbpasswd -a tom
New SMB password:
Retype new SMB password:
Added user tom.
```

> 注意：samba 用户只需名字与系统用户相同，密码无需相同。访问 samba 共享验证身份时要输入 samba 账户的密码。

### 第三步 使用 tom 账户访问共享

我们会发现，使用 `tom` 账户可以正常访问 `Share` 共享，但由于系统中的 `tom` 用户对 `/mnt/storage/share` 目录没有写权限，因此只能查看目录中的文件。