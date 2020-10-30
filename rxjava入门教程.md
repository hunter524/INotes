# 响应式（rxjava)编程入门

  响应式编程作为一种编程范式在开源社区具有一种怎样的实践?响应式编程给我们带来了什么?更简洁的代码逻辑?更好的性能?更少的代码实现各种复杂的异步操作?本次我们基于 ReactiveX 系列开源项目中 java 语言实现 rxjava 来入门响应式编程.该系列的开源项目还有其他语言的实现如:RxJS,RxSwift,RxCpp,RxLua 等等,它们秉承着相同的思想在不同的语言平台上加以实践.

## 响应式编程简介

------

### 什么是响应式编程

- 响应式编程
  
  执行者作为观察者,等待指定事件,当指定事件触发才去执行动作.响应式编程调用方法不是直接去执行操作,而只是去注册事件监听器.因此响应式编程具有等待稍后执行的语义,执行行为只是对事件的一种响应,故此称为响应式编程.因此响应式编程其实是一种异步编程,当然也可以做到同步模式,但是这就失去了其真正的意义.也就是说响应式编程实质上基于事件驱动的异步编程模式.

  根本上响应式编程也是非阻塞式编成,可以提升程序的整体性能.

- 命令式编程
  
  调用方法即直接执行操作.(类似要x人立即去做xxx事情,故此叫命令式)

- reactivex 项目的简述
  
  Reactive Extensions for Async Programming (异步编程的响应式扩展)

- rxjava 项目的简述
  
  RxJava – Reactive Extensions for the JVM – a library for composing asynchronous and event-based programs using observable sequences for the Java VM.(一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库)

那么现有的异步编程模式出现了什么问题？不好用？不好读？容易出错?对初学者不友好?

### RxJava 好在哪儿

换句话说，『同样是做异步，为什么人们用它，而不用现成的 ScheduledExecutorService,ExecutorService,ForkJoinPool,Callable,FutureTask,CountDownLatch,CyclicBarrier,Semaphore,LinkedBlockingQueue,SynchronousQueue,PriorityBlockingQueue 等』

- 简洁
  
  异步操作很关键的一点是程序的简洁性，因为在调度过程比较复杂的情况下，异步代码经常会既难写也难被读懂。但是 rxjava 随着程序逻辑变得越来越复杂，它依然能够保持简洁。该处需要强调的是逻辑复杂度变得简洁而不是代码量减少.(逻辑变得简洁才能提升代码的读写速度),使用rxjava 编写的异步程序可以一会儿排成人字一会儿排成一字.

### 响应式编程的基础

- 观察者模式,
  
  Callback. 对于观察者模式则存在 Observable (被观察者) 和 Observer(观察者) 或者可以称之为 Subscriber (订阅者).对于回调我们需要有注册中心/事件中心/事件分发器也就是被观察者, CallBack 则是接受被观察者的调用.(一旦谈到回调,就无法回避 Callback hell 这个话题)

- 代理模式/装饰者模式/责任链模式

  Observable 构建成的转换链实际上像是一种责任链.很多操作符的实现对 Observable 和 Observer 的包装/增强/限制 则更像是装饰者模式和代理模式.

### 函数式编程

函数式编程基础概念<https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/>

rxjava 很多概念和API 的设计上参考了函数式编程的基础概念,如 Maybe,map等操作符号传入的函数接口则对应于高阶函数的概念,Observable的转换链的建立则和柯里化的概念很像.

简单提一下柯里化(curry):

curry 的概念很简单：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12

// lodash 组建提供的柯里化操作

var curry = require('lodash').curry;

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});

match(/\s+/g, "hello world");
// [ ' ' ]

match(/\s+/g)("hello world");
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces("hello world");
// [ ' ' ]

hasSpaces("spaceless");
// null

filter(hasSpaces, ["tori_spelling", "tori amos"]);
// ["tori amos"]

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(["tori_spelling", "tori amos"]);
// ["tori amos"]

var noVowels = replace(/[aeiou]/ig);
// function(replacement, x) { return x.replace(/[aeiou]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiou]/ig, "*") }

censored("Chocolate Rain");
// 'Ch*c*l*t* R**n'

```

在 js 中函数可能算是一等公民,kotlin 中也可以便捷写出柯里化模式的方法.java 对象是一等公民的语言实现柯里化则比较费劲.但是java其实也在积极的引入函数式编程. java 中的 lambda,函数式接口,方法引用,stream流,引入invokedynamic 指令.

下面的 rxjava 代码均以 rxjava2 版本作为基础.虽然 rxjava2 将于 2021.02 停止维护.目前最新版本 rxjava3.[rxjava1 rxjava2 向 rxjava3 的迁移手册]<https://github.com/ReactiveX/RxJava/wiki/What's-different-in-3.0> rxjava1 rxjava2 rxjava3 在同一个项目中是可以并存的.rxjava1 的实现由于历史原因实现的并不好,但是 rxjava1 制定出的操作附语义,基础高性能非阻塞算法并没有太多改变.rxjava2 的实现由于响应式编程规范接口的诞生,同时根据rxjava1在实践中产生的问题,修改了很多操作符实现的架构模式,并且修复了 rxjava1 由于架构问题而没有办法修复的问题.本质上 rxjava2 对于 rxjava1 则是一次比较大的改动,改动主要在实现的架构模式上和对接新的异步编程规范接口上. rxjava3 相对于 rxjava2 所做的工作则更多的是功能的增强,优化.

rxjava1 中没有实现响应式编程的规范接口,因为该规范出现在 2014年1月(开始制定),而rxjava1则是从2012年3月开始开发的.

## rxjava 实战(异步编程)

------

### 解决 Callback hell

```java
package com.github.hunter524.rxjava.start;

