# AIDL

> 主讲人：梅静

## 前言
     
   * 在Android中每一个应用都拥有自己独立的jvm(java虚拟机)，都有其独立的内存地址空间,用于数据操作，但与其他应用不能直接进行通信，从而保证应用程序的数据安全性以及稳定性。

### 如何解决跨进程间的通信（两个应用之间进行数据通信）？

   * 采用AnoirdIPC机制实现进程间通信
   
   *  什么是IPC?    

           (Inter-Process Communication) 跨进程通信，是进程间/跨进程间进行数据交互的一个过程。

   * 在Android进行数据共享的方式要哪些?
   
     * socket   套接字  
   
            用于数据交互，可跨进程，支持多并发(多线程)，但需要网络支持
   
     * 共享文件SharedPreferences

           用于简单的数据交换，可跨进程，不支持用多并发（多线程） 

     *  intent的Bunde数据传递  
 
            用于bundle支持的数据交互,不可跨进程，不支持多并发

     *  BroadcastReceiver
     
            用于数据交互，可跨进程，支持多并发，但是不够安全，任意应用都可拦截

     *  ContentProvider
    
             用于数据交互，可跨进程（不直接与其他进程交互），支持多并发操作，底层基于binder
  
     *  基于Binder的Message
   
            用于bundle支持的数据交互，不可跨进程，对高并发的处理不太好
 
     *  基于Binder的AIDL
             
              用于数据交互，可跨进程（不直接与其他进程交互），支持多并发操作。   


      
   * 进程间可直接进行数据共享，跨进程可以直接进行数据共享吗?Why?  
     
      * 跨进程间不可直接进行数据共享

      * 所有运行在不同进程中的四大组件，只要通过内训共享数据都会失败，主要是多进程带来的影响。

     	     静态成员和单例模式完全失效（（由独立虚拟机造成））
	         线程同步机制完全失效（由独立虚拟机造成）
	         SharedPreferences的可靠性下降（存在并发读写的问题）
	         Application会多次创建（新的进程中又会导致进程所在的Application在新的虚拟机中再次创建）
    



    
 * 跨进程通信是如何实现的？
    
      * 采用Binder机制实现进程通信

 
## Binder是神马？ 
   
  * 定义 
    * 是Android系统中提供的一种进程间通信的机制，采用c/s架构模型，它提供了远程过程调用功能。这个机制有几部分完成，Client，Server，ServiceManager，bindDriver，bindDriver是binder的驱动程序，运行在Liunx内核空间（系统），Client，Service,ServiceManager运行在用户空间 

     * ServiceManager
    
           是一个Liunx级的进程，是Service的管理器，任何的Service在使用之前，均需要向ServiceManager注册，

           当客户端需要访问某个Service时，应该首先向SM查询该服务是否存在，若存在会将Service取出，返回给客户端一个Service的引用。

 

