# WidgetLayout 介绍
**WidgetLayout**是一组继承于`ViewGroup`的自定义容器集合，均支持按百分比测绘,限定最大最小宽高，和gravity 属性，自身描边，大部分支持内容分割线，权重等，目前实现了以下实用容器(`kotlin` 实现见[kotlin_master分支][kotlin-widgetlayout]):

1. `ColumnLayout` 以等分列方式布局，每列可设置内容居左，中，右，及铺满，可设置最小最大列宽高限定。
2. `NestRefreshLayout` 一个精简强大的支持任意类型`View`的下拉刷新，上拉加载更多,可添加头部和尾部且可设置悬停模式。
3. `NestFloatLayout` 支持列表的嵌套滑动和指定子 `View` 悬停顶部，类似`NestScrollView` 。
4. `PageScrollView` 可水平垂直方向布局和滑动吸顶等，无需嵌套，支持`ScrollView`和`ViewPager`的交互和接口。
5. `PageScrollTab` 在`PageScrollView`上扩展支持`Tab`场景交互和各种UI定制。
6. `WrapLayout` 支持水平布局，并自适应换行，单行或列时支持`weight`属性，可限定每行最少和最多Item数，行内容可水平和垂直居中。
7. `LabelLayout` 继承自 `WrapLayout`,以`ItemProvider` 方式提供内容，有简单的回收复用机制，有Item点击监听。
8. `HierarchyLayout` 一个展示 `View`树层级关系和工具容器，可画出结束依赖图和3D层级图，计算出平均层级和最近一次`measure,layout,draw`的时间。