import io.reactivex.*;
import io.reactivex.functions.Consumer;
import io.reactivex.functions.Function;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class CallBackHell {
    public static ExecutorService CACHED_EXECUTOR_SERVICE = Executors.newCachedThreadPool();

    public static class PersonInfo {

        public String token;

        public PersonInfo(String token) {
            this.token = token;
        }
    }

    public static class TradeData {
        public String id;

        public TradeData(String id) {
            this.id = id;
        }
    }

    public static class TradeDetail {
        public String content;

        public TradeDetail(String content) {
            this.content = content;
        }
    }

    public interface GetPersonCallBack {
        public void onSucces(PersonInfo personInfo);

        public void onError();
    }

    public interface GetTradeDetailCallBack {
        public void onSucces(TradeDetail tradeDetail);

        public void onError();
    }

    public interface GetTradeDataCallBack {
        public void onSucces(TradeData tradedata);

        public void onError();
    }

    public static void getPersonInfo(String acc, String pwd, GetPersonCallBack getPersonCallBack) {
        CACHED_EXECUTOR_SERVICE.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                getPersonCallBack.onSucces(new PersonInfo("token"));
            }
        });
    }

    public static void getTradeData(String token, GetTradeDataCallBack getTradeDataCallBack) {
        CACHED_EXECUTOR_SERVICE.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                getTradeDataCallBack.onSucces(new TradeData("id"));
            }
        });
    }

    public static void getTradeDetail(String id, GetTradeDetailCallBack getTradeDetailCallBack) {
        CACHED_EXECUTOR_SERVICE.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                getTradeDetailCallBack.onSucces(new TradeDetail("content"));
            }
        });
    }

    public static void main(String[] args) {
//        旧的 callbackHell
        getPersonInfo("acc", "pwd", new GetPersonCallBack() {
            @Override
            public void onSucces(PersonInfo personInfo) {
                getTradeData(personInfo.token, new GetTradeDataCallBack() {
                    @Override
                    public void onSucces(TradeData tradedata) {
                        getTradeDetail(tradedata.id, new GetTradeDetailCallBack() {
                            @Override
                            public void onSucces(TradeDetail tradeDetail) {
                                System.out.println("Call Back hell Detail:" + tradeDetail.content);
                            }

                            @Override
                            public void onError() {

                            }
                        });
                    }

                    @Override
                    public void onError() {

                    }
                });
            }

            @Override
            public void onError() {

            }
        });


//        rxjava wrapped
        getPersonInfoObWrapp("acc", "pwd")
                .flatMap(new Function<PersonInfo, ObservableSource<TradeData>>() {
                    @Override
                    public ObservableSource<TradeData> apply(PersonInfo personInfo) throws Exception {
                        return getTradeDataObWrapp(personInfo.token);
                    }
                })
                .flatMap(new Function<TradeData, ObservableSource<TradeDetail>>() {
                    @Override
                    public ObservableSource<TradeDetail> apply(TradeData tradeData) throws Exception {
                        return getTradeDetailObWrapp(tradeData.id);
                    }
                })
                .observeOn(Schedulers.trampoline())
                .subscribe(new Consumer<TradeDetail>() {
                    @Override
                    public void accept(TradeDetail tradeDetail) throws Exception {
                        System.out.println("RxJava Wrap Detail:" + tradeDetail.content);
                    }
                });
// no wrap rxjava

        getPersonInfoOb("acc", "pwd")
                .flatMap(new Function<PersonInfo, ObservableSource<TradeData>>() {
                    @Override
                    public ObservableSource<TradeData> apply(PersonInfo personInfo) throws Exception {
                        return getTradeDataOb(personInfo.token);
                    }
                })
                .flatMap(new Function<TradeData, ObservableSource<TradeDetail>>() {
                    @Override
                    public ObservableSource<TradeDetail> apply(TradeData tradeData) throws Exception {
                        return getTradeDetailOb(tradeData.id);
                    }
                })
                .observeOn(Schedulers.trampoline())
                .subscribe(new Consumer<TradeDetail>() {
                    @Override
                    public void accept(TradeDetail tradeDetail) throws Exception {
                        System.out.println("RxJava NoWrap Detail:" + tradeDetail.content);
                    }
                });

    }
    // 包装老的接口进入 rxjava 的模式
    public static Observable<PersonInfo> getPersonInfoObWrapp(String acc, String pwd) {
        return Observable.<PersonInfo>create(new ObservableOnSubscribe<PersonInfo>() {
            @Override
            public void subscribe(ObservableEmitter<PersonInfo> emitter) throws Exception {
                getPersonInfo(acc, pwd, new GetPersonCallBack() {
                    @Override
                    public void onSucces(PersonInfo personInfo) {
                        emitter.onNext(personInfo);
                        emitter.onComplete();
                    }

                    @Override
                    public void onError() {
                        emitter.onError(new Throwable());
                    }
                });
            }
        });
    }

    public static Observable<TradeData> getTradeDataObWrapp(String token) {
        return Observable.<TradeData>create(new ObservableOnSubscribe<TradeData>() {
            @Override
            public void subscribe(ObservableEmitter<TradeData> emitter) throws Exception {
                getTradeData(token, new GetTradeDataCallBack() {
                    @Override
                    public void onSucces(TradeData tradeData) {
                        emitter.onNext(tradeData);
                        emitter.onComplete();
                    }

                    @Override
                    public void onError() {
                        emitter.onError(new Throwable());
                    }
                });
            }
        });
    }

    public static Observable<TradeDetail> getTradeDetailObWrapp(String id) {
        return Observable.<TradeDetail>create(new ObservableOnSubscribe<TradeDetail>() {
            @Override
            public void subscribe(ObservableEmitter<TradeDetail> emitter) throws Exception {
                getTradeDetail(id, new GetTradeDetailCallBack() {
                    @Override
                    public void onSucces(TradeDetail tradeDetail) {
                        emitter.onNext(tradeDetail);
                        emitter.onComplete();
                    }

                    @Override
                    public void onError() {
                        emitter.onError(new Throwable());
                    }
                });
            }
        });
    }