![](http://img0.ph.126.net/686W8rp1RTOBsMMmxbg3wA==/6632224451933151135.png)


说明：
* SM的注册过程：
   
  *   Server在自己的进程中向binder驱动程序申请创建一个作为自己Service的binder实体
  *    Binder驱动程序则会为这个Service创建位于内核中的binder实体节点和binder引用，然后将Service的名称和binder的引用传给ServiceManager，通知ServiceMnager注册相应的服务
  *    ServiceManger接收到数据之后，根据Service的名称和引用将其填写入一张表中。
  *    若Client向Service服务请求时，ServiceManager就会通过服务名称。查找到对应服务的binder引用返回给Client。
  
      
		  数据接收方（Client）：持有Binder应用
		  数据发送方 (Server):持有Binder实体   


* Binder的工作机制：

   * Client发送请求给	Server,在Server为返回结果前，Client处于阻塞状态，请求的数据和相应的操作Client的Proxy处理
   *  proxy代理接口的方法会将client传递的参数打包成为Parcel对象，发送给内核的binder driver（transact()）
   *  Binder driver将数据发送给Server，Server对数据进行解析，并根据相应的操作标识进行对应的处理。
   *  Server操作完成后，将执行之后的数据进行封装，然后将数据交给binder driver进行远程程序调度（onTransact()）
   *   binder driver(Binder驱动程序)将结果数据交给Client的Proxy代理，Proxy代理将数据解析后将结果返回给Client
   *   Client接收数据之后，结束阻塞状态、


*  数据共享

    * 数据的发送和接收是通过Binder驱动程序调度完成。
    * Binder驱动处理完发送方（Server）请求操作之后,会把数据写入到缓存区
    *  接收方（Client）会去读取缓存区的数据，，当读取到数据后，会将数据进行封装。在未读取到数据之前，接收方一直处于阻塞状态。
    

    接收方和发送方通过建立的公共缓存区，进行读写操作来进行数据共享的
    

## AIDL的定义

   * Android Interface Definition Language（Android 接口定义语言），定义了客户端与服务端的一个标准。
   
### 特点 

    优点：实现多应用之间进行跨进程通信  

    缺点：耗内存

 
#### 开发流程

* 1.定义Aidl文件(先在服务端定义，然后再客户端定义，两端保持一致,其实所有的跨进程对象传递都是对象的序列化与反序列化  所以必须包名一致)
* 2.服务端实现接口。
* 3.客户端调用接口

#### 语法规则 

* 1.语法与java的接口相似
* 2.只支持方法声明，不支持静态成员的声明
* 3.除了默认的基本数据类型外，其他的数据类型需要手动导包。

### 特别注意
* 1.AIDL Client请求后会被挂起

      当客户端发起远程请求时,由于当前线程会被挂起直至服务端进程返回数据，所以如果一个远程方法是耗时的，那么不能在ui线程发起远程请求。
* 2.除了基本数据,String类型以外；其他数据类型的参数必须标注：in/out/inout

		   in:输入型参数
		   out:输出型参数
		   inout：输入输出型参数

* 3.AIDL文件不是实现Binder的必须品

		我们完全可以不提供AIDL文件即可实现Binder，之所以提供AIDL文件，是为了方便系统为我们生成代码


* 4.Aidl所对应的java文件是自动生成的，外部不能修改




 

 ## 如何实现AIDL?下面我们根据几个例子来看一下（基于Android Studio开发）

 #### 实例1:有两个应用应用A（客户端），应用B（服务端）。应用B用于当获取到客户端传过来的数据后进行数据计算并将结果返回给客户端，提供远程计算功能；应用A用于输入数据，并将数据传给应用B，获取返回值进行界面更新，提供原始数据，并显示功能。
 ![](http://i.imgur.com/CtdtXzs.png)

 * 服务端 
  
 1.创建一个项目，之后右键项目的main文件夹，出现菜单选择Aidl选项，创建Aidl文件(类名要和文件名相同)。
  ![](http://i.imgur.com/LaMw23c.png)

   说明：在新建的文件中，会有自动生成的一个接口方法，这个就是向我们展示了他支持基本数据类型，当然它也支持其他的类型，将在后续说明。

```
	    /**
	     * Demonstrates some basic types that you can use as parameters
	     * and return values in AIDL.
	     */
	    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
	            double aDouble, String aString);
```



  2.我们将默认提供的方法删除，实现我们自己的接口方法

         //计算两个数据的和
        int   add(int num1,int num2);

![](http://i.imgur.com/j7s99VL.png)


  3.因为Android Studio对Aidl不能实现自动编译功能，所以写好之后aidl接口之后，需要将项目重新编译一下。编译之后我可以在项目的app\build\generated\source\aidl文件夹下查看到aidl文件所对应的java文件，它类似与R文件，我们无需创建，系统将自动为我们创建，从而使我们可以调用。
![](http://i.imgur.com/ZpoTlu5.png)



  4.当我们写好aidl之后，我们需要实现其接口。那么如何让客户端去访问呢?那我们就需要Service的IBinder，因为当客户端绑定这个服务之后IBinder才会被执行，此时客户端则可以访问服务端了。

![](http://i.imgur.com/O57oOya.png)


  5.我们需要将实现aidl接口的服务在清单文件注册。

```xml
        <service android:name=".ICalculateService"
            android:enabled="true"
            android:exported="true">
        </service>
```

## 说明：注册该服务一定要设置这两个属性，否则客户端运行时将绑定不到服务，从而可能出现异常。
 ```    
    android:enabled="true"  设置该服务能否被实例化 默认值为  true  true：可以被实例化  false:不允许被实例化

    android:exported="true" 设置该服务能否被其他应用组件调用或进行交互，其默认值依赖于该服务的包含的过滤器。若没有过滤器，则该服务只能被应用程序内部调用 此时默认值为false；若包含过滤器，则该服务可以被外部程序所调用。
```     

* 客户端

1.创建一个新的项目，实现其界面展示。

![](http://i.imgur.com/kSb5s0s.png)

2.将服务端的aidl包拷贝到客户端中来，注意包名，类名，文件名一定要与服务端的完全一致。

![](http://i.imgur.com/WXbWbpT.png)


  3.因为Android Studio对Aidl不能实现自动编译功能，所以写好之后aidl接口之后，需要将项目重新编译一下。编译之后我可以在项目的app\build\generated\source\aidl文件夹下查看到aidl文件所对应的java文件，它类似与R文件，我们无需创建，系统将自动为我们创建，从而使我们可以调用。（与服务端一样）


  4.客户端绑定服务端的服务。

 ### 注意: Android5.0以后,android不允许通过隐示视图启动服务，必须通过显示意图才可启动服务。

```java
		    private String TAG="AidlCalculate";
		    private CalculateAidl mCalculateAidl;
		    private ServiceConnection connectionm=new ServiceConnection() {
		
		        /**
		         * 当服务连接后调用
		         * @param componentName
		         * @param iBinder
		         */
		        @Override
		        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
		            Toast.makeText(AidlCommunicateToCalculateActivity.this,"连接到远程服务",Toast.LENGTH_SHORT);
		            Log.i(TAG,"连接到远程服务");
		            mCalculateAidl= CalculateAidl.Stub.asInterface(iBinder);
		            //在客户端绑定远程服务之后，给Binder设置死亡代理：
		            try {
		                iBinder.linkToDeath(mDeathRecipient, 0);
		            } catch (RemoteException e) {
		                e.printStackTrace();
		                Log.i(TAG,"给Binder设置死亡代理失败:"+e.getMessage());
		            }
		        }
		
		        /**
		         * 当服务端开后实现
		         * @param componentName
		         */
		        @Override
		        public void onServiceDisconnected(ComponentName componentName) {
		            Toast.makeText(AidlCommunicateToCalculateActivity.this,"与远程服务断开连接",Toast.LENGTH_SHORT);
		            Log.i(TAG,"与远程服务断开连接");
		            //资源回收
		            mCalculateAidl=null;
		
		        }
		    };
		
		   /*
		   * 当远程服务挂掉后调用
		   *
		   */
		    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
		        @Override
		        public void binderDied() {
		            // TODO: 这里重新绑定远程Service。
		            if(mCalculateAidl == null)
		                return;
		            mCalculateAidl.asBinder().unlinkToDeath(mDeathRecipient, 0);
		            mCalculateAidl = null;
		
		        }
		    };


		    /**
		     * 绑定服务
		     * 采用显示意图调用   需要  包名    包名+类名
		     */
		    private void bindService() {
		        Intent intent=new Intent();
		        intent.setComponent(new ComponentName("com.project.aidltocalculate","com.project.aidltocalculate.ICalculateService"));
		        bindService(intent,connectionm, Service.BIND_AUTO_CREATE);
		        Log.i(TAG,"绑定服务");
		    }
		
		
		    @Override
		    protected void onDestroy() {
		        super.onDestroy();
		        unbindService(connectionm);
		    }

```

5.传递数据给服务端，并获取返回结果，更新界面。
	
```java

	   @Override
	    public void onClick(View view) {
	        switch (view.getId()) {
	            case R.id.btn_add:
	                 String result1=etNum1.getText().toString();
	                 String result2=etNum2.getText().toString();
	                if(TextUtils.isEmpty(result1)){
	                    result1="0";
	                }
	                if(TextUtils.isEmpty(result2)){
	                    result2="0";
	                }
	                 int num1=Integer.valueOf(result1);
	                 int num2=Integer.valueOf(result2);
	                try {
	                   if(mCalculateAidl!=null) {
	                       int data = mCalculateAidl.add(num1, num2);
	                       tvData.setText("数据之和:" + data);
	                   }else {
	                       Toast.makeText(MainActivity.this," 与远程服务断开连接",Toast.LENGTH_SHORT);
	                   }
	                } catch (RemoteException e) {
	                    Log.i(TAG,"没有连接到服务:"+e.getMessage());
	                    Toast.makeText(MainActivity.this,"没找到该服务",Toast.LENGTH_SHORT);
	                }

 ```

6.运行服务端和客户端，进行测试

###   服务端：
![](http://i.imgur.com/Ktq3Mfj.png)


###  客户端
![](http://i.imgur.com/dHMyO1i.png)


###  说明: 
1.客户端访问服务端的前提是服务端应用要在客户端应用之前启用，否则将无法连接到服务；同时客户端能够调用服务端的方法，是因为客户端和服务端带运行中，一旦服务端被系统kill后，客户端将与其失去连接，从而无法服务访问器数据;

2.Binder运行在服务端进程，如果服务端进程由于某些原因异常终止，这个时候我们到服务端的Binder连接断裂，会导致我们的远程调用失败。Binder提供了两个配对的方法linkToDeath和unlinkToDeath，通过linkToDeath我们可以给Binder设置一个死亡代理，当Binder死亡时，我们会收到通知，这个时候我们就可以重新发起连接请求从而恢复连接。



* AIDL支持的数据类型：
   
   *  基本类型：  
   
               Byte  Int  Long  Float  Double    Boolean   Char
               String
               List集合 

   * 其他
               
              支持实现Parcelable的实体数据    
   

 ### 说明： 1.Aidl只支持方法，不能定义静态成员；对于其他数据类型需要导包</br>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 2.对于基本数据类型，aidl不支持short类型。</br>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3.对于集合支持List的集合，不支持Map集合,必须要指明是输入端（in输入型参数 ）/输出端（ou输出型参数）/inout:输入输出型参数</br>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4.支持实体数据，但是该实体必须实现Parcelable接口，不支持Serializable，必须要指明是输入端（in输入型参数 ）/输出端（ou输出型参数）/inout:输入输出型参数</br>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;5. 对于List集合，必须要指明其实输入端还是输出端，因为当客户端将集合数据传递给服务端的过程实际上传递给系统底层，系统底层对于数据操作是最基本的，所以需要将数据拆分，这个过程叫打包，当系统根据客户端端的请求找到服务端将数据给服务端是，需要将数据组合成原始数据，这个过程叫拆包、为了保证不过多小号内存，我们必须指明数据的是输入端（in输入型参数 ）/输出端（ou输出型参数）/inout:输入输出型参数，否则会报错。


  错误写法：

```java
			// BaseDataAidlInterface.aidl
			package com.project.aidltocalculate;
			
			// Declare any non-default types here with import statements
			
			interface BaseDataAidlInterface {
			    /**
			     * Demonstrates some basic types that you can use as parameters
			     * and return values in AIDL.
			     */
			    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
			            double aDouble, String aString,List<String> list);
			}
```

报错异常如下图：

![](http://i.imgur.com/2nd6idt.png)


正确写法：in List<String> list   标识这个是输入端

```java
		// BaseDataAidlInterface.aidl
			package com.project.aidltocalculate;
			
			// Declare any non-default types here with import statements
			
			interface BaseDataAidlInterface {
			    /**
			     * Demonstrates some basic types that you can use as parameters
			     * and return values in AIDL.
			     */
			    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
			            double aDouble, String aString,in List<String> list);

			}

```

 #### 实例2:有两个应用应用A（客户端），应用B（服务端）。应用B用于当获取到客户端传过来的数据后进行组合后将结果返回给客户端，提供数据重新组合功能；应用A用于将数据传给应用B，获取返回值进行界面更新，提供原始数据，并显示功能。（基于实例1）

 * 服务端 
1.在项目中新建一个实体类，实现Parcelable接口
![](http://i.imgur.com/YVmGj1s.png)


2.在Aidl文件加载下新建一个与实体类同名的aidl文件，用于向aidl描述该对象是实体类并实现了Parcelable接口

![](http://i.imgur.com/LDjNeLl.png)


3.在Aidl文件夹下新建一个aidl的接口文件，注意对于list集合和实体数据必须指明参数输入输出类型，否则无法编译通过

![](http://i.imgur.com/BmbCP8G.png)

4.对项目进行编译，之后实现其接口实现服务。

![](http://i.imgur.com/FMn5ppb.png)

5.在清单文件中注册该服务

```
        <service android:name=".IDataTypeService"
            android:enabled="true"
            android:exported="true">
        </service>
```

 * 客户端
 
1.创建一个新的项目，实现其界面展示。

![](http://i.imgur.com/vCknac7.png)

2.将服务端的aidl包拷贝到客户端中来，注意包名，类名，文件名一定要与服务端的完全一致。

 ![](http://i.imgur.com/KTtkCGk.png)



  3.因为Android Studio对Aidl不能实现自动编译功能，所以写好之后aidl接口之后，需要将项目重新编译一下。编译之后我可以在项目的app\build\generated\source\aidl文件夹下查看到aidl文件所对应的java文件，它类似与R文件，我们无需创建，系统将自动为我们创建，从而使我们可以调用。（与服务端一样）


  4.客户端绑定服务端的服务。

```java

			     private String TAG="DataType";
			    private BaseDataAidlInterface mCalculateAidl;
			    private ServiceConnection connectionm=new ServiceConnection() {
			
			        /**
			         * 当服务连接后调用
			         * @param componentName
			         * @param iBinder
			         */
			        @Override
			        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
			            Toast.makeText(AidlCommunicateToDataTypeActivity.this,"连接到远程服务",Toast.LENGTH_SHORT);
			            Log.i(TAG,"连接到远程服务");
			            mCalculateAidl= BaseDataAidlInterface.Stub.asInterface(iBinder);
			            //在客户端绑定远程服务之后，给Binder设置死亡代理：
			            try {
			                iBinder.linkToDeath(mDeathRecipient, 0);
			            } catch (RemoteException e) {
			                e.printStackTrace();
			                Log.i(TAG,"给Binder设置死亡代理失败:"+e.getMessage());
			            }
			
			        }
			
			        /**
			         * 当服务端开后实现
			         * @param componentName
			         */
			        @Override
			        public void onServiceDisconnected(ComponentName componentName) {
			            Toast.makeText(AidlCommunicateToDataTypeActivity.this,"与远程服务断开连接",Toast.LENGTH_SHORT);
			            Log.i(TAG,"与远程服务断开连接");
			            //资源回收
			            mCalculateAidl=null;
			
			        }
			    };
			
			    /*
			     * 当远程服务挂掉后调用
			     *
			      */
			    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
			        @Override
			        public void binderDied() {
			            // TODO: 这里重新绑定远程Service。
			            if(mCalculateAidl == null)
			                return;
			            mCalculateAidl.asBinder().unlinkToDeath(mDeathRecipient, 0);
			            mCalculateAidl = null;
			
			        }
			    };

		
		    /**
		     * 绑定服务
		     * 采用显示意图调用   需要  包名    包名+类名
		     */
		    private void bindService() {
		        Intent intent=new Intent();
		        intent.setComponent(new ComponentName("com.project.aidltocalculate","com.project.aidltocalculate.IDataTypeService"));
		        bindService(intent,connectionm, Service.BIND_AUTO_CREATE);
		        Log.i(TAG,"绑定服务");
		    }
		
		
		    @Override
		    protected void onDestroy() {
		        super.onDestroy();
		        unbindService(connectionm);
		    }

```

**
    5.传递数据给服务端，并获取返回结果，更新界面。

```java

	    @Override
	    public void onClick(View view) {
	        switch (view.getId()){
	            case R.id.btn_get_remote_data_way1:
	                try {
	                    if(mCalculateAidl!=null) {
	                        List<String>  list=new ArrayList<>();
	                        list.add("list的集合");
	                        String data = mCalculateAidl.basicTypes(1,99999,true,3.14f,0.9999999,"字符串数据",list,"aa");
	                        tvWay1.setText("数据之和:" + data);
	                    }else {
	                        Toast.makeText(AidlCommunicateToDataTypeActivity.this," 与远程服务断开连接",Toast.LENGTH_SHORT);
	                    }
	                } catch (RemoteException e) {
	                    Log.i(TAG,"没有连接到服务:"+e.getMessage());
	                    Toast.makeText(AidlCommunicateToDataTypeActivity.this,"没找到该服务",Toast.LENGTH_SHORT);
	                }
	
	                break;
	            case R.id.btn_get_remote_data_way2:
	                try {
	                if(mCalculateAidl!=null) {
	                    Person person=new Person();
	                    person.setAge(21);
	                    person.setName("张三");
	                    List<Person> result = mCalculateAidl.addPersonData(person);
	                    tvWay2.setText("数据之和:" + result.toString());
	                }else {
	                    Toast.makeText(AidlCommunicateToDataTypeActivity.this," 与远程服务断开连接",Toast.LENGTH_SHORT);
	                }
	        } catch (RemoteException e) {
	            Log.i(TAG,"没有连接到服务:"+e.getMessage());
	            Toast.makeText(AidlCommunicateToDataTypeActivity.this,"没找到该服务",Toast.LENGTH_SHORT);
	        }
	
	        break;
	
	        }
	    }

```

   6.运行和测试
![](http://i.imgur.com/dDCQvRl.png)

## AIDL的底层实现原理解析


 ![](http://img1.ph.126.net/lERyHsQ9MvF9nKBjuc-oHQ==/6632468543514492859.png)


### 说明：

1.服务端创建Aidl。编译生成所对应的java文件

2.生成的java文件其实是一个接口A，继承了android.os.Iinterface接口，里面声明实现了asBinder()方法，用于返回一个binder对象

3.接口A里面有一个内部抽象类（Stub）继承类android.os.Binder,同时实现了接口A。

4.当我们在service中 new  这个抽象类（Stub）时，其构造函数调用了`this.attachInterface(this, DESCRIPTOR)`方法，将该接口A的实例存储到底层中。

5.当客户端调用asinterface()时，会将服务端Binder的对象转换成客户端所需的AIDL接口类型的对象，但是转换的过程是区分进程的，若客户端和服务端处于同一进程，那么此方法将返回从obj.queryLocalInterface()中获取对象，否则返回是系统封装后的proxy代理对象
      
6.客户端获取实例后，调用方法进行数据传递时，该方法运行在客户端，首先会创建所需要的输入型Parcel对象_date，输出型Parcel对象_reply以及需要返回的结果对象，然后将客户端传入的数据写入到_data中，接着调用transact方法发起RPC（远程过程调用）请求，同时当前线程挂起（处于阻塞状态）；这时候服务端的onTransact()方法会被调用，知道RPC过程返回，当前继续执行，并从_reply中取出RPC过程返回的结果给客户端

7.当服务端调用onTransact()时,该方法运行在服务端的Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层分装好后交由此方法处理，服务端通过code可以确定客户端所应求的目标方法是什么，然后从_data中取出目标方法所需要的参数，之后执行目标方法，当目标执行完成后向_reply写入数据，并返回boolean确定这次请求是否成功。


 ## 参考链接：
 
- [http://blog.csdn.net/zizidemenghanxiao/article/details/50341773](http://blog.csdn.net/zizidemenghanxiao/article/details/50341773)
- [http://blog.csdn.net/coding_glacier/article/details/7520199](http://blog.csdn.net/coding_glacier/article/details/7520199 "Android Binder机制")
- [ https://juejin.im/entry/58257bd28ac247004f3a5129]( https://juejin.im/entry/58257bd28ac247004f3a5129 "binder机制")



 



 




 




  

 






   