以下是各容器实现的结构类图， **[直接看示例请点击](#demo演示效果)[`演示效果`](#demo演示效果)**

![WrapLayout&LabelLayout][classlayer]

# 实现背景及使用场景

**1. 现有系统容器下开发界面经常遇到以下尴尬问题:**
   * 实现特定的组合布局，我们会用功能鲜明的4大常用布局去嵌套实现，**增加了严重OverDraw的机率**；
   * `RelativeLayout` 布局功能强大但是`measure`过程复杂(每次执行`onMeasure` **所有直接子`View`会有两次measure**)
   * 为实现复杂交互工作量比开发业务繁重，比如**滑动控件的悬停或联动**；
   * **分割线或是边界线**很多人常用`View`来实现，即耗内存，又影响测绘时间。
   * **等分布局**，使用多层不同方向`LinearLayout`来实现，嵌套太多。
   * 常用容器控件还没有对自身做最大宽高限定的，也无对子 View 做最大宽高限定。
   
**2. 适合的使用场景举例按需要选择，可减少布局嵌套和复杂的交互代码。**
   + **容器自身或对直接子`View`的最大宽高限定**，容器内容和直接子`View`的`gravity`灵活支持。
   + 容器需要**描边**或子`View`间画**分割线**的，或有各种间距要求，或按下自带激变层效果可使用继承于`PressViewGroup`的容器，如`WrapLayout,LabelLayout,ColumnLayout`。
   + 常见的**表格**布局或**动态等分**布局可用`ColumnLayout`，支持每列的`Align`方式（左中右上下）和全铺满。
   + 水平或垂直布局，标签布局，或需自适应换行可选用`WrapLayout`和`LabelLayout`，可设置行最少最多的`View` 个数和行居中，
   + 列表需要**嵌套滑动**和悬停吸顶可用`NestFloatLayout`，类似NestScrollView 。
   + `PageScrollView` 可取代`ScrollView&HorizontalScrollView` 少嵌套，可设置任意子`View`滑动悬停在开始和结束位置，可不限定子`View`大小像 `ViewPager` 一样选中居中和滑动的交互。
   + `WrapLayout,ColumnLayout`是完全可替代支持不同方向的`LinearLayout`并能提供更多的布局约束，和**背景，描边，分割等额外装饰**。
   + 以上容器都有一个共通的基类，便于统一监控性能打点如`layout`,`measure`过程等。 

# Demo演示效果

`WrapLayout`和`LabelLayout` `ColumnLayout`的演示效果。的演示效果。

![WrapLayout&LabelLayout][wraplabel]
![ColumnLayout][column]

<br/>

`PageScrollView`和`PageScrollTab`的使用示例。

![无需嵌套LinearLayout > scrollview.gif][scrollview]
![ViewPager 模式 > viewpager.gif][viewpager]

<br/>

Demo 入口 和 `NestFloatLayout`的演示效果。

![NestFloatLayout][entry]
![NestFloatLayout][entry]


<br/>

`HiearchyLayout`的静态图，实际是可随手势改变 3D 形态的。

![hierarchyview] [层级3D图]
![hierarchyviewnode] [层级关系树]



# 如何使用：XML 属性和 API 简介

###集成需要提供support-v4包，然后再gradle 脚本中添加依赖如下。 
```
 compile 'com.rexy.android:widgetlayout:1.0.0'
```

### 通用属性说明和介绍
**注;所有xml 中使用自定义属性的地方，请在根标签中加上`xmlns:app="http://schemas.android.com/apk/res-auto"`**

**1. 所有容器自身和子 `View` 对于 `maxWidth,maxHeight,gravity，widthPercent,heightPercent` 支持。**

 a. 容器控件自身标签下使用 `android:gravity,android:maxWidth ,android:maxHeight`,即可支持容器内容的align 属和最大宽与高的限制。
 java 代码可通过 `setGravity` ,`setMaxWidth`,`setMaxHeight` 来支持。

 b. 容器直接子`View`使用`android:layout_gravity,android:maxWidth,android:maxHeight` 即可支持直接子`View`在容器内的`Align`和自身大小的限制。
 java 代码可通过 `BaseViewGroup.LayoutParams lp=(BaseViewGroup.LayoutParams)child.getLayoutParams(); lp.gravity=Gravity.CENTER;lp.maxWidth=100;lp.maxHeight=200`


**2. 部分容器`FloatDrawable`和`DividerMargin`的应用，仅限于继承于`PressViewGroup`的容器**

  a. xml 中使用支持`FloatDrawable`属性和解释如下,java 都有对应的set 和get 方法:

``` xml
  -hover drawable 忽略手势滑动到自身之外取消按下状态-->
  <attr name="ignoreForegroundStateWhenTouchOut" format="boolean"/>
  <!--hover drawable 颜色-->
  <attr name="foregroundColor" format="color"/>
  <!--hover drawable  圆角-->
  <attr name="foregroundRadius" format="dimension"/>
  <!--hover drawable 动画时间-->
  <attr name="foregroundDuration" format="integer"/>
  <!--hover drawable 最小不透明度-->
  <attr name="foregroundAlphaMin" format="integer"/>
  <!--hover drawable 最大不透明度-->
  <attr name="foregroundAlphaMax" format="integer"/>
```

   b. xml 中使用支持`DividerMargin`属性和解释如下,java 都有对应的set 和get 方法:

```xml
  <!--左边线的颜色，宽度，和边线padding-->
  <attr name="borderLeftColor" format="color"/>
  <attr name="borderLeftWidth" format="dimension"/>
  <attr name="borderLeftMargin" format="dimension"/>
  <attr name="borderLeftMarginStart" format="dimension"/>
  <attr name="borderLeftMarginEnd" format="dimension"/>

  <!--上边线的颜色，宽度，和边线padding-->
  <attr name="borderTopColor" format="color"/>
  <attr name="borderTopWidth" format="dimension"/>
  <attr name="borderTopMargin" format="dimension"/>
  <attr name="borderTopMarginStart" format="dimension"/>
  <attr name="borderTopMarginEnd" format="dimension"/>

  <!--右边线的颜色，宽度，和边线padding-->
  <attr name="borderRightColor" format="color"/>
  <attr name="borderRightWidth" format="dimension"/>
  <attr name="borderRightMargin" format="dimension"/>
  <attr name="borderRightMarginStart" format="dimension"/>
  <attr name="borderRightMarginEnd" format="dimension"/>

  <!--下边线的颜色，宽度，和边线padding-->
  <attr name="borderBottomColor" format="color"/>
  <attr name="borderBottomWidth" format="dimension"/>
  <attr name="borderBottomMargin" format="dimension"/>
  <attr name="borderBottomMarginStart" format="dimension"/>
  <attr name="borderBottomMarginEnd" format="dimension"/>

  <!--内容四边的间距，不同于padding -->
  <attr name="contentMarginLeft" format="dimension"/>
  <attr name="contentMarginTop" format="dimension"/>
  <attr name="contentMarginRight" format="dimension"/>
  <attr name="contentMarginBottom" format="dimension"/>
  <!--水平方向和垂直方向Item 的间距-->
  <attr name="contentMarginHorizontal" format="dimension"/>
  <attr name="contentMarginVertical" format="dimension"/>


  <!--水平分割线颜色-->
  <attr name="dividerColorHorizontal" format="color"/>
  <!--水平分割线宽-->
  <attr name="dividerWidthHorizontal" format="dimension"/>
  <!--水平分割线开始和结束padding-->
  <attr name="dividerPaddingHorizontal" format="dimension"/>
  <attr name="dividerPaddingHorizontalStart" format="dimension"/>
  <attr name="dividerPaddingHorizontalEnd" format="dimension"/>

  <!--垂直分割线颜色-->
  <attr name="dividerColorVertical" format="color"/>
  <!--垂直分割线宽-->
  <attr name="dividerWidthVertical" format="dimension"/>
  <!--垂直分割线开始 和结束padding-->
  <attr name="dividerPaddingVertical" format="dimension"/>
  <attr name="dividerPaddingVerticalStart" format="dimension"/>
  <attr name="dividerPaddingVerticalEnd" format="dimension"/>
```


### 具体容器组件的属性和使用介绍

1.`ColumnLayout` xml 属性支持属性如下：java 都有对应的set和get方法就不给示例了，内容分割线间距见以上通用属性。

``` xml
  <!--列个数-->
  <attr name="columnNumber" format="integer" />
  <!--每行内容垂直居中-->
  <attr name="columnCenterVertical" format="boolean"/>

  <!--列内内容全展开的索引 * 或 1,3,5 类似列索引0 开始-->
  <attr name="stretchColumns" format="string" />
  <!--列内内容全靠中间 * 或 1,3,5 类似列索引0 开始-->
  <attr name="alignCenterColumns" format="string" />
  <!--列内内容全靠右 * 或 1,3,5 类似列索引0 开始-->
  <attr name="alignRightColumns" format="string" />

  <!--列宽和高的最大最小值限定-->
  <attr name="columnMinWidth" format="dimension" />
  <attr name="columnMaxWidth" format="dimension" />
  <attr name="columnMinHeight" format="dimension" />
  <attr name="columnMaxHeight" format="dimension" />
```

2.`NestRefreshLayout` 在xml 中可像`ScrollView`一样用，只支持一个内容。可动态设置头部和尾部。

``` java
   //可选，设置头部headerView，true 表示悬停在内容上。
   refreshLayout.setHeaderView(headerView, true);
   
   //可选，设置尾部footerView，false 表示不需要悬停在内容上。
   refreshLayout.setFooterView(footerView, false);
   
   //可选，设置覆盖内容的蒙层 View。
   refreshLayout.setMaskView(maskView, true);
   
   //设置下拉刷新的指示器。
   refreshLayout.setRefreshPullIndicator(new DefaultRefreshIndicator(getActivity()));
   //设置上拉加载更多的指示器。
   refreshLayout.setRefreshPushIndicator(new DefaultRefreshIndicator(getActivity()));
   
   设置下拉或上推的状态和刷新动作的监听
   mRefreshLayout.setOnRefreshListener(new NestRefreshLayout.OnRefreshListener() {
      @Override
      public void onRefreshStateChanged(NestRefreshLayout parent, int state, int preState, int moveAbsDistance) {
      }
      @Override
      public void onRefresh(NestRefreshLayout parent, boolean refresh) {
      }
   });
```

3.`NestFloatLayou` xml 属性支持属性和 java 代码如下：

``` xml
 <!--实现了嵌套滑动NestScrollingChild 接口的滑动的 View 所在的直接子 View 索引-->
 <attr name="nestViewIndex" format="integer"/>```
 <!--需要吸顶到顶部的 View 所在的直接子 View 索引-->
 <attr name="floatViewIndex" format="integer"/>`java`代码可如下设置：
```

``` java
 mLastFloatLayout.setNestViewId(viewId);
 mLastFloatLayout.setFloatViewId(viewId);
 或
  mLastFloatLayout.setNestViewIndex(viewIndex);
  mLastFloatLayout.setFloatViewIndex(viewIndex);
```

4.`PageScrollView,PageScrollTab` 使用.

a. 支持的xml 属性，对应都有java 相应的set 和 get;

```xml
  <!--布局方向，也决定了手势方向，仅支持水平和垂直之一。-->
  <attr name="layoutDirection"/>
  <!--滑动交互 ViewPager 方式-->
  <attr name="viewPagerStyle" format="boolean"/>
  <!--所有的child居中-->
  <attr name="childCenter" format="boolean"/>
  <!--所有的child填充整个父容器-->
  <attr name="childFillParent" format="boolean"/>
  <!--内容item 的间距-->
  <attr name="middleMargin" format="dimension"/>
  <!--item 的size 按父容器的size 百分比-->
  <attr name="sizeFixedPercent" format="float"/>
  <!--快速滑动松手后的回弹距离-->
  <attr name="overFlingDistance" format="dimension"/>
  <!--滑动悬停到开始位置的child 索引-->
  <attr name="floatViewStartIndex" format="integer"/>
  <!--滑动悬停到结束位置的child 索引-->
  <attr name="floatViewEndIndex" format="integer"/>
```

b.java 其它接口设置

``` java
//接着上面
  mPageScrollView.setPageHeadView(headerView); //设置头部 View
  mPageScrollView.setPageFooterView(footerView); 设置尾部 View
  //设置 PageTransformer 动画，实现滑动视图的变换。
  mPageScrollView.setPageTransformer(new PageScrollView.PageTransformer() {
  @Override
  public void transformPage(View view, float position, boolean horizontal) {
  //在这里根据滑动相对偏移量 position,实现该视图的动画效果。
  }
  @Override
  public void recoverTransformPage(View view, boolean horizontal) {
  //清除视图的动画效果，在setPageTransformer(null)时会调用。
  }
  });
  PageScrollView.OnPageChangeListener pagerScrollListener = new PageScrollView.OnPageChangeListener() {
  @Override
  public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
  // ViewPager 滑动视图时，相对偏移适时回调。
  }
  @Override
  public void onPageSelected(int position, int oldPosition) {
  // ViewPager 模式时 选中回调。
  }
  @Override
  public void onScrollChanged(int scrollX, int scrollY, int oldScrollX, int oldScrollY) {
  //视图滑动回调 View.onScrollChanged
  }
  @Override
  public void onScrollStateChanged(int state, int oldState) {
  //state 的取值如下，标明着容器的滑动状态。
  // SCROLL_STATE_IDLE = 0; // 滑动停止状态。
  // SCROLL_STATE_DRAGGING = 1;//用户正开始拖拽滑动 。
  // SCROLL_STATE_SETTLING = 2;//开始松开手指快速滑动。
  }
  };
  mPageScrollView.setOnPageChangeListener(pagerScrollListener);
  // 设置视图滚动的监听。
  mPageScrollView.setOnScrollChangeListener(pagerScrollListener);
  //设置可见子 View 发生变化时 可见索引区间的监听。
  mPageScrollView.setOnVisibleRangeChangeListener(new OnVisibleRangeChangeListener(){
      public void onVisibleRangeChanged(int firstVisible, int lastVisible, int oldFirstVisible, int oldLastVisible){
      }
  });
  //设置动画初始化滑动到第二个 View ，-1 表示动画时间内部计算，如无需动画传0
  mPageScrollView.scrollTo(1,0,-1);
