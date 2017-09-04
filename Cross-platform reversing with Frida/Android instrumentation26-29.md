##安卓工具类方法

![Alt text](/pediySnowGroup/Android instrumentation.png)

##和java相辅相成

* java.available- 是现在运行时？
* java.perform(fn)与java虚拟机从给定的回调互动
* java.cast(ptr('0x1234)),java.use("android.os.Handler"))-互相作用于内存地址为0x1234的对象
* 构造器像new（）方法一样被暴露，并且可以用任何方法重新加载：Handler.$new.overload("android.os.Looper").call(Handler,looper)
* java.enumerateLoadedClasses()-全部加载的类
* 指定.implementation替代一个方法
* java.choose()-遍历堆查找java实例对象

##钩住java方法

![Alt text](/pediySnowGroup/HookingJavamethods.png)

##简单的工具类方法
1.pid = frida.spawn(["/bin/ls"])
2.session = frida.attach(pid)
3.script = session.create_script("your script")
4.<apply instrumentation>–推荐使用RPC的这个: script.exports.init()	
5.frida.resume(pid) - 应用程序的主线程将进入main()

针对手机app是定它的identifier：spawn([com.apple.AppStore]) 忘记identifier是什了？使用frida-ps-ai查看
