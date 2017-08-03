# RxJava使用介绍
>主讲人：阳石柏


- RxJava基本概念
- 背压概念介绍
- RxJava 2.0版本介绍及更新




## 一.RxJava基本概念


>RxJava 在 GitHub 主页上的自我介绍是 "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）,这就是RxJava。然而对于刚接触Rxjava的人来说,这种概括显然变得晦涩难懂,那么RxJava到底是什么？其实初学RxJava只要把握两点：观察者模式和异步,就基本可以熟练使用RxJava了。

>异步在这里并不需要做太多的解释，因为在概念和使用上，并没有太多高深的东西。大概就是你脑子里想能到的那些多线程，线程切换这些东西。我会在后面会讲解它的用法。我们先把观察者模式说清楚,“按下开关，台灯灯亮”

#### 在这个事件中，台灯作为观察者，开关作为被观察者，台灯透过电线来观察开关的状态来并做出相应的处理

![image](http://i.imgur.com/zBcYcQe.png)

观察上图，其实已经很明了了，不过需要指出一下几点（对于下面理解RxJava很重要）：

### 开关（被观察者）作为事件的产生方（生产“开”和“关”这两个事件），是主动的，是整个开灯事理流程的起点。
### 台灯（观察者）作为事件的处理方（处理“灯亮”和“灯灭”这两个事件），是被动的，是整个开灯事件流程的终点。
### 在起点和终点之间，即事件传递的过程中是可以被加工，过滤，转换，合并等等方式处理的（上图没有体现，后面会讲到）。

>这三点对于我们理解RxJava非常重要。因为上述三条分别对应了RxJava中被观察者(Observable),观察者(Observer)和操作符的职能。而观察者模式又是RxJava程序运行的骨架。RxJava也是基于观察者模式来组建自己的程序逻辑的，就是构建被观察者（Observable），观察者（Observer/Subscriber），然后建立二者的订阅关系（就像那根电线，连接起台灯和开关）实现观察，在事件传递过程中还可以对事件做各种处理。



### 创建被观察者

#### 正常模式：

``` java
   Observable switcher=Observable.create(new Observable.OnSubscribe<String>(){

            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("On");
                subscriber.onNext("Off");
                subscriber.onNext("On");
                subscriber.onNext("On");
                subscriber.onCompleted();
            }
        });

```

这是最正宗的写法，创建了一个开关类，产生了五个事件，分别是：开，关，开，开，结束。

### 偷懒模式1

``` java
    Observable switcher=Observable.just("On","Off","On","On");

```
### 偷懒模式2

``` java
    String [] kk={"On","Off","On","On"};
    Observable switcher=Observable.from(kk);
```  
    
#### 偷懒模式是一种简便的写法，实际上也都是被观察者把那些信息"On","Off","On","On"，包装成onNext（"On"）这样的事件依次发给观察者，当然，它自己补上了onComplete()事件。

#### 以上是最常用到的创建方式，好了，我们就创建了一个开关类。


### 创建观察者

#### 正常模式

```java       
        Subscriber light=new Subscriber<String>() {
            @Override
            public void onCompleted() {
            //被观察者的onCompleted()事件会走到这里;
             Log.d("DDDDDD","结束观察...\n";
            }

            @Override
            public void onError(Throwable e) {
                    //出现错误会调用这个方法
            }
            @Override
            public void onNext(String s) {
                //处理传过来的onNext事件
                Log.d("DDDDD","handle this---"+s)
            }
            
```
#### 偷懒模式(非正式写法)

```java
        Action1 light =new Action1<String>() {
                @Override
                public void call(String s) {
                    Log.d("DDDDD","handle this---"+s)
                }
            }

```         
#### 之所以说它是非正式写法，是因为Action1是一个单纯的人畜无害的接口，和Observer没有啥关系，只不过它可以当做观察者来使，专门处理onNext 事件，这是一种为了简便偷懒的写法。当然还有Action0，Action2,Action3...,0,1,2,3分别表示call()这个方法能接受几个参数。如果你还不懂，可以暂时跳过。后面我也会尽量使用new Subscriber方式，创建正统的观察者，便于你们理解。

### 订阅

#### 现在已经创建了观察者和被观察者，但是两者还没有联系起来
```java
    switcher.subscribe(light);
    
    这里有很多初学者的疑问就是明明是台灯观察开关，正常来看就应是light.subscribe(switcher)才对，之所以“开关订阅台灯”，是为了保证流式API调用风格
```   
### 啥是流式API调用风格？

```java
    //这就是RxJava的流式API调用
    Observable.just("On","Off","On","On")
        //在传递过程中对事件进行过滤操作
         .filter(new Func1<String, Boolean>() {
                    @Override
                    public Boolean call(String s) {
                        return s！=null;
                    }
                })
        .subscribe(mSubscriber);
```
       
#### 上面就是一个非常简易的RxJava流式API的调用：同一个调用主体一路调用下来，一气呵成。

>由于被观察者产生事件，是事件的起点，那么开头就是用Observable这个主体调用来创建被观察者，产生事件，为了保证流式API调用规则，就直接让Observable作为唯一的调用主体，一路调用下去。

>一句话，背后的真实的逻辑依然是台灯订阅了开关，但是在表面上，我们让开关“假装”订阅了台灯，以便于保持流式API调用风格不变。

好了，现在分解动作都完成了，已经架构了一个基本的RxJava事件处理流程。

我们再来按照观察者模式的运作流程和流式Api的写法复习一遍：

![image](http://i.imgur.com/LTOXDM3.png)

结合流程图的相应代码实例如下：
```java
//创建被观察者，是事件传递的起点

    Observable.just("On","Off","On","On")
        //这就是在传递过程中对事件进行过滤操作
         .filter(new Func1<String, Boolean>() {
                    @Override
                    public Boolean call(String s) {
                        return s！=null;
                    }
                })
                
        //实现订阅
        .subscribe(
        //创建观察者，作为事件传递的终点处理事件    
                  new Subscriber<String>() {
                        @Override
                        public void onCompleted() {
                            Log.d("DDDDDD","结束观察...\n");
                        }

                        @Override
                        public void onError(Throwable e) {
                            //出现错误会调用这个方法
                        }
                        @Override
                        public void onNext(String s) {
                            //处理事件
                            Log.d("DDDDD","handle this---"+s)
                        }
        );
        
```        
#### 至此,基本上我们就把RxJava的骨架就讲完了，总结一下：

>1.创建被观察者，产生事件

>2.设置事件传递过程中的过滤，合并，变换等加工操作。

>3.订阅一个观察者对象，实现事件最终的处理。

Tips: 当调用订阅操作（即调用Observable.subscribe()方法）的时候，被观察者才真正开始发出事件。

### RxJava的操作符

#### Map操作

>比如被观察者产生的事件中只有图片文件路径；,但是在观察者这里只想要bitmap,那么就需要类型变换。

```
    Observable.just(getFilePath())
           //指定了被观察者执行的线程环境
          .subscribeOn(Schedulers.newThread())
          //将接下来执行的线程环境指定为io线程
          .observeOn(Schedulers.io())
            //使用map操作来完成类型转换
            .map(new Func1<String, Bitmap>() {
              @Override
              public Bitmap call(String s) {
                //显然自定义的createBitmapFromPath(s)方法，是一个极其耗时的操作
                  return createBitmapFromPath(s);
              }
          })
            //将后面执行的线程环境切换为主线程
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                 //创建观察者，作为事件传递的终点处理事件    
                  new Subscriber<Bitmap>() {
                        @Override
                        public void onCompleted() {
                            Log.d("DDDDDD","结束观察...\n");
                        }

                        @Override
                        public void onError(Throwable e) {
                            //出现错误会调用这个方法
                        }
                        @Override
                        public void onNext(Bitmap s) {
                            //处理事件
                            showBitmap(s)
                        }
                    );

```
                
> 实际上在使用map操作时，new Func1() 就对应了类型的转变的方向，String是原类型，Bitmap是转换后的类型。在call()方法中，输入的是原类型，返回转换后的类型而进行读取文件，创建bitmap可能是一个耗时操作，那么就应该在子线程中执行，主线程应该仅仅做展示。那么线程切换一般就会是比较复杂的事情了。但是在Rxjava中，是非常方便的。

### 异步（线程调度）

>异步是相对于主线程来讲的子线程操作，在这里我们不妨使用线程调度这个概念更加贴切。

首先介绍一下RxJava的线程环境有哪些选项：

![image](http://i.imgur.com/EDirPvB.png)


使用代码解释更加简洁方便:
```java
    //new Observable.just()执行在新线程
    Observable.just(getFilePath())
    //指定在新线程中创建被观察者
        .subscribeOn(Schedulers.newThread())
    //将接下来执行的线程环境指定为io线程
        .observeOn(Schedulers.io())
    //map就处在io线程
        .map(mMapOperater)
    //将后面执行的线程环境切换为主线程，
    //但是这一句依然执行在io线程
        .observeOn(AndroidSchedulers.mainThread())
    //指定线程无效，但这句代码本身执行在主线程
       .subscribeOn(Schedulers.io())
    //执行在主线程
       .subscribe(mSubscriber);

```
#### 实际上线程调度只有subscribeOn（）和observeOn（）两个方法。对于初学者，只需要掌握两点：

>1.subscribeOn（）它指示Observable在一个指定的调度器上创建（只作用于被观察者创建阶段）。只能指定一次，如果指定多次则以第一次为准

>2.observeOn（）指定在事件传递（加工变换）和最终被处理（观察者）的发生在哪一个调度器。可指定多次，每次指定完都在下一步生效。

## 二.背压概念介绍

>RxJava是一个观察者模式的架构，当这个架构中被观察者(Observable)和观察者(Subscriber)处在不同的线程环境中时，由于者各自的工作量不一样，导致它们产生事件和处理事件的速度不一样，这就会出现两种情况：

>1.被观察者产生事件慢一些，观察者处理事件很快。那么观察者就会等着被观察者发送事件，（好比观察者在等米下锅，程序等待，这没有问题）。

>2.被观察者产生事件的速度很快，而观察者处理很慢。那就出问题了，如果不作处理的话，事件会堆积起来，最终挤爆你的内存，导致程序崩溃。（好比被观察者生产的大米没人吃，堆积最后就会烂掉）。
下面我们用代码演示一下这种崩溃的场景：

```java

    //被观察者在主线程中，每1ms发送一个事件
    Observable.interval(1, TimeUnit.MILLISECONDS)
    //将观察者的工作放在新线程环境中
    .observeOn(Schedulers.newThread())
    //观察者处理每1000ms才处理一个事件
    .subscribe(new Action1() {
        @Override
        public void call(Long aLong) {
           try {
                 Thread.sleep(1000);
              } catch (InterruptedException e) {
                 e.printStackTrace();
                          }
                Log.w("TAG","---->"+aLong);
                      }
                  });

```
#### 在上面的代码中，被观察者发送事件的速度是观察者处理速度的1000倍

这段代码运行之后：

```java
    ...
    Caused by: rx.exceptions.MissingBackpressureException
    ...
    ...
```
>抛出MissingBackpressureException往往就是因为，被观察者发送事件的速度太快，而观察者处理太慢，而且你还没有做相应措施，所以报异常.

>简而言之:背压是指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略,即背压是流速控制的一种策略。

有两点要强调的是：

>1.背压策略的一个前提是异步环境，也就是说，被观察者和观察者处在不同的线程环境中。

>2.背压（Backpressure）并不是一个像flatMap一样可以在程序中直接使用的操作符，他只是一种控制事件流速的策略。

那么我们再回看上面的程序异常就很好理解了，就是当被观察者发送事件速度过快的情况下，我们没有做流速控制，导致了异常。


#### 响应式拉取（reactive pull）

>由之前我们可以了解到，在RxJava的观察者模型中，被观察者是主动的推送数据给观察者，观察者是被动接收的。而响应式拉取则反过来，观察者主动从被观察者那里去拉取数据，而被观察者变成被动的等待通知再发送数据。其结构示意图如下:
![image](http://i.imgur.com/ZPJLdgl.png)

总的来说:

>1.背压是一种策略，具体措施是下游观察者通知上游的被观察者发送事件

>2.背压策略很好的解决了异步环境下被观察者和观察者速度不一致的问题

>3.在RxJava1.X中，同样是Observable，有的不支持背压策略，导致某些情况下，显得特别麻烦，出了问题也很难排查，使得RxJava的学习曲线变得十份陡峭。

## 三.Rxjava 2.0版本介绍及更新

首先要强调的一点是，RxJava以观察者模式为骨架，在2.0中依然如此。

不过此次更新中，出现了两种观察者模式：

>Observable(被观察者)/Observer（观察者）

>Flowable(被观察者)/Subscriber(观察者)


![image](http://i.imgur.com/1BB6uLF.png)

### RxJava2.X中，Observeable用于订阅Observer，是不支持背压的，而Flowable用于订阅Subscriber，是支持背压(Backpressure)的。

#### Observable/Observer(这种观察者模型是不支持背压的)
```java
Observable正常用法：

    Observable mObservable=Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onComplete();
            }
        });

    Observer mObserver=new Observer<Integer>() {
    //这是新加入的方法，在订阅后发送数据之前，
    //回首先调用这个方法，而Disposable可用于取消订阅
       @Override
        public void onSubscribe(Disposable d) {
            }
       @Override
       public void onNext(Integer value) {

            }

        @Override
        public void onError(Throwable e) {

        }
        @Override
        public void onComplete() {
            }
        };
        mObservable.subscribe(mObserver);

```
#### Flowable/Subscriber

```java
        Flowable.range(0,10)
        .subscribe(new Subscriber<Integer>() {
            Subscription sub;
            //当订阅后，会首先调用这个方法，其实就相当于onStart()，
            //传入的Subscription s参数可以用于请求数据或者取消订阅
            @Override
            public void onSubscribe(Subscription s) {
                Log.w("TAG","onsubscribe start");
                sub=s;
                sub.request(1);
                Log.w("TAG","onsubscribe end");
            }

            @Override
            public void onNext(Integer o) {
                Log.w("TAG","onNext--->"+o);
                sub.request(1);
            }
            @Override
            public void onError(Throwable t) {
                t.printStackTrace();
            }
            @Override
            public void onComplete() {
                Log.w("TAG","onComplete");
            }
        });

 ```       
        
>输出如下：

>onsubscribe start

>onNext--->0

>onNext--->1

>onNext--->2

>...

>onNext--->9

>onComplete

>onsubscribe end

#### 操作符相关

>这一块其实可以说没什么改动，大部分之前你用过的操作符都没变，即使有所变动，也只是包名或类名的改动。大家可能经常用到的就是Action和Function。

#### Action相关

>在1.0中，这类接口是从Action0，Action1...往后排（数字代表可接受的参数），现在做出了改动

>Rx1.0-----------Rx2.0

>Action0--------Action

>Action1--------Consumer

>Action2--------BiConsumer

>后面的Action都去掉了，只保留了ActionN

#### Function相关

同上，也是命名方式的改变

>上面那两个类，和RxJava1.0相比，他们都增加了throws Exception，也就是说，在这些方法做某些操作就不需要try-catch。

例如：
```java

    Flowable.just("file.txt")
    .map(name -> Files.readLines(name))
    .subscribe(lines -> System.out.println(lines.size()), Throwable::printStackTrace);

```
>Files.readLines(name)这类io方法本来是需要try-catch的，现在直接抛出异常，就可以放心的使用lambda ，保证代码的简洁优美。
