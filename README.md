#3D Touch Study Notes

A collection of developmental information on 3D Touch introduced by Apple. 3D Touch is only available on iPhone 6s and iPhone 6s Plus running iOS 9.

##Home Screen Quick Actions

It presents a set of quick actions to user when pressing an app icon. Developer can define **static or dynamic** quick actions, both types of quick action can display **up to 2 lines of text along with an optional icon**. iOS 9 displays **up to 4 quick actions**.

You can combine both types of quick action. Static quick actions are shown first, and if static items do not exhaust the limit and you have also defined dynamic ones, then dynamic ones will be displayed until limit is reached.

###Defining Static Quick Actions

Static quick actions are defined in an app's **Info.plist** file in the **UIApplicationShortcutItem** array. Here are the keys that developer can use to define a single quick action:

- **UIApplicationShortcutItemType**: use it to disambiguate among action types your app receives (_required_)
- **UIApplicationShortcutItemTitle**: quick action title, if it’s too long and no subtitle string provide, the system displays the title on 2 lines (_required_)
- **UIApplicationShortcutItemSubtitle**: quick action subtitle
- **UIApplicationShortcutItemIconType**: icon from system-provided library
- **UIApplicationShortcutItemIconFile**: icon from app’s bundle, should be single color,  35x35 point, if this key is specified, then system ignores the UIApplicationShortcutItemIconType key
- **UIApplicationShortcutItemUserInfo**: app-defined dictionary, one usage is to provide app version information

###UIApplicationShortcutItemUserInfo Usage

When your app is installed and has not been launched, only static quick actions are displayed. Dynamic quick actions are displayed only after the first launch and if there is room in the list. If your app is updated and has not yet been launched, dynamic quick actions of previous version is displayed. One way to gracefully handle this scenario is to include app version information in UIApplicationShortcutItemUserInfo dictionary of a quick action.

###Handling Quick Actions

When user selects a home screen quick action, the following callback is invoked:

```swift
func application(application: UIApplication, performActionForShortcutItem shortcutItem: UIApplicationShortcutItem, completionHandler: Bool -> Void) {
	let handled = handleShortcutItem(shortcutItem)
	completionHandler(handled)
}
```

 The above callback will be skipped if `application(\_:,willFinishLaunchingWithOptions:)` or `application(\_:,didFinishLaunchingWithOptions:)` returns false. Apple recommends handling shortcut items in those callbacks and return false if possible.

###Defining Dynamic Quick Actions

Developer can create dynamic quick actions using **UIApplicationShortcutItem**, **UIMutableApplicationShortcutItem**, and **UIApplicationShortcutIcon** classes, and then assign them to app’s shared UIApplication object using the `shortcutItems` property.

##UIKit Peek and Pop

It allows users to easily peek into a piece of content by pressing lightly, then when pressed little harder it pops into the actual content with full view. Apple refers to this interaction as **“peek and pop”**, a terminology used for end-users. For developers, the corresponding term is **“preview and commit”**, which aligns with API documents.

###Checking 3D Touch and Registering for Previewing

```swift
if traitCollection.forceTouchCapability == .Available {
	registerForPreviewingWithDelegate(self, sourceView: view)
} else {
	print(“3D Touch is not available.”)
	// Provide alternatives such as touch and hold,
	// implemented with UILongPressGestureRecognizer class
}
```

###UIViewControllerPreviewingDelegate Methods

```swift
func previewingContext(previewingContext: UIViewControllerPreviewing, viewControllerForLocation location:CGPoint) -> UIViewController? {
	…
	vc.preferredContentSize = CGSize(width: 0.0, height: 200.0)
	previewingContext.sourceRect = cell.frame
	…
}
```

The above delegate method is invoked during the preview. Once a fully configured UIViewController is created, setting the `previewingContext.sourceRect` accomplishes the blurring effect, in which all other UI elements on screen is blurred but the content under user’s touch remains visually sharp.

```swift
func previewingContext(previewingContext: UIViewControllerPreviewing, commitViewController viewControllerToCommit: UIViewController) {
	showViewController(viewControllerToCommit, sender: self)
}
```

The above delegate method is invoked when user wants to commit to viewing the content.

###Unregistering for Previewing

```swift
func unregisterForPreviewingWithContext(_ previewing: UIViewControllerPreviewing)
```

The above method is called automatically when a 3D Touch-registered view controller is deallocated. In certain cases, you can call this method explicitly.

##Optional Peek Quick Actions

User can obtain peek quick actions by swiping a peek upward if they are configured in the preview view controller. Developer can create a preview quick action with **UIPreviewAction** class, a group of preview quick actions with **UIPreviewActionGroup**, and both classes conform to **UIPreviewActionItem** protocol. Preview quick actions will be shown by simply override method `previewActionItems()` on UIViewController class, and return an array of objects conforming UIPreviewActionItem.

##Web View Peek and Pop

In **WKWebView** and **UIWebView**, peek and pop for links can be enabled by using `allowsLinkPreview` property.

**SFSafariViewController** automatically supports peek and pop for links and detected data.

##Force Properties

In iOS 9, the **UITouch** class offers 2 new properties: `force` and `maximumPossibleForce`. These properties enables developers to detect and respond touch pressures.

##Current Development Limitations

1. With Xcode 7.0 you must develop on a device that supports 3D Touch. **Simulator does not support 3D Touch**.
2. With Xcode 7.0 you must implement peek and pop view controllers in code. Interface Builder does not provide graphical support for configuring view controller or transitions for 3D Touch.

##References
- [Adopting 3D Touch on iPhone](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/Adopting3DTouchOniPhone/index.html#//apple_ref/doc/uid/TP40016543)
- [Property List Key Reference](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW36)
- [ApplicationShortcuts Sample Code](https://developer.apple.com/library/prerelease/ios/samplecode/ApplicationShortcuts/Introduction/Intro.html#//apple_ref/doc/uid/TP40016545)
- [ViewControllerPreview Sample Code](https://developer.apple.com/library/prerelease/ios/samplecode/ViewControllerPreviews/Introduction/Intro.html#//apple_ref/doc/uid/TP40016546)
- [Static Shortcut Items](https://littlebitesofcocoa.com/79)
- [Dynamic Shortcut Items](https://littlebitesofcocoa.com/88)
- [View Controller Previews](https://littlebitesofcocoa.com/80)
- [3D Touch](https://littlebitesofcocoa.com/95)