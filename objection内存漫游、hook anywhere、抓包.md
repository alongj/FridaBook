# objection:内存漫游、hook anywhere、抓包

1.  objection 安装

```bash
pip install objection
```

2. 启动app

```bash
objection -g 包名 explore
```

		1.  查看当前程序调用的所有so

```
memory list modules
```

2. 查看到其所有so 并保存位json

```bash	
memory list modules --json test.json
```

3.  内存查找

```bash
memory search
```

4. 打印所有 activities 

```bash
android hooking list activies
```

5. 强行启动指定activiy

```bash
android intent launch_activity com.android.settings.DisplaySettings
```

6. 打印和筛选类

```bash
android hooking list classes 打印所有 classes
android hooking search classes *** 筛选指定类
```

7. 打印方法和hook

```bash
android hooking list class_methods sun.util.locale.LanguageTag 打印方法体
android hooking watch class_method com.xxx.xxx --dump-args --dump-backtrace --dump-return 快速hook（来看是否代码走到这里）
```

eg: 抖音hook实例

```bash
android hooking list class_method com.ss.android.ugc.aweme.im.sdk.share.helper.l
android hooking watch class_method com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ --dump-args --dump-return
```

回显

```bash
(agent) Attempting to watch class com.ss.android.ugc.aweme.im.sdk.share.helper.l and method LIZ.
(agent) Hooking com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(java.util.List, java.lang.String, java.util.List, boolean, com.ss.android.ugc.aweme.base.f)
(agent) Hooking com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(java.util.List)
(agent) Hooking com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(android.content.Context, com.ss.android.ugc.aweme.im.service.model.IMContact, com.ss.android.ugc.aweme.sharer.ui.SharePackage, com.ss.android.ugc.aweme.im.sdk.chat.model.BaseContent, com.ss.android.ugc.aweme.base.f)
...
(agent) Registering job 791094. Type: watch-method for: com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ
```

方法调用后

```bash
(agent) [791094] Called com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(java.util.List, java.lang.String, com.ss.android.ugc.aweme.sharer.ui.SharePackage, com.ss.android.ugc.aweme.im.sdk.chat.model.BaseContent, java.lang.String)
(agent) [791094] Arguments com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ([object Object], (none), com.ss.android.ugc.aweme.sharer.ui.SharePackage@31c2, TextContent{createdAt=1637239163837type=700, prevId=7031888647930660383, rootId=7031888647930660383, isCard=false, extContent=null, msgHint='}, 88fa8181-3745-4c55-8a30-8aabf38ef90b)
(agent) [791094] Called com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(java.util.List, java.lang.String, com.ss.android.ugc.aweme.sharer.ui.SharePackage, com.ss.android.ugc.aweme.im.sdk.chat.model.BaseContent, java.lang.String, java.lang.Runnable)
(agent) [791094] Arguments com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ([object Object], (none), com.ss.android.ugc.aweme.sharer.ui.SharePackage@31c2, TextContent{createdAt=1637239163837type=700, prevId=7031888647930660383, rootId=7031888647930660383, isCard=false, extContent=null, msgHint='}, 88fa8181-3745-4c55-8a30-8aabf38ef90b, (none))
(agent) [791094] Called com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(java.util.List, java.lang.String, com.ss.android.ugc.aweme.im.sdk.chat.model.BaseContent, com.ss.android.ugc.aweme.im.sdk.model.c, com.ss.android.ugc.aweme.sharer.ui.SharePackage, java.util.List, boolean, java.lang.Runnable)
(agent) [791094] Arguments com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ([object Object], (none), TextContent{createdAt=0type=700, prevId=7031888647930660383, rootId=7031888647930660383, isCard=false, extContent=null, msgHint='}, com.ss.android.ugc.aweme.im.sdk.model.c@d4a7bd3, com.ss.android.ugc.aweme.sharer.ui.SharePackage@31c2, [object Object], (none), (none))
(agent) [791094] Called com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(java.util.List, java.lang.String, com.ss.android.ugc.aweme.im.sdk.chat.model.BaseContent, com.ss.android.ugc.aweme.im.sdk.model.c, java.lang.Runnable, java.util.Map)
(agent) [791094] Arguments com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ([object Object], (none), TextContent{createdAt=0type=700, prevId=7031888647930660383, rootId=7031888647930660383, isCard=false, extContent=null, msgHint='}, com.ss.android.ugc.aweme.im.sdk.model.c@d4a7bd3, (none), (none))
(agent) [791094] Return Value: (none)
(agent) [791094] Called com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(com.ss.android.ugc.aweme.sharer.ui.SharePackage, java.util.List, boolean)
(agent) [791094] Arguments com.ss.android.ugc.aweme.im.sdk.share.helper.l.LIZ(com.ss.android.ugc.aweme.sharer.ui.SharePackage@31c2, [object Object], (none))
(agent) [791094] Return Value: (none)
(agent) [791094] Return Value: (none)
(agent) [791094] Return Value: true
(agent) [791094] Return Value: true
```

* 附加模式hook

  ```bash
  objection -g packageName explore --startup-command "android hooking watch com.xxx.xxx"
  ```

* 设置返回值

  ```bash
  android hooking set return_value com.xxx.xxx.methodName false
  ```

  

8. 提取内存信息

   1. 查看库的导出函数

      ```bash
      memory list exports libart.so --json /root/libart.json
      ```

   2. 提取内存

      ```bash	
      memory dump all from_base
      ```

   3. 搜索整个内存

      ```bash
      memory search --string --offsets-only
      ```

   4. 查找类实例

      ```bash
      android hooking list activities
      android heap search instances com.ss.android.ugc.aweme.main.MainActivity
      ```

      ```bash
      Class instance enumeration complete for com.ss.android.ugc.aweme.main.MainActivity
       Hashcode  Class                                       toString()
      ---------  ------------------------------------------  --------------------------------------------------
      194448101  com.ss.android.ugc.aweme.main.MainActivity  com.ss.android.ugc.aweme.main.MainActivity@b970ae5
      ```

   5. 调用实例方法

      ```bash
      android heap execute 194448101 finish
      ```

      ```bash
      Handle 194448101 is to class
              com.ss.android.ugc.aweme.main.MainActivity
      Executing method: finish()
      ```

      编辑调用方法时的js

      ```bash
      android heap evaluate 120061934
      ```

      进入编辑模式，输入完成 ESC+Enter

      ```js
      (The hashcode at `120061934` will be available as the `clazz` variable.)
      console.log("evaluate result" + clazz.getBasePackageName());
      JavaScript capture complete. Evaluating...
      Handle 120061934 is to class
              com.ss.android.ugc.aweme.main.MainActivity
      evaluate resultcom.ss.android.ugc.aweme
      ```

9. 启动Activity和Service

   ```bash
   直接启动activity
   android intent launch_activity 强制启动界面，可配合android hooking list activities 获取界面
   
   直接启动service
   android hooking list services 查看当前开启的服务
   android intent launch_service com.服务  开启服务
   ```

10. Java内存查找

    ```bash
    列出内存所有的类
    android hooking list classes (打印所有 classes)
    搜索类
    android hooking search classes display
    搜索所有的方法
    android hooking search methods display
    列出类的所有方法
    android hooking list class_methods com.xxx.xxx
    ```

11. 查看当前hook工作列表

    ```bash
    jobs list
    ```

12. 屏蔽SSL 校验, 使得ssl pinning失效

    ```bash
    android sslpinning disable
    ```

    















