# JDK安装
某JDK装不上去装的OpenJDK，要设置JAVA_HOME。

JAVA地址在/usr/lib/jvm/下如果是1.8版本就是/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el7_9.x86_64

编辑一个jdk.sh的文件存放在/etc/profile.d/jdk.sh

```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el7_9.x86_64
export PATH=$JAVA_HOME/bin:$PATH
```

最后执行```source /etc/profile.d/jdk.sh```即可

# Jenkins安装
到官网https://pkg.jenkins.io/redhat-stable/ 按官网指示，但是需要注意的是
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# 应该要加一个参数，否则卡进度条
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo --no-check-certificate
```