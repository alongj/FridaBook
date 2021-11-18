# Frida与Android

1. 创建并运行frida脚本，与Android上的frida-server通信
   1. 编写运行在frida-server上的js脚本jscode
   2. 获取usb设备，attach到android上的进程长
   3. 使用jscode创建脚本对象script，并设置js的消息回调
   4. script.load()

```python	
import frida, sys

def on_message(message, data):
    if message['type'] == 'send':
        print("[*] {0}".format(message['payload']))
    else:
        print(message)

jscode = """
Java.perform(function () {
  // Function to hook is defined here
  var MainActivity = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity');

  // Whenever button is clicked
  var onClick = MainActivity.onClick;
  onClick.implementation = function (v) {
    // Show a message to know that the function got called
    send('onClick');

    // Call the original onClick handler
    onClick.call(this, v);

    // Set our values after running the original onClick handler
    this.m.value = 0;
    this.n.value = 1;
    this.cnt.value = 999;

    // Log to the console that it's done, and we should have the flag!
    console.log('Done:' + JSON.stringify(this.cnt));
  };
});
"""

process = frida.get_usb_device().attach('com.example.seccon2015.rock_paper_scissors')
script = process.create_script(jscode)
script.on('message', on_message)
print('[*] Running CTF')
script.load()
sys.stdin.read()
```

