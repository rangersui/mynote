# 问题描述

安装samba和ssh，方便虚拟机传文件。

# 解决方案

```bash
sudo apt install ssh samba
```

添加samba用户

```bash
sudo smbpasswd -a username # 将用户添加到samba, 可以是现有用户
# 如果是新用户则需要用useradd和passwd先添加系统用户并设密码
sudo smbpasswd -e username # enable user
```

配置/etc/samba/smb.conf: 

```conf
server role = standalone server
...
passdb backend = tdbsam
...
[username]
comment = 说明
path = /home/mx #path
browsable = yes
writeable = yes
read only = no
```

安装完毕后运行`ifconfig`获取网络地址

## 在windows访问

打开文件管理器，在地址栏输入：`\\虚拟机的网络地址` 即可访问

# 相关问题

安装后可以ping到ubuntu的IP, 但是出现不能访问samba共享的目录:"The specified network name is no longer available"

原因：部分samba组件安装不完整,执行以下命令重新安装组件即可

```bash
sudo apt-get install --reinstall libsmbclient libsmbclient-dev libtevent0 libtalloc2 # 重装组件
sudo service smbd restart  # 重启服务
```