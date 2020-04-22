# Preferred Frame Rate API Explained

## Problem Statement
A web developer currently has no way of indicating to the browser the rate at which they want to update the page content. The browser generally schedules rendering lifecyle updates at the same frequency as the display's default refresh rate, which is also the frequency at which requestAnimationFrame is dispatched. However, a page may wish to animate at a lower frame rate for multiple reasons : 

* If the app can not consistently produce frames at the display's refresh rate given the workload for each frame update.
* If reducing the update frequency for power efficieny is a better tradeoff for the user experience.

The common patterns developers can use to control the frequency of their content updates are:

* Use a timer to schedule the next animation update.
* Use requestAnimationFrame but skip rendering in callbacks at a fixed frequency.

The above methods can allow the application to control the rate of updates driven solely by script on the main event loop. However, it doesn't enable aligning the presentation of these updates by the browser to the same vsync as other fixed rate animations (such as video). An explicit hint from the app could allow the browser to align all such animations to the same frame and reduce overall composition overhead.

## Proposed API
The primary API being proposed here is as follows:

```javascript
window.setPreferredFrameRate(30, "lower");
```

The API allows the developer to provide 2 parameters to indicate their frame rate preference:

* The first is an int specifying the preferred frame rate value. A value of 0 implies that the desired frame rate is the max value supported by the display.
* The second parameter is a string with 2 possible values: "lower" or "higher". It specifies whether the browser should pick a lower or higher value than the provided rate if needed. This is needed because the actual frame rate chosen by the browser could depend on multiple factors including the refresh rates supported by the physical display and the preferred frame rate of other content sources animating onscreen. This means that it may not be optimal to animate the page content at the exact provided rate and it needs to be adjusted.

Once a preference is specified, the browser will schedule rendering lifecycle updates in response to DOM mutations and dispatch of requestAnimationFrame at a rate as close as possible to the specified rate. But it is important to note that this is a performance hint and not a guarantee.

## Open Questions
Its not clear how this frame rate preference should affect other content animations such as:
* Input gestures, primarily scrolling/pinch zoom.
* Fixed rate video playback.
* Offscreen canvas requestAnimationFrame on a worker thread.

## Non Goals
The web also lacks APIs for a developer to query the refresh rate capabilities of the user's physical display. The most common way to infer this is by observing the rate of requestAnimationFrame callbacks. It seems ergonomic to provide APIs to explicitly query this info, similar to other [screen](https://developer.mozilla.org/en-US/docs/Web/API/Window/screen) properties.

## References
The API proposed here draws heavily from similar APIs on other platforms including the [preferredFramesPerSecond](https://developer.apple.com/documentation/quartzcore/cadisplaylink/1648421-preferredframespersecond) property on Mac and [setFrameRate](https://developer.android.com/ndk/reference/group/a-native-window#anativewindow_setframerate) API on Android.
