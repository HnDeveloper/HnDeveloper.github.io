>主讲人：许佳辉  

# **设计原理+设计模式**

## 1.1单一原则

### 一个类一种职责，避免重复 
假如：
一个适配器用来加载首页简单的数据列表，但有多个类似的。如果是直接复制的话，一旦修改，就会容易陷入重复逻辑，一个地方更新代码，很容易忘记更新另一个地方的代码。

### 一个类包含很多种职责会容易引发各种问题。
例如：
适配器会把各种职责分类，获取数目的，获取布局的，配置数据的，配置类型的。如果把所有的代码写在了一起，
1. 找起来不方便
2. 代码累赘难以区分
3. 3如果修改多次的话，容易把代码混淆在一起，有些方法是需要注意先后顺序这样容易出问题。

应用：加载布局的就独立一个加载布局的方法，不要加一个布局填写一个布局。如果布局多了，修改次数频繁，容易出现问题。还有一些配置、和事件处理等等。



----------

## 1.2闭合原则

对个一个类添加新功能是封闭的，通过接口形式去调用扩展

### 问题由来： 
在软件的生命周期内，因为变化、升级和维护等原因需要对软件原有代码进行修改时，可能会给旧代码中引入错误，也可能会使我们不得不对整个功能进行重构，并且需要原有代码经过重新测试。
 
### 解决方案：
当软件需要变化时，尽量通过扩展软件实体的行为来实现变化，而不是通过修改已有的代码来实现变化。

### 通俗的说：新建模块添加功能，避免影响其他代码。

>理解：对于与扩张是开放的，对于修改是封闭的。

例如：一个获取图片的方法的方法，需要判断哪一种缓存缓存然后获取图片。
如果有很多很多缓存机制的话，不断通过if/swith等判断获取，代码会变得很累赘
1. 这时我们可以通过一个抽象类或者抽象方法实现生成图片这个公共方法。然后每一个种缓存类实现。
2. 获取图片方法就无需通过判断，直接把新建的父类/接口传进去实现方法即可。这就叫避免影响其他代码。
3. 其他子类可以实现不同的方法。这就叫新建模块添加功能。


----------

## 1.3里氏替换原则

子类可以扩展父类的功能，但不能改变父类原有的功能。

1.原本是这样的

```java
	class A{  
	    public int func1(int a, int b){  
	        return a-b;   //进行减法
	    }  
	}  
  
	public class Client{ 
	    public static void main(String[] args){  
	        A a = new A();  
	        System.out.println("100-50="+a.func1(100, 50));  //50
	        System.out.println("100-80="+a.func1(100, 80));  //20
	    }  
} 
```

2.我们需要增加一个新的功能：完成两数相加，然后再与100求和，由类B来负责。
```java
	class B extends A{  
	    public int func1(int a, int b){  //重载
	        return a+b;  //问题出现在此，修改了父类的方法
	    }  
	}  

	public class Client{  
	    public static void main(String[] args){  
	        B b = new B();  
	        System.out.println("100-50="+b.func1(100, 50));  // 100-50 = 150
	        System.out.println("100-80="+b.func1(100, 80));  // 100-80 = 180
	    }  
	} 
```
子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。


----------



## 1.4接口分离原则

接口可以通过多个接口分离，然后通过继承等方式减少方法体的累赘。
接口分离是典型的解耦方式
```java
	interface 接口{  
	    public void 接口方法1();  
	    public void 接口方法2();  
	    public void 接口方法3();   
	}  

	class A{  
	    public void 方法A1(){  
	        接口.接口方法1();  
	    }  
	    public void 方法A2(){  
	  	接口.接口方法2();
	    } 
	}  

	class B implements 接口{
	    @Override  
	    public void 接口方法1() {  
	 		方法B1();
	    }  
	    @Override  
	    public void 接口方法2() {  
	 		方法B2();
	    }  
	    //对于类B来说， 接口方法3不是必需的，但是由于接口A中有这两个方法，  
	    @Override  
	    public void 接口方法3() {  
	     ......
	    }  
	
	    public void 方法B1() {  
	        System.out.println("方法A1的实现");  
	    }  
	    public void 方法B2() {  
	        System.out.println("方法A2的实现");  
	    }  
	}  
```

A类直接调用B类方法，耦合度高。删除B类A直接报错，但通过接口分离的话，删除B类对A是没任何影响。

如果有C和D类去通过I进行实现的话，会有多余的接口

客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。

### 问题由来：
类A通过接口I依赖类B，
类C通过接口I依赖类D，如果接口I对于类A和类B来说不是最小接口，则类B和类D必须去实现他们不需要的方法。 

解决方案：将臃肿的接口I拆分为独立的几个接口，类A和类C分别与他们需要的接口建立依赖关系。也就是采用接口隔离原则。 

通俗的说：避免不需要的接口，建立接口类继承除去多余不用的接口。


----------


## 1.5迪米特法则 

场景 A认识B  ,  B认识C  现在A需要解决一件事，但是只能C完成
```java
	public class A{
		public String name;
		public A(String name){
			this.name = name;
		}
		public B getB(String name){
			return new B(name);
		}
		public void work(){
			B b = getB("李四");
			C c = b.getC("王五");
			c.work();
		}
		public void work(){ //使用了迪米特
			B b = getB("李四");
			b.work();
		}
	}
	
	public class B{
		private String name;
		publicB(String name){
			this.name = name;
		}
		public C getC(String name){
			return new C(name);
		}
	public  void  work(){
			C c = getC("王五");
			c.work();
		}
	}
	
	public classC{
		public String name;
		public C (String name){
			this.name = name;
		}
		public  void  work(){
			System.out.println(name + "把这件事做好了");
		}
	}

	public class  Client{
		public  static  void  main(String[] args){
			A a = new A("张三");
			a.work(); 
		}
	}
```
就是常见的getXXX().getXXX().getXXX()，那就考虑下是不是违反迪米特法则

 一个对象应该对其他对象保持最少的了解。 
因为A是不认识C的，怎么知道C里面是什么方法。

### 问题由来：
类与类之间的关系越密切，耦合度越大，当一个类发生改变时，对另一个类的影响也越大。

### 解决方案：
尽量降低类与类之间的耦合。


----------


## 1.6依赖倒置原则 

通过接口来调用方法而不是直接调用，将细节方法抽象出来。
在开闭原则也可以使用，就是将方法抽取做一个公共的接口，其他子类去实现

```java
	class Book{  
	    public String getContent(){  
	        return "很久很久以前有一个阿拉伯的故事……";  
	    }  
	}  
	class Mother{  
	    public void narrate(Book book){  
	        System.out.println("妈妈开始讲故事");  
	        System.out.println(book.getContent());  
	    }  
	}  
	public class Client{  
	    public static void main(String[] args){  
	        Mother mother = new Mother();  
	        mother.narrate(new Book());  
	    }  
	}
```
妈妈开始讲故事
很久很久以前有一个阿拉伯的故事……

现在多了一份报纸，上面妈妈只会读书
```java
	class Newspaper{  
	    public String getContent(){  
	        return "林书豪38+7领导尼克斯击败湖人……";  
	    }  
	}
	定义一个接口
	interface IReader{  
	    public String getContent();  
	} 
	让这两个类实现
	class Newspaper implements IReader {  
	    public String getContent(){  
	        return "林书豪17+9助尼克斯击败老鹰……";  
	    }  
	}  
	class Book implements IReader{  
	    public String getContent(){  
	        return "很久很久以前有一个阿拉伯的故事……";  
	    }  
	}  
	
	public class Client{  
	    public static void main(String[] args){  
	        Mother mother = new Mother();  
	        mother.narrate(new Book());  
	        mother.narrate(new Newspaper());  
	    }  
	}  
```
妈妈开始讲故事
很久很久以前有一个阿拉伯的故事……
妈妈开始讲故事
林书豪17+9助尼克斯击败老鹰……

