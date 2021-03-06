## Overview

Traditionally transitions between different activities or fragments involved enter and exit transitions that animated entire view hierarchies independent to each other. Example of such transitions are a fade transition, slide transition or the newly introduced explode transition.

Default Activity Transition:

![Imgur](https://i.imgur.com/N9m3iBq.gif)

However, many times, there are elements common to both activities and providing the ability to transition these shared elements separately emphasizes continuity between transitions and breaks activity boundaries as the user navigates the app.

![SharedElement](https://i.imgur.com/LydorEa.png)

The nature of this transition forces the human eye to focus on the content and its representation in the new activity instead of the actual activity frame sliding or fading which makes the experience a lot more seamless.

![Imgur](https://i.imgur.com/rUkkfUZ.gif)
![Shared Element Transition](https://i.imgur.com/1cjqsSA.gif)

## Activity Shared Elements Transitions

Note that the shared element transitions require Android 5.0 (API level 21) and above and will be ignored for any lower API versions. Be sure to [check the version at runtime](https://developer.android.com/training/material/compatibility.html#CheckVersion) before using API 21 specific features.

### 1. Enable Window Content Transitions

Enable Window Content Transitions in your `styles.xml` file:

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- Customize your theme here. -->
    <item name="android:windowContentTransitions">true</item>
    ...
</style>
```

### 2. Assign a Common Transition Name

Assign a common transition name to the shared elements in both layouts. Use the   `android:transitionName` attribute.

For e.g. in `MainActivity.xml`:

```xml
<android.support.v7.widget.CardView
  ...>
      <ImageView
          android:id="@+id/ivProfile"
          android:transitionName="profile"
          android:scaleType="centerCrop"
          android:layout_width="match_parent"
          android:layout_height="160dp" />
      ...
</android.support.v7.widget.CardView>
```

In `DetailActivity.xml`:

```xml
<LinearLayout
  ...>
      <ImageView
          android:id="@+id/ivProfile"
          android:transitionName="profile"
          android:scaleType="centerCrop"
          android:layout_width="match_parent"
          android:layout_height="380dp" />
      ...
</LinearLayout>
```

Note that it doesn't matter if the `android:id` is different or where in the layout hierarchy the source and target views exist.

### 3. Start Activity

Start the target activity by specifying a bundle of those shared elements and views from the source.

```java
Intent intent = new Intent(this, DetailsActivity.class);
// Pass data object in the bundle and populate details activity.
intent.putExtra(DetailsActivity.EXTRA_CONTACT, contact);
ActivityOptionsCompat options = ActivityOptionsCompat.
    makeSceneTransitionAnimation(this, (View)ivProfile, "profile");
startActivity(intent, options.toBundle());
```

Thats it! Specifying the source view along with the transition name ensures that even if you have multiple views with the same transition name in the source hierarchy, it will essentially be able to pick the right view to start the animation from.

To reverse the scene transition animation when you finish the second activity, call the `Activity.supportFinishAfterTransition()` method instead of `Activity.finish()`. Also, you will need to override the behavior of the home button in the `ToolBar/ ActionBar` for such cases:

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        // Respond to the action bar's Up/Home button
        case android.R.id.home:
            supportFinishAfterTransition();
            return true;
    }
    return super.onOptionsItemSelected(item);
}
```

### 4. Multiple Shared Elements

Sometimes, you might want to animate multiple elements from the source view hierarchy. This can be achieved by using distinct transition names in the source and target layout xml files.

```java
Intent intent = new Intent(context, DetailsActivity.class);
intent.putExtra(DetailsActivity.EXTRA_CONTACT, contact);
Pair<View, String> p1 = Pair.create((View)ivProfile, "profile");
Pair<View, String> p2 = Pair.create(vPalette, "palette");
Pair<View, String> p3 = Pair.create((View)tvName, "text");
ActivityOptionsCompat options = ActivityOptionsCompat.
    makeSceneTransitionAnimation(this, p1, p2, p3);
startActivity(intent, options.toBundle());
```

**Note:** By default `android.util.Pair` will be imported but we want to select the `android.support.v4.util.Pair` class instead.

Be careful to not overdo transitions between shared elements. While it can make sense to have one cohesive unit animate from one screen to another (which may or may not contain multiple shared elements), having too many shared elements will result in a distracting animation which makes the experience more jarring.

### 5. Customizing Shared Elements Transition

In Android L, shared elements transition defaults to a combination of [ChangeBounds](https://developer.android.com/reference/android/transition/ChangeBounds.html), [ChangeTransform](https://developer.android.com/reference/android/transition/ChangeTransform.html), [ChangeImageTransform](https://developer.android.com/reference/android/transition/ChangeImageTransform.html), and [ChangeClipBounds](https://developer.android.com/reference/android/transition/ChangeClipBounds.html). This works well for most typical cases. However, you may customize this behavior or even define your own custom transition. 

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- enable window content transitions -->
    <item name="android:windowContentTransitions">true</item>

    <!-- specify enter and exit transitions -->
    <!-- options are: explode, slide, fade -->
    <item name="android:windowEnterTransition">@transition/change_image_transform</item>
    <item name="android:windowExitTransition">@transition/change_image_transform</item>

    <!-- specify shared element transitions -->
    <item name="android:windowSharedElementEnterTransition">
      @transition/change_image_transform</item>
    <item name="android:windowSharedElementExitTransition">
      @transition/change_image_transform</item>
</style>
```

The `change_image_transform` transition in this example is defined as follows:

```xml
<!-- res/transition/change_image_transform.xml -->
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
  <changeImageTransform/>
</transitionSet>
```

To enable window content transitions at runtime instead, call the `Window.requestFeature()` method:

```java
// inside your activity (if you did not enable transitions in your theme)
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
// set an enter transition
getWindow().setEnterTransition(new Explode());
// set an exit transition
getWindow().setExitTransition(new Explode());
```

See this official guide on [Defining Custom Animations](https://developer.android.com/training/material/animations.html#Transitions) for more details.

## Fragment Shared Elements Transitions

Leveraging shared element transitions works with fragments too in a similar way as was shown above for activities.

Note that the shared element transitions require Android 5.0 (API level 21) and above and will be ignored for any lower API versions. Be sure to [check the version at runtime](https://developer.android.com/training/material/compatibility.html#CheckVersion) before using API 21 specific features.

### Assign a Common Transition Name

Within two fragments let's assign a common transition name to the shared elements in both layouts. Use the `android:transitionName` attribute and put the view inside both `FirstFragment` and `SecondFragment`:

```xml
<android.support.v7.widget.CardView
  ...>
      <ImageView
          android:id="@+id/ivProfile"
          android:transitionName="profile"
          android:scaleType="centerCrop"
          android:layout_width="match_parent"
          android:layout_height="160dp" />
      ...
</android.support.v7.widget.CardView>
```

### Define the Transition

Add a transition to the `res/transition` folder named `change_image_transform.xml` with the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
    <changeImageTransform />
</transitionSet>
```

### Animate Transition in FragmentTransaction

Now within the activity, we can trigger the transition as part of any `FragmentTransaction`:

```java
// Get access to or create instances to each fragment
FirstFragment fragmentOne = ...;
SecondFragment fragmentTwo = ...;
// Check that the device is running lollipop
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    // Inflate transitions to apply
    Transition changeTransform = TransitionInflater.from(this).
          inflateTransition(R.transition.change_image_transform);
    Transition explodeTransform = TransitionInflater.from(this).
          inflateTransition(android.R.transition.explode);
 
    // Setup exit transition on first fragment
    fragmentOne.setSharedElementReturnTransition(changeTransform);
    fragmentOne.setExitTransition(explodeTransform);

    // Setup enter transition on second fragment
    fragmentTwo.setSharedElementEnterTransition(changeTransform);
    fragmentTwo.setEnterTransition(explodeTransform);

    // Find the shared element (in Fragment A)
    ImageView ivProfile = (ImageView) findViewById(R.id.ivProfile);

    // Add second fragment by replacing first 
    FragmentTransaction ft = getFragmentManager().beginTransaction()
            .replace(R.id.container, fragmentTwo)
            .addToBackStack("transaction")
            .addSharedElement(ivProfile, "profile");
    // Apply the transaction
    ft.commit();
}
else {
    // Code to run on older devices
}
```

Note that we need use methods on the exiting fragment such as `setSharedElementReturnTransition` and `setExitTransition`. On the entering fragment, we call `setSharedElementEnterTransition` and `setEnterTransition`. Finally we need to find the instance of the shared element and then call `addSharedElement(view, transitionName)` as part of building the `FragmentTransaction`. Additional external resources for fragment-to-fragment shared element transitions include:

 * [Android Authority article](http://www.androidauthority.com/using-shared-element-transitions-activities-fragments-631996/) covering the basics
 * [Medium article on fragment shared element transitions](https://medium.com/@bherbst/fragment-transitions-with-shared-elements-7c7d71d31cbb#.gx3casxjb)
 * [Useful stackoverflow post](http://stackoverflow.com/questions/26561579/how-to-start-shared-element-transition-using-fragments/27173907#27173907) for more details
 * [Sample repo with working code](https://github.com/bherbst/FragmentTransitionSample)
 * [More useful sample code](http://stackoverflow.com/a/31800026/313399)

With that you can apply these handy transitions to fragments as well as to activities. 

## References

* <https://developer.android.com/training/material/animations.html>
* <http://java.dzone.com/articles/material-design-activity>
* <https://www.youtube.com/watch?v=97SWYiRtF0Y&t=1403>