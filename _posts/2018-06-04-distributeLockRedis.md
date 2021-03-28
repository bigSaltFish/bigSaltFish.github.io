---
layout: default
title: 分布式锁之redis实现
---
  &ensp;&ensp;对“锁”大家肯定都不陌生，锁是针对多线程情况下对资源访问的控制，初学java时候，就知道synchronize和lock，synchronize是重量级锁，lock是轻量级，锁巴拉巴拉。。。，但是都是针对单个jvm来说的，现在稍微大点的网站都是多台服务器，通过nginx等负载均衡到多个服务器节点，这样的话，就得使用分布式锁了，常见的分布式锁有基于redis或者zookeeper，这里讨论下基于redis实现分布式锁及其会遇到的问题。

# 1.0 setnx
  &ensp;&ensp;根据redis提供的setnx命令，可以当做互斥锁来使用，成功了会返回1，失败返回0，然后做一些操作，最后删除key，伪代码如下：

	if(1 == setnx(key,value)){  
	    dosomething;  
	    del(key);  
	}

  &ensp;&ensp;考虑的更完善点呢，可以加入“锁超时”的情况，假如一个获得锁权限的节点，因为一些其他因素导致迟迟没有执行del(key)的操作，导致锁无法释放，会阻塞依赖该key的业务流程。伪代码如下：
    
	if(1 == setnx(key,value)){  
	    expire(key,timeOutSecond)  
	    dosomething;  
	    del(key);  
	}

&ensp;&ensp;加入了锁超时的代码，看似是“很完善”了，但是，还是有一些问题的，setnx和expire不是原子性、del会误删。

# 2.0 原子性和误删操作
## 2.1 操作的原子性
   &ensp;&ensp;先说下原子性，在1.0中的代码，setnx和expire不是原子性，极端情况，第一行代码，setnx(key,value)执行完之后，再执行第二行expire的时候，节点GG思密达了，第二行代码指令就没有抵达redis，那这样的话，之前设置的key就永恒不朽了。<br>
&ensp;&ensp;如何处理这个问题呢？redis 2.6.12及其以上版本，提供了原子性操作的api：

`set(key,value,ex second,nx)`

set四个参数的这种等于 setnx+expire两行代码，还保证了原子性<br>
   &ensp;&ensp;这里吐槽一下啊，redis不仅提供了setnx、还提供了setex，还还提供了psetex，一个set(key,value,ex|px second,nx)命令不就搞定了吗，弄这么多干嘛呢？

## 2.2 误删key
  &ensp;&ensp;再说下误删操作，还是1.0的伪代码，假如线程A获得了锁，超时时间是30秒，假如线程A在30秒内未执行完成，则锁超时，锁会被其他线程获取；假如获取到锁的是线程B，线程B在执行代码块的过程中，线程A终于执行完了并删除了锁，那这时就是误删了，这个时间线程A删的锁其实是线程B的锁了。<br>
  &ensp;&ensp;如何处理这种误删操作呢？可以把当前线程的线程ID作为value，在删除锁之前比对下key的value是不是当前线程的线程ID，如果是，才会去删除锁。伪代码如下：
```
String currendThreadId = Thread.currendThread.getId();  
if("ok".equals(set(key,currendThreadId,ex second,nx))){  
  doSomething;  
    
  if(currendThreadId.equals(get(key))){  
      del(key)  
   }  
}  
```
  &ensp;&ensp;上面的代码，其实还是隐含一个问题--判断锁的值和删除锁，不是原子性，要达到效果，redis的现成api是没有的，得需要使用redis脚本了
```
String script_lua = "if (redis.call('get',KEYS[1]) == redis.call('get',ARGV[1])) then return redis.call('del',KEYS[1]) else return 0 end";  
redisClient.eval(script_lua,Collections.singletoList(key),Collections.singletoList(currendThreadId));  
```
这行代码的效果等同于在redis-cli中执行
![客户端执行效果图片]({{site.url}}/assets/2018-06-04-distributeLockRedis/redis_cli.jpg)

## 2.3 存在并发可能性
  &ensp;&ensp;虽然使用脚本的方式可以有效防止误删的操作，但是还是会有并发问题--线程A和线程B同时都在操作，这显然不合理，为解决这个问题，那只有限制B不能进入了。<br>
  &ensp;&ensp;这里就需要引入一个“守护线程”的东西了，守护线程会在线程A即将过期的时候，去主动给线程A“续命”，当线程A执行完毕，再关掉守护线程；即使线程A所在的节点GG思密达了，因为线程A和守护线程是在一个节点的，守护线程也会GG的，锁到期后，会自动释放。

  &ensp;&ensp;到这里才算是把redis的分布式锁比较全面的说完。