高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。
 
### 问题由来：
类A直接依赖类B，假如要将类A改为依赖类C，则必须通过修改类A的代码来达成。这种场景下，类A一般是高层模块，负责复杂的业务逻辑；类B和类C是低层模块，负责基本的原子操作；假如修改类A，会给程序带来不必要的风险。

### 解决方案：
将类A修改为依赖接口I，类B和类C各自实现接口I，类A通过接口I间接与类B或者类C发生联系，则会大大降低修改类A的几率。



----------

## 2.1Build模式

链式构建结构，结构清晰。

//显示基本的AlertDialog  
```java
	    private void showDialog(Context context) {  
	        AlertDialog.Builder builder = new AlertDialog.Builder(context);  
	        builder.setIcon(R.drawable.icon);  
	        builder.setTitle("Title");  
	        builder.setMessage("Message");  
	        builder.setPositiveButton("Button1",  
	                new DialogInterface.OnClickListener() {  
	                    public void onClick(DialogInterface dialog, int whichButton) {  
	                        setTitle("点击了对话框上的Button1");  
	                    }  
	                });  
	        builder.setNeutralButton("Button2",  
	                new DialogInterface.OnClickListener() {  
	                    public void onClick(DialogInterface dialog, int whichButton) {  
	                        setTitle("点击了对话框上的Button2");  
	                    }  
	                });  
	        builder.setNegativeButton("Button3",  
	                new DialogInterface.OnClickListener() {  
	                    public void onClick(DialogInterface dialog, int whichButton) {  
	                        setTitle("点击了对话框上的Button3");  
	                    }  
	                });  
	        builder.create().show();  // 构建AlertDialog， 并且显示
	    } 
```

一个类需要很多参数，如一个对话框需要设置内容问题，这时我们可以直接通过构造函数传入内容文本，然后进行配置。这时有个需求，确认文本需要更换，难道我们还是通过构造函数传入？如果还有更多东西要改的话，这时构造函数会变得非常多，看起来不知道那个才是需要用到。

```java
	Person p5=new Person("赵六",17,170,65.4);

	举例build设置模式       
	                                                                          
	Person.Builder builder=new Person.Builder();
	Person person=builder
			.name("张三")
			.age(18)
			.height(178.5)
			.weight(67.4)
			.build();
	person.run();
	person.eat();
```
这时Builder是作为人的基本配置，而Person可以做eat,sleep,run等动作，也就是有人称为设计者，而Builder是作为搬运工，做基础配置。如果是按照规范的话，最好使用build。如果构造不多的话就可

```java
	new Person() = new  Person()
		.name("张三")
		.age(18)
		.height(178.5)
		.weight(67.4) 
		.run();
```			
这就是要build的原因。		


----------


## 2.2原型模式

用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。

