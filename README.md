# Inthegame SmartTV - SDK for Android TV, Fire TV, and Android mobile

This SDK allows you to easily integrate the In The Game engagement platform in a Smart TV or mobile app.\
It is compatible with Amazon Fire TV, Android TV, and Android mobile phones.\
The repository includes an example app that shows how to use the framework. The TV example app is based on Android's Leanback TV examples.


## Installation

In your Android project, choose **File > New > New module > Import from .aar file**. 

Select the **inthegametv.aar** file (included in this repo).

Add the following imports (if missing) to your app's build.gradle:

```
    implementation project(':inthegametv')
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation 'androidx.core:core-ktx:1.3.2'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.squareup.picasso:picasso:2.71828'
    implementation ('io.socket:socket.io-client:1.0.1') {
        exclude group: 'org.json', module: 'json'
    }
```

## Usage

To display interactive content, you insert the ITGOverlayView over your video player.
Define a variable:

```
private var mOverlay: ITGOverlayView? = null
```

And configure it on the fragment's `onViewCreated`:
```
val overlay = ITGOverlayView(context)
overlay.load("<your_channel_id>", "<your_broadcaster_name>")
overlay.listener = this
(view as ViewGroup).addView(overlay)
mOverlay = overlay
```

The overlay needs to be informed whenever the video is paused, starts playing, or if the user seeks to a different time position, so that it knows when to show content.\
So you call these methods when the video player state is changed:
```
mOverlay?.videoPlaying(time)
mOverlay?.videoPaused()
```

To allow the user to close interactions with the back button, you pass your back button events to the overlay.\
The method will return true if the overlay uses the event to close content. If false, you should perform your app's default back action.
```
val usedEvent = mOverlay?.handleBackPressIfNeeded() == true
```
Some interactive content requires a pause in the video. To use this feature, override the ITGOverlayView's listener methods to detect when it requests these events.
```
fun overlayRequestedPause()
fun overlayRequestedPlay()
```

The overlay will request focus when showing content, so that the user can interact with the buttons. If the focus transfer is not automatic on your setup, use these listener methods to set the focus accordingly.\
`focusView` is the overlay element that should be focused. `popMessage` indicates whether this is a good time to show user engagement content.

```
fun overlayRequestedFocus(focusView: View)
fun overlayReleasedFocus(popMessage: Boolean)
```

If you want to customize the frequency of the `popMessage` variable (in terms of how many times user must answer content to trigger it) you can configure `overlay.infoPopupFrequencySequence` .

There is an optional button to show external content on custom interfaces, if you wish to use it this is the listener method:
```
fun overlayClickedUserArea()
```

If, during runtime, you want to check if an activity is showing, or close the current activity, these methods are available:
```
overlay.isDisplayingInteraction()
overlay.closeCurrentInteraction()
```
       
If you want to connect ITG content with your user's account, there are variables in the `load()` method where you can specify your user's ID and display name. The variables are called `userBroadcasterForeignID` and `userInitialName`.


## Optional Customization

You can also adjust the bottom padding for the content, and the type of animation:
```
overlay.setBottomPaddingDp(30)
overlay.animationType = ITGAnimationType.FROM_RIGHT
```

If you want to replace our content visuals with your own customizable interfaces, you can implement the layout listener:
```
overlay.layoutListener = this
```
Then you can create subclasses of our content views with customized layouts, and provide them in the listener methods. A full working example is included in the repository.\
The example includes custom layouts for 3 different formats - television, mobile landscape and mobile portrait. You can use only those you need - if your app is for TV only, use the layout-television files on your main layout folder.\
The listener methods are: 
```
fun customPollView(): ITGPollView?
fun customRatingView(): ITGRatingView?
fun customTriviaView(): ITGTriviaView?
fun customWikiView(): ITGWikiView?
fun customNoticeView(): ITGNotice?
```

You can adjust the behaviour of the win notifications with these variables:
```
overlay.showNotices = false
overlay.showNoticeAsActivity = true
```

You can also hide content when a phone is in portrait mode:
```
overlay.hideInPortraitMode = true
```
