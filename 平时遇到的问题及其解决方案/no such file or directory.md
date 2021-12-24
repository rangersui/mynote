通过ls命令确认文件确实存在，并且执行命令没有打错，但运行后提示：

-bash: ./xxx: No such file or directory

这是因为64位Ubuntu缺少32位运行库导致的错误。

安装32位运行库：

```shell
apt-get install lib32ncurses5
apt-get install lib32z1
```