```java
public class Person{
    private String name;
    private int age;
    private double height;
    private double weight;
     .....忽略部分
}
```
1.实现Cloneable接口
public class Person implements Cloneable{
......忽略部分
2.重写Object的clone方法

```java
@Override
public Object clone(){
3.实现clone方法中的拷贝逻辑
Person person=null;
	try {
		person=(Person)super.clone();
		person.name=this.name;
		person.weight=this.weight;
		person.height=this.height;
		person.age=this.age;
	} catch (CloneNotSupportedException e) {
		e.printStackTrace();
	} 
	return person;
}
}
```

实现流程
```java
	public class Main {
	    public static void main(String [] args){
	        Person p=new Person();
	        p.setAge(18);
	        p.setName("张三");
	        p.setHeight(178);
	        p.setWeight(65);
	        System.out.println(p);
	
	        Person p1= (Person) p.clone();
	        System.out.println(p1);
	
	        p1.setName("李四");
	        System.out.println(p);
	        System.out.println(p1);
	    }
	}

	Person{name=’张三’, age=18, height=178.0, weight=65.0}
	Person{name=’张三’, age=18, height=178.0, weight=65.0}
	Person{name=’张三’, age=18, height=178.0, weight=65.0}
	Person{name=’李四’, age=18, height=178.0, weight=65.0} 并不会影响原本的对象
```
Intent类，该类也实现了Cloneable接口。OkHttp也应用了原型模式。

```java
	@Override
	public Object clone() {
		return new Intent(this);
	}
	
	public Intent(Intent o) {
		this.mAction = o.mAction;
		this.mData = o.mData;
		this.mType = o.mType;
		this.mPackage = o.mPackage;
		this.mComponent = o.mComponent;
		this.mFlags = o.mFlags;
		this.mContentUserHint = o.mContentUserHint;
		if (o.mCategories != null) {
			this.mCategories = new ArraySet<String>(o.mCategories);
		}
		if (o.mExtras != null) {
			this.mExtras = new Bundle(o.mExtras);
		}
		if (o.mSourceBounds != null) {
			this.mSourceBounds = new Rect(o.mSourceBounds);
		}
		if (o.mSelector != null) {
			this.mSelector = new Intent(o.mSelector);
		}
		if (o.mClipData != null) {
			this.mClipData = new ClipData(o.mClipData);
		}
	}

```

----------


## 2.3 单例模式
```java
	public class Singleton {
	    private static volatile Singleton instance = null;
	
	    private Singleton(){
	    }
	 
	    public static Singleton getInstance() {
	        if (instance == null) { 
	            synchronized (Singleton.class) {
	                if (instance == null) {
	                    instance = new Singleton();
	                }
	            }
	        }
	        return instance;
	    }
	}
```

1. 必须防止外部可以调用构造函数进行实例化，因此构造函数必须私有化。
2. 必须定义一个静态函数获得该单例
3. 单例使用volatile修饰
4. 使用synchronized 进行同步处理，并且双重判断是否为null，我们看到synchronized (Singleton.class)里面又进行了是否为null的判断，这是因为一个线程进入了该代码，如果另一个线程在等待，这时候前一个线程创建了一个实例出来完毕后，另一个线程获得锁进入该同步代码，实例已经存在，没必要再次创建，因此这个判断是否是null还是必须的。 


比如EventBus中的单例

```java
	
	private static volatile EventBus defaultInstance;
	public static EventBus getDefault() {
		if (defaultInstance == null) {
			synchronized (EventBus.class) {
				if (defaultInstance == null) {
					defaultInstance = new EventBus();
				}
			}
		}
		return defaultInstance;
	}
```
当这个类的对象在多个地方创建的时候，使得内部的方法多次调用，但是我们希望只要一个对象操作这个方法，或者不希望多个地方同时调用这个方法，我们需要保持这个方法的单一性质，我们就用单利模式吧。



----------


## 2.4 工厂模式 + 2.5 抽象工厂模式


简单案例：

原本是这样

```java
	Client client = new Client();
	后来改成这样
		public class Factory {
			public static Client getClient() {
				return new Client();
			}
		}
	Client client = Factory.getClient();

	换个实际案例：

	public abstract class Fruits {
		public abstract void color();
		public abstract void weight();
	}

	//实现类 

	public class Banana extends Fruits {
		@Override
		public void color() {
			System.out.println("Banana is yellow");
		}
		@Override
		public void weight() {
			System.out.println("Weight 0.3kg");
		}
	}
	public class Cucumber extends Fruits {
		....同上
	}
	public class Sugarcane extends Fruits {
	....同上
	}
	//没用到工厂模式
	public class Client {
		public static void main(String[] args) {
			Fruits banana = new Banana();
			banana.color();
			banana.weight();
			
			Fruits cucumber = new Cucumber();
			cucumber.color();
			cucumber.weight();
			
			Fruits sugarcane = new Sugarcane();
			sugarcane.color();
			sugarcane.weight();
		}
	}
	//工厂模式
	public class Grower {
		public <T extends Fruits> T getFruits(Class<T> clz) {
			try {
				Fruits fruits = (Fruits) Class.forName(clz.getName()).newInstance();//反射
				return (T) fruits;
			} catch (Exception e) {
				return null;
			}
		}
	}

	public class Client {
		public static void main(String[] args) {
			Grower grower = new Grower();//这里没有抽象其工厂类，实际上可以继续抽象
	
			Fruits banana = grower.getFruits(Banana.class);
			banana.color();
			banana.weight();
	
			Fruits cucumber = grower.getFruits(Cucumber.class);
			cucumber.color();
			cucumber.weight();
	
			Fruits sugarcane = grower.getFruits(Sugarcane.class);
			sugarcane.color();
			sugarcane.weight();
		}
	}
```


Set，List是抽象工厂
HashSet,ArraryList是实体工厂
Iterator是抽象产物
ArrayListIterator 是实体产物

1、 关于ArrayList，HashSet，与 Iterator 之间都能算是一种工厂方法；

```java
	Set<string> set = new HashSet<string>();
	Iterator<string> iterator = set.iterator();
	while(iterator.hasNext()){
	    // 具体操作
	}
	List<string> list =new ArrayList<string>();
	Iterator<string> it = list.iterator();
	while(it.hasNext()){
	    // 具体操作
	}
```

2.其中List和Set都是工厂接口

```java
	public interface Set<e> extends Collection<e>{
	    public Iterator<e> iterator();
	}
	public interface List<e> extends Collection<e>{
	    public Iterator<e> iterator();
	}
```
3.ArrayList与HashMap 都有关于其中的实现了。

```java
	public abstract class AbstractList extends AbstractCollection implements List {.....}
	public class ArrayList<e> extends AbstractList<e> 
	implements Cloneable, Serializable, RandomAccess{
	 
	      public Iterator<E> iterator() {//重写其抽象类
	             return new Itr();
	         }
	
		private class Itr implements Iterator<E> {
		    protected int limit = ArrayList.this.size;
		    int cursor; 
		    int lastRet = -1;
		    int expectedModCount = modCount;
		    public boolean hasNext() {
		        return cursor < limit;
		    }
		    @SuppressWarnings("unchecked")
		    public E next() {
		      ..........
		    }
		    public void remove() {
		       ..........
		    }
		    @Override
		    @SuppressWarnings("unchecked")
		    public void forEachRemaining(Consumer<? super E> consumer) {
		        .....
		    }
	
	           .........
	}
```


----------


## 3.1 适配器模式 

目标接口（Target）：普通插头
需要适配的类（Adaptee）：欧洲插座
适配器（Adapter）：转换插
```java

// 已存在的、具有特殊功能、但不符合我们既有的标准接口的类  

	class Adaptee {  
	    public void specificRequest() {  
	        System.out.println("被适配类具有 特殊功能...");  
	    }  
	}  

// 目标接口，或称为标准接口  

	interface Target {  
	    public void request();  
	}  

// 具体目标类，只提供普通功能  

	class ConcreteTarget implements Target {  
	    public void request() {  
	        System.out.println("普通类 具有 普通功能...");  
	    }  
	}  

// 适配器类，继承了被适配类，同时实现标准接口  

	class Adapter extends Adaptee implements Target{  
	    public void request() {  
	        super.specificRequest();  
	    }  
	} 

// 测试类

	public class Client {  
		public static void main(String[] args) {  
	        // 使用普通功能类  
	        Target concreteTarget = new ConcreteTarget();  
	        concreteTarget.request();  
	        // 使用特殊功能类，即适配类  
	        Target adapter = new Adapter();  
	        adapter.request();  
	    }  
	}  
	
//另一种方式

	class Adapter implements Target{  
	    private Adaptee adaptee;  
	    public Adapter(Adaptee adaptee)  {
			this.adptee = adptee; 
	    }
	    public void request() {  
	        adptee .specificRequest();  //原理是一样的
	    }  
	} 

```
----------


## 3.2 桥接模式

	

如果直接新建实体对象的话需要建12个



这种写法相当于 7个类



但为了设计需求，通常添加接口来

```java
	public abstract class Shape {
	    Color color;
	    public void setColor(Color color) {
	        this.color = color;
	    }
	    public abstract void draw();
	}
	
	public class Square extends Shape{
	    public void draw() {
	        color.bepaint("正方形");
	    }
	}
	......
	public interface Color {
	    public void bepaint(String shape);
	}
	
	public class White implements Color{
	    public void bepaint(String shape) {
	        System.out.println("白色的" + shape);
	    }
	}
	.....
	
	public class Client {
	    public static void main(String[] args) {
	        //白色
	        Color white = new White();
	        //正方形
	        Shape square = new Square();
			//长方形
	        Shape rectange = new Rectangle();
	        //白色的正方形
	        square.setColor(white);
	        square.draw();
	        //白色的长方形
	        rectange.setColor(white);
	        rectange.draw();
	    }
	}
```

总的来说就是通过两个以上的接口来转换两边的对象


----------


## 3.3 组合模式

组合模式定义了如何将容器对象和叶子对象进行递归组合，使得客户在使用的过程中无须进行区分，可以对他们进行一致的处理。
```java
	
	public abstract class 	{  
	    String name;  
	    public File(String name){  
	        this.name = name;  
	    }  
	    public String getName() {  
	        return name;   
	    }  
	    public void setName(String name) {  
	        this.name = name;  
	    }  
	    public abstract void display();  
	}
	
	public class Folder extends File{  
	    private List<File> files;  
	    public Folder(String name){  
	        super(name);  
	        files = new ArrayList<File>();  
	    }
	    public void display() {  
	        for(File file : files){  
	            file.display();  
	        }  
	    }  
	    public void add(File file){  
	        files.add(file);  
	    }  
	    public void remove(File file){  
	        files.remove(file);  
	    }  
	}
	
	//分支（叶子）
	public class TextFile extends File{  
	    public TextFile(String name) {  
	        super(name);  
	    }  
	    public void display() {  
	        System.out.println("这是文本文件，文件名：" + super.getName());  
	    }  
	}
	
	//分支（叶子）
	public class ImagerFile extends File{  
	    public ImagerFile(String name) {  
	        super(name);  
	    }  
	    public void display() {  
	        System.out.println("这是图像文件，文件名：" + super.getName());  
	    }  
	}
	
	//分支（叶子）
	public class VideoFile extends File{  
	    public VideoFile(String name) {  
	        super(name);  
	    }  
	    public void display() {  
	        System.out.println("这是影像文件，文件名：" + super.getName());  
	    }  
	}
	
	
	public class Client {  
	    public static void main(String[] args) {  
	        //总文件夹  
	        Folder fileA = new Folder("总文件夹");  
	        //向总文件夹中放入三个文件：1.txt、2.jpg、1文件夹  
	        TextFile aText= new TextFile("a.txt");  
	        ImagerFile bImager = new ImagerFile("b.jpg"); 
	        Folder fileB = new Folder("B文件夹");  
	          
	        fileA.add(aText);  
	        fileA.add(bImager);  
	        fileA.add(cFolder);  
	          
	        //向C文件夹中添加文件：c_1.txt、c_1.rmvb、c_1.jpg   
	        TextFile cText = new TextFile("c_1.txt");  
	        ImagerFile cImage = new ImagerFile("c_1.jpg");  
	        VideoFile cVideo = new VideoFile("c_1.rmvb");  
	          
	        fileB.add(cText);  
	        fileB.add(cImage);  
	        fileB.add(cVideo);  
	
	        fileA.display();  
	        //将c_1.txt删除  
	        fileB.remove(cText);  
	        fileB.display();  
	    }  
	}
	
```

----------

## 3.4 装饰者模式

装饰者模式，动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更加有弹性的替代方案。
```java
	//汉堡
	public abstract class Humburger {    
	    protected  String name ;    
	    public String getName(){    
	        return name;    
	    }    
	    public abstract double getPrice();    
	}
	
	//汉堡-鸡腿堡
	public class ChickenBurger extends Humburger {    
	    public ChickenBurger(){    
	        name = "鸡腿堡";    
	    }    
	    @Override    
	    public double getPrice() {    
	        return 10;    
	}
	//配料
	public abstract class Condiment extends Humburger {    
	    public abstract String getName();    
	}
	
	
	//配料 生菜
	public class Lettuce extends Condiment {    
	    Humburger humburger;    
	    public Lettuce(Humburger humburger){    
	        this.humburger = humburger;    
	    }    
	    @Override    
	    public String getName() {    
	        return humburger.getName()+" 加生菜";    
	    }    
	    @Override    
	    public double getPrice() {    
	        return humburger.getPrice()+1.5;    
	    }    
	} 
	//配料 辣椒
	public class Chilli extends Condiment {    
	    Humburger humburger;    
	    public Chilli(Humburger humburger){    
	        this.humburger = humburger;    
	            
	    }    
	    @Override    
	    public String getName() {    
	        return humburger.getName()+" 加辣椒";    
	    }    
	    @Override    
	    public double getPrice() {    
	        return humburger.getPrice();  //辣椒是免费的哦    
	    }    
	}
	
	public class Test {    
	    public static void main(String[] args) {    
	        Humburger humburger = new ChickenBurger();    
	        System.out.println(humburger.getName()+"  价钱："+humburger.getPrice());    
	        Lettuce lettuce = new Lettuce(humburger);    
	        System.out.println(lettuce.getName()+"  价钱："+lettuce.getPrice());    
	        Chilli chilli = new Chilli(humburger);    
	        System.out.println(chilli.getName()+"  价钱："+chilli.getPrice());    
	        Chilli chilli2 = new Chilli(lettuce);    
	        System.out.println(chilli2.getName()+"  价钱："+chilli2.getPrice());    
	    }    
	}  

```
>鸡腿堡  价钱：10.0    
鸡腿堡 加生菜  价钱：11.5    
鸡腿堡 加辣椒  价钱：10.0    
鸡腿堡 加生菜 加辣椒  价钱：11.5   


----------

## 3.5 外观模式

外观模式提供了一个统一的接口，用来访问子系统中的一群接口。

```java

	public class Television {  
	    public void on(){  
	        System.out.println("打开了电视....");  
	    }  
	    public void off(){  
	        System.out.println("关闭了电视....");  
	    }  
	}
	
	public class Light {  
	    public void on(){  
	        System.out.println("打开了电灯....");  
	    }  
	    public void off(){  
	        System.out.println("关闭了电灯....");  
	    }  
	}
	
	public class AirCondition {  
	    public void on(){  
	        System.out.println("打开了空调....");  
	    }  
	    public void off(){  
	        System.out.println("关闭了空调....");  
	    }  
	}
	
	public class Screen {  
	    public void up(){  
	        System.out.println("升起银幕....");  
	    }  
	    public void down(){  
	        System.out.println("下降银幕....");       
	    }  
	}
	
	public class WatchTvSwtichFacade {  
	    Light light;  
	    AirCondition ac;  
	    Television tv;  
	    Screen screen;  
	      
	    public WatchTvSwtichFacade(Light light,AirCondition ac,Television tv,Screen screen){  
	        this.light = light;  
	        this.ac = ac;  
	        this.tv = tv;  
	        this.screen = screen;  
	    }  
	      
	    public void on(){  
	        light.on();       //首先开灯  
	        ac.on();          //然后是打开空调  
	        screen.down();    //把银幕降下来  
	        tv.on();          //最后是打开电视  
	    }  
	      
	    public void off(){  
	        tv.off();         //首先关闭电视机  
	        screen.up();      //银幕升上去  
	        ac.off();         //空调关闭  
	        light.off();      //最后关灯  
	    }  
	}
	
	public class Client {  
	    public static void main(String[] args) {  
	        //实例化组件  
	        Light light = new Light();  
	        Television tv = new Television();  
	        AirCondition ac = new AirCondition();  
	        Screen screen = new Screen();  
	          
	        WatchTvSwtichFacade watchTv = new WatchTvSwtichFacade(light, ac, tv, screen);  
	          
	        watchTv.on();  
	        System.out.println("--------------可以看电视了.........");  
	        watchTv.off();  
	        System.out.println("--------------可以睡觉了...........");  
	    }  
	}
```

总的来说外观模式就是综合管理



----------


## 3.6 亨元模式

享元模式就是运行共享技术有效地支持大量细粒度对象的复用。系统使用少量对象,而且这些都比较相似，状态变化小，可以实现对象的多次复用。

享元模式的核心在于享元工厂类，享元工厂类的作用在于提供一个用于存储享元对象的享元池，用户需要对象时，首先从享元池中获取，如果享元池中不存在，则创建一个新的享元对象返回给用户，并在享元池中保存该新增对象。
```java
	public class FlyweightFactory{  
	    static Map<String, Shape> shapes = new HashMap<String, Shape>();  //存储对象
	    public static Shape getShape(String key){  //工厂模式
	        Shape shape = shapes.get(key);  
	        //如果shape==null,表示不存在,则新建,并且保持到共享池中  
	        if(shape == null){  //单例模式
	            shape = new Circle(key);  
	            shapes.put(key, shape);  //存储进去
	        }  
	        return shape;  
	    }  
	    public static int getSum(){   //获取对象
	        return shapes.size();  
	    }  
	}
	
	public abstract class Shape {  
	    public abstract void draw();  
	}
	
	public class Circle extends Shape{  
	    private String color;  
	    public Circle(String color){  //标识 
	        this.color = color;  
	    }  
	    public void draw() {  
	        System.out.println("画了一个" + color +"的圆形");  
	    }    
	}
	
	public class Client {  
	    public static void main(String[] args) {  
	        Shape shape1 = FlyweightFactory.getShape("红色");  
	        shape1.draw();  
	          
	        Shape shape2 = FlyweightFactory.getShape("灰色");  
	        shape2.draw();  
	          
	        Shape shape3 = FlyweightFactory.getShape("绿色");  
	        shape3.draw();  
	          
	        Shape shape4 = FlyweightFactory.getShape("红色");  
	        shape4.draw();  
	          
	        Shape shape5 = FlyweightFactory.getShape("灰色");  
	        shape5.draw();  
	          
	        Shape shape6 = FlyweightFactory.getShape("灰色");  
	        shape6.draw();  
	          
	        System.out.println("一共绘制了"+FlyweightFactory.getSum()+"中颜色的圆形");  
	    }  
	}

```
享元模式=单例模式+工厂模式（黑白棋）

----------

## 3.7 代理模式


代理模式就是给一个对象提供一个代理，并由代理对象控制对原对象的引用。


	
//聚合式静态代理  都实现了同一个接口，可实现灵活多变。
```java
	public interface Manager {  
	    void doSomething();  
	} 
	
	public class Admin implements Manager {  //目标对象
	    @Override
	    public void doSomething() {  
	        System.out.println("Admin do something.");  
	    }  
	}  
	
	public class AdminPoly implements Manager{  //这是代理对象
	    private Admin admin;  
	    public AdminPoly(Admin admin) {  //或者写成(Manager  admin)
	        super();  
	        this.admin = admin;  
	    }  
	   @Override
	    public void doSomething() {  
	        System.out.println("Log:admin操作开始");  
	        admin.doSomething();  //执行代理
	        System.out.println("Log:admin操作结束");  
	    }  
	}  
	
	Admin admin = new Admin();  
	Manager m = new AdminPoly(admin);  
	m.doSomething();
	
	//继承式静态代理  继承式的实现方式则不够灵活。
	public class AdminProxy extends Admin {  
	    @Override  
	    public void doSomething() {  
	        System.out.println("Log:admin操作开始");  
	        super.doSomething();  
	        System.out.println("Log:admin操作开始");  
	    }  
	}
	AdminProxy proxy = new AdminProxy();  
	proxy.doSomething();  
```
不违反里斯替换，并不更改父类的内容，只是在原有基础上添加东西。

.容易扩展,代理者和被代理者都接口化了,代理者更改业务后只要接口不变,被代理者可以不做任何修改.



----------

## 4.1 职责链模式
将对象组成一条链，发送者将请求发给链的第一个接收者，并且沿着这条链传递，直到有一个对象来处理它或者直到最后也没有对象处理而留在链末尾端。
```java
//　抽象处理者角色类

	public abstract class Handler {
	    //持有下一个处理请求的对象
	    protected Handler successor = null;
	    // 取值方法
	    public Handler getSuccessor() {
	        return successor;
	    }
	    //设置下一个处理请求的对象
	    public void setSuccessor(Handler successor) {
	        this.successor = successor;
	    }
	    //处理聚餐费用的申请
	    public abstract String handleFeeRequest(String user , double fee);
	}
	
	
//　具体处理者角色

	public class ProjectManager extends Handler {
	    @Override
	    public String handleFeeRequest(String user, double fee) {
	        String str = "";
	        //项目经理权限比较小，只能在500以内
	        if(fee < 500)  {
	            //为了测试，简单点，只同意张三的请求
	            if("张三".equals(user)){
	                str = "成功：项目经理同意【" + user + "】的聚餐费用，金额为" + fee + "元";    
	            }else {
	                //其他人一律不同意
	                str = "失败：项目经理不同意【" + user + "】的聚餐费用，金额为" + fee + "元";
	            }
	        }else{
	            //超过500，继续传递给级别更高的人处理
	            if(getSuccessor() != null){
	                return getSuccessor().handleFeeRequest(user, fee);
	            }
	        }
	        return str;
	    }
	}

//　具体处理者角色

	public class DeptManager extends Handler {
	    @Override
	    public String handleFeeRequest(String user, double fee) {
	        String str = "";
	        //部门经理的权限只能在1000以内
	        if(fee < 1000) {
	            //为了测试，简单点，只同意张三的请求
	            if("张三".equals(user)) {
	                str = "成功：部门经理同意【" + user + "】的聚餐费用，金额为" + fee + "元";    
	            }else {
	                //其他人一律不同意
	                str = "失败：部门经理不同意【" + user + "】的聚餐费用，金额为" + fee + "元";
	            }
	        }else {
	            //超过1000，继续传递给级别更高的人处理
	            if(getSuccessor() != null) {
	                return getSuccessor().handleFeeRequest(user, fee);
	            }
	        }
	        return str;
	    }
	}
	
//　具体处理者角色

	public class GeneralManager extends Handler {
	    @Override
	    public String handleFeeRequest(String user, double fee) {
	        String str = "";
	        //总经理的权限很大，只要请求到了这里，他都可以处理
	        if(fee >= 1000){
	            //为了测试，简单点，只同意张三的请求
	            if("张三".equals(user)) {
	                str = "成功：总经理同意【" + user + "】的聚餐费用，金额为" + fee + "元";    
	            }else {
	                //其他人一律不同意
	                str = "失败：总经理不同意【" + user + "】的聚餐费用，金额为" + fee + "元";
	            }
	        }else  {
	            //如果还有后继的处理对象，继续传递
	            if(getSuccessor() != null)  {
	                return getSuccessor().handleFeeRequest(user, fee);
	            }
	        }
	        return str;
	    }
	}

//客户端
	
	public class Client {
	    public static void main(String[] args) {
	        //先要组装责任链
	        Handler h1 = new GeneralManager();
	        Handler h2 = new DeptManager();
	        Handler h3 = new ProjectManager();
	        h3.setSuccessor(h2);
	        h2.setSuccessor(h1);
	        
	        //开始测试
	        String test1 = h3.handleFeeRequest("张三", 300);
	        System.out.println("test1 = " + test1);//项目经理同意 500
	        String test2 = h3.handleFeeRequest("李四", 300);
	        System.out.println("test2 = " + test2);//不同意
	        
	        String test3 = h3.handleFeeRequest("张三", 700);
	        System.out.println("test3 = " + test3);//部门经理同意 1000
	        String test4 = h3.handleFeeRequest("李四", 700);
	        System.out.println("test4 = " + test4);//不同意
	        System.out.println("---------------------------------------");
	        
	        String test5 = h3.handleFeeRequest("张三", 1500);
	        System.out.println("test5 = " + test5);;//总经理同意
	        String test6 = h3.handleFeeRequest("李四", 1500);
	        System.out.println("test6 = " + test6);;//不同意
	    }
	}
```
总的来说就是讲根据判断是否可以处理，不可处理则往下传



----------


## 4.2 命令模式
所以命令模式将请求封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。同时命令模式支持可撤销的操作。
```java
抽象命令类

	public interface Command {  
	    //Command命令接口，为所有的命令声明一个接口。所有的命令都应该实现它 
	    public void execute();  
	} 

电视机类

	public class Television {  
	    public void open(){  
	        System.out.println("打开电视机......");  
	    }  
	      
	    public void close(){  
	        System.out.println("关闭电视机......");        
	    }  
	      
	    public void changeChannel(){  
	        System.out.println("切换电视频道......");  
	    }  
	} 

遥控器类

	public class Controller {  
	    private Command openTVCommand;  
	    private Command closeTVCommand;  
	    private Command changeChannelCommand;  
	    public Controller(Command openTvCommand,Command closeTvCommand,Command changeChannelCommand){  
	        this.openTVCommand = openTvCommand;  
	        this.closeTVCommand = closeTvCommand;  
	        this.changeChannelCommand = changeChannelCommand;  
	    }   
	    //打开电视剧
	    public void open(){  
	        openTVCommand.execute();  
	    }  
	    //关闭电视机 
	    public void close(){  
	        closeTVCommand.execute();  
	    }  
	    //换频道
	    public void change(){  
	        changeChannelCommand.execute();  
	    }  
	}
	
//打开电视机

	public class OpenTvCommand implements Command{  
	    private Television tv;  
	    public OpenTvCommand(){  
	        tv = new Television();  
	    }  
	    public void execute() {  
	        tv.open();  
	    }  
	}

//换频道
  
	public class ChangeChannelCommand implements Command{  
	    private Television tv;  
	    public ChangeChannelCommand(){  
	        tv = new Television();  
	    }  
	    public void execute() {  
	        tv.changeChannel();  
	    }  
	}

//关闭电视机
  
	public class CloseTvCommand implements Command{  
	    private Television tv;  
	    public CloseTvCommand(){  
	        tv = new Television();  
	    }  
	    public void execute() {  
	        tv.close();  
	    }  
	}

客户端

	public class Client {  
	        public static void main(String a[])   {  
	            Command openCommand,closeCommand,changeCommand;  
	              
	            openCommand = new OpenTvCommand();  
	            closeCommand = new CloseTvCommand();  
	            changeCommand = new ChangeChannelCommand();  
	              
	            Controller control = new Controller(openCommand,closeCommand,changeCommand);  
	              
	            control.open();           //打开电视机  
	            control.change();         //换频道  
	            control.close();          //关闭电视机  
	        }  
	} 
```
### 优点
 1. 降低了系统耦合度
 2. 新的命令可以很容易添加到系统中去。
### 缺点
 使用命令模式可能会导致某些系统有过多的具体命令类。


----------

## 4.3 解释器模式

所谓解释器模式就是定义语言的文法，并且建立一个解释器来解释该语言中的句子。
```java
	public interface Expression {
	   public boolean interpret(String context);
	}

//非终结表达式

	public class TerminalExpression implements Expression {
	   private String data;
	   public TerminalExpression(String data){
	      this.data = data;
	   }
	   @Override
	   public boolean interpret(String context) {
	      if(context.contains(data)){
	         return true;
	      }
	      return false;
	   }
	}

//终结表达式

	public class OrExpression implements Expression {  
	   private Expression expr1 = null;
	   private Expression expr2 = null;
	   public OrExpression(Expression expr1, Expression expr2) {
	      this.expr1 = expr1;
	      this.expr2 = expr2;
	   }
	   @Override
	   public boolean interpret(String context) {      
	      return expr1.interpret(context) || expr2.interpret(context);
	   }
	}

//终结表达式

	public class AndExpression implements Expression {  
	   private Expression expr1 = null;
	   private Expression expr2 = null;
	   public AndExpression(Expression expr1, Expression expr2) {
	      this.expr1 = expr1;
	      this.expr2 = expr2;
	   }
	   @Override
	   public boolean interpret(String context) {      
	      return expr1.interpret(context) &amp;&amp; expr2.interpret(context);
	   }
	}

//终结表达式

	public class InterpreterPatternDemo {
	 
	   //规则：Robert 和 John 是男性
	   public static Expression getMaleExpression(){
	      Expression robert = new TerminalExpression("Robert");
	      Expression john = new TerminalExpression("John");
	      return new OrExpression(robert, john);       
	   }
	 
	   //规则：Julie 是一个已婚的女性
	   public static Expression getMarriedWomanExpression(){
	      Expression julie = new TerminalExpression("Julie");
	      Expression married = new TerminalExpression("Married");
	      return new AndExpression(julie, married);    
	   }
	 
	   public static void main(String[] args) {
	      Expression isMale = getMaleExpression();
	      Expression isMarriedWoman = getMarriedWomanExpression();
	 
	      System.out.println("John is male? " + isMale.interpret("John"));
	      System.out.println("Julie is a married women? "
	      + isMarriedWoman.interpret("Married Julie"));
	   }
	}
```
>John is male? true
Julie is a married women? tru

### 优点
 1、 可扩展性比较好，灵活。
 2、 增加了新的解释表达式的方式。
 3、 易于实现文法。
### 缺点
  1、 执行效率比较低，可利用场景比较少。
  2、 对于复杂的文法比较难维护。

----------

## 4.4 迭代器模式
所谓迭代器模式就是提供一种方法顺序访问一个聚合对象中的各个元素，而不是暴露其内部的表示。

```java
迭代器接口 
 
	public interface Iterator<T> {  
	    //是否还有下一个元素 
	    boolean hasNext();  
	    //返回当前位置的元素并将位置移至下一位 
	     T next();  
	} 
	
具体迭代器 

	public class ConcreteIterator<T> implements Iterator{  
	    private List<T> list = new ArrayList<>();  
	    private int cursor = 0;  
	    @Override  
	    public boolean hasNext() {  
	        return cursor != list.size();  
	    }  
	    @Override  
	    public Object next() {  
	        T obj = null;  
	        if(this.hasNext()){  
	            obj = this.list.get(cursor++);  //取出后移除
	        }  
	        return obj;  
	    }  
	}
	
容器接口 

	public interface Aggregate<T> {  
	    // 添加一个元素 
	    void add(T obj);  
	    // 移除一个元素 
	    void remove(T obj);  
	    // 获取容器的迭代器 
	    Iterator<T>  iterator();  
	} 
	
	
具体容器接口 

	public class ConcreteAggregate<T> implements Aggregate<T>{  
	    private List<T> list = new ArrayList<>();  
	    @Override  
	    public void add(T obj) {  
	        list.add(obj);  
	    }  
	    @Override  
	    public void remove(T obj) {  
	        list.remove(obj);  
	    }  
	    @Override  
	    public Iterator<T> iterator() {  
	        return new ConcreteIterator<>();  
	    }  
	}  
	
	public class Client {  
	    public static void main(String[] args){  
	        Aggregate<String> a = new ConcreteAggregate<>();  
	        a.add("Aige");  
	        a.add("Studio\n");  
	        a.add("SM");  
	        a.add("Brother");  
	        Iterator<String> i = a.iterator();  
	        while(i.hasNext()){  
	            System.out.println(i.next());  
	        }  
	    }  
	}  
```
对于迭代模式来说，其自身优点很明显也很单一，支持以不同的方式去遍历一个容器对象，也可以有多个遍历，弱化了容器类与遍历算法之间的关系。

而缺点就是对类文件的增加。



----------


## 4.5 中介者模式

所谓中介者模式就是用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。
```java
//处理抽象类

	abstract class AbstractColleague {  
	    protected int number;  
	    public int getNumber() {  
	        return number;  
	    }  
	    public void setNumber(int number){  
	        this.number = number;  
	    }  
	    //注意这里的参数不再是同事类，而是一个中介者  
	    public abstract void setNumber(int number, AbstractMediator am);  //传入中介者
	}  

//存放A数据以及对于处理方法

	class ColleagueA extends AbstractColleague{  
	    @Override  
	    public void setNumber(int number, AbstractMediator am) {  
	        this.number = number;  
	        am.AaffectB();  
	    }  
	} 

//存放B数据以及对于处理方法

	class ColleagueB extends AbstractColleague{  
	    @Override  
	    public void setNumber(int number, AbstractMediator am) {  
	        this.number = number;  
	        am.BaffectA();  
	    }  
	}

//中介抽象类

	abstract class AbstractMediator {  
	    protected AbstractColleague A;  
	    protected AbstractColleague B;  
	    public AbstractMediator(AbstractColleague a, AbstractColleague b) {  
	        A = a;  
	        B = b;  
	    }  
	    public abstract void AaffectB();  
	    public abstract void BaffectA();  
	}

//中介实体类

	class Mediator extends AbstractMediator {  
	    public Mediator(AbstractColleague a, AbstractColleague b) {  
	        super(a, b);  
	    }  
	    //处理A对B的影响  
	    public void AaffectB() {  
	        int number = A.getNumber();  
	        B.setNumber(number*100);  
	    }  
	    //处理B对A的影响  
	    public void BaffectA() {  
	        int number = B.getNumber();  
	        A.setNumber(number/100);  
	    }  
	}
	
	
	        AbstractColleague collA = new ColleagueA();  
	        AbstractColleague collB = new ColleagueB();  
	        AbstractMediator am = new Mediator(collA, collB);  
	          
	        System.out.println("==========通过设置A影响B==========");  
	        collA.setNumber(1000, am);  
	        System.out.println("collA的number值为："+collA.getNumber());  
	        System.out.println("collB的number值为A的10倍："+collB.getNumber());  
	  
	        System.out.println("==========通过设置B影响A==========");  
	        collB.setNumber(1000, am);  
	        System.out.println("collB的number值为："+collB.getNumber());  
	        System.out.println("collA的number值为B的0.1倍："+collA.getNumber());  
```
### 使用中介者模式的优点：
1.降低了系统对象之间的耦合性，使得对象易于独立的被复用。
2.提高系统的灵活性，使得系统易于扩展和维护。
### 使用中介者模式的缺点：
中介者模式的缺点是显而易见的，因为这个“中介“承担了较多的责任，所以一旦这个中介对象出现了问题，那么整个系统就会受到重大的影响。


----------


## 4.6 备忘录模式

所谓备忘录模式就是在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态。

```java
//角色类

	class Originator {  
	    private String state = "";  //如果需要多状态的话，在这里添加
	    public String getState() {  
	        return state;  
	    }  
	    public void setState(String state) {  //设置
	        this.state = state;  
	    }  
	    public Memento createMemento(){  //创建
	        return new Memento(this.state);  
	    }  
	    public void restoreMemento(Memento memento){  //恢复
	        this.setState(memento.getState());  
	    }  
	}  

//记忆

	class Memento {  
	    private String state = "";  
	    public Memento(String state){  
	        this.state = state;  
	    }  
	    public String getState() {  
	        return state;  
	    }  
	    public void setState(String state) {  
	        this.state = state;  
	    }  
	} 
 
//管理者

	class Caretaker {  
	    private Memento memento;  
	    public Memento getMemento(){  //获取
	        return memento;  
	    }  
	    public void setMemento(Memento memento){  //设置
	        this.memento = memento;  
	    }  
	}  

//客户端

	public class Client {  
	    public static void main(String[] args){  
	        Originator originator = new Originator();  
	        originator.setState("状态1");  //角色状态改变
	        System.out.println("初始状态:"+originator.getState());  
	
	        Caretaker caretaker = new Caretaker();  
	        caretaker.setMemento(originator.createMemento());  //存储第一个记忆
	
	        originator.setState("状态2");  //角色状态改变
	        System.out.println("改变后状态:"+originator.getState());  
	
	        originator.restoreMemento(caretaker.getMemento());  //恢复
	        System.out.println("恢复后状态:"+originator.getState());  
	    }  
	}
```
### 优点
 1、 给用户提供了一种可以恢复状态的机制。可以是用户能够比较方便地回到某个历史的状态。
 2、 实现了信息的封装。使得用户不需要关心状态的保存细节。
### 缺点
消耗资源。如果类的成员变量过多，势必会占用比较大的资源，而且每一次保存都会消耗一定的内存。


----------

## 4.7 观察者模式

何谓观察者模式？观察者模式定义了对象之间的一对多依赖关系，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并且自动更新。
```java
	public interface Observer {
	    public void update(String message);
	}
//观察者

	public class WeixinUser implements Observer {
	    // 微信用户名
	    private String name;
	    public WeixinUser(String name) {
	        this.name = name;
	    }
	    @Override
	    public void update(String message) {
	        System.out.println(name + "-" + message);
	    }
	}
	
	public interface Subject {
	    /** 增加订阅者 */
	    public void attach(Observer observer);
	    /** 删除订阅者 */
	public void detach(Observer observer);
	    /** 通知订阅者更新消息 */
	    public void notify(String message);
	}

//被观察者

	public class SubscriptionSubject implements Subject {
	    //储存订阅公众号的微信用户
	    private List<Observer> weixinUserlist = new ArrayList<Observer>();
	    @Override
	    public void attach(Observer observer) {
	        weixinUserlist.add(observer);
	    }
	    @Override
	    public void detach(Observer observer) {
	        weixinUserlist.remove(observer);
	    }
	    @Override
	    public void notify(String message) {
	        for (Observer observer : weixinUserlist) {
	            observer.update(message);
	        }
	    }
	}
	
//客户端

	public class Client {
	    public static void main(String[] args) {
	        SubscriptionSubject mSubscriptionSubject=new SubscriptionSubject();
	        //创建微信用户
	        WeixinUser user1=new WeixinUser("杨影枫");
	        WeixinUser user2=new WeixinUser("月眉儿");
	        WeixinUser user3=new WeixinUser("紫轩");
	        //订阅公众号
	        mSubscriptionSubject.attach(user1);
	        mSubscriptionSubject.attach(user2);
	        mSubscriptionSubject.attach(user3);
	        //公众号更新发出消息给订阅的微信用户
	        mSubscriptionSubject.notify("刘望舒的专栏更新了");
	    }
	}
```
>杨影枫-刘望舒的专栏更新了
月眉儿-刘望舒的专栏更新了
紫轩-刘望舒的专栏更新了


----------

## 4.8 状态模式
所以状态模式就是允许对象在内部状态发生改变时改变它的行为，对象看起来好像修改了它的类。

```java	
	public interface State {
	    void handle();
	}
	 
	class Booked implements State {
	    @Override
	    public void handle() {
	        System.out.println("您已下单！");
	    }
	}
	 
	class Payed implements State {
	    @Override
	    public void handle() {
	        System.out.println("您已付款！");
	    }
	}
	 
	class Sended implements State {
	    @Override
	    public void handle() {
	        System.out.println("已发货！");
	    }
	}
	class InWay implements State {
	    @Override
	    public void handle() {
	        System.out.println("送货中。。。");
	    }
	}
	 
	class Recieved implements State {
	    @Override
	    public void handle() {
	        System.out.println("已确认收货！");
	    }
	}
	 
	public class Context {
	    private State state;
	    public Context() {}
	    public Context(State state) {
	        this.state = state;
	    }
	     
	    public void setState(State state) {
	        System.out.println("订单信息已更新！");
	        this.state = state;
	        this.state.handle();
	    }
	 
	}
	 
//客户端

	public class Client {
	    public static void main(String[] args) {
	        Context context = new Context();
	        context.setState(new Booked());
	        context.setState(new Payed());
	        context.setState(new Sended());
	        context.setState(new InWay());
	        context.setState(new Recieved());
	    }
	}
```
>订单信息已更新！
您已下单！
订单信息已更新！
您已付款！
订单信息已更新！
已发货！
订单信息已更新！
送货中。。。
订单信息已更新！
已确认收货！


----------

## 4.9 策略模式

所以策略模式就是定义了算法族，分别封装起来，让他们之前可以互相转换，此模式然该算法的变化独立于使用算法的客户。


首先我们需要知道策略模式与状态模式是如此的相似，就犹如一对双胞胎一样。只不过状态模式是通过改变对象内部的状态来帮助对象控制自己的行为，而策略模式则是围绕可以互换的算法来创建成功业务的。两者都可用于解决同一个问题：带有大量的if..else…等条件判断语句来进行选择的。
```java
	public interface Sort{
	    public abstract int[] sort(int arr[]);
	}
	
//冒泡排序

	public class BubbleSort implements Sort{
	    public int[] sort(int arr[]){
	       int len=arr.length;
	       for(int i=0;i<len;i++){
	           for(int j=i+1;j<len;j++){
	              int temp;
	              if(arr[i]>arr[j]){
	                  temp=arr[j];
	                  arr[j]=arr[i];
	                  arr[i]=temp;
	              }             
	           }
	        }
	        System.out.println("冒泡排序");
	        return arr;
	    }
	}

//插入排序

	public class InsertionSort implements Sort {
	    public int[] sort(int arr[]) {
	        int len = arr.length;
	        for (int i = 1; i < len; i++) {
	            int j;
	            int temp = arr[i];
	            for (j = i; j > 0; j--) {
	                if (arr[j - 1] > temp) {
	                    arr[j] = arr[j - 1];
	
	                } else
	                    break;
	            }
	            arr[j] = temp;
	        }
	        System.out.println("插入排序");
	        return arr;
	    }
	
	}

//选择排序

	public class SelectionSort implements Sort {
	    public int[] sort(int arr[]) {
	        int len = arr.length;
	        int temp;
	        for (int i = 0; i < len; i++) {
	            temp = arr[i];
	            int j;
	            int samllestLocation = i;
	            for (j = i + 1; j < len; j++) {
	                if (arr[j] < temp) {
	                    temp = arr[j];
	                    samllestLocation = j;
	                }
	            }
	            arr[samllestLocation] = arr[i];
	            arr[i] = temp;
	        }
	        System.out.println("选择排序");
	        return arr;
	    }
	}
	
//管理类

	public class ArrayHandler{
	    private Sort sortObj;
	    
	    public int[] sort(int arr[])
	    {
	        sortObj.sort(arr);
	        return arr;
	    }
	
	    public void setSortObj(Sort sortObj) {
	        this.sortObj = sortObj; 
	    }
	}
	
//客户端

	public class Client{
	    public static void main(String args[]) {
	       int arr[]={1,4,6,2,5,3,7,10,9};
	       int result[];
	       ArrayHandler ah=new ArrayHandler();
	       Sort sort = new SelectionSort();    //使用选择排序
	       ah.setSortObj(sort); //设置具体策略
	       result=ah.sort(arr);
	       for(int i=0;i<result.length;i++){
	               System.out.print(result[i] + ",");
	       }
	    }
	}
```
### 优点
1、策略模式提供了对“开闭原则”的完美支持，用户可以在不修改原有系统的基础上选择算法或行为，也可以灵活地增加新的算法或行为。
2、策略模式提供了可以替换继承关系的办法。
3、使用策略模式可以避免使用多重条件转移语句。
### 缺点
1、客户端必须知道所有的策略类，并自行决定使用哪一个策略类。
2、策略模式将造成产生很多策略类，



----------

## 4.10 模板方法模式


所谓模板方法模式就是在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。
```java	 
	public abstract class Beverage {  
	    /** 准备饮料的过程，烧水，冲泡，倒杯，搭配糕点 */  
	    final public void prepare() {  
	        boilWater();  
	        brew();  
	        pourInCup();  
	        food();  
	    }  
	    final public void boilWater() {  
	        System.out.println("烧水");  
	    }  
	    abstract void brew();  
	    final public void pourInCup() {  
	        System.out.println("将饮料倒入杯子里");  
	    }  
	    public void food() {  
	        System.out.println("搭配一点糕点");  
	    }  
	}
	
/**  咖啡，实现了brew方法  */  

	public class Coffee extends Beverage {  
	    @Override  
	    void brew() {  
	        System.out.println("冲咖啡");  
	    }  
	}  
	
/**  茶，实现了brew方法，重写了food方法 */  

	public class Tea extends Beverage {  
	    @Override  
	    public void food() {  
	        System.out.println("喝茶不需要搭配糕点");  
	    }  
	    @Override  
	    void brew() {  
	        System.out.println("冲茶");  
	    }  
	}
	
/**  测试类 */  

	public class Test {  
	    public static void main(String[] args) {  
	        System.out.println("---茶---");  
	        Beverage tea = new Tea();  
	        tea.prepare();  
	        System.out.println("---咖啡---");  
	        Beverage coffee = new Coffee();  
	        coffee.prepare();  
	    }  
	}
```	  
   > ---茶---  
    烧水  
    冲茶  
    将饮料倒入杯子里  
    喝茶不需要搭配糕点  
    ---咖啡---  
    烧水  
    冲咖啡  
    将饮料倒入杯子里  
    搭配一点糕点  

### 优点：
1)模板方法模式在一个类中形式化地定义算法，而由它的子类实现细节的处理。
2)模板方法是一种代码复用的基本技术。它们在类库中尤为重要，它们提取了类库中的公共行为。
3)模板方法模式导致一种反向的控制结构，这种结构有时被称为“好莱坞法则” ，即“别找我们，,我们找你”通过一个父类调用其子类的操作(而不是相反的子类调用父类)，通过对子类的扩展增加新的行为，符合“开闭原则”
### 缺点：
每个不同的实现都需要定义一个子类，这会导致类的个数增加，系统更加庞大，设计也更加抽象，但是更加符合“单一职责原则”，使得类的内聚性得以提高。






