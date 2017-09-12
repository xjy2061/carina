+++
date = "2017-09-10T20:09:00+08:00"
title = "Android memory leak"
draft = false
tags = ["Android", "Memory leak"]
categories = [ "Post" ]
+++

## Leak because of system bug

### `PhoneStateListener` leak
Below 7.0 a non static inner class `IPhoneStateListener.Stub` callback in `PhoneStateListener` references to outside `PhoneStateListener`, even caller has been destroyed and "un-registered" the `PhoneStateListener`, the references coming from: Native Stack --> PhoneStateListener --> Context(Activity).

A wrapper class wraps a weak reference of `PhoneStateListener` can be used to avoid this memory leak.

> Caution: The original `PhoneStateListener` object must be referenced by caller class, otherwise the weak reference in the wrapper class will get `null`.

```java
public class PhoneStateListenerWrapper extends PhoneStateListener {
    private WeakReference<PhoneStateListener> mPhoneStateListenerRef;

    public PhoneStateListenerWrapper(PhoneStateListener phoneStateListener) {
        mPhoneStateListenerRef = new WeakReference<>(phoneStateListener);
    }

    @Override
    public void onCallStateChanged(int state, String incomingNumber) {
        super.onCallStateChanged(state, incomingNumber);
        PhoneStateListener phoneStateListener = mPhoneStateListenerRef.get();
        if (phoneStateListener != null) {
            phoneStateListener.onCallStateChanged(state, incomingNumber);
        }
    }
}

private PhoneStateListener mPhoneStateListener = new PhoneStateListener() {
    @Override
    public void onCallStateChanged(int state, String incomingNumber) {
        super.onCallStateChanged(state, incomingNumber);
        //...
    }
};
private PhoneStateListenerWrapper mPhoneStateListenerWrapper = new PhoneStateListenerWrapper(mPhoneStateListener);

((TelephonyManager) mContext.getSystemService(Context.TELEPHONY_SERVICE)).listen(mPhoneStateListenerWrapper, PhoneStateListener.LISTEN_CALL_STATE);
((TelephonyManager) mContext.getSystemService(Context.TELEPHONY_SERVICE)).listen(mPhoneStateListenerWrapper, PhoneStateListener.LISTEN_NONE);
```

### `AudioManager` leak
Below 6.0 `AudioManager` reference to the incoming context of constructor directly, the related issue is [here](https://issuetracker.google.com/issues/37024105).

A simple workaround is using `Application` context to get `AudioManager`.

```java
mAudioManager = (AudioManager) context.getApplicationContext().getSystemService(Context.AUDIO_SERVICE);
```

### `InputMethodManager` leak
Refer to [issue 37043700](https://issuetracker.google.com/issues/37043700) and [issue 37090736](https://issuetracker.google.com/issues/37090736), `mCurRootView` or `mServedView` or `mNextServedView` in `InputMethodManager` may reference to last focused view on a wide system version range. But I have not found this kind of leak in official system. However, `mLastSrvView` of `InputMethodManager` may cause leak on huawei 6.0 and above device, and `mLastSrvView` field is not found in official source code, I think this is a leak for certain manufacturer.

To fix this kind of leak, a straight way is set the relative field of `InputMethodManager` to `null` when activity destroyed. Considering there may be multiple activities have this leak, a better way is fix this leak in the `onActivityDestroyed(Activity activity)` method of `ActivityLifecycleCallbacks`.

```java
@Override
public void onActivityDestroyed(Activity activity) {
	MemoryLeakFixer.fixInputMethodManagerLeak(activity);
}

public class MemoryLeakFixer {

    public static void fixInputMethodManagerLeak(Activity activity) {
        //Surround with a appropriate if condition to make sure execute fix when leak real exists is better.
        InputMethodManager inputMethodManager = null;
        try {
            inputMethodManager = (InputMethodManager) activity.getApplicationContext().getSystemService(Context.INPUT_METHOD_SERVICE);
        } catch (Throwable t) {
            t.printStackTrace();
        }
        if (inputMethodManager!= null) {
            String[] fieldNames = new String[]{"mCurRootView", "mServedView", "mNextServedView", "mLastSrvView"/*huawei*/};
            Class clazz = inputMethodManager.getClass();
            for (String fieldName : fieldNames) {
                try {
                    Field field = clazz.getDeclaredField(fieldName);
                    if (field != null) {
                        if (!field.isAccessible()) {
                            field.setAccessible(true);
                        }
                        Object obj = field.get(inputMethodManager);
                        if (obj instanceof View) {
                            View view = (View) obj;
                            if (view.getContext() == activity) {
                                field.set(inputMethodManager, null);
                            }
                        }
                    }
                } catch (Throwable t) {
                    t.printStackTrace();
                }
            }
        }
    }
}
```

### `TextLine` leak
An activity context may be referenced by `sCached` in `TextLine`, a simple solution is set every element of `sCached` to `null` in activity's `onDestroy` method.

```java
public static void fixTextLineLeak() {
    try {
        Field field = Class.forName("android.text.TextLine").getDeclaredField("sCached");
        field.setAccessible(true);
        Object[] sCached = (Object[]) field.get(null);
        if (sCached != null) {
            for (int i = 0; i < sCached.length; i++) {
                sCached[i] = null;
            }
        }
    } catch (Throwable t) {
        t.printStackTrace();
    }
}
```

## Leak because of misuse

### Unremoved `ViewTreeObserver` listener leak
The scenario is after create a new item view for a `ViewPager` by the `instantiateItem` method of `PageAdapter`, a listener reference to the item view indirectly have been add to the `ViewTreeObserver` of the item view, and the listener never be removed from the `ViewTreeObserver`, cause the item view leak even the view has been removed from the view hierarchy in the `destroyItem` method of `PageAdapter`.

**Never forgot remove listeners from `ViewTreeObserver`.**

```java
final ViewTreeObserver observer = view.getViewTreeObserver();
if (observer != null) {
	observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
	    @Override
	    public void onGlobalLayout() {
    	    //do something
			if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
				observer.removeOnGlobalLayoutListener(this);
			} else {
				observer.removeGlobalOnLayoutListener(this);
			}
		}
	});
}
```
