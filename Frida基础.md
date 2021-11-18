## Frida基础

1. Frida的安装

   1. 前置依赖安装

      1. python - last 3.x +
      2. window, macOS, GNU/Linux

   2. 安装命令

      ```bash
      pip install frida-tools
      ```

      [注]:  pip search 官方已经禁用了, 查找安装包，需要去[pypi.org]([PyPI · The Python Package Index](https://pypi.org/))

   3. 测试安装成功

      ``` bash
      frida --version
      ```

   4.  逆向安装还需要下载 frida-server

      在github的frida下载页([Releases · frida/frida (github.com)](https://github.com/frida/frida/releases)) 中，找到对应版本的frida-server-versionX-android-arm/arm64.xz比如(frida-server-15.1.10-android-arm64.xz)),解压下来以备使用

2. Frida连接到设备，并通过命令交互

   1. 将上面下载的frida-server-xxx.server重命名为frida-server

      ```bash	
      adb push frida-server /data/lcoal/tmp/frida-server
      ```

   2. 启动Android端的frida服务

      ```bash
      adb shell "/data/local/tmp/frida-server &"
      ```

   3. adb端口转发

      ```bash
      adb forward tcp:27042 tcp:27042
      adb forward tcp:27043 tcp:27043
      ```

      如果要取消转发，使用如下命令

      ```bash	
      adb forward --remove tcp:27042
      adb forward --remove tcp:27043
      ```

   4. 连接设备

      ```bash
      //连接到本地usb
      frida-ps -U
      //连接到远程
      frida-ps -R
      ```

      

      

      

      