2. Java bridge

   1. 创建java.lang.String对象

   ```js
   var JavaString = Java.use('java.lang.String');
   var exampleString1 = JavaString.$new('Hello World, this is an example string in Java.');
   ```

   2. 使用console.log打印对象

   ```js
   console.log('[+] exampleString1: ' + exampleString1);
   ```

   3. 调用Java对象的方法

   ```js
   console.log('[+] exampleString1.length(): ' + exampleString1.length());
   ```

   4. 创建CharSet的实例，初始化为defalut character set

   ```js
   // Create an instance of java.nio.charset.Charset, and initialize the default character set.
   const Charset = Java.use('java.nio.charset.Charset');
   var charset = Charset.defaultCharset();
   ```

   5. 使用JavaScript的string创建数组

   ```js
   // Create a byte array of a Javascript string
   const charArray = "This is a Javascript string converted to a byte array.".split('').map(function(c) {
     return c.charCodeAt(0);
   })
   ```

   6. 使用overload创建String对象

   ```js
   exampleString2 = JavaString.$new.overload('[B', 'java.nio.charset.Charset').call(JavaString, charArray, charset)
   ```

   7. 拦截StrtingBuilder的构造方法，并打印参数的一部分

   ```js
    const StringBuilder = Java.use('java.lang.StringBuilder');
   //We need to overwrite .$init() instead of .$new(), since .$new() = .alloc() + .init()
   StringBuilder.$init.overload('java.lang.String').implementation = function (arg) {
     var partial = "";
     var result = this.$init(arg);
     if (arg !== null) {
       partial = arg.toString().replace('\n', '').slice(0,10);
     }
     // console.log('new StringBuilder(java.lang.String); => ' + result)
     console.log('new StringBuilder("' + partial + '");')
     return result;
   }
   ```

   8. 拦截StringBuilder的toString()方法

   ```js
    StringBuilder.toString.implementation = function () {
      var result = this.toString();
      var partial = "";
      if (result !== null) {
        partial = result.toString().replace('\n', '').slice(0,10);
      }
      console.log('StringBuilder.toString(); => ' + partial)
      return result;
    }
   ```

   9. 给Java对象的属性赋值

   ```js
   const Person = Java.use('com.xxx.Person')
   var p = Person.$init();
   p.name.value = 'xiaohei';
   ```

   如果属性和方法重名，使用this._m.value=0，给属性赋值

   10. 打印调用堆栈

   ```js
   function printStack(){
   	console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));
   }
   ```

   11. 打印BYTE数组

   ```js
   function printByteArray(byteArray){
       var content = '';
       for(var i = 0;i < byteArray.length; i++){
           if(byteArray[i] > 32 && byteArray[i] < 126){
               content += String.fromCharCode(byteArray[i])
           }
       }
       console.log(content);
   }
   ```

   12. gson

   ```js
   /**
    * 通过肉丝姐编译的gson转换Object成json对象,需要用时需要把函数体内的代码迁移到相应位置即可
    */
   function objectToGson(obj){
       Java.openClassFile('/data/local/tmp/r0gson.dex').load();
       const gson = Java.use('com.r0ysue.gson.Gson');
       var result = gson.$new().toJson(obj);
       return result;
   }
   ```

   13.  Java.array

   ```js
   // 构建一个byte[]
   var byte = Java.array('byte', [1, 2, 3, 9]);
   ```

   14. 查看hook的类名

   ```js
   xxx.$init.implementation = function(){
       var result = this.$init();
       console.log(result.$ClassName);
       return result;
   }
   ```

   15. 强转 Java.cast

   ```js
   var textContext = Java.cast(baseContent,classTextContent);
   ```

   16. 自定义接口实现, 使用Java.registerClass

   ```js
   /**
    * 通过现有接口实现一个类
    */
   function implementClass(){
       var beer = Java.registerClass({
           // 我们要实现的类将要在位置
           name: 'com.r0ysue.a0526printout.Beer',
           implements: [Java.use('com.r0ysue.a0526printout.liquid')],
           methods: {
               // 要实现的接口的函数,注意，接口的话一定要实现全部哦
               flow() {
                   console.log('hello beer!');
               return Java.use('java.lang.String').$new("beer");
             },
           }
         });
         console.log(beer.$new().flow());
   }
   ```

   17. 集合的迭代

   ```js
   /**
    * 打印map,方法1：通过iterator()来打印
    * @param {} obj 
    */
   function printMap1(){
       Java.choose('java.util.HashMap', {
           onMatch: function(instance){
               var keySet = instance.keySet();
               var iterator = keySet.iterator();
   			var result = "";
               while(iterator.hasNext()){
   				var keystr = iterator.next().toString();
   				var valuestr = instance.get(keystr).toString();
   				var map = keystr + ':' + valuestr + '||';
   				result += map;
               }
                   console.log('HashMap instance is:', instance, ', it args:', result);
           },onComplete:function(){console.log('search complete!');}
       })
   }
   
   /**
    * 打印Map，通过toString
    */
   function printMap2(){
       Java.choose('java.util.HashMap', {
           onMatch: function(instance){
               console.log('find the Map instance', instance);
               console.log('it args:', instance.toString());
           }
       })
   }
   ```

   18. non-ascii 方法名hook,先打印原始的编码，然后再通过这个编码去hook(*)

   ```js
   function x() {
     var targetClass = "com.example.hooktest.MainActivity";
   
     var hookCls = Java.use(targetClass);
     var methods = hookCls.class.getDeclaredMethods();
   
     for (var i in methods) {
       console.log(methods[i].toString());
       console.log(encodeURIComponent(methods[i].toString().replace(/^.*?\.([^\s\.\(\)]+)\(.*?$/, "$1")));
     }
   
     hookCls[decodeURIComponent("%D6%8F")]
       .implementation = function (x) {
       console.log("original call: fun(" + x + ")");
       var result = this[decodeURIComponent("%D6%8F")](900);
       return result;
     }
   }
   ```

   19. 枚举所有的类并打印

   ```js
   Java.enumerateLoadedClasses({
       onMatch:function(name, handle){
           console.log('name:', name);
       },onComplete(){}
   })
   ```

   查找接口的实现类

   ```js
   Java.enumerateLoadedClasses({
       onMatch: function(className){
           if (className.indexOf(包名) < 0){
               return;
           }
           var hookCls = Java.use(className);
           // getInterfaces()可以把当前加载的类，类所实现的interface全都打印出来。
           var interFaces = hookCls.class.getInterfaces();
           if (interFaces.length > 0){
               console.log(className);
               for (var i in interFaces){
                   console.log('\t', interFaces[i].toString())
               }
           }
       }
   })
   ```

   Frida遍历查找API

   ```js
   const groups = Java.enumerateMethods("*youtube*!on*")
   console.log(JSON.stringify(groups, null, 2));
   ```





