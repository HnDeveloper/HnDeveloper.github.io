# Android数据存储
> 作者：阳石柏

## 1.SP存储

## 2.文件存储

## 3.DiskLruchCache的使用

## 4.数据库存储(GreenDao使用)





### 一、SP存储

##### 简介: 
SharedPreference类提供了一个总体框架，可以保存和检索的任何基本数据类型（ boolean, float, int, long, string）的持久键-值对（基于XML文件存储的“key-value”键值对数据）;通常用来存储程序的一些配置信息。其存储在“data/data/程序包名/shared_prefs目录下

##### 使用方式: 

getSharedPreferences (String name, int mode);

Mode分为四种模式:

1. MODE_PRIVATE: 私有方式存储,其他应用无法访问
2. MODE_WORLD_READABLE: 表示当前文件可以被其他应用读取
3. MODE_WORLD_WRITEABLE: 表示当前文件可以被其他应用写入
4. MODE_MULTI_PROCESS:适用于多进程访问(目前已被废弃，google官方推荐使用ContentProvider来实现进程间共享访问)


示例:

存储数据:

        SharedPreferences sp = mContext.getSharedPreferences(KEY, Context.MODE_PRIVATE);
        SharedPreferences.Editor et = sp.edit();
        et.putString(key, value);
        et.commit();

取数据:

       SharedPreferences sp = mContext.getSharedPreferences(KEY, Context.MODE_PRIVATE);
        String data =  sp.getString(key, "");

SP优缺点:

SP使用起来很方便，简洁。但存储数据类型比较单一（只有基本数据类型），无法进行条件查询，只能在不复杂的存储需求下使用，比如保存配置信息等。

