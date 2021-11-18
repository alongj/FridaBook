# Frida常用命令

1. 连接设备

```bash
# Connect Frida to an iPad over USB and list running processes
$ frida-ps -U

# List running applications
$ frida-ps -Ua

# List installed applications
$ frida-ps -Uai

# Connect Frida to the specific device
$ frida-ps -D 0216027d1d6d3a03
```

2. 获取attach的设备信息

```bash
frida-ls-devices
```

3. trace 目标进程的函数

```bash
frida-trace -U -j "*!*method*/isu" processName
```

4. 启动指定进程

```bash
frida -U -f com.ss.android.ugc.aweme --no-pause
```



