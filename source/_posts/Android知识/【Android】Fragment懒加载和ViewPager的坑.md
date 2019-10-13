---
title: 【Android】Fragment懒加载和ViewPager的坑
date: 2016/10/02 21:30:25
categories:
- Android知识
---

> **本篇文章已授权微信公众号 安卓巴士Android开发者门户 独家发布**  

#效果  

老规矩，先来看看效果  

![效果图](http://upload-images.jianshu.io/upload_images/1924341-f63733e001d0dd6b.gif?imageMogr2/auto-orient/strip)

ANDROID和福利两个Fragment是设置的Fragment可见时加载数据，也就是懒加载。圆形的旋转加载图标只有一个，所以，如果当前Fragment正处于加载状态，在离开该Fragment时需要隐藏加载动画，因为另一个Fragment并不一定处于加载状态，当返回Fragment时，如果还是处于加载状态，则要可以实现自动显示加载动画，如果数据已经加载完毕则不需要再显示出来。  
  
以上效果就是今天要介绍和分享的，那么开始往下看吧。  

#懒加载  
懒加载意思也就是当需要的时候才会去加载  
  
那么，为什么Fragment需要懒加载呢，一般我们都会在**onCreate()**或者**onCreateView()**里去启动一些数据加载操作，比如从本地加载或者从服务器加载。大部分情况下，这样并不会出现什么问题，但是当你使用**ViewPager + Fragment**的时候，问题就来了，这时就应该考虑是否需要实现懒加载了。  

#ViewPager + Fragment 的坑  

ViewPager为了让滑动的时候可以有很好的用户的体验，也就是防止出现卡顿现象，因此它有一个缓存机制。默认情况下，ViewPager会提前创建好当前Fragment旁的两个Fragment，举个例子说也就是如果你当前显示的是编号3的Fragment，那么其实编号2和4的Fragment也已经创建好了，也就是说这**3个Fragment**都已经执行完 **onAttach() -> onResume()** 这之间的生命周期函数了。  


![日志图1](http://upload-images.jianshu.io/upload_images/1924341-121926ffcf7c58f2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1924341-45b4060da633517d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本来Fragment的 **onResume()**表示的是当前Fragment处于可见且可交互状态，但由于ViewPager的缓存机制，它已经失去了意义，也就是说我们只是打开了“福利”这个Fragment，但其实“休息视频”和“拓展资源”这两个Fragment的数据也都已经加载好了。  

如果加载数据的操作都比较耗时或者都是类似图片的占用大量内存，这时就应该考虑想想是否该实现懒加载。也就是，当我打开哪个Fragment的时候，它才会去加载数据。  

#懒加载实现？  

##setUserVisibleHint(boolean isVisibleToUser) 可以？  

当你去网上查找相关资料时，你会发现很多人推荐说把加载数据的操作放在这个函数里，**isVisibleToUser**表示当前Fragment是否可见。那么，是否真的可以就这样做呢？先来看个日志：  


![日志图2](http://upload-images.jianshu.io/upload_images/1924341-a3fa159face2d9a1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

题主是从 **DayDataFragment** 跳转到 **MeiziDataFragment** 的，所以可以看到日志里面：**DayDataFragment**打出了**false**，表示它不可见了。而**MeiziDataFragment**却先打出了**false**，然后才打出**true**，这是因为**setUserVisibleHint()**在Fragment实例化时会先调用一次，并且默认值是false，当选中当前显示的Fragment时还会再调用一次。  

所以，看上面的日志，除了**DayDataFragment**外，其他三个Fragment均没有实例化，所以当打开**MeiziDataFragment**时，因为ViewPager的缓存机制，会同时创建三个Fragment的实例，所以打印了三条**isVisibleToUeser: false**的日志，因为选中的是**MeiziDataFragment**，所以它还会触发一次**setUserVisibleHint()**，并且打印出true。  

那么，是否可以在**setUserVisibleHint(boolean isVisibleToUser)**里进行数据加载操作来实现懒加载呢？  

可以是可以，如果你只是需要数据的懒加载的话，但如果你还有以下的需求，那么这种方式就不行了：  

#####1、如果你在Fragment可见时需要进行一些控件的操作，比如显示加载控件  
#####2、如果你还需要在Fragment从 “可见 -> 不可见” 时进行一些操作的话，比如取消加载控件显示  


这边再提一下，**setUserVisibleHint()**可能会在Fragment的生命周期之外被调用，也就是可能在view创建前就被调用，也可能在destroyView后被调用，所以如果涉及到一些控件的操作的话，可能会报 null 异常，因为控件还没初始化，或者已经摧毁了。  


#进一步封装  

题主稍微进行了一些封装，自定义了一个新的回调函数  **onFragmentVisibleChange(boolean isVisible)**  ，可以实现的效果有：  

#####1、只有两种情况会触发该函数  
#####2、一种是Fragment从“不可见 -> 可见” 时触发，并传入 isVisible = true  
#####3、一种是Fragment从“可见   -> 不可见” 时触发，并传入 isVisible = false
#####4、可以在该函数内进行控件的操作，不会报null异常    


![日志图3](http://upload-images.jianshu.io/upload_images/1924341-231a7fe2289090ac.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

题主这次仍旧是从**DayDataFragment** 跳转到 **MeiziDataFragment**， 但跟上上面的日志图片不同，这里只打印了两条日志，也就是说即使有三个Fragment被实例化了，但只有显示的那个Fragment和离开的那个Fragment才会触发回调函数，这样就可以支持我们在可见状态变化时进行一些操作，因为不会有多余的false触发。  

另外，因为ViewPager缓存机制，所以题主进行了view的复用，防止**onCreateView()**重复的创建view，其实也就是将view设置为成员变量，创建view时先判断是否为null。因为ViewPager里对Fragment的回收和创建时，如果Fragment已经创建过了，那么只会调用 **onCreateView() -> onDestroyView()** 生命函数，**onCreate()和onDestroy**并不会触发，所以关于变量的初始化和赋值操作可以在**onCreate()**里进行，这样就可以避免重复的操作。  

#代码    

***  
2016-04-21 更新：该博客封装的懒加载实现有些不足，比如不支持数据只有第一次打开Fragment时才进行加载的应用场景，因此重新写了篇博客，可以移步至此观看：[再来一篇Fragment的懒加载（只加载一次哦）](http://www.jianshu.com/p/254dc5ddffea)  

***  

最后附上代码，另外注意一下，题主是从项目里抽出代码，进行一些修改，让它尽量可以直接复制粘贴使用，但并没有进行过测试，所以如果不行的话可以留言，题主会查看。或者你直接到我原项目里去查看，[代码已托管至Github上](https://github.com/woshidasusu/Meizi/blob/master/app/src/main/java/coder/dasu/meizi/view/fragment/GankDataFragment.java)，因为项目是针对具体需求的，所以类里面会增加很多其他无关的代码。再或者，你可以尝试自己进行封装下，代码很少，不到50行，理解思路就行了。  

```  

/**
 * Created by dasu on 2016/9/27.
 * https://github.com/woshidasusu/Meizi
 *
 * Viewpager + Fragment情况下，fragment的生命周期因Viewpager的缓存机制而失去了具体意义
 * 该抽象类自定义一个新的回调方法，当fragment可见状态改变时会触发的回调方法，介绍看下面
 *
 * @see #onFragmentVisibleChange(boolean)
 */
public abstract class ViewPagerFragment extends Fragment {

    /**
     * rootView是否初始化标志，防止回调函数在rootView为空的时候触发
     */
    private boolean hasCreateView;
    
    /**
     * 当前Fragment是否处于可见状态标志，防止因ViewPager的缓存机制而导致回调函数的触发
     */
    private boolean isFragmentVisible;
    
    /**
     * onCreateView()里返回的view，修饰为protected,所以子类继承该类时，在onCreateView里必须对该变量进行初始化
     */
    protected View rootView;

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        Log.d(getTAG(), "setUserVisibleHint() -> isVisibleToUser: " + isVisibleToUser);
        if (rootView == null) {
            return;
        }
        hasCreateView = true;
        if (isVisibleToUser) {
            onFragmentVisibleChange(true);
            isFragmentVisible = true;
            return;
        }
        if (isFragmentVisible) {
            onFragmentVisibleChange(false);
            isFragmentVisible = false;
        }
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        initVariable();
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        if (!hasCreateView && getUserVisibleHint()) {
            onFragmentVisibleChange(true);
            isFragmentVisible = true;
        }
    }

    private void initVariable() {
        hasCreateView = false;
        isFragmentVisible = false;
    }

    /**************************************************************
     *  自定义的回调方法，子类可根据需求重写
     *************************************************************/

    /**
     * 当前fragment可见状态发生变化时会回调该方法
     * 如果当前fragment是第一次加载，等待onCreateView后才会回调该方法，其它情况回调时机跟 {@link #setUserVisibleHint(boolean)}一致
     * 在该回调方法中你可以做一些加载数据操作，甚至是控件的操作，因为配合fragment的view复用机制，你不用担心在对控件操作中会报 null 异常
     *
     * @param isVisible true  不可见 -> 可见
     *                  false 可见  -> 不可见
     */
    protected void onFragmentVisibleChange(boolean isVisible) {
        Log.w(getTAG(), "onFragmentVisibleChange -> isVisible: " + isVisible);
    }
}


```  

#用法  

  新建类ViewPagerFragment，将上面代码复制粘贴进去，添加需要的import语句 -> 新建你需要的Fragment类，继承ViewPagerFragment，在onCreateView()里对rootView进行初始化 -> 重写onFragmentVisibleChange()，在这里进行你需要的操作，比如数据加载，控制显示等。  
  
```  
public class MyFragment extends ViewPagerFragment{
    
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        if (rootView == null) {
            rootView = inflater.inflate(R.layout.fragment_android, container, false);
        
        }
        return rootView;
    } 
    
    @Override
    protected void onFragmentVisibleChange(boolean isVisible) {
        super.onFragmentVisibleChange(isVisible);
        if (isVisible) {
        //   do things when fragment is visible    
        //    if (ListUtils.isEmpty(mDataList) && !isRefreshing()) {
        //        setRefresh(true);
        //        loadServiceData(false);
            } else {
        //        setRefresh(false);
            }
        }
    }
}
  
```  
  
##[项目Github 地址](https://github.com/woshidasusu/Meizi)  

***  
  
  
##PS  
  
  以上就是这次的内容了，最近题主想利用 **Gank**公开的api，做一个类似于Meizi的应用出来，虽然这个App已经有无数人都做过了，但确实是一个很值得学习的项目，题主仍然是小白一个，所以还是好好学习下。drakeet的Meizi项目用到了很多高级技术，比如Rxjava之类的，题主看不懂，其他Github上一些比较出名的Meizi App要么是MVP架构，要么还是用到了目前小白的我看不懂的技术，所以这次就决定自己用最基础的MVC，一些简单常用的第三方库来做这个App，毕竟路要一步一步走，如果这个完成了，收获和体验应该会很多（这不就收获了这篇随笔了吗O(∩_∩)O），所以，如果有兴趣的话，欢迎Start，欢迎指点，欢迎拍砖，大家一起学习进步。  