```


c.`PageScrollTab` 继承于 `PageScrollView` ，额外支持以下xml 属性（java 均有get 和set 对应）。

```xml
 <!--tab item 的背景-->
   <attr name="tabItemBackground" format="reference"/>
   <attr name="tabItemBackgroundFirst" format="reference"/>
   <attr name="tabItemBackgroundLast" format="reference"/>
   <attr name="tabItemBackgroundFull" format="reference"/>

   <!--底部指示线-->
   <attr name="tabIndicatorColor" format="color"/>
   <attr name="tabIndicatorHeight" format="dimension"/>
   <attr name="tabIndicatorOffset" format="dimension"/>
   <attr name="tabIndicatorWidthPercent" format="float"/>

   <!--顶部水平分界线-->
   <attr name="tabTopLineColor" format="color"/>
   <attr name="tabTopLineHeight" format="dimension"/>

   <!--底部水平分界线-->
   <attr name="tabBottomLineColor" format="color"/>
   <attr name="tabBottomLineHeight" format="dimension"/>

   <!-- item 之间垂直分割线-->
   <attr name="tabItemDividerColor" format="color"/>
   <attr name="tabItemDividerWidth" format="dimension"/>
   <attr name="tabItemDividerPadding" format="dimension"/>


   <!-- item 的最小 Padding 设置-->
   <attr name="tabItemMinPaddingHorizontal" format="dimension"/>
   <attr name="tabItemMinPaddingTop" format="dimension"/>
   <attr name="tabItemMinPaddingBottom" format="dimension"/>

   <!--item文字大写开-->
   <attr name="tabItemTextCaps" format="boolean"/>

   <!--item 文字颜色-->
   <attr name="tabItemTextColor" format="reference"/>