//    新写的接口直接 rxjava 模式

    public static Observable<PersonInfo> getPersonInfoOb(String acc, String pwd) {
        return Observable.just("start")
                         .map(new Function<String, PersonInfo>() {
                             @Override
                             public PersonInfo apply(String s) throws Exception {
                                 Thread.sleep(1000);
                                 return new PersonInfo("token");
                             }
                         })
                         .subscribeOn(Schedulers.io());
    }

    public static Observable<TradeData> getTradeDataOb(String token) {
        return Observable.just(token)
                         .map(new Function<String, TradeData>() {
                             @Override
                             public TradeData apply(String s) throws Exception {
                                 Thread.sleep(1000);
                                 return new TradeData("id");
                             }
                         })
                         .subscribeOn(Schedulers.io());
    }

    public static Observable<TradeDetail> getTradeDetailOb(String id) {

        return Observable.just(id)
                         .map(new Function<String, TradeDetail>() {
                             @Override
                             public TradeDetail apply(String s) throws Exception {
                                 Thread.sleep(1000);
                                 return new TradeDetail("content");
                             }
                         })
                         .subscribeOn(Schedulers.io());
    }

}
```

解决回调地狱在很多语言中都提供了一些解决方案:

- JS 中的 Promise,async,await
  
- Kotlin 中以协程为基础的 suspend fun (挂起方法)

### 解决频繁执行线程切换的需求

```java
package com.github.hunter524.rxjava.start;

import io.reactivex.Observable;
import io.reactivex.Observer;
import io.reactivex.disposables.Disposable;
import io.reactivex.functions.Function;
import io.reactivex.schedulers.Schedulers;

public class ThreadChange {

    public static void main(String[] args) throws Throwable {
        Observable.just("start")
                  .observeOn(Schedulers.io())
                  .map(new Function<String, String>() {
                      @Override
                      public String apply(String s) throws Exception {
                          System.out.println("Map1 Thread:" + Thread.currentThread());
                          return s + " map1 io ";
                      }
                  })
                  .observeOn(Schedulers.computation())
                  .map(new Function<String, String>() {
                      @Override
                      public String apply(String s) throws Exception {
                          System.out.println("Map2 Thread:" + Thread.currentThread());
                          return s + "map2 computation";
                      }
                  })
                  .observeOn(Schedulers.newThread())
                  .map(new Function<String, String>() {
                      @Override
                      public String apply(String s) throws Exception {
                          System.out.println("Map3 Thread:" + Thread.currentThread());
                          return s + " map3 newThread";
                      }
                  })
                  .observeOn(Schedulers.single())
                  .map(new Function<String, String>() {
                      @Override
                      public String apply(String s) throws Exception {
                          System.out.println("Map4 Thread:" + Thread.currentThread());
                          return s + " map4 single";
                      }
                  })
                  .subscribe(new Observer<String>() {
                      @Override
                      public void onSubscribe(Disposable d) {

                      }

                      @Override
                      public void onNext(String s) {
                          System.out.println("onNext Thread:" + Thread.currentThread());
                          System.out.println("onNext:" + s);
                      }

                      @Override
                      public void onError(Throwable e) {

                      }

                      @Override
                      public void onComplete() {

                      }
                  });
        Thread.sleep(1000);
    }
}
//    OUT_PUT:
//    Map1 Thread:Thread[RxCachedThreadScheduler-1,5,main]
//    Map2 Thread:Thread[RxComputationThreadPool-1,5,main]
//    Map3 Thread:Thread[RxNewThreadScheduler-1,5,main]
//    Map4 Thread:Thread[RxSingleScheduler-1,5,main]
//    onNext Thread:Thread[RxSingleScheduler-1,5,main]
//    onNext:start map1 io map2 computation map3 newThread map4 single
```

### 同步两个异步请求/并发计算同步回调

```java
package com.github.hunter524.rxjava.start;

import com.google.common.collect.Lists;
import io.reactivex.Observable;
import io.reactivex.functions.BiFunction;
import io.reactivex.functions.Consumer;
import io.reactivex.functions.Function;
import io.reactivex.schedulers.Schedulers;

import java.util.Arrays;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

//要实现 先请求个人信息,然后并发请求交易概括和交易详情,然后将交易详情组合进入交易概括内部
//用 java 实现则需要使用 CountDownLatch
public class ZipObservable {

    public static ExecutorService CACHED_EXECUTOR_SERVICE = Executors.newCachedThreadPool();


