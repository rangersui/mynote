# 安装docker

windows上下载docker for windows即可，linux上使用sudo apt install docker同理。

## 故障排除

windows版本的docker需要使用WSL2，但是根据提示下载安装后可能会报错`Failed to set version to docker-desktop: exit code: -1`，这一情况可能是由于代理软件和wsl2之间的端口有冲突造成的，该情况可以通过`netsh winsock reset`解决。

# 配置opengauss

下载enmotech/opengauss:1.1.0，不要下载最新版本，存在未知错误，使用1.1.0版本可以正常使用。

运行指令`docker run --name gauss --privileged=true -d -e GS_PASSWORD=password@123 -p 5432:5432 enmotech/opengauss:1.1.0`

使用数据库连接工具，默认账号为gaussdb