### 二、文件存储

    关于文件存储，Activity提供了openFileOutput()方法可以用于把数据输出到文件中。

    文件可用来存放大量数据，如文本、图片、音频等。

    默认位置：/data/data/<包>/files/***.***。

    public void save()
      {
        try {
            FileOutputStream outStream=this.openFileOutput("a.txt",Context.MODE_WORLD_READABLE);
            outStream.write(text.getText().toString().getBytes());
            outStream.close();
            Toast.makeText(MyActivity.this,"Saved",Toast.LENGTH_LONG).show();
        } catch (FileNotFoundException e) {
            return;
        }
        catch (IOException e){
            return ;
        }
     } 

    openFileOutput()方法的第一参数用于指定文件名称，不能包含路径分隔符“/” ，如果文件不存在，Android 会自动创建它。

    创建的文件保存在/data/data/<package name>/files目录，如： /data/data/com.test/files/a.txt 



    openFileOutput()方法的第二参数用于指定操作模式，有四种模式，分别为：

    Context.MODE_PRIVATE = 0

    Context.MODE_APPEND = 32768

    Context.MODE_WORLD_READABLE = 1

    Context.MODE_WORLD_WRITEABLE = 2

    Context.MODE_PRIVATE：为默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容，如果想把新写入的内容追加到原文件中。可以使用Context.MODE_APPEND

    Context.MODE_APPEND：模式会检查文件是否存在，存在就往文件追加内容，否则就创建新文件。

    Context.MODE_WORLD_READABLE和Context.MODE_WORLD_WRITEABLE用来控制其他应用是否有权限读写该文件。

    MODE_WORLD_READABLE：表示当前文件可以被其他应用读取；

    MODE_WORLD_WRITEABLE：表示当前文件可以被其他应用写入。



    如果希望文件被其他应用读和写，可以传入： openFileOutput("test.txt", Context.MODE_WORLD_READABLE + Context.MODE_WORLD_WRITEABLE); android有一套自己的安全模型，当应用程序(.apk)在安装时系统就会分配给他一个userid，当该应用要去访问其他资源比如文件的时候，就需要userid匹配。默认情况下，任何应用创建的文件，sharedpreferences，数据库都应该是私有的（位于/data/data/<package name>/files），其他程序无法访问。除非在创建时指定了Context.MODE_WORLD_READABLE或者Context.MODE_WORLD_WRITEABLE ，只有这样其他程序才能正确访问。



    读取文件示例：



    public void load()
    {
     try {
        FileInputStream inStream=this.openFileInput("a.txt");
        ByteArrayOutputStream stream=new ByteArrayOutputStream();
        byte[] buffer=new byte[1024];
        int length=-1  

        while((length=inStream.read(buffer))!=-1)   {
            stream.write(buffer,0,length);
           }

        stream.close();
        inStream.close();
        text.setText(stream.toString());
        Toast.makeText(MyActivity.this,"Loaded",Toast.LENGTH_LONG).show();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    catch (IOException e){
        return ;
    }
     }  

对于私有文件只能被创建该文件的应用访问，如果希望文件能被其他应用读和写，可以在创建文件时，指定Context.MODE_WORLD_READABLE和Context.MODE_WORLD_WRITEABLE权限。  


    Activity还提供了getCacheDir()和getFilesDir()方法：
    getCacheDir()方法用于获取/data/data/<package name>/cache目录。

    getFilesDir()方法用于获取/data/data/<package name>/files目录。



    把文件存入SDCard：

    使用Activity的openFileOutput()方法保存文件，文件是存放在手机空间上，一般手机的存储空间不是很大，存放些小文件还行，如果要存放像视频这样的大文件，是不可行的。对于像视频这样的大文件，我们可以把它存放在SDCard。  

 在程序中访问SDCard，你需要申请访问SDCard的权限。


    在AndroidManifest.xml中加入访问SDCard的权限如下:

    <!-- 在SDCard中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>

    <!-- 往SDCard写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/> 

    要往SDCard存放文件，程序必须先判断手机是否装有SDCard，并且可以进行读写。


         if(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)){ 

         File sdCardDir = Environment.getExternalStorageDirectory();/获取SDCard目录     

         File saveFile = new File(sdCardDir, “a.txt”);
         FileOutputStream outStream = new FileOutputStream(saveFile);
         outStream.write("test".getBytes());
         outStream.close();
     } 


    Environment.getExternalStorageState()方法用于获取SDCard的状态，如果手机装有SDCard，并且可以进行读写，那么方法返回的状态等于Environment.MEDIA_MOUNTED。  

    Environment.getExternalStorageDirectory()方法用于获取SDCard的目录，当然要获取SDCard的目录，你也可以这样写：

            File saveFile = new File("/sdcard/a.txt");

            FileOutputStream outStream = new FileOutputStream(saveFile);

            outStream.write("test".getBytes());

            outStream.close();

### 三、DiskLruchCache的使用


首先DiskLruCache是不能new出实例的，如果我们要创建一个DiskLruCache的实例，则需要调用它的open()方法


        public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)

     open()方法接收四个参数，第一个参数指定的是数据的缓存地址，第二个参数指定当前应用程序的版本号，第三个参数指定同一个key可以对应多少个缓存文件，基本都是传1，第四个参数指定最多可以缓存多少字节的数据。



缓存地址通常都会存放在 /sdcard/Android/data/<application package>/cache 这个路径下面，但同时我们又需要考虑如果这个手机没有SD卡，或者SD正好被移除了的情况，因此专门写一个方法来获取缓存地址：

    public File getDiskCacheDir(Context context, String uniqueName) {  
    String cachePath;  
    if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())  
            || !Environment.isExternalStorageRemovable()) {  
        cachePath = context.getExternalCacheDir().getPath();  
    } else {  
        cachePath = context.getCacheDir().getPath();  
    }  
    return new File(cachePath + File.separator + uniqueName);  
    }  

当SD卡存在或者SD卡不可被移除的时候，就调用getExternalCacheDir()方法来获取缓存路径，否则就调用getCacheDir()方法来获取缓存路径。前者获取到的就是 /sdcard/Android/data/<application package>/cache 这个路径，而后者获取到的是 /data/data/<application package>/cache 这个路径。


因此一个open()方法应该如下所示:

        //得到缓存文件
        File diskCacheDir = getDiskCacheDir(mContext, "Bitmap");
        //如果文件不存在 直接创建
        if (!diskCacheDir.exists()) {
            diskCacheDir.mkdirs();
        }

        try {
            mDiskCache = DiskLruCache.open(diskCacheDir, 1, 1, 1024 * 1024 * 20);
        } catch (IOException e) {
            e.printStackTrace();
        }




 有了DiskLruCache的实例之后，我们就可以对缓存的数据进行操作了,首先进行写入操作:

     /**
     * 将Bitmap写入缓存
     */
        private Bitmap addBitmapToDiskCache(String url) throws IOException {


        if (mDiskCache == null) {
            return null;
        }

        //设置key，并根据URL保存输出流的返回值决定是否提交至缓存
        String key = hashKeyFormUrl(url);
        //得到Editor对象
        DiskLruCache.Editor editor = mDiskCache.edit(key);
        if (editor != null) {
            OutputStream outputStream = editor.newOutputStream(0);
            if (downloadUrlToStream(url, outputStream)) {
                //提交写入操作
                editor.commit();
            } else {
                //撤销写入操作
                editor.abort();
            }
            mDiskCache.flush();
        }
        return getBitmapFromDiskCache(url);
    }

