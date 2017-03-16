title: "Android 实现心跳动画"
date:  2015-11-30 12:00:00
---
## 踩到巨坑

今天需要为一个 ImageView 实现心跳的效果。所谓心跳效果，就是每隔 1s 左右进行一次先收缩再扩张的动画。想到这是一个 view animation，就在 `/res/anim` 中建立了一个 `heart_beat.xml`，写了类似这样的一些东西：

```xml
<set
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:interpolator/accelerate_decelerate">
    
    <scale
        android:duration="100"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fromXScale="1.0"
        android:fromYScale="1.0"
        android:toXScale="0.8"
        android:toYScale="0.8"
        android:repeatCount="1"
        android:repeatMode="reverse" />

</set>
```

收缩再扩张的效果是没问题了，但是图片一直抽搐个不停，要怎么在两次动画之间加上间隔时间呢？

于是修改成了这样：

```xml
<set
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:interpolator/accelerate_decelerate"
    android:ordering="sequentially"
    android:repeatCount="infinite">
    
    <scale
        android:startOffset="1200"
        android:duration="100"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fromXScale="1.0"
        android:fromYScale="1.0"
        android:toXScale="0.8"
        android:toYScale="0.8" />
    
    <scale
        android:duration="100"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fromXScale="0.8"
        android:fromYScale="0.8"
        android:toXScale="1.0"
        android:toYScale="1.0" />

</set>
```

`<set>` 中的两个 `<scale>` 分别对应收缩和扩张的动画，其中第一个 `<scale>` 延时 1.2s，再将 `<set>` 设为顺序播放 + 无限循环，看起来应该没什么问题，但是运行的时候发现不仅动画的顺序错了，循环根本就没有生效。

翻了一下文档，发现 `ordering` 是 Animator 而不是 Animation 的属性，而 Animator 对应的是 property animation。再加上 [AnimationSet 的文档里][1]说 `repeatCount` 对 AnimationSet 无效，好吧，改用 Animator 重写一下应该就没问题了吧。

在 `/res/animator` 中建立一个 `heart_beat.xml`，又写了一些类似这样的东西：

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="sequentially"
    android:interpolator="@android:interpolator/accelerate_decelerate">
    
    <set>
        <objectAnimator
            android:propertyName="scaleX"
            android:valueFrom="1.0"
            android:valueTo="0.8"
            android:duration="100" />
        <objectAnimator
            android:propertyName="scaleY"
            android:valueFrom="1.0"
            android:valueTo="0.8"
            android:duration="100" />
    </set>
    
    <set
        android:startOffset="1200">
        <objectAnimator
            android:propertyName="scaleX"
            android:valueFrom="0.8"
            android:valueTo="1.0"
            android:duration="100" />
        <objectAnimator
            android:propertyName="scaleY"
            android:valueFrom="0.8"
            android:valueTo="1.0"
            android:duration="100" />
    </set>

</set>
```

似乎没什么大问题了，收缩和扩张的顺序没错，间隔时间也差不多。但是在收缩和扩张的时候，为什么横向和纵向不是一起缩放的呢？说好的 `<set>` 的默认播放顺序是 `together` 呢？

[ObjectAnimator 的文档里][2]说可以在 `<objectAnimator>` 里用 `<propertyValuesHolder>` 来让多个属性同时发生变化，就试着写了一下，然后直接被丢了个错误说没有 `<propertyValuesHolder>` 这个标签……

## Programmatically

所以还是老老实实地用代码来写吧。仍然是之前的方法，用多个 PropertyValuesHolder 来同时改变对象的多个值，再用 ObjectAnimator 封装好各个动画，最后用 AnimatorSet 将多个动画连接起来。

```java
PropertyValuesHolder pvhIncreaseScaleX =
        PropertyValuesHolder.ofFloat("scaleX", 1.0f, 0.8f);
PropertyValuesHolder pvhIncreaseScaleY =
        PropertyValuesHolder.ofFloat("scaleY", 1.0f, 0.8f);
PropertyValuesHolder pvhDecreaseScaleX =
        PropertyValuesHolder.ofFloat("scaleX", 0.8f, 1.0f);
PropertyValuesHolder pvhDecreaseScaleY =
        PropertyValuesHolder.ofFloat("scaleY", 0.8f, 1.0f);

ObjectAnimator heartBeatIncreaseAnimator = ObjectAnimator.ofPropertyValuesHolder(
        mImageView, pvhIncreaseScaleX, pvhIncreaseScaleY
);
heartBeatIncreaseAnimator.setStartDelay(1200);
heartBeatIncreaseAnimator.setDuration(100);
heartBeatIncreaseAnimator.setInterpolator(new AccelerateDecelerateInterpolator());

ObjectAnimator heartBeatDecreaseAnimator = ObjectAnimator.ofPropertyValuesHolder(
        mImageView, pvhDecreaseScaleX, pvhDecreaseScaleY
);
heartBeatDecreaseAnimator.setDuration(100);
heartBeatDecreaseAnimator.setInterpolator(new AccelerateDecelerateInterpolator());

