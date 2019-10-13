---
title: 【Android】属性动画的使用理解
date: 2016/09/08 21:30:25
categories:
- Android知识
---

> **本篇文章已授权微信公众号 dasu_Android（大苏）独家发布**  

*** 
属性动画的教程网上已经特别多了，本篇也不打算再去各种详解知识点，主要就是记录题主学习属性动画时的碰到的一些困惑，以及后来自己的理解。如果有人也碰到相似的问题，正好可以一起讨论下。  

*** 

#概要 
本篇主要涉及的知识点包括：  
1. ObjectAnimator  
1. ValueAnimator  

老规矩，首先先来看下效果图：  

![](http://upload-images.jianshu.io/upload_images/1924341-8f099a69c991d2fc.gif?imageMogr2/auto-orient/strip)


这种折叠/展开，隐藏/显示的动画在很多地方都会有用到，如果再加上使用5.0后引进的Z属性，实现各种酷炫的立体动画就更吸引人了。所以，还是先掌握好这基础的属性动画吧。  

#分析  

如果你还对属性动画不太明白，或者没用过ObjectAnimator、ValueAnimator的话，建议先去看下[郭神的这篇](http://blog.csdn.net/guolin_blog/article/details/43536355)。  

从上图很容易可以看出，这需要用到**translationX/Y**属性，即平移的属性。也许你会觉得，这不是很简单吗，不就设置下平移的起止值，动画时长，搞定。  

没错，是很简单，就是这么实现的。但其实，对于新手来说，知道怎么做和把它做出来其实还是两码事。题主也还是个初学者，当初也是觉得这很简单啊，然后自己做的时候却出现了各种问题。下面就来讲讲题主做的过程中碰到的一些问题吧。  

##1、平移的距离如何确定？  

先来看那个竖直收缩/扩展的效果，每个控件都平移到最底下控件的位置，然后消失。有时候我们的需求就是这样，不要求将控件全部移出屏幕，只移到某个指定位置，然后消失之类的。如果是移出屏幕，那么距离很容易设定，但像这种情况下，我们要如何去设置每个控件应该平移多长的距离呢？  

很多博客，在对属性动画介绍时，给出的示例代码都是简单的设置某个具体的数值，然后让我们看效果。但这里还能继续用写死的固定值吗，显然不行，那么就需要我们在代码中动态的来计算两个控件之间的距离，然后再来确定控件应该平移的距离。  

经过一番查找，题主找到可以用**View.getLocationOnScreen()**这个方法来实现。  

```  
    /**
     * 计算两个view的距离
     * @param v1
     * @param v2
     * @return 返回new int[2], [0]横坐标距离，[1]纵坐标的距离
     */
    private int[] calculateWidgetsDistance(View v1, View v2){
        int[] location1 = new int[2];
        int[] location2 = new int[2];
        int[] ret = new int[2];

        v1.getLocationOnScreen(location1);
        v2.getLocationOnScreen(location2);

        ret[0] = Math.abs(location1[0] - location2[0]);
        ret[1] = Math.abs(location1[1] - location2[1]);
        return ret;
    }

```  

##2、setTranslationX(float translationX) 参数值的含义  

如果我们使用**ValueAnimator**来实现动画效果，那么我们就需要接触到**setTranslationX()**这类方法了，如下：  
```  
        ValueAnimator animator = ValueAnimator.ofFloat(mView.getTranslationY(),300.0f);
           animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    float value = (float) animation.getAnimatedValue();
                    mView.setTranslationY(value);
                }
         });
       animator.setDuration(1000);
       animator.start();

```  

那么好，问题来了。上面动画的效果是什么？或者说 **300.0f**代表的是什么含义？ 

先来说说动画的效果，是将mView从当前位置，沿Y轴平移到Y坐标300的地方？还是从当前位置沿Y正方向平移300？我们看下效果是什么：  

![](http://upload-images.jianshu.io/upload_images/1924341-5d11ed8ae7429086.gif?imageMogr2/auto-orient/strip)


好像是沿Y平移了300，那么真的是这样吗？如果上面代码的效果表示的意思真是从当前位置沿Y平移300，那么当我们再次点击按钮时，应该继续往下移300，不断的点击就不断的往下移才对，但很明显，从上图中我们看出，当再次点击时没有任何动画效果了。所以，上面代码的动画效果显然不是沿Y平移300.  

那么到底是什么效果呢？我们来将代码稍微做些改动，先**复制**上面代码，然后把**300.0f改成200.0f**,然后把复制的这个动画绑定到其他按钮（如下图的FAB)上，这样当我们先点击FAB，再点击按钮本身，也就是先启动平移200f动画，再启动平移300f的动画。看看会有什么效果：  

![](http://upload-images.jianshu.io/upload_images/1924341-63629b4b61d85f80.gif?imageMogr2/auto-orient/strip)


注意看上图里的点击顺序，为了更方便讲解，我们这里标好步骤：  
1. 点击FAB时，控件往下平移一段距离  
1. 再点击控件本身时，控件继续往下平移一段距离，但比第一次平移的距离短  
1. 然后不断点击按钮本身时，没任何动画效果  
1. 但是当再点击FAB时，按钮往上平移了  
1. 此时再点击按钮本身时，咦！发现有效果了，往下平移了  
1. 然后再点击按钮本身发现又没任何效果了。但是再点击FAB时，按钮又往上平移了！发现没有，当按钮处于最底时，点击FAB会将按钮返回到第2个步骤了。  

我稍微的对上面那图做些备注，你们就很容易明白为什么是这个动画效果，以及最初那几个问题（300.0f代表什么含义）。  

![](http://upload-images.jianshu.io/upload_images/1924341-f9d6d736922ab1fc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


明白了没有，**300.0f表示的是相对于控件最初最初位置的一个距离**，因为这里是Y轴平移，所以上面那代码的动画效果就是**将mView控件从当前位置，沿Y轴平移到距离控件最初位置300的地方**。  

所以，当我们改动代码后才会有那个效果，因为点击FAB，是将控件平移到距离最初起始位置为200的地方。然后再点击按钮本身时，代码意思是将控件从当前位置平移到距离最初位置300的地方，但此时控件的位置并不是在最初的位置，而是已经经过一次平移，处于距离最初位置200的地方，当前控件要平移到300的地方，只需要再平移100就够了。所以第二次控件下移的距离才会比第一次短。之后的效果就不要我再来讲解了吧，记住300.f和200.0f都是相对于最初位置的距离，然后就可以很好的理解上图的动画了。  

花这么多力气说这个，是因为题主觉得，对于初学者来说，要确切的理解参数的含义，这样才可以根据自己想要实现的动画效果来计算需要传递进去的数值是多少。  

好了，如果我们现在要实现这样一个动画效果，让控件从当前位置沿Y轴平移到距离最初位置200的地方，那么代码该怎么写？

```  
    ValueAnimator animator = ValueAnimator.ofFloat(mView.getTranslationY(),200.0f);
    ...
```  

现在再来实现，很简单，对吧。那么，再来，如果我们要实现，让控件从当前位置沿Y轴平移200呢？  

```  
   ValueAnimator animator = ValueAnimator.ofFloat(mView.getTranslationY() , mView.getTranslationY() + 200.0f );
   ... 

```  

怎么样，想对了吗。注意这里的需求是要相对于当前位置移动200，所以数值要怎么计算明白了吧。  

理解了参数的含义，想要实现各种动画效果就更有可能了。以上，均为题主学习中碰到的问题和自己的理解，如果有错误的地方，还望告知，不然误导了别人可就不好了。  


##ObjectAnimator  

题主是先接触的ValueAnimator，然后才接触ObjectAnimator的，基本的动画效果用这两个都能实现，而且ObjectAnimator实现起来，比ValueAnimator方便多了，反正题主现在是喜欢用ObjectAnimator就是了。  

给你们看下，上面贴出来的代码实现的动画效果，用ObjectAnimator该怎么写：  

```  
       ObjectAnimator animator = ObjectAnimator.ofFloat(mView,"translationY",mView.getTranslationY(),300.0f); 
       animator.setDuration(1000);
       animator.start();

```  
一句代码就搞定，简单多了。虽然简单，但也有几点需要注意的，数值的确定问题跟上面一样，上面理解了这里就可以直接用了，就不再多说了。  

那么，就只剩下第二个参数**"translationY"**这个问题了。它的作用就是指定要实现的是哪个动画属性，说白点，属性动画就是通过不断修改属性值来达到效果的，这点在上面分析的第二点给出的代码上也可以很容易看出来。  

那么，这个属性值到底有哪些，这个字符串的参数可以传递哪些进去？不知道有没有初学者跟题主一样，刚接触时都有这个困惑。  

去网上查找，你会发现，很多大神都给列举出了其他一些取值，比如"alpha"、"rotationX/Y"等等，那么这些值是从哪来的呢？可以看一看[郭神的这一篇](http://blog.csdn.net/guolin_blog/article/details/43816093)。这里就稍微提一下，如果你突然忘记某个动画单词该怎么拼，或者不知道它支不支持使用这个方法，可以利用AS的查看源码方式到View里面去查找一下setXXX()和getXXX()方法，如果有，则支持。  



#Github  

最后附上Demo源代码地址，有兴趣可以看看，代码很粗糙，只是为了理解怎么用而写的，大家就忽略掉这个问题吧。  

[AnimatorDemo:https://github.com/woshidasusu/AnimatorDemo](https://github.com/woshidasusu/AnimatorDemo)  

