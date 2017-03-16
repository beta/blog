title: "Make your BottomSheetDialog noncancelable"
date:  2016-08-30 12:00:00
tags:
- Android
---
Update: I wrote a standalone library for solving this issue. You can check it out at [https://github.com/beta/android-lockable-bottom-sheet][1].

- - -

Android Design Support Library 23.2 comes with the support for [bottom sheets][2] of Material Design, and developers can now create a materialized bottom sheet easily with the help of BottomSheetDialog or BottomSheetDialogFragment. Problem comes: you cannot make a persistent bottom sheet by using the `Dialog#setCancelable` or `DialogFragment#setCancelable` method, so that your bottom sheet dialog will always be canceled or dismissed no matter when the user clicks outside the dialog or when a swipe-down gesture is performed. But there are times when we need to make it stuck there to let the user wait before he/she can dismiss it or perform other actions. Here’s a workaround to make your BottomSheetDialog noncancelable.

Let’s find out what makes our bottom sheets canceled first. As mentioned above there are two ways to cancel a BottomSheetDialog, by clicking outside the dialog or by swiping down on the dialog. A normal Dialog could be set persistent with `setCancelable(false)`, but a BottomSheetDialog could simply not, as it’s implemented in a different approach. You can find a layout resource named *design_bottom_sheet_dialog* in the package of design library, and there is a View component with id set to `touch_outside`, which is apparently used for detecting clicks outside the dialog.

```xml design_bottom_sheet_dialog.xml
<View
    android:id="@+id/touch_outside"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:soundEffectsEnabled="false"/>
```

And there is an OnClickListener set in method `wrapInBottomSheet` of BottomSheetDialog.

```java BottomSheetDialog.java
private View wrapInBottomSheet(int layoutResId, View view, ViewGroup.LayoutParams params) {
    ...
    if (shouldWindowCloseOnTouchOutside()) {
        coordinator.findViewById(R.id.touch_outside).setOnClickListener(
                new View.OnClickListener() {
                    @Override
                    public void onClick(View view) {
                        if (isShowing()) {
                            cancel();
                        }
                    }
                });
    }
}
```

That’s the first way of dismissing your bottom sheet dialog. It’s easy to remove the OnClickListener of view `touch_outside`:

```java
View touchOutsideView = getDialog().getWindow().getDecorView().findViewById(android.support.design.R.id.touch\_outside);
touchOutsideView.setOnClickListener(null);
```

Now when you click outside the sheet, it will not be dismissed anymore. However when you swipe down on the sheet, it will also be canceled. You may have noticed the name of the method I’ve just mentioned, which is wrapInBottomSheet. What is wrapped in a bottom sheet? Where’s the bottom sheet? Well, there’s a FrameLayout named `design_bottom_sheet` in the layout XML file, and the `layout_behavior` attribute is set to `bottom_sheet_behavior`, which is associated with class BottomSheetBehavior bundled in the design library.

```xml design_bottom_sheet_dialog.xml
<FrameLayout
    android:id="@+id/design_bottom_sheet"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal|top"
    android:clickable="true"
    app:layout_behavior="@string/bottom_sheet_behavior"
    style="?attr/bottomSheetStyle"/>
```

The BottomSheetBehavior handles all the touch events of the bottom sheet dialog, and switches the dialog to different state according to the current position and moving direction of the dialog. It provides an abstract class BottomSheetCallback which acts like a listener for the state changing and sliding events. You can find a callback instance at the end of class BottomSheetDialog, which is used to dismiss the dialog when there is a swipe-down gesture performed and the state of the dialog is set to `STATE_HIDDEN`.

```java BottomSheetDialog.java
private BottomSheetBehavior.BottomSheetCallback mBottomSheetCallback
        = new BottomSheetBehavior.BottomSheetCallback() {
    @Override
    public void onStateChanged(@NonNull View bottomSheet,
            @BottomSheetBehavior.State int newState) {
        if (newState == BottomSheetBehavior.STATE_HIDDEN) {
            dismiss();
        }
    }
    
    @Override
    public void onSlide(@NonNull View bottomSheet, float slideOffset) {
    }
};
```

Luckily, neither do we need to overwrite the callback instance, nor we has to try to remove the callback. BottomSheetBehavior provides a convenient method for us, which is `BottomSheetBehavior#setHideable`. To call this method you need to first pass a View object with the `bottom_sheet_behavior` attribute using method `BottomSheetBehavior#from`.

```java
View bottomSheetView = getDialog().getWindow().getDecorView().findViewById(android.support.design.R.id.design_bottom_sheet);
BottomSheetBehavior.from(bottomSheetView).setHideable(false);
```

With `setHideable(false)`, you cannot swipe down to dismiss the bottom sheet when it at the collapsed state. However you can still switch it between the collapsed state and the expanded state.

## Solution

Time to write a new `setCancelable` for your dialog (the code snippet below is used in a derived class of BottomSheetDialogFragment instead of BottomSheetDialog).

```java
@Override
public void setCancelable(boolean cancelable) {
    final Dialog dialog = getDialog();
    View touchOutsideView = dialog.getWindow().getDecorView().findViewById(android.support.design.R.id.touch_outside);
    View bottomSheetView = dialog.getWindow().getDecorView().findViewById(android.support.design.R.id.design_bottom_sheet);
    
    if (cancelable) {
        touchOutsideView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (dialog.isShowing()) {
                    dialog.cancel();
                }
            }
        });
        BottomSheetBehavior.from(bottomSheetView).setHideable(true);
    } else {
        touchOutsideView.setOnClickListener(null);
        BottomSheetBehavior.from(bottomSheetView).setHideable(false);
    }
}
```

[1]: https://github.com/beta/android-lockable-bottom-sheet
[2]: https://material.google.com/components/bottom-sheets.html
