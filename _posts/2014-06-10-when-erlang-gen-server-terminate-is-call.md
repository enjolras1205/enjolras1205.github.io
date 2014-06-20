---
layout: post
title: "when erlang gen-server's terminate is call"
---

gen_server定义了6个callback接口：
{% highlight erlang linenos %}
init/1
handle_call/3
handle_cast/2
handle_info/2
terminate/2
code_change/3
{% endhighlight %}
对于callback的实现者来说，理解callback函数的触发点是最重要的，本文只讨论terminate的调用。
 
terminate的被调用有如下几种情况：

+ handle_call返回 {stop, Reason, Reply, StateN} -> terminate(Reason, StateN)
+ handle_info | handle_cast返回 {stop, Reason, StateN} -> terminate(Reason, StateN)
+ handle_call | handle_info | handle_cast 抛出异常(在gen_server.erl中被捕获为{'EXIT', What}) -> terminate(What, State)
+ handle_call | handle_info | handle_cast 返回值非法 -> terminate({bad_return_value, Reply}, State)
+ 收到来自Parent的{'EXIT', Parent, Reason}消息：  
设置了一个timeout或者无限等待的情况下，supervisor是通过exit(Pid, shutdown)来通知子进程退出的，所以，若supervisor下的gen_server worker进程没有设为系统进程，worker进程不会收到来自Parent的Exit消息，故terminate不会被调用。

结论：

terminate被调用的方式有如下几种：

+ 返回值触发
+ 代码错误抛出异常被捕获
+ 系统gen_server收到来自Parent的Exit消息

如果希望gen_server进程崩溃时terminate一定被调用到(exit(Pid, kill)除外)，设为system进程即可。

以上仅仅是结论，通过查看源码，以及写测试代码得出。这里有一些用得到的测试代码:

[测试代码](https://github.com/enjolras1205/erlang_learning/tree/master/erl_server/"测试代码")