    public static void main(String[] args) throws InterruptedException {

//        java 实现
        final List<String>[] overView = new List[]{null};
        final List<String>[] tradeDetail = new List[]{null};

        CountDownLatch countDownLatch = new CountDownLatch(2);
        getPersonInfo("acc", "pwd", new GetPersonCallBack() {
            @Override
            public void onSucces(PersonInfo personInfo) {
                getTradeDetails(personInfo.token, new GetTradeDetailsCallBack() {
                    @Override
                    public void onSucces(TradeDetails tradedetails) {
                        tradeDetail[0] = tradedetails.details;
                        countDownLatch.countDown();
                    }

                    @Override
                    public void onError() {

                    }
                });

                getTradeOverView(personInfo.token, new GetTradeOverViewCallBack() {
                    @Override
                    public void onSucces(TradeOverView tradeoverview) {
                        overView[0] = tradeoverview.overviews;
                        countDownLatch.countDown();
                    }

                    @Override
                    public void onError() {

                    }
                });
            }

            @Override
            public void onError() {

            }
        });

        countDownLatch.await();
//        合并 detail 进入 Overview
        System.out.println("OverView:" + Arrays.deepToString(overView[0].toArray()) + "Details:" + Arrays.deepToString(tradeDetail[0].toArray()));

//        rxjava 实现
        getPersonInfoOb("acc", "pwd")
                .flatMap(
                        new Function<PersonInfo, Observable<String>>() {
                            @Override
                            public Observable<String> apply(PersonInfo personInfo) throws Exception {
                                return Observable.zip(getTradeOverViewOb(personInfo.token), getTradeDetailOb(personInfo.token),
                                        new BiFunction<TradeOverView, TradeDetails, String>() {
                                            @Override
                                            public String apply(TradeOverView tradeOverView, TradeDetails tradeDetails) throws Exception {
                                                System.out.println("Rx OverView:" + Arrays.deepToString(tradeOverView.overviews.toArray())
                                                        + "Details:" + Arrays.deepToString(tradeDetails.details.toArray()));

                                                return "success";
                                            }
                                        });
                            }
                        })
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(String s) throws Exception {
                        System.out.println("RxResult:" + s);
                    }
                });

    }

    public static class PersonInfo {

        public String token;

        public PersonInfo(String token) {
            this.token = token;
        }
    }

    public static class TradeOverView {
        public List<String> overviews;

        public TradeOverView(List<String> overviews) {
            this.overviews = overviews;
        }
    }

    public static class TradeDetails {
        public List<String> details;

        public TradeDetails(List<String> details) {
            this.details = details;
        }
    }

    public interface GetTradeOverViewCallBack {
        public void onSucces(TradeOverView tradeoverview);

        public void onError();
    }

    public interface GetTradeDetailsCallBack {
        public void onSucces(TradeDetails tradedetails);

        public void onError();
    }

    public interface GetPersonCallBack {
        public void onSucces(PersonInfo personInfo);

        public void onError();
    }

    public static void getPersonInfo(String acc, String pwd, GetPersonCallBack getPersonCallBack) {
        CACHED_EXECUTOR_SERVICE.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                getPersonCallBack.onSucces(new PersonInfo("token"));
            }
        });
    }

    public static void getTradeOverView(String token, GetTradeOverViewCallBack getTradeOverViewCallBack) {
        CACHED_EXECUTOR_SERVICE.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                getTradeOverViewCallBack.onSucces(new TradeOverView(Lists.newArrayList("over 11", "over 22", "over 33")));
            }
        });
    }

    public static void getTradeDetails(String token, GetTradeDetailsCallBack getTradeDetailsCallBack) {
        CACHED_EXECUTOR_SERVICE.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                getTradeDetailsCallBack.onSucces(new TradeDetails(Lists.newArrayList("detail 11", "detail 22", "detail 33")));
            }
        });
    }


    //    新写的接口直接 rxjava 模式

    public static Observable<PersonInfo> getPersonInfoOb(String acc, String pwd) {
        return Observable.just("start")
                         .map(new Function<String, PersonInfo>() {
                             @Override
                             public PersonInfo apply(String s) throws Exception {
                                 Thread.sleep(1000);
                                 return new PersonInfo("token");
                             }
                         })
                         .subscribeOn(Schedulers.io());
    }

    public static Observable<TradeOverView> getTradeOverViewOb(String token) {
        return Observable.just(token)
                         .map(new Function<String, TradeOverView>() {
                             @Override
                             public TradeOverView apply(String s) throws Exception {
                                 Thread.sleep(1000);
                                 return new TradeOverView(Lists.newArrayList("over 11", "over 22", "over 33"));
                             }
                         })
                         .subscribeOn(Schedulers.io());
    }

    public static Observable<TradeDetails> getTradeDetailOb(String id) {

        return Observable.just(id)
                         .map(new Function<String, TradeDetails>() {
                             @Override
                             public TradeDetails apply(String s) throws Exception {
                                 Thread.sleep(1000);
                                 return new TradeDetails(Lists.newArrayList("detail 11", "detail 22", "detail 33"));
                             }
                         })
                         .subscribeOn(Schedulers.io());
    }
}
```

- zip 与 CountDownLatch 实现的区别
  
  zip 是基于非阻塞算法进行的, CountDownLatch 则是在 CountDownLatch#await 时会使用 LockSupport#park 操作阻塞线程,虽然不占用太多cpu资源,但是占用了进程的线程资源(系统对于每个进程创建的最大线程是有限制的)

- 如何给异步请求添加超时机制
  
  对 Zip 操作的每个 Observable添加一个 timeout 操作符即可.

### 实现通用的事件中心(多播/单播/重播/最新的事件)

在 Android 中需要使用 LocalBroadcastManager (可以实现复杂的事件下发机制).在 java 中则需要基于 java.util.Observable 和 java.util.Observer 实现.或者引入别人实现的功能强大的事件总线库.

- 基于 JDK 实现的事件中心

```java
package com.github.hunter524.rxjava.start;

import java.util.Observable;
import java.util.Observer;

public class JavaEventCenter {
    public static void main(String[] args) {

//        下发时要设置 setChanged 否则该事件会被忽略

        Observable eventCenter = new Observable() {
            @Override
            public void notifyObservers(Object arg) {
                setChanged();
                super.notifyObservers(arg);
            }
        };

        Observer manObserver = new Observer() {
            @Override
            public void update(Observable o, Object arg) {
                System.out.println("Man Observer Type:" + arg.getClass()
                                                             .getSimpleName());
                if (arg instanceof Man) {
                    System.out.println("Man Receive:" + ((Man) arg).name);
                }
            }
        };

        Observer woManObserver = new Observer() {
            @Override
            public void update(Observable o, Object arg) {
                System.out.println("Woman Observer Type:" + arg.getClass()
                                                             .getSimpleName());
                if (arg instanceof Woman) {
                    System.out.println("Woman Receive:" + ((Woman) arg).name);
                }
            }
        };
        eventCenter.notifyObservers(new Human("human 1"));

        eventCenter.addObserver(manObserver);

        eventCenter.notifyObservers(new Human("Human 1"));
        eventCenter.notifyObservers(new Man("Man 1"));
        eventCenter.notifyObservers(new Woman("Woman 1"));

        eventCenter.addObserver(woManObserver);

        eventCenter.notifyObservers(new Human("Human 2"));
        eventCenter.notifyObservers(new Man("Man 2"));
        eventCenter.notifyObservers(new Woman("Woman 2"));

    }

    public static class Human {
        public String name;

        public Human(String name) {
            this.name = name;
        }
    }

    public static class Man extends Human {
        public Man(String name) {
            super("Man:" + name);
        }
    }

    public static class Woman extends Human {
        public Woman(String name) {
            super("Woman:" + name);
        }
    }
}

