---
id: building-for-tv
title: Building For TV Devices
---

TV devices support has been implemented with the intention of making existing React Native applications work on Apple TV and Android TV, with few or no changes needed in the JavaScript code for the applications.

<div class="toggler">
  <ul role="tablist" id="toggle-platform">
    <li id="android" class="button-android" aria-selected="false" role="tab" tabindex="0" aria-controls="androidtab" onclick="displayTab('platform', 'android')">
      Android
    </li>
    <li id="ios" class="button-ios" aria-selected="false" role="tab" tabindex="0" aria-controls="iostab" onclick="displayTab('platform', 'ios')">
      🚧 iOS
    </li>
  </ul>
</div>

<block class="ios" />

> **Deprecated.** Use [react-native-tvos](https://github.com/react-native-community/react-native-tvos) instead. For the details please check the [0.62 release blog post](https://reactnative.dev/blog/#moving-apple-tv-to-react-native-tvos).

The RNTester app supports Apple TV; use the `RNTester-tvOS` build target to build for tvOS.

## Build changes

- _Native layer_: React Native Xcode projects all now have Apple TV build targets, with names ending in the string '-tvOS'.

- _react-native init_: New React Native projects created with `react-native init` will have Apple TV target automatically created in their XCode projects.

- _JavaScript layer_: Support for Apple TV has been added to `Platform.ios.js`. You can check whether code is running on AppleTV by doing

```jsx
var Platform = require('Platform');
var running_on_tv = Platform.isTV;

// If you want to be more specific and only detect devices running tvOS
// (but no Android TV devices) you can use:
var running_on_apple_tv = Platform.isTVOS;
```

<block class="android" />

## Build changes

- _Native layer_: To run React Native project on Android TV make sure to make the following changes to `AndroidManifest.xml`

```xml
  <!-- Add custom banner image to display as Android TV launcher icon -->
 <application
  ...
  android:banner="@drawable/tv_banner"
  >
    ...
    <intent-filter>
      ...
      <!-- Needed to properly create a launch intent when running on Android TV -->
      <category android:name="android.intent.category.LEANBACK_LAUNCHER"/>
    </intent-filter>
    ...
  </application>
```

- _JavaScript layer_: Support for Android TV has been added to `Platform.android.js`. You can check whether code is running on Android TV by doing

```js
var Platform = require('Platform');
var running_on_android_tv = Platform.isTV;
```

<block class="ios android" />

## Code changes

<block class="ios" />

- _General support for tvOS_: Apple TV specific changes in native code are all wrapped by the TARGET_OS_TV define. These include changes to suppress APIs that are not supported on tvOS (e.g. web views, sliders, switches, status bar, etc.), and changes to support user input from the TV remote or keyboard.

- _Common codebase_: Since tvOS and iOS share most Objective-C and JavaScript code in common, most documentation for iOS applies equally to tvOS.

- _Access to touchable controls_: When running on Apple TV, the native view class is `RCTTVView`, which has additional methods to make use of the tvOS focus engine. The `Touchable` mixin has code added to detect focus changes and use existing methods to style the components properly and initiate the proper actions when the view is selected using the TV remote, so `TouchableWithoutFeedback`, `TouchableHighlight` and `TouchableOpacity` will work as expected. In particular:

  - `onFocus` will be executed when the touchable view goes into focus
  - `onBlur` will be executed when the touchable view goes out of focus
  - `onPress` will be executed when the touchable view is actually selected by pressing the "select" button on the TV remote.

<block class="android" />

- _Access to touchable controls_: When running on Android TV the Android framework will automatically apply a directional navigation scheme based on relative position of focusable elements in your views. The `Touchable` mixin has code added to detect focus changes and use existing methods to style the components properly and initiate the proper actions when the view is selected using the TV remote, so `TouchableWithoutFeedback`, `TouchableHighlight`, `TouchableOpacity` and `TouchableNativeFeedback` will work as expected. In particular:

  - `onFocus` will be executed when the touchable view goes into focus
  - `onBlur` will be executed when the touchable view goes out of focus
  - `onPress` will be executed when the touchable view is actually selected by pressing the "select" button on the TV remote.

<block class="ios" />

- _TV remote/keyboard input_: A new native class, `RCTTVRemoteHandler`, sets up gesture recognizers for TV remote events. When TV remote events occur, this class fires notifications that are picked up by `RCTTVNavigationEventEmitter` (a subclass of `RCTEventEmitter`), that fires a JS event. This event will be picked up by instances of the `TVEventHandler` JavaScript object. Application code that needs to implement custom handling of TV remote events can create an instance of `TVEventHandler` and listen for these events, as in the following code:

<block class="android" />

- _TV remote/keyboard input_: A new native class, `ReactAndroidTVRootViewHelper`, sets up key events handlers for TV remote events. When TV remote events occur, this class fires a JS event. This event will be picked up by instances of the `TVEventHandler` JavaScript object. Application code that needs to implement custom handling of TV remote events can create an instance of `TVEventHandler` and listen for these events, as in the following code:

<block class="ios android" />

```jsx
var TVEventHandler = require('TVEventHandler');

class Game2048 extends React.Component {
  _tvEventHandler: any;

  _enableTVEventHandler() {
    this._tvEventHandler = new TVEventHandler();
    this._tvEventHandler.enable(this, function(cmp, evt) {
      if (evt && evt.eventType === 'right') {
        cmp.setState({board: cmp.state.board.move(2)});
      } else if(evt && evt.eventType === 'up') {
        cmp.setState({board: cmp.state.board.move(1)});
      } else if(evt && evt.eventType === 'left') {
        cmp.setState({board: cmp.state.board.move(0)});
      } else if(evt && evt.eventType === 'down') {
        cmp.setState({board: cmp.state.board.move(3)});
      } else if(evt && evt.eventType === 'playPause') {
        cmp.restartGame();
      }
    });
  }

  _disableTVEventHandler() {
    if (this._tvEventHandler) {
      this._tvEventHandler.disable();
      delete this._tvEventHandler;
    }
  }

  componentDidMount() {
    this._enableTVEventHandler();
  }

  componentWillUnmount() {
    this._disableTVEventHandler();
  }
```

<block class="ios" />

- _Dev Menu support_: On the simulator, cmd-D will bring up the developer menu, similar to iOS. To bring it up on a real Apple TV device, make a long press on the play/pause button on the remote. (Please do not shake the Apple TV device, that will not work :) )

- _TV remote animations_: `RCTTVView` native code implements Apple-recommended parallax animations to help guide the eye as the user navigates through views. The animations can be disabled or adjusted with new optional view properties.

- _Back navigation with the TV remote menu button_: The `BackHandler` component, originally written to support the Android back button, now also supports back navigation on the Apple TV using the menu button on the TV remote.

<block class="android" />

- _Dev Menu support_: On the simulator, cmd-M will bring up the developer menu, similar to Android. To bring it up on a real Android TV device, press the menu button or long press the fast-forward button on the remote. (Please do not shake the Android TV device, that will not work :) )

- _Known issues_:

  - `TextInput` components do not work for now (i.e. they cannot receive focus automatically, see [this comment](https://github.com/facebook/react-native/pull/16500#issuecomment-629285638)). 
    - It is however possible to use a ref to manually trigger `inputRef.current.focus()`. 
    - You can wrap your input inside a `TouchableWithoutFeedback` component and trigger focus in the `onFocus` event of that touchable. This enables opening the keyboard via the arrow keys.
    - The keyboard might reset its state after each keypress (this might only happen inside the Android TV emulator).
  - The content of `Modal` components cannot receive focus, see [this issue](https://github.com/facebook/react-native/issues/24448) for details.
