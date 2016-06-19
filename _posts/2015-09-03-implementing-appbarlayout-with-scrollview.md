---
title: Implementing AppbarLayout scrolling behavior with ScrollView
date: 2015-09-03
published: true
---

As there were some bugs in AppbarLayout scrolling behavior, I got the requirement to implement same behavior with scroll view. So here is how we can do it.

### Expected Output

![](/public/images/10.gif){: .center-image }

**Step 1: Not reinventing the wheel**


[This](https://github.com/ksoichiro/Android-ObservableScrollView) library is very good as starting point for our final goal, and [here](https://github.com/ksoichiro/Android-ObservableScrollView/blob/master/docs/basic/flexible-space-with-image.md) you will find detail explanation about how to create this.

![](/public/images/2.gif){: .center-image }

Let's see the basic structure of the this implementation.

{% highlight xml%}
<FrameLayout>

    <ImageView
        android:id="@+id/image"
        android:layout_height="240dp" />

    <View
        android:id="@+id/overlay"
        android:layout_height="240dp" />

    <ObservableScrollView android:id="@+id/scroll">

        <LinearLayout android:orientation="vertical">

            <View
                android:id="@+id/paddingtop"
                android:layout_height=“240dp”/>

            <TextView android:id="@+id/main_content" />
        </LinearLayout>
    </ObservableScrollView>

    <LinearLayout
        android:id="@+id/titletext_container"
        android:orientation="vertical">
        android:paddingLeft="@dimen/margin_standard">

        <TextView
            android:id="@+id/title"
            android:minHeight="?attr/actionBarSize" />

        <View
            android:id="@+id/animating_area"
            android:minHeight="184dp" />
    </LinearLayout>

    <FloatingActionButton android:id="@+id/fab" />
</FrameLayout>
{% endhighlight %}

  * *FrameLayout* is used for moving children separately.
  * *ImageView(@id/image)* is the image that will be translated with
    parallax effect.
  * *View(@id/overlay)* is an overlay view.You can see the image is fading in
    and out. This view overlaps with the image and its opacity is changed by scroll position.

![Output](/public/images/3.png){: .center-image }

  * *ObservableScrollView* provide vertical scroll value which is used to
    scroll other views
  * *LinearLayout* holds the actual content.
  * *View(@id/paddingtop)* provide top padding of 240dp so that main content
    start below header view.
  * *TextView(@id/main_content)* this represent main content below header.

![Output](/public/images/4.png){: .center-image }

  * *LinearLayout(@id/titletext_container)* holds title text and the
    animating area
  * *TextView(@id/titletext)* holds title text which will be scaled and translated on top of the action bar, its minimum  height is equal to action bar height.
  * *View(@id/animating_space)* This view provides space for title text
    to animate. its height is (header height - action bar height).

![Output](/public/images/5.png){: .center-image }

**Step 2: Identifying what need to be done**

  * The toolbar should stick at the top.
  * Title text should stick at the top.
  * The status bar should be transparent initially and fade in with the primary color.
  * Image View should start below the status bar.

**Step 3: Toolbar should stick at top**

  * As Toolbar should be on top of scrolling content, so place it below ObservableScrollView. Also make toolbar transparent initially.

  {% highlight xml %}
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/transparent"
        android:minHeight="?attr/actionBarSize"
        app:popupTheme="@style/Theme.AppCompat.Light.DarkActionBar"
        app:theme="@style/Toolbar"
      />
  {% endhighlight %}

  ![Output](/public/images/100.gif){: .center-image }

  * As we can see title text is overlapping with up icon, this can be corrected by adding left padding of 56dp(toolbar home icon width) to Title text.

  {% highlight xml %}
  <LinearLayout
        android:paddingLeft="@dimen/toolbar_margin_start">
        <TextView />
        <View />
     </LinearLayout>
  {% endhighlight %}

 ![Output](/public/images/54.gif){: .center-image }

  * If user scrolled the content more then flexibleRange, change the Toolbar color to primary else keep it transparent.

 {% highlight xml %}
  @Override
    public void onScrollChanged(int scrollY, boolean firstScroll, boolean dragging) {
        float flexibleRange = mFlexibleSpaceImageHeight - mActionBarSize;
        if(scrollY>=flexibleRange){
            mToolbarView.setBackgroundColor(getResources().getColor(R.color.primary));
        }
        else{
            mToolbarView.setBackgroundColor(Color.TRANSPARENT);
        }
    }
 {% endhighlight %}

   ![Output](/public/images/53.gif){: .center-image }

  **Step 4: Status bar color change**

  * From our final output we can see that initially we need to set status bar color as transparent, so in onCreate we can set below code.

    {% highlight xml %}
      getWindow().setStatusBarColor(Color.TRANSPARENT);
    {% endhighlight %}


But this will result in following output

  ![Output](/public/images/97.png){: .center-image.normal }


  Basically we need to move image below status bar, this can be done with below code

  {% highlight xml %}
    getWindow().getDecorView().setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
  {% endhighlight %}

  ![Output](/public/images/95.png){: .center-image.normal }

  You can see Toolbar overlapped with status bar, we need to set top padding to toolbar and it should be of status bar height.

  {% highlight xml %}
     mToolbarView.setPadding(0, getStatusBarHeight(), 0, 0);
  {% endhighlight %}

  ![Output](/public/images/toolbar_padding.png){: .center-image.normal }

  But this cause another issue, i.e Toolbar color change will be visible with a flicker as shown below.

 ![Output](/public/images/94.gif){: .center-image }

 This is because we increased action height by adding extra padding but didn't updated mActionBarSize value so add following

 {% highlight xml %}
    mActionBarSize = getActionBarSize() + getStatusBarHeight();
 {% endhighlight %}

  ![Output](/public/images/92.gif){: .center-image }


  **Step 5: Title text should stick at top**

   * Translate Title text till it reaches status bar height, this will stick textview at top.

  *Instead of*
       {% highlight xml %}                                      
         int titleTranslationY = maxTitleTranslationY - scrollY;
       {% endhighlight %}

  *Do this*

       {% highlight xml %}                                      
         int titleTranslationY = Math.max(maxTitleTranslationY - scrollY,getStatusBarHeight());  
       {% endhighlight %}    


  And hear is our final output

   ![Output](/public/images/91.gif){: .center-image.normal }

   You can find the code [here](https://github.com/shekarrex/AppbarLayoutBehaviorWithScrollView.git)


   <br>
