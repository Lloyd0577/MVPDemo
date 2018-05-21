# MVPDemo

> 声明：原创作品，转载请注明出处[http://www.jianshu.com/p/7b839b7c5884](http://www.jianshu.com/p/7b839b7c5884)

之前公司的项目用到了MVP+Retrofit+RxJava的框架进行网络请求，所以今天特此写一篇文章以做总结。相信很多人都听说过MVP、Retrofit、以及RxJava，有的人已经开始用了，有的人可能还不知道这是什么，以及到底怎么用。不过没关系，接下来我将为你一一揭开他们的神秘面纱，然后利用这三个家伙搭建一个网络请求框架
>**1.什么是MVP？**

MVP（Model View Presenter）其实就是一种项目的整体框架，能让你的代码变得更加简洁，说起框架大家可能还会想到MVC、MVVM。由于篇幅原因，这里我们先不讲MVVM，先来看一下MVC。其实Android本身就采用的是MVC（Model View Controllor）模式、其中Model指的是数据逻辑和实体模型；View指的是布局文件、Controllor指的是Activity。对于很多Android初学者可能会有这样的经历，写代码的时候，不管三七二十一都往Activity中写，当然我当初也是这么干的，根本就没有什么框架的概念，只要能实现某一个功能就很开心了，没有管这么多。当然项目比较小还好，一旦项目比较大，你会发现，Activity所承担的任务其实是很重的，它既要负责页面的展示和交互，还得负责数据的请求和业务逻辑之类的工作，相当于既要打理家庭，又要教育自己调皮的孩子，真是又当爹又当妈。。。那该怎么办呢？这时候Presenter这个继父来到了这个家庭。Presenter对Activity说，我来了，以后你就别这么辛苦了，你就好好打理好View这个家，我专门来负责教育Model这孩子，有什么情况我会向你反映的。这时Activity流下了幸福的眼泪，从此，Model、View（Activity）、Presenter一家三口过上了幸福的生活。。。好了磕个药继续，由于Presenter（我们自己建的类）的出现，可以使View（Activity）不用直接和Model打交道，View（Activity）只用负责页面的显示和交互，剩下的和Model交互的事情都交给Presenter做，比如一些网络请求、数据的获取等，当Presenter获取到数据后再交给View（Activity）进行展示，这样，Activity的任务就大大减小了。这便是MVP（Model 还是指的数据逻辑和实体模型，View指的是Activity，P就是Presenter）框架的工作方式。

>**2.什么是Retrofit？**


接下来我们看一下什么是Retrofit。在官网对Retrofit的描述是这样的
`A type-safe HTTP client for Android and Java`说人话就是“一个类型安全的用于Android和Java网络请求的客户端”，其实就是一个封装好的网络请求库。接下来就来看一下这个库该怎么用。首先我在网上找了一个API接口用于测试：`https://api.douban.com/v2/book/search?q=金瓶梅&tag=&start=0&count=1`这是一个用于查询一本书详细信息的一个请求接口。如果直接用浏览器打开的话会返回以下内容：

![浏览器中返回内容](http://upload-images.jianshu.io/upload_images/3057657-a6a81eae622ca561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来我们来看看如何用Retrofit将上面的请求下来。为了在Android Studio中添加Retrofit库，我们需要添加如下依赖：

>compile 'com.squareup.retrofit2:retrofit:2.1.0'

好了，添加完该库，我们再来看看如何使用，首先我们来建一个实体类Book，用于装网络请求后返回的数据。这里顺带说一下，有的人建一个实体类时可能会根据浏览器中返回中的数据一行一行敲，其实这样非常麻烦，这里教大家一个简单的方法，瞬间生成一个实体类。没错有的人可能用过，我们需要一个插件`GsonFormat`。它的使用也很简单，首先需要在Android Studio中下载，点击左上角菜单栏中的`File`,然后点击`Settings`，在弹窗中选择`Plugins`,然后点击下方的`Browse repositories...`
![](http://upload-images.jianshu.io/upload_images/3057657-aa502d14b5fd3fb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在新打开的窗口中搜索`GsonFormat`，点击右侧绿色按钮就可以下载安装了，安装完需要重启下studio，就可以用了。
![](http://upload-images.jianshu.io/upload_images/3057657-2852cb1ed273df1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

它的用法也很简单，比如你先建立一个新的空类取名Book，然后在里面按`Alt+insert`,会有个小弹窗选择`GsonFormat`,之后在弹出的编辑框中拷入在浏览器中请求下来的那一坨东西，然后一直点ok就会自动生成字段，以及set和get方法，一会儿我们用Retrofit请求下来的数据都会保存在这个实体类中，还是挺方便的。最后我们里面添加一个toString()方法，用于后面显示方便。

接下来，回到我们的Retrofit中上，实体类已经建好了，我们来看看这个Retrofit如何进行网络请求，其实代码也很简单。首先我们需要定义一个接口，取名RetrofitService ：
```
public interface RetrofitService {
    @GET("book/search")
    Call<Book> getSearchBook(@Query("q") String name, 
                             @Query("tag") String tag, 
                             @Query("start") int start, 
                             @Query("count") int count);
}
```
额。。想必有人要问了，这是什么玩意？跟我们平时定义的接口类很像，但又不一样。别心急，我来一一解释下，和别的接口类一样，我们在其中定义了一个方法`getSearchBook`，那么这个方法是做什么的呢？其实它干的事很简单，就是拼接一个URL然后进行网络请求。这里我们拼接的URL就是上文提到的测试URL:`https://api.douban.com/v2/book/search?q=金瓶梅&tag=&start=0&count=1`。聪明的你一定看出来了，在这个URL中book/search就是GET后的值，而?后的q、tag、start、count等入参就是这个方法的入参。有的朋友可能要问了，`https://api.douban.com/v2/`这么一大串跑哪去了？其实我们在进行网络请求时，在URL中前一部分是相对不变的。什么意思呢，比如你打开`间书`网站，在`间书`中你打开不同的网页，虽然它的URL不同，但你会发现，每个URL前面都是以`http://www.jianshu.com/`开头,我们把这个不变的部分，也叫做baseUrl提出来，放到另一个地方,在下面我们会提到。这样我们一个完整的URL就拼接好了。在方法的开头我们可以看到有个GET的注释，说明这个请求是GET方法，当然你也可以根据具体需要用POST、PUT、DELETE以及HEAD。他们的区别如下：
 - GET ----------查找资源（查）
 - POST --------修改资源（改）
 - PUT ----------上传文件（增）
 - DELETE ----删除文件（删）
 - HEAD--------只请求页面的首部

然后我们来看一下这个方法的返回值，它返回Call实体，一会我们要用它进行具体的网络请求，我们需要为它指定泛型为Book也就是我们数据的实体类。接下来，你会发现这个方法的入参和我们平时方法的入参还不大一样。在每个入参前还多了一个注解。比如第一个入参`@Query("q") String name `，`Query`表示把你传入的字段拼接起来，比如在测试url中我们可以看到`q=金瓶梅`的入参，那么`Query`后面的值必须是q，要和url中保持不变，然后我们定义了String类型的name，当调用这个方法是，用于传入字符串，比如可以传入“金瓶梅”。那么这个方法就会自动在q后面拼上这个字符串进行网络请求。以此类推，这个url需要几个入参你就在这个方法中定义几个入参，每个入参前都要加上`Query`注解。当然Retrofit除了Query这个注解外，还有其他几个比如：@QueryMap、@Path、@Body、@FormUrlEncoded/@Field、@Header/@Headers。我们来看一下他们的区别:
####@Query(GET请求):
用于在url后拼接上参数，例如：
```
@GET("book/search")
Call<Book> getSearchBook(@Query("q") String name);//name由调用者传入
```
相当于：
```
@GET("book/search?q=name")
Call<Book> getSearchBook();
```
####@QueryMap(GET请求):
当然如果入参比较多，就可以把它们都放在Map中，例如：
```
@GET("book/search")
Call<Book> getSearchBook(@QueryMap Map<String, String> options);
```
####@Path(GET请求):
用于替换url中某个字段，例如：
```
@GET("group/{id}/users")
Call<Book> groupList(@Path("id") int groupId);
```
像这种请求接口，在group和user之间有个不确定的id值需要传入，就可以这种方法。我们把待定的值字段用`{}`括起来，当然  `{}`里的名字不一定就是id，可以任取，但需和`@Path`后括号里的名字一样。如果在user后面还需要传入参数的话，就可以用Query拼接上，比如：
```
@GET("group/{id}/users")
Call<Book> groupList(@Path("id") int groupId,@Query("sort") String sort);
```
当我们调用这个方法时，假设我们groupId传入1，sort传入“2”，那么它拼接成的url就是`group/1/users?sort=2`，当然最后请求的话还会加上前面的baseUrl。

####@Body(POST请求):
可以指定一个对象作为HTTP请求体,比如：
```
@POST("users/new")
Call<User> createUser(@Body User user);
```
它会把我们传入的User实体类转换为用于传输的HTTP请求体，进行网络请求。
####@Field(POST请求):
用于传送表单数据：
```
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```
注意开头必须多加上`@FormUrlEncoded`这句注释，不然会报错。表单自然是有多组键值对组成，这里的first_name就是键，而具体传入的first就是值啦。
####@Header/@Headers(POST请求):
用于添加请求头部：
```
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```
表示将头部Authorization属性设置为你传入的authorization；当然你还可以用@Headers表示,作用是一样的比如：
```
@Headers("Cache-Control: max-age=640000")
@GET("user")
Call<User> getUser()
```
当然你可以多个设置：
```
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("user")
Call<User> getUser()
```
好了，这样我们就把上面这个RetrofitService 接口类解释的差不多了。我觉得，Retrofit最主要的也就是这个接口类的定义了。好了，有了这个接口类，我们来看一下，到底如何使用这个我们定义的接口来进行网络请求。代码如下：
```
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("https://api.douban.com/v2/")
        .addConverterFactory(GsonConverterFactory.create(new GsonBuilder().create()))
        .build();
RetrofitService service = retrofit.create(RetrofitService.class);
Call<Book> call =  service.getSearchBook("金瓶梅", null, 0, 1);
call.enqueue(new Callback<Book>() {
    @Override
    public void onResponse(Call<Book> call, Response<Book> response) {
        text.setText(response.body()+"");
    }
    @Override
    public void onFailure(Call<Book> call, Throwable t) {
    }
});
```
这里我们可以看到，先新建了一个Retrofit对象，然后给它设置一个我们前面说的baseUrl`https://api.douban.com/v2/`.因为接口返回的数据不是我们需要的实体类，我们需要调用addConverterFactory方法进行转换。由于返回的数据为json类型，所以在这个方法中传入Gson转换工厂`GsonConverterFactory.create(new GsonBuilder().create())`，这里我们需要在studio中添加Gson的依赖：
> compile 'com.squareup.retrofit2:converter-gson:2.1.0'

然后我们调用retrofit的create方法并传入上面我们定义的接口的文件名RetrofitService.class，就可以得到RetrofitService 的实体对象。有了这个对象，我们就可以调用里面之前定义好的请求方法了。比如：
```
Call<Book> call =  service.getSearchBook("金瓶梅", null, 0, 1);
```
它会返回一个Call实体类，然后就可以调用Call的enqueue方法进行异步请求，在enqueue方法中传入一个回调CallBack，重写里面的onResponse和
onFailure方法，也就是请求成功和失败的回调方法。当成功时，它会返回Response，里边封装了请求结果的所有信息，包括报头，返回码，还有主体等。比如调用它的body()方法就可获得Book对象，也就是我们需要的数据。这里我们就把返回的Book，显示屏幕上。如下图：


![Book中的数据](http://upload-images.jianshu.io/upload_images/3057657-370847c4f1e78557.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好了，到这里我们就基本了解了Retrofit的整个工作流程。

>####**3.RxJava**

我们这篇文章主要介绍搭建整体网络请求框架，所以关于RxJava的基础知识，我这就不再详细介绍了，网上也有很多文章，对RxJava还不是很了解的同学，推荐你看一下扔物线的这篇文章[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)

下面我们来看一下RxJava和retrofit的结合使用，为了使Rxjava与retrofit结合，我们需要在Retrofit对象建立的时候添加一句代码`addCallAdapterFactory(RxJavaCallAdapterFactory.create())`，当然你还需要在build.gradle文件中添加如下依赖：
>compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'

完整的代码如下：
```
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("https://api.douban.com/v2/")
        .addConverterFactory(GsonConverterFactory.create(new GsonBuilder().create()))
        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())//支持RxJava
        .build();
```
然后我们还需要修改RetrofitService 中的代码：
```
public interface RetrofitService {
    @GET("book/search")
    Observable<Book> getSearchBook(@Query("q") String name,
                                    @Query("tag") String tag, @Query("start") int start,
                                    @Query("count") int count);

```
可以看到，在原来的RetrofitService 中我们把getSearchBook方法返回的类型Call改为了Observable，也就是被观察者。其他都没变。然后就是创建RetrofitService 实体类：
```
RetrofitService service = retrofit.create(RetrofitService.class);
```
和上面一样，创建完RetrofitService ，就可以调用里面的方法了：
```
Observable<Book> observable =  service.getSearchBook("金瓶梅", null, 0, 1);
```
其实这一步，就是创建了一个rxjava中observable，即被观察者,有了被观察者，就需要一个观察者，且订阅它：
```
observable.subscribeOn(Schedulers.io())//请求数据的事件发生在io线程
          .observeOn(AndroidSchedulers.mainThread())//请求完成后在主线程更显UI
          .subscribe(new Observer<Book>() {//订阅
              @Override
              public void onCompleted() {
                  //所有事件都完成，可以做些操作。。。
              }
              @Override
              public void onError(Throwable e) {
                  e.printStackTrace(); //请求过程中发生错误
              }
              @Override
              public void onNext(Book book) {//这里的book就是我们请求接口返回的实体类    
              }
           }
```
在上面中我们可以看到，事件的消费在Android主线程，所以我们还要在build.gradle中添加如下依赖：
>compile 'io.reactivex:rxandroid:1.2.0'

这样我们就引入了RxAndroid，RxAndroid其实就是对RxJava的扩展。比如上面这个Android主线程在RxJava中就没有，因此要使用的话就必须得引用RxAndroid。

>##**4.实践**

接下来我们就看看，在一个项目中上面三者是如何配合的。我们打开Android Studio，新建一个项目取名为MVPDemo。这个demo的功能也很简单，就是点击按钮调用上面的那个测试接口，将请求下来书的信息显示在屏幕上。首先我们来看一下这个工程的目录结构：

![工程目录](http://upload-images.jianshu.io/upload_images/3057657-4ed4d34dad709ec9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们可以看到，在项目的包名下，我们建了三个主要的文件夹：app、service、ui。当然根据项目的需要你也可以添加更多其他的文件夹，比如一些工具类等。其中app文件夹中可以建一个Application类，用于设置应用全局的一些属性，这里为了使项目更加简单就没有添加；然后，我们再来看看ui文件夹下，这个文件夹下主要放一些关于界面的东西。在里面我们又建了三个文件夹：activity、adapter、fragment，我想看名字你就清楚里面要放什么了。最后我们在重点看看service文件夹中的东西。首先我们来看看里面重要的两个类：RetrofitHelper和RetrofitService。RetrofitHelper主要用于Retrofit的初始化：
```
public class RetrofitHelper {

    private Context mCntext;

    OkHttpClient client = new OkHttpClient();
    GsonConverterFactory factory = GsonConverterFactory.create(new GsonBuilder().create());
    private static RetrofitHelper instance = null;
    private Retrofit mRetrofit = null;
    public static RetrofitHelper getInstance(Context context){
        if (instance == null){
            instance = new RetrofitHelper(context);
        }
        return instance;
    }
    private RetrofitHelper(Context mContext){
        mCntext = mContext;
        init();
    }

    private void init() {
        resetApp();
    }

    private void resetApp() {
        mRetrofit = new Retrofit.Builder()
                .baseUrl("https://api.douban.com/v2/")
                .client(client)
                .addConverterFactory(factory)
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
    }
    public RetrofitService getServer(){
        return mRetrofit.create(RetrofitService.class);
    }
}

```
代码并不复杂，其中resetApp方法，就是前面介绍的Retrofit的创建，getServer方法就是为了获取RetrofitService接口类的实例化。然后定义了一个静态方法getInstance用于获取自身RetrofitHelper的实例化，并且只会实例化一次。

接下来，看一下RetrofitService，其中代码还是上面一样：
```
public interface RetrofitService {
    @GET("book/search")
    Observable<Book> getSearchBooks(@Query("q") String name,
                                    @Query("tag") String tag, @Query("start") int start,
                                    @Query("count") int count);
}
```

然后我们依次来看一下service文件夹下的四个文件夹：entity、manager、presenter和view。其中entity下放我们请求的实体类，这里就是Book。接下来我们来看一下manager中DataManager。这个类其实就是为了让你更方便的调用RetrofitService 中定义的方法：
```
public class DataManager {
    private RetrofitService mRetrofitService;
    public DataManager(Context context){
        this.mRetrofitService = RetrofitHelper.getInstance(context).getServer();
    }
    public  Observable<Book> getSearchBooks(String name,String tag,int start,int count){
        return mRetrofitService.getSearchBooks(name,tag,start,count);
    }
}
```
可以看到，在它的构造方法中，我们得到了RetrofitService 的实例化，然后定义了一个和RetrofitService 中同名的方法，里面其实就是调用RetrofitService 中的这个方法。这样，把RetrofitService 中定义的方法都封装到DataManager 中，以后无论在哪个要调用方法时直接在DataManager 中调用就可以了，而不是重复建立RetrofitService 的实例化，再调用其中的方法。

好了，我们再来看一下presenter和view，我们在前面说过，presenter主要用于网络的请求以及数据的获取，view就是将presenter获取到的数据进行展示。首先我们先来看view，我们看到我们建了两个接口类View和BookView,其中View是空的，主要用于和Android中的View区别开来：
```
public interface View {
}
```
然后让BookView继承自我们自己定义的View ：
```
public interface BookView extends View {
    void onSuccess(Book mBook);
    void onError(String result);
}
```
可以看到在里面定义两个方法，一个onSuccess，如果presenter请求成功，将向该方法传入请求下来的实体类，也就是Book，view拿到这个数据实体类后，就可以进行关于这个数据的展示或其他的一些操作。如果请求失败，就会向这个view传入失败信息，你可以弹个Toast来提示请求失败。通常这两个方法比较常用，当然你可以根据项目需要来定义一些其他的方法。接下来我们看看presenter是如何进行网络请求的 。我们也定义了一个基础Presenter：
```
public interface Presenter {
    void onCreate();

    void onStart();//暂时没用到

    void onStop();

    void pause();//暂时没用到

    void attachView(View view);

    void attachIncomingIntent(Intent intent);//暂时没用到
}

```
里面我们可以看到，定义了一些方法，前面几个onCreate、onStart等方法对应着Activity中生命周期的方法，当然没必要写上Activity生命周期中所有回调方法，通常也就用到了onCreate和onStop，除非需求很复杂，在Activity不同生命周期请求的情况不同。接着我们定义了一个attachView方法，用于绑定我们定义的View。也就是，你想把请求下来的数据实体类给哪个View就传入哪个View。下面这个attachIncomingIntent暂且没用到，就不说了。好了，我们来看一下BookPresenter具体是怎么实现的：
```
public class BookPresenter implements Presenter {
    private DataManager manager;
    private CompositeSubscription mCompositeSubscription;
    private Context mContext;
    private BookView mBookView;
    private Book mBook;
    public BookPresenter (Context mContext){
        this.mContext = mContext;
    }
    @Override
    public void onCreate() {
        manager = new DataManager(mContext);
        mCompositeSubscription = new CompositeSubscription();
    }

    @Override
    public void onStart() {

    }

    @Override
    public void onStop() {
        if (mCompositeSubscription.hasSubscriptions()){
            mCompositeSubscription.unsubscribe();
        }
    }

    @Override
    public void pause() {

    }

    @Override
    public void attachView(View view) {
        mBookView = (BookView)view;
    }

    @Override
    public void attachIncomingIntent(Intent intent) {
    }
    public void getSearchBooks(String name,String tag,int start,int count){
        mCompositeSubscription.add(manager.getSearchBooks(name,tag,start,count)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Book>() {
                    @Override
                    public void onCompleted() {
                        if (mBook != null){
                            mBookView.onSuccess(mBook);
                        }
                    }

                    @Override
                    public void onError(Throwable e) {
                        e.printStackTrace();
                        mBookView.onError("请求失败！！");
                    }

                    @Override
                    public void onNext(Book book) {
                        mBook = book;
                    }
                })
        );
    }
}
```
BookPresenter实现了我们定义的基础Presenter，在onCreate中我们创建了DataManager的实体类，便于调用RetrofitService中的方法，还新建了一个CompositeSubscription对象，CompositeSubscription是用来存放RxJava中的订阅关系的。注意请求完数据要及时清掉这个订阅关系，不然会发生内存泄漏。可在onStop中通过调用CompositeSubscription的unsubscribe方法来取消这个订阅关系，不过一旦调用这个方法，那么这个CompositeSubscription也就无法再用了，要想再用只能重新new一个。然后我们可以看到在attachView中，我们把BookView传进去。也就是说我们要把请求下来的实体类交给BookView来处理。接下来我们定义了一个方法getSearchBooks，名字和入参都和请求接口RetrofitService中的方法相同。这里的这个方法也就是请求的具体实现过程。其实也很简单，就是向CompositeSubscription添加一个订阅关系。上面我们已经说过manager.getSearchBooks就是调用RetrofitService的getSearchBooks方法，而这个方法返回的是一个泛型为Book的Observable，即被观察者，然后通过subscribeOn(Schedulers.io())来定义请求事件发生在io线程，然后通过observeOn(AndroidSchedulers.mainThread())来定义事件在主线程消费，即在主线程进行数据的处理，最后通过subscribe使观察者订阅它。在观察者中有三个方法：onNext、onCompleted、onError。当请求成功话，就会调用onNext，并传入请求返回的Book实体类，我们在onNext中，把请求下来的Book实体类存到内存中，当请求结束后会调用onCompleted，我们把请求下来的Book实体类交给BookView处理就可以了，如果请求失败，那么不会调用onCompleted而调用onError，这样我们可以向BookView传递错误消息。

好了，这样我们我们就可以调用这个接口方法来进行网络的请求了，我们先写一下页面的布局：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:orientation="vertical"
    android:paddingTop="@dimen/activity_vertical_margin">

    <TextView
        android:id="@+id/text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />
    <Button
        android:id="@+id/button"
        android:onClick="getFollowers"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="请求"/>
</LinearLayout>
```
界面很简单，一共两个控件，一个Button，点击时进行网络请求，一个TextView，用于显示请求下来的数据。然后我么看一下Activity中代码：
```
public class MainActivity extends AppCompatActivity {

    private TextView text;
    private Button button;
    private BookPresenter mBookPresenter = new BookPresenter(this);
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        text = (TextView)findViewById(R.id.text);
        button = (Button)findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mBookPresenter.getSearchBooks("金瓶梅", null, 0, 1);
            }
        });
        mBookPresenter.onCreate();
        mBookPresenter.attachView(mBookView);
    }

    private BookView mBookView = new BookView() {
        @Override
        public void onSuccess(Book mBook) {
            text.setText(mBook.toString());
        }

        @Override
        public void onError(String result) {
            Toast.makeText(MainActivity.this,result, Toast.LENGTH_SHORT).show();
        }
    };
    @Override
    protected void onDestroy(){
        super.onDestroy();
        mBookPresenter.onStop();
    }
}
```
逻辑并不复杂，我们先创建了一个BookPresenter 对象，然后调用它的onCreate方法进行初始化，接着调用attachView来绑定BookView。BookView的实现也很简单，在onSuccess方法中将Book 中内容显示在TextView上，在onError中弹出一个Toast提示。然后点击按钮的时候就调用BookPresenter中getSearchBooks方法，同时传入必要的入参。这样网络请求就开始了，如果请求成功就会回调BookView 中的onSuccess方法，失败就回调onError方法。当活动销毁时记得调用BookPresenter的onStop方法来释放订阅关系，防止内存泄漏。

最后别忘了在AndroidManifest中添加网络权限：
>    <uses-permission android:name="android.permission.INTERNET"/>

好了，我们运行一下看一下效果：


![演示](http://upload-images.jianshu.io/upload_images/3057657-ac0f5e24d5fdd501.gif?imageMogr2/auto-orient/strip)