----------

## 4.11 访问者模式


访问者模式即表示一个作用于某对象结构中的各元素的操作，它使我们可以在不改变各元素的类的前提下定义作用于这些元素的新操作。
```java	  
//抽象访问者

	public abstract class Visitor {  
	    protected String name;  
	    public void setName(String name) {  
	        this.name = name;  
	    }  
	    public abstract void visitor(MedicineA a);  
	    public abstract void visitor(MedicineB b);  
	}

//具体访问者：划价员

	public class Charger extends Visitor{  
	    @Override  
	    public void visitor(MedicineA a) {  
	        System.out.println("划价员：" + name +"给药" + a.getName() +"划价:" + a.getPrice());  
	    }  
	    @Override  
	    public void visitor(MedicineB b) {  
	        System.out.println("划价员：" + name +"给药" + b.getName() +"划价:" + b.getPrice());  
	    }    
	} 

// 具体访问者：药房工作者

	public class WorkerOfPharmacy extends Visitor{  
	    @Override  
	    public void visitor(MedicineA a) {  
	        System.out.println("药房工作者：" + name + "拿药 ：" + a.getName());  
	    }  
	    @Override  
	    public void visitor(MedicineB b) {  
	        System.out.println("药房工作者：" + name + "拿药 ：" + b.getName());  
	    }  
	} 

//抽象元素

	public abstract class Medicine {  
	    protected String name;  
	    protected double price;  
	    public Medicine (String name,double price){  
	        this.name = name;  
	        this.price = price;  
	    }  
	    public String getName() {  
	        return name;  
	    }  
	    public void setName(String name) {  
	        this.name = name;  
	    }  
	    public double getPrice() {  
	        return price;  
	    }  
	    public void setPrice(double price) {  
	        this.price = price;  
	    }  
	    public abstract void accept(Visitor visitor);  
	}
	
//具体元素A

	public class MedicineA extends Medicine{  
	    public MedicineA(String name, double price) {  
	        super(name, price);  
	    }  
	    public void accept(Visitor visitor) {  
	        visitor.visitor(this);  
	    }  
	}
	//具体元素B
	public class MedicineA extends Medicine{ ... }
	//药单
	public class Presciption {  
	    List<Medicine> list = new ArrayList<Medicine>();  
	    public void accept(Visitor visitor){  
	        Iterator<Medicine> iterator = list.iterator();  
	        while (iterator.hasNext()) {  
	            iterator.next().accept(visitor);  
	        }  
	    }  
	    public void addMedicine(Medicine medicine){  
	        list.add(medicine);  
	    }  
	    public void removeMedicien(Medicine medicine){  
	        list.remove(medicine);  
	    }  
	} 

//客户端

	public class Client {  
	    public static void main(String[] args) {  
			//1.定义两种药
	        Medicine a = new MedicineA("板蓝根", 11.0);  
	        Medicine b = new MedicineB("感康", 14.3);  
	        //2.加入当访问管理（药单）
	        Presciption presciption = new Presciption();  
	        presciption.addMedicine(a);  
	        presciption.addMedicine(b);  
	        //3.访问者（划价员）
	        Visitor charger = new Charger();  
	        charger.setName("张三");  
	        //3.访问者（药房管理者）
	        Visitor workerOfPharmacy = new WorkerOfPharmacy();  
	        workerOfPharmacy.setName("李四");  
	        //4.具体操作
	        presciption.accept(charger);  //划价员的操作
	        System.out.println("-------------------------------------");  
	        presciption.accept(workerOfPharmacy);  //
	    }  
	}
```
>划价员：张三给药板蓝根划价:11.0
划价员：张三给药感康划价:14.3
药房工作者：李四拿药：板蓝根
药房工作者：李四拿药：感康


### 优点
 1、使得新增新的访问操作变得更加简单。
 2、能够使得用户在不修改现有类的层次结构下，定义该类层次结构的操作。
 3、将有关元素对象的访问行为集中到一个访问者对象中，而不是分散搞一个个的元素类中。
### 缺点
1、增加新的元素类很困难。在访问者模式中，每增加一个新的元素类都意味着要在抽象访问者角色中增加一个新的抽象操作，并在每一个具体访问者类中增加相应的具体操作，违背了“开闭原则”的要求。
2、破坏封装。当采用访问者模式的时候，就会打破组合类的封装。
3、比较难理解。貌似是最难的设计模式了。