```

java 额外的接口：

``` java
  //设置ItemProvider，初始化选中第0 个索引， 类似上面的 LabelLayout 的初始化。
  mPageScrollTab.setTabProvider(mItemProvider,0);
  mPageScrollTab.setTabClickListener(new PageScrollTab.ITabClickEvent() {
      @Override
      public boolean onTabClicked(PageScrollTab parent, View cur, int curPos, View pre, int prePos) {
          return false;
      }
  });
```




5.`WrapLayout` xml 属性支持属性如下：java 都有对应的set 和get 方法就不给示例了。当作`LinearLayout`如果要支持`weight`布局，需要设置 `setSupportWeight`为true见`weightSupport`.
且垂直布局需设置lineMaxItemCount为1，水平布局时lineMinItemCount可不设，也可设置为一个大于或等于子视图数的值或设置小于1 的值. 只有这样才可以应用权重去做垂直或是水平的布局。

``` xml
  <!--每行内容水平居中-->
  <attr name="lineCenterHorizontal" format="boolean"/>
  <!--每行内容垂直居中-->
  <attr name="lineCenterVertical" format="boolean"/>

  <!--每一行最少的Item 个数-->
  <attr name="lineMinItemCount" format="integer"/>
  <!--每一行最多的Item 个数-->
  <attr name="lineMaxItemCount" format="integer"/>
  
  <!-- 支持weight 属性，前提条件是单行或单列的情况-->
  <attr name="weightSupport" format="boolean" />