AnimatorSet heartBeatAnimatorSet = new AnimatorSet();
heartBeatAnimatorSet.playSequentially(heartBeatIncreaseAnimator, heartBeatDecreaseAnimator);
heartBeatAnimatorSet.start();
```

然而 AnimatorSet 还是不能循环播放，不过可以通过监听 AnimatorSet 的结束事件，间接地实现循环。

这么长的一坨代码丢在 Activity 里肯定不好，而且考虑到复用的话，封装成一个工具类似乎不错：

```java
import android.animation.Animator;
import android.animation.AnimatorSet;
import android.animation.ObjectAnimator;
import android.animation.PropertyValuesHolder;
import android.view.View;
import android.view.animation.AccelerateDecelerateInterpolator;

/\*\*
 * An utility class for playing "heart beat" animation on a certain view object.
 \*/
public class HeartBeatAnimationUtil {

    private View mTarget;
    private long mDuration = 100;
    private long mDelay = 1200;
    private float mFromScale = 1.0f;
    private float mToScale = 0.8f;
    
    private HeartBeatAnimationUtil(View target) {
        mTarget = target;
    }
    
    /**
     * Creates a new instance and specifies a View to apply animation.
     *
     * @param target View to apply animation
     * @return The instance itself
     */
    public static HeartBeatAnimationUtil with(View target) {
        return new HeartBeatAnimationUtil(target);
    }
    
    /**
     * Specifies the duration.
     *
     * @param duration Duration of the animation
     * @return The instance itself
     */
    public HeartBeatAnimationUtil in(long duration) {
        mDuration = duration;
        return this;
    }
    
    /**
     * Specifies the time of delay.
     *
     * @param delay Time of delay between two animations
     * @return The instance itself
     */
    public HeartBeatAnimationUtil after(long delay) {
        mDelay = delay;
        return this;
    }
    
    /**
     * Sets the original scale.
     *
     * @param fromScale The original scale of the view
     * @return The instance itself
     */
    public HeartBeatAnimationUtil scaleFrom(float fromScale) {
        mFromScale = fromScale;
        return this;
    }
    
    /**
     * Sets the targeted scale.
     *
     * @param toScale The targeted scale of the view
     * @return The instance itself
     */
    public HeartBeatAnimationUtil scaleTo(float toScale) {
        mToScale = toScale;
        return this;
    }
    
    /**
     * Starts the animation.
     */
    public void start() {
        PropertyValuesHolder pvhIncreaseScaleX =
                PropertyValuesHolder.ofFloat("scaleX", mFromScale, mToScale);
        PropertyValuesHolder pvhIncreaseScaleY =
                PropertyValuesHolder.ofFloat("scaleY", mFromScale, mToScale);
        PropertyValuesHolder pvhDecreaseScaleX =
                PropertyValuesHolder.ofFloat("scaleX", mToScale, mFromScale);
        PropertyValuesHolder pvhDecreaseScaleY =
                PropertyValuesHolder.ofFloat("scaleY", mToScale, mFromScale);
    
        ObjectAnimator heartBeatIncreaseAnimator = ObjectAnimator.ofPropertyValuesHolder(
                mTarget, pvhIncreaseScaleX, pvhIncreaseScaleY
        );
        heartBeatIncreaseAnimator.setStartDelay(mDelay);
        heartBeatIncreaseAnimator.setDuration(mDuration);
        heartBeatIncreaseAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
    
        ObjectAnimator heartBeatDecreaseAnimator = ObjectAnimator.ofPropertyValuesHolder(
                mTarget, pvhDecreaseScaleX, pvhDecreaseScaleY
        );
        heartBeatDecreaseAnimator.setDuration(mDuration);
        heartBeatDecreaseAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
    
        AnimatorSet heartBeatAnimatorSet = new AnimatorSet();
        heartBeatAnimatorSet.playSequentially(heartBeatIncreaseAnimator, heartBeatDecreaseAnimator);
        heartBeatAnimatorSet.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {
            }
    
            @Override
            public void onAnimationEnd(Animator animation) {
                animation.start();
            }
    
            @Override
            public void onAnimationCancel(Animator animation) {
            }
    
            @Override
            public void onAnimationRepeat(Animator animation) {
            }
        });
        heartBeatAnimatorSet.start();
    }

}
```

简单地实现了一下流畅接口，调用方法是：

```java
HeartBeatAnimationUtil.with(mImageLogo).start();

HeartBeatAnimationUtil.with(mImageLogo)
        .scaleFrom(1.0f)
        .scaleTo(0.8f)
        .in(100)
        .after(1200)
        .start();
```

[1]: https://developer.android.com/reference/android/view/animation/AnimationSet.html
[2]: https://developer.android.com/reference/android/animation/ObjectAnimator.html