此方法就是访问urlS中传入的网址，并通过outputStream写入到本地。有了这个方法之后，下面我们就可以使用DiskLruCache来进行写入了，写入的操作是借助DiskLruCache.Editor这个类完成的。类似地，这个类也是不能new的，需要调用DiskLruCache的edit()方法来获取实例,


 而edit()方法接收一个参数key，这个key将会成为缓存文件的文件名，并且必须要和图片的URL是一一对应的.因此将图片的URL进行MD5编码，编码后的字符是唯一的，并且只会包含0-F字符，也符合文件的命名规则.


    /**
     * 将URL转换成key
     */
    private String hashKeyFormUrl(String url) {
        String cacheKey;
        try {
            final MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(url.getBytes());
            cacheKey = bytesToHexString(mDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            cacheKey = String.valueOf(url.hashCode());
        }
        return cacheKey;
    }

    /**
     * 将Url的字节数组转换成哈希字符串
     */
    private String bytesToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }


     /**
     * 将URL中的图片保存到输出流中
     */
    private boolean downloadUrlToStream(String urlString, OutputStream outputStream) {
        HttpURLConnection urlConnection = null;
        BufferedOutputStream out = null;
        BufferedInputStream in = null;
        try {
            final URL url = new URL(urlString);
            urlConnection = (HttpURLConnection) url.openConnection();
            in = new BufferedInputStream(urlConnection.getInputStream(), 1024 * 8);
            out = new BufferedOutputStream(outputStream, 1024 * 8);
            int b;
            while ((b = in.read()) != -1) {
                out.write(b);
            }
            return true;
        } catch (final IOException e) {
            e.printStackTrace();
        } finally {
            if (urlConnection != null) {
                urlConnection.disconnect();
            }
            try {
                if (out != null) {
                    out.close();
                }
                if (in != null) {
                    in.close();
                }
            } catch (final IOException e) {
                e.printStackTrace();
            }
        }
        return false;
    }





  缓存成功写入之后就需要读取缓存中的内容,如下所示:

    /**
     * 从缓存中取出Bitmap
     */
    private Bitmap getBitmapFromDiskCache(String url) throws IOException {

        //如果缓存中为空  直接返回为空2017/4/6 18:53:47 2017/4/6 18:53:47 
        if (mDiskCache == null) {
            return null;
        }

        //通过key值在缓存中找到对应的Bitmap
        Bitmap bitmap = null;
        String key = hashKeyFormUrl(url);
        //通过key得到Snapshot对象
        DiskLruCache.Snapshot snapShot = mDiskCache.get(key);
        if (snapShot != null) {
            //得到文件输入流
            InputStream ins = snapShot.getInputStream(0);

            bitmap = BitmapFactory.decodeStream(ins);
        }
        return bitmap;
    }

### 四、数据库存储(GreenDao使用)


关于GreenDao

greenDao是一个将对象映射到SQLite数据库中的轻量且快速的ORM解决方案。关于greenDAO的概念可以看官网
http://greenrobot.org/greendao/

greenDAO 优势

1、一个精简的库
2、性能最大化
3、内存开销最小化
4、易于使用的 APIs
5、对 Android 进行高度优化

GreenDao 3.0使用

GreenDao 3.0采用注解的方式来定义实体类，通过gradle插件生成相应的代码。

一，在as中导入相关的包

    compile'org.greenrobot:greendao:3.0.1'  
    compile'org.greenrobot:greendao-generator:3.0.0'