```

6.`LabelLayout` 继承`WrapLayout`有其所有功能接口。

  不同是支持 `android:textSize,android:textColor` 在xml 中设置Label 的字号和字色。同样可用java 代码设置字号字色。
  使用可通过`ItemProvider` 接口来初始化 `Label`,本工程中的示例初始化如下。

  ``` java
  final String[] mLabels = new String[]{
          "A", "B", "C", "D", "E", "F", "G", "H"
  };
  labelLayout.setItemProvider(new ItemProvider.ViewProvider() {
      @Override
      public int getViewType(int position) {
          return 0;
      }

      @Override
      public View getView(int position, View convertView, ViewGroup parent) {
          return buildView(getTitle(position),true);
      }

      @Override
      public CharSequence getTitle(int position) {
          return mLabels[position];
      }

      @Override
      public Object getItem(int position) {
          return mLabels[position];
      }

      @Override
      public int getCount() {
          return mLabels == null ? 0 : mLabels.length;
      }
  });
  final LabelLayout.OnLabelClickListener mLabelClicker = new LabelLayout.OnLabelClickListener() {
      @Override
      public void onLabelClick(LabelLayout parent, View labelView) {
          Object tag = labelView.getTag();
          CharSequence text = tag == null ? null : String.valueOf(tag);
          if (text == null && labelView instanceof TextView) {
              text = ((TextView) labelView).getText();
          }
          if (text != null) {
              Toast.makeText(getActivity(), text, Toast.LENGTH_SHORT).show();
          }
      }
  };
  mLabelLayout.setOnLabelClickListener(mLabelClicker);
  ```

# 最后感谢大家关注和多提建议或是**issue**，如果很感兴趣请 star 和 follower .

[classlayer]:image/classlayer.jpg "NestFloatLayout 的示例"
[scrollview]:image/example_scrollview.gif "ScrollView type but no need to nest a single ViewGroup,just use as a LinearLayout"
[viewpager]:image/example_viewpager.gif "ViewPager type but not support PageAdapter"
[wraplabel]:image/wraplabel.gif "WrapLayout 和 LabelLayout 的示例"
[column]:image/column.gif "ColumnLayout 的示例"
[nestlist]:image/nestlist.gif "NestFloatLayout 的示例"
[entry]:image/entry.jpg " 演示入口"
[hierarchyview]:image/hierarchyView.jpeg " 演示入口"
[hierarchyviewnode]:image/hierarchyViewNode.jpeg " 演示入口"
[kotlin-widgetlayout]:https://github.com/rexyren/WidgetLayout/tree/kotlin_master "kotlin project"