// OUTPUT:
// Man Observer Type:Human
// Man Observer Type:Man
// Man Receive:Man:Man 1
// Man Observer Type:Woman
// Woman Observer Type:Human
// Man Observer Type:Human
// Woman Observer Type:Man
// Man Observer Type:Man
// Man Receive:Man:Man 2
// Woman Observer Type:Woman
// Woman Receive:Woman:Woman 2
// Man Observer Type:Woman

```

后面加入的监听者无法收到前面已经下发的事件?如果后面加入的监听者要接收到最新下发的事件是不是就又要自己写代码了?如果后面加入的监听者要收到之前下发的所有事件是不是又要重新写代码?让 Observer 收到所有事件自己过滤要不要处理是不是不安全?

- Rxjava 的事件中心

```java
package com.github.hunter524.rxjava.start;

import io.reactivex.Observable;
import io.reactivex.functions.Consumer;
import io.reactivex.subjects.BehaviorSubject;
import io.reactivex.subjects.PublishSubject;
import io.reactivex.subjects.Subject;

// 示例创建一个后添加的监听者也能接收到最近的一个事件的事件中心
// 并且事件的监听过滤收纳在 Observable 内部,不向外暴露.
public class RxJavaEventCenter {

    static Subject<Human> EVENT_CENTER = BehaviorSubject.create();

    static Observable<Human> HUMAN_CENTER = EVENT_CENTER.ofType(Human.class);

    static Observable<Man> MAN_CENTER = EVENT_CENTER.ofType(Man.class);

    static Observable<Woman> WOMAN_CENTER = EVENT_CENTER.ofType(Woman.class);


    public static void main(String[] args) {
        EVENT_CENTER.onNext(new Human("human 0"));

        HUMAN_CENTER.subscribe(new Consumer<Human>() {
            @Override
            public void accept(Human human) throws Exception {
                System.out.println("Observer Human:" + human.name);
            }
        });
        EVENT_CENTER.onNext(new Human("human 1"));
        EVENT_CENTER.onNext(new Man("man 0"));
        EVENT_CENTER.onNext(new Man("man 1"));
        EVENT_CENTER.onNext(new Woman("woman 0"));
        EVENT_CENTER.onNext(new Man("man 1"));

        MAN_CENTER.subscribe(new Consumer<Man>() {
            @Override
            public void accept(Man man) throws Exception {
                System.out.println("Observer Man:" + man.name);
            }
        });

        EVENT_CENTER.onNext(new Man("man 2"));
// 由于最近一个事件并非 Woman 因此该处订阅收不到任何事件
        WOMAN_CENTER.subscribe(new Consumer<Woman>() {
            @Override
            public void accept(Woman woman) throws Exception {
                System.out.println("Observer Woman:" + woman.name);
            }
        });
    }

    public static class Human {
        public String name;

        public Human(String name) {
            this.name = name;
        }
    }

    public static class Man extends Human {
        public Man(String name) {
            super("Man:" + name);
        }
    }

    public static class Woman extends Human {
        public Woman(String name) {
            super("Woman:" + name);
        }
    }
}

//OUTPUT:
//Observer Human:human 0
//Observer Human:human 1
//Observer Human:Man:man 0
//Observer Human:Man:man 1
//Observer Human:Woman:woman 0
//Observer Human:Man:man 1
//Observer Man:Man:man 1
//Observer Human:Man:man 2
//Observer Man:Man:man 2
```

- AsyncSubject

订阅者只能接受 onComplete 之前的最新的一个事件,或者onError

![AsyncSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/AsyncSubject.png)

- BehaviorSubject

![BehaviorSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/S.BehaviorSubject.png)

- CompletableSubject

![CompletableSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/CompletableSubject.png)

- MaybeSubject
  
![MaybeSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/MaybeSubject.png)

- PublishSubject
  
![PublishSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/PublishSubject.png)

- ReplaySubject
  
![ReplaySubject ERROR](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/ReplaySubject.ue.png)

![ReplaySubject Normal](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/ReplaySubject.u.png)

- SerializedSubject

只是为了 onXXX 串行化调用.

- SingleSubject
  
![SingleSubject](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/SingleSubject.png)

- UnicastSubject
  
![UnicastSubject](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/UnicastSubject.png)

### 异步事件序列转换成同步操作/异步任务转换成为异步事件序列

```java
package com.github.hunter524.rxjava.start;

import io.reactivex.Observable;
import io.reactivex.ObservableEmitter;
import io.reactivex.ObservableOnSubscribe;
import io.reactivex.functions.Consumer;
import io.reactivex.schedulers.Schedulers;

import java.util.Arrays;
import java.util.List;

public class RxAsyncToSync {
    public static void main(String[] args) throws Throwable {
//        异步的 Observable
        Observable<String> one = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                Thread.sleep(1000);
                emitter.onNext("one");
                emitter.onNext("two");
                emitter.onComplete();
            }
        }).subscribeOn(Schedulers.io());
        System.out.println("one async start subscribe @"+System.currentTimeMillis());
        one.subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                System.out.println("one async onNext @"+System.currentTimeMillis()+"Value:"+s);
            }
        });
        System.out.println("one async end subscribe @"+System.currentTimeMillis());

//        异步的 Observable 使其同步的返回数据
        System.out.println("one sync block start Get @"+System.currentTimeMillis());
        List<String> blockingGet = one.toList().blockingGet();
        System.out.println("one sync block end Get @"+System.currentTimeMillis()+"Values:"+ Arrays.deepToString(blockingGet.toArray()));

        Thread.sleep(3000);
    }
//    OUT_PUT:
//    one async start subscribe @1604025816577
//    one async end subscribe @1604025816596
//    one sync block start Get @1604025816596
//    one async onNext @1604025817613Value:one
//    one async onNext @1604025817613Value:two
//    one sync block end Get @1604025817630Values:[one, two]
}
```

```java
package com.github.hunter524.rxjava.start;

import io.reactivex.Observable;
import io.reactivex.Observer;
import io.reactivex.disposables.Disposable;
import io.reactivex.functions.Consumer;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class AsyncToRxAsync {

    public static ExecutorService CACHED_EXECUTOR_SERVICE = Executors.newCachedThreadPool();


    public static void main(String[] args) throws Throwable {
        Future<String> future = CACHED_EXECUTOR_SERVICE.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
                Thread.sleep(1000);
//                throw new IllegalArgumentException("args illegal!");
                return "callable return";
            }
        });
        System.out.println("subscribe @:" + System.currentTimeMillis());
        Observable.fromFuture(future, Schedulers.io())
                  .subscribe(new Observer<String>() {
                      @Override
                      public void onSubscribe(Disposable d) {

                      }

                      @Override
                      public void onNext(String s) {
                          System.out.println("onNext @:" + System.currentTimeMillis() + "Value:" + s);
                      }

                      @Override
                      public void onError(Throwable e) {
                          e.printStackTrace();
//                          e.getCause().printStackTrace();
                      }

                      @Override
                      public void onComplete() {

                      }
                  });
    }

//    OUT_PUT:
//    subscribe @:1604025972172
//    onNext @:1604025973184Value:callable return
}
```

当然也可以将 RxAsync 转换成为 Future .只需要使用 Observable#toFuture

### 并发竞争/及时取消

![Observable#amb](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/amb.png)

### 防抖

![debounce](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/debounce.png)

![throttleFirst](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/throttleFirst.png)

![throttleLast](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/throttleLast.png)

## rxjava 实战(函数式编程)

### 缓存/过滤/分组/叠加/数学求和/均值/

这些操作其实并非 rxjava 的核心目标,只是顺带实现了这些函数式编程已经提供的基础方法.需要引入rxjava 作者自己实现的扩展库<https://github.com/akarnokd/RxJavaExtensions>

## reactive-streams 规范接口

Reactive Streams <http://www.reactive-streams.org/> <https://github.com/reactive-streams/reactive-streams-jvm>

项目核心代码只定义了四个接口,但是大部分响应式编程的库都引用该库,遵守该接口规范.

### Publisher

表示无限事件序列的生产者,rxjava2 中的 Flowable 实现了该接口,在 rxjava1 中没有实现该规范,因为该规范出现在 2014年1月(开始制定),而rxjava1则是从2012年3月开始开发的.

### Subscriber

表示事件的订阅者, Subscriber 通过 Publisher#subscribe 方法向生产者进行订阅. Publisher 则通过 Subscriber#onSubscribe,Subscriber#onNext,Subscriber#onError,Subscriber#onComplete 分别下发订阅关系,事件,error 状态,结束状态.

### Subscription

表示订阅链的订阅关系.Subscriber 通过 Subscriber#onSubscribe 下发的 Subscription .通过 Subscription#cancel 可以取消订阅关系.通过 Subscription#request 消费者可以协调生产者的生产速率(如果生产者支持的话).

### Processor

同时继承了 Subscriber/Publisher,既是生产者也是消费者.

## rxjava 基础组件简介

### Observable(创建/被观察者)

0..N flows, no backpressure,

```java
        Observable.create(
                new ObservableOnSubscribe<String>() {
                    @Override
                    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                        ObservableEmitter<String> serialize = emitter.serialize();
                        serialize.onNext("one");
                        serialize.onNext("two");
                        serialize.onNext("three");
                        serialize.onComplete();
                    }
                })
```

```java
        Observable.just("one")
                  .subscribe(new Consumer<String>() {
                      @Override
                      public void accept(String s) throws Exception {
                          System.out.println("Accetp Data From ObservableEmitter:" + s);
                      }
                  })
```

```java
String[] words = {"Hello", "Hi", "Aloha"};
Observable observable = Observable.from(words);
// 将会依次调用：
// onNext("Hello");
// onNext("Hi");
// onNext("Aloha");
// onCompleted();
```

### Subscriber/Observer(观察者)

均为含有订阅者/观察者的语义.

rxjava1 的 Subscriber 继承了 Subscription 因此还含有订阅关系的语义表示.

在 rxjava2 rxjava3 则保持相同的设计 Observer 只含有订阅者/观察者的语义.订阅关系的语义则由 Disposable 单独表述.移除了Subscriber 这个类.但是也新增了 DisposableObserver 这个类,用于表示 rxjava1 中的 Subscriber 语义.

选择 Observer 和 Subscriber 是完全一样的。它们的区别对于使用者来说主要有两点：

- onStart(): 这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求onStart() 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。

- unsubscribe()
  
  这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态。 unsubscribe() 这个方法很重要，因为在 subscribe() 之后， Observable 会持有 Subscriber 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方调用 unsubscribe() 来解除引用关系，以避免内存泄露的发生。
  
  在 rxjava2 rxjava3 中则对应于 Disposable#dispose 和 Disposable#isDisposed 两个方法.

与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 onNext()之外，还定义了两个特殊的事件：onCompleted() 和 onError()。

- onCompleted()
  
  事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发 onCompleted() 方法作为标志。
  
- onError()
  
  事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。
  
在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

- onSubscribe

  rxjava2 添加在 Observer 中的方法,调用时会传入 Disposable 用于解除订阅关系 ,解决 rxjava1 同步订阅时无法取消的问题.

```java
package com.github.hunter524.rxjava.start;

import com.google.common.base.Ascii;
import com.google.common.base.Strings;
import com.google.common.collect.Lists;
import io.reactivex.*;
import io.reactivex.disposables.Disposable;
import io.reactivex.functions.BiFunction;
import io.reactivex.functions.Consumer;
import io.reactivex.functions.Function;
import io.reactivex.schedulers.Schedulers;