二，在build.gradle中进行配置：

    apply plugin: 'org.greenrobot.greendao'
    buildscript { 
    repositories { 
        mavenCentral()    
    }    
    dependencies {
    classpath 'org.greenrobot:greendao-gradle-plugin:3.0.0'    
    }
    }

在gradle的根模块中加入上述代码。

三，自定义路径

    greendao {
    schemaVersion 1
    daoPackage 'com.yangshibai.savetest.gen'
    targetGenDir 'src/main/java'
    }

    在gradle的根模块中加入上述代码，就完成了我们的基本配置了。
    属性介绍：
    schemaVersion--> 指定数据库schema版本号，迁移等操作会用到    ;
    daoPackage --> dao的包名，包名默认是entity所在的包；
    targetGenDir --> 生成数据库文件的目录;

四，创建一个User的实体类


    @Entity  //表示这个实体类一会会在数据库中生成对应的表
    public class User {
    @Id //表示该字段是id
    private Long id; 
    private String name; 
    @Transient //表示这个属性将不会作为数据表中的一个字段
    private int tempUsageCount; // not persisted  
        }

五，MakeProject

编译项目，User实体类会自动编译，生成get、set方法并且会在com.anye.greendao.gen目录下生成三个文件；


greenDao
GreenDao使用

    public class MyApplication extends Application {
    private DaoMaster.DevOpenHelper mHelper;
    private SQLiteDatabase db;
    private DaoMaster mDaoMaster;
    private DaoSession mDaoSession;
    public static MyApplication instances;
    @Override    public void onCreate() {
     super.onCreate();
     instances = this;
     setDatabase();
    }
    public static MyApplication getInstances(){
     return instances;
    }

        /**
        * 设置greenDao
        */
    private void setDatabase() {
    // 通过 DaoMaster 的内部类 DevOpenHelper，你可以得到一个便利的 SQLiteOpenHelper 对象。
    // 可能你已经注意到了，你并不需要去编写「CREATE TABLE」这样的 SQL 语句，因为 greenDAO 已经帮你做了。

    mHelper = new DaoMaster.DevOpenHelper(this, "notes-db", null);
    db = mHelper.getWritableDatabase();
    // 注意：该数据库连接属于 DaoMaster，所以多个 Session 指的是相同的数据库连接。 
    mDaoMaster = new DaoMaster(db); 
    mDaoSession = mDaoMaster.newSession();
    }   
    public DaoSession getDaoSession() {
      return mDaoSession;
    }
    public SQLiteDatabase getDb() {
      return db;
    }
    }
获取UserDao对象：

    mUserDao = MyApplication.getInstances().getDaoSession().getUserDao();



简单的增删改查实现：



1. 增

        /**
        * 增加数据
         */
        private void addDate() {
        mUser = new User((long) i, "测试" + i);
        mUserDao.insert(mUser);//添加一个
        mContext.setText(mUser.getName());
        Toast.makeText(GreenDaoActivity.this, "增加索引为" + i + ",内容为:" + mUser.getName(), Toast.LENGTH_SHORT).show();
        i++;
    }




2. 删

    /**
     * 删除数据
     */
        private void deleteDate() {
        deleteUserById(j);
        Toast.makeText(GreenDaoActivity.this, "删除索引为" + j, Toast.LENGTH_SHORT).show();
        j++;
        }

    /**
     * 根据主键删除User
     *
     * @param id User的主键Id
     */
    public void deleteUserById(long id) {
        mUserDao.deleteByKey(id);
    }



3. 改

       /**
       * 更改数据
       */
        private void updateDate() {

        mUser = new User((long) k, "id为" + k + "的数据已经更改");
        mUserDao.update(mUser);
        Toast.makeText(GreenDaoActivity.this, "更改索引为" + k, Toast.LENGTH_SHORT).show();
        k++;
    }



4. 查

        /**
        * 查找数据
        */

        private void findDate() {
        List<User> users = mUserDao.loadAll();
        String userName = "";
        for (int i = 0; i < users.size(); i++) {
            userName += users.get(i).getName() + ",";
        }
            mContext.setText("查询全部数据==>" + userName);
        }
