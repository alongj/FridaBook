# Frida RPC

1. send——js向python发送数据send()

```js
Java.perform(function(){
    var ai = Java.use('com.tencent.mm.ui.chatting.c.ai');

    ai.eS.implementation = function(str, i) {
        send('str=' + str + ', i=' + i);
        var b = this.eS(str, i);
        return b;
    };
});
```

2. recv(function(){}).wait() 接口python传过来的数据

```js
Java.use(...).methodname.overload(...).impelementation = function(...) {
  send("");
  recv(function(received_json_objection) {
  	string_to_recv = received_json_objection.my_data;
    console.log("string_to_recv:" + string_to_recv);
  }).wait();
}
```

3. python调用Js的方法

   1. 提供Js端可被远程调用的函数

   ```js
   rpc.exports =  {
       sendmsg:function(str) {
           send('hello,' + str);
           Java.perform(function() {
               const IMConversation = Java.use('com.ss.android.ugc.aweme.im.service.model.IMConversation');
               var mIMConversation = IMConversation.$new();
               mIMConversation.mConversationType.value = 2;
               mIMConversation.mConversationId.value = "7029144708374970910";
               console.log('mIMConversation=' + mIMConversation.toString());
           })
       }
   }
   ```

   2.  python端调用这个rpc方法，script.exorts.methodxxx()

   ```python
   javaScript = get_javascript(jsFile)
   script = session.create_script(javaScript)
   
   script.on("message", on_message)
   print('[*] Running CTF')
   script.load()
   //调用script.export#method()
   script.exports.sendmsg('world')
   ```

   