public class CreateObservable {
    public static void main(String[] args) throws Throwable {
        Observable.create(
                new ObservableOnSubscribe<String>() {
                    @Override
                    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                        ObservableEmitter<String> serialize = emitter.serialize();
                        serialize.onNext("one");
                        serialize.onNext("two");
                        serialize.onNext("three");
                        serialize.onComplete();
                    }
                })
//                  小写变大写
                  .map(new Function<String, String>() {
                      @Override
                      public String apply(String s) throws Exception {
                          return Ascii.toUpperCase(s);
                      }
                  })
//                  String 流变 char 流
                  .flatMap(new Function<String, Observable<Character>>() {
                      @Override
                      public Observable<Character> apply(String s) throws Exception {
                          return Observable.fromIterable(Lists.charactersOf(s));
                      }
                  })
//                  char 流去重 ONETWHR
                  .distinct()
//                  再排序 E H N O R T W
                  .sorted()
                  .subscribeOn(Schedulers.io())
                  .subscribe(new Observer<Character>() {
                      @Override
                      public void onSubscribe(Disposable d) {

                      }

                      @Override
                      public void onNext(Character character) {
                          System.out.println(character);
                      }

                      @Override
                      public void onError(Throwable e) {

                      }

                      @Override
                      public void onComplete() {

                      }
                  });
        Thread.sleep(1000);
    }
}
// OUT_PUT:
// E
// H
// N
// O
// R
// T
// W

```

![rxjava 事件流](rxjavaimg/rxjava事件流概括.png)
![rxjava 订阅流程](rxjavaimg/rxjava订阅执行流程.png)

![Map示意图](rxjavaimg/map.jpg)

![FlatMap 示意图](rxjavaimg/flatmap.jpg)

图中的 OnSubscribe 为 rxjava1 的遗留概念,rxjava2 中不存在.对应的功能被合并进入了 Observable#subscribeActual.有兴趣的可以对比一下 rxjava1 rxjava2 中 Observable#just 操作附的实现.

![Observable#map](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/map.png)

![Observable#flatMap](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/flatMap.png)

### Subscription/Disposable(订阅/订阅关系)

rxjava1 中 Observable#subscribe 订阅方法的返回值则为 Subscription 订阅关系.

rxjava2/rxjava3 中 Observable#subscribe 的订阅者如果为 Observer 则该订阅方法的返回值为 void,订阅关系的描述 Disposable 则通过 Observer#onSubscribe 下发进入 Observer 内部用于取消订阅关系.

订阅关系取消的条件有两种:

- Observer 认为自己不需要再关注 Observable 下发的事件了,调用 Subscription#unsubscribe 或者 Disposable#dispose

- Observable 认为自己不再发送数据了即数据已经发送完毕,或者在数据的产生过程/处理过程中出现了未捕获的异常.则会分别调用 Observer#onComplete 或 Observer#onError 同时解除订阅关系.

订阅关系在不再需要时需要及时取消,否则会占用资源.

### Scheduler

传递给 Observable#observeOn,Observable#subscribeOn 的线程任务调度器用于切换订阅者观察者的执行线程.

内置线程调度器分为:

- IO

  底层为线程池的线程最大数量设置为 int 的最大值.

- COMPUTATION

  底层为根据 cpu 核心数配置的固定线程数的线程池.

- SINGLE

  底层为只有一个线程的线程池

- TRAMPOLINE

  蹦床模式,在当前线程执行该任务但是并不是立即执行,会稍后执行.实现为使当前线程会进入事件循环模式.

- NEW_THREAD

  每次调度任务执行都新建一个线程执行.

- MAIN

  客户端的特殊的主线程

### Flowable

0..N flows, supporting Reactive-Streams and backpressure

rxjava1 中不存在

### Single

a flow of exactly 1 item or an error,

### Maybe

a flow with no items, exactly one item or an error.

rxjava1 中不存在.

### Completable

a flow without items but only a completion or error signal,

### Subject

既是 Observable 也是 Observer.其实可以理解为事件中转站,自己作为 Observer 接收上游下发的事件,同时自己也作为 Observable 可以被订阅,向订阅了自己的订阅者下发之前接收到的事件.

### subscribeOn/observeOn

- subscribeOn

  切换 subscribe 操作的线程.

- observeOn

  切换响应者的线程.

### Flowable/Observable/Single/Maybe/Completable

是可以相互转换的,产生的事件序列依次从多到少.

## 高级概念

### 背压

生产者的生产效率大于消费者的消费效率.因此需要配置仓库容纳生产者生产的产品,或者直接按照配置的规则丢弃生产者生产的产品,再或者要求生产者放慢生产效率,按照需求生产产品.如果一直存储则会最终打爆内存导致崩溃.

rxjava1:在创建安全 Observable 时则要求传入背压策略进行背压控制.

rxjava2:Observable 不再支持背压控制策略语义,创建安全 Observable 时使用串行发射器默认的背压策略时无限缓存.只有创建 Flowable 时才会要求和可以进行背压策略控制.rxjava1 中不存在 Flowable 的概念.
*如果不需要串行化调用 onNext 方法,则不存在背压的问题*

### 串行访问

Observer#onNext/Observer#onComplete/Observer#onError 的串行化访问

rxjava1:

创建自定义的Observable分为两种:Observable#unsafeCreate和Observable#create,前者是不保证串行访问的,后者则会保证串行访问.同时前者也不保证 onError 和 onCompleted 只能被调用一次,当 onError 和 onComplete 被调用之后不能再调用 onNext 的语义规则.

rxjava2,rxjava3:

Observable#create 创建的 Observable 不再保证串行化访问.如果需要串行化访问则需要使用 ObservableEmitter#serialize 获得串行化发射器,使用串行化发射器才能保证下游数据的串行访问. Observable 不提供串行访问也就不存在背压问题.一旦串行化之后内部便会提供一个SPSC 队列用于缓存来不及消费的元素.

*队列漏和发射循环是实现串行化访问语义的基础算法.*

### 冷/热 Observable

冷: 每次订阅都产生事件序列,只在有订阅者的时候产生事件序列.每次产生的事件序列可能相同也可能不同,与作者的实现有关.

热: 不管有没有订阅者都按照既定的规则产生事件序列,可以被多次订阅,但后面的订阅者无法收到前面已经错过的事件序列.主要使用在不想订阅者订阅一次就要求执行一次操作的场景,避免过多的资源耗费.

```java
package com.github.hunter524.rxjava.start;

import io.reactivex.Observable;
import io.reactivex.ObservableEmitter;
import io.reactivex.ObservableOnSubscribe;
import io.reactivex.Observer;
import io.reactivex.disposables.Disposable;
import io.reactivex.observables.ConnectableObservable;
import io.reactivex.schedulers.Schedulers;

public class HotColdObservable {
    public static void main(String[] args) throws Throwable {
        Observable<String> coldOb = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("1-"+"TIME:"+System.nanoTime());
                emitter.onNext("2-"+"TIME:"+System.nanoTime());
                emitter.onNext("3-"+"TIME:"+System.nanoTime());
                emitter.onNext("4-"+"TIME:"+System.nanoTime());
            }
        });

        Observable<String> hotOb = Observable
                .create(new ObservableOnSubscribe<String>() {
                    @Override
                    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                        emitter.onNext("1-"+"TIME:"+System.nanoTime());
                        Thread.sleep(1000);
                        emitter.onNext("2-"+"TIME:"+System.nanoTime());
                        Thread.sleep(1000);
                        emitter.onNext("3-"+"TIME:"+System.nanoTime());
                        Thread.sleep(1000);
                        emitter.onNext("4-"+"TIME:"+System.nanoTime());
                    }
                })
                .subscribeOn(Schedulers.io())
                .publish();

        ((ConnectableObservable<String>) hotOb).connect();


        coldOb.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(String s) {
                System.out.println("cold subscribe 1 ====:"+s);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });

        coldOb.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(String s) {
                System.out.println("cold subscribe 2====:"+s);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });

        Thread.sleep(1100);

        hotOb.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(String s) {
                System.out.println("hot subscribe 1 ====:"+s);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
        Thread.sleep(1100);

        hotOb.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(String s) {
                System.out.println("hot subscribe 2 ====:"+s);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
        Thread.sleep(5000);
    }
//    OUT_PUT
//    cold subscribe 1 ====:1-TIME:39156321281444
//    cold subscribe 1 ====:2-TIME:39156321810264
//    cold subscribe 1 ====:3-TIME:39156321970437
//    cold subscribe 1 ====:4-TIME:39156322094173

//    cold subscribe 2====:1-TIME:39156322506015
//    cold subscribe 2====:2-TIME:39156322684361
//    cold subscribe 2====:3-TIME:39156322796437
//    cold subscribe 2====:4-TIME:39156322914416

//    hot subscribe 1 ====:3-TIME:39158324861822
//    hot subscribe 1 ====:4-TIME:39159325097997

//    hot subscribe 2 ====:4-TIME:39159325097997
}
```

### Operator/Observable#lift/ObservableTransformer/Observable#compose

rxjava1 遗留的概念,在 rxjava1 中很多内置操作符语义都是通过 Operator 和 Observable#lift 进行实现的.有点复杂不够直观.
前者用来转换 Observer 后者用来转换 Observable.

rxjava2 和 rxjava3 中则不再使用 Operator 实现内置的操作符语义.转而使用包装 Observable 和 Observer 实现操作符语义.提供给用户使用 Observable#as 操作符进行 Observable 的包装.[Uber AutoDispose][https://uber.github.io/AutoDispose/] 则是基于该操作符进行的扩展,从而实现根据生命周期自动解除订阅的功能.

### Producer

rxjava1 遗留的概念,用于消费者协调生产者的生产效率,解决背压问题. rxjava2 中只有 Flowable 流中存在对应的概念.但是已经不是像 rxjava1 中使用自己定义的 Producer 而是使用 org.reactivestreams 规范中定义的 Subscription 接口.

### RxJavaHooks/RxJavaPlugins 机制

人工提供的切面,可以在事件下发,onError回调,线程调度提供了一个监控和替换的切面.

### 响应式编程的定义/实现

- 社区界的接口定义/规范定义 <https://github.com/reactive-streams/>

- 规范官网 <https://www.reactive-streams.org/>

- 主流的各种语言的实现 <https://github.com/ReactiveX> 其中 rxjava 43.6k 个 star.rxjs 23.2k 个 star.

- Spring 也支持响应试编程 <https://spring.io/reactive> 只不过其推荐了另外一种实现方式<https://github.com/reactor/>作为其生态的一部分。但是其也实现了 reactive-streams 规范.

- jdk 9 增加了 flow 响应式编程的支持 <https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.Publisher.html>

### 谁在使用

reactivex 官网下滑 <http://reactivex.io>

### Reference

- 扔物线的 rxjava 入门教程 <https://gank.io/post/560e15be2dca930e00da1083> *本人入门时看的第一篇教程*

- rxjava1 源码分析 <https://blog.piasy.com/AdvancedRxJava/index.html> *阅读 rxjava 源码是参考的系列文章,读懂了 rxjava1 的源码 rxjava2 rxjava3 的源码根本不存在问题,因为 rxjava1 的实现机制更复杂,不够简洁*

- 串行访问,非阻塞算法 (emitter-loop)发射循环<https://blog.piasy.com/AdvancedRxJava/2016/05/06/operator-concurrency-primitives/index.html>

- 串行访问,非阻塞算法 (queue-drain)队列漏]<https://blog.piasy.com/AdvancedRxJava/2016/05/13/operator-concurrency-primitives-2/index.html>

- JCTools Java Concurrent Tools <https://github.com/JCTools/JCTools>

- rxjava 函数式扩展 <https://github.com/akarnokd/RxJavaExtensions>

- rxjava 操作附汇总 <https://github.com/ReactiveX/RxJava/wiki/Alphabetical-List-of-Observable-Operators>

- rxjava 发起者,akarnokd 匈牙利,布达佩斯 匈牙利科学院的工程学博士<https://akarnokd.blogspot.com/>作者只主写了 rxjava. 其他语言的版本均由开源社区的其他人员创作.其也是 reactive-streams 规范的制定者之一.
