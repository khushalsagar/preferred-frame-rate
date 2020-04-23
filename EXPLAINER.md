# Preferred Frame Rate API Explained

## Problem Statement
A web developer currently has no way of indicating to the browser the rate at which they want to update the page content. The browser generally schedules rendering lifecyle updates at the same frequency as the display's default refresh rate, which is also the frequency at which requestAnimationFrame is dispatched. However, a page may wish to animate at a lower frame rate for multiple reasons : 

* If the page can not consistently produce frames at the display's refresh rate given the workload for each frame update.
* If reducing the update frequency for power efficieny is a better tradeoff for the user experience.

The common patterns developers can use to control the frequency of their content updates are:

* Use a timer to schedule the next animation update.
* Use requestAnimationFrame but skip rendering in callbacks at a fixed frequency.

The above methods can allow the page to control the rate of updates driven solely by script on the main event loop. However, it doesn't enable aligning the presentation of these updates by the browser to the same vsync as other fixed rate animations (such as video). An explicit hint from the app could allow the browser to align all such animations to the same frame and reduce overall composition overhead.

## Proposed API
The API being proposed here is as follows:

```javascript
window.setPreferredAnimationFrameRate(30, "lower");
```

The API allows the developer to provide 2 parameters to indicate their frame rate preference:

* The first is an int specifying the preferred frame rate value. A value of 0 implies no preference, which is equivalent to not using this API. This value is capped at the maximum refresh rate supported by the display. There should likely be a minimum acceptable value but that is TBD at this point.

* The second parameter is a string with 2 possible values: "lower" or "higher". It specifies whether the browser should pick a lower or higher value than the provided rate if using the exact value is not possible. This is needed because the actual frame rate chosen by the browser could depend on multiple factors including the refresh rates supported by the physical display and the preferred frame rate of other content sources animating onscreen. This means that it may not be optimal to animate the page content at the exact provided rate and it needs to be adjusted.

Once a preference is specified, the browser will dispatch requestAnimationFrame callbacks at a rate as close as possible to the specified rate. The browser may also use this as a performance hint to change the rate at which updates to the DOM outside of requestAnimationFrame are painted, but the behaviour for that case is undefined. The page should prefer using requestAnimationFrame to ensure a consistent frame rate for all script driven animations.

## Impact on Other Animations
The use of this API should not have an impact on other browser driven animations. This includes declarative animations like [css animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations), animated images, videos, etc. In such cases, the frame rate is already known through the metadata associated with each of these animations. For instance the rate of a css animation can be controlled using a [steps](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timing-function) timing function.

Similarly this preference does not apply to the frame rate of input gestures processed by the browser, such as scrolling, pinch zoom, etc. The common default behaviour is to animate these gestures at the display's maximum refresh rate, which also tends to be the rate at which associated event handlers are dispatched. If the page wants to render in response to these events at the same rate, the recommended behaviour is to remove the frame rate preference to ensure requestAnimationFrame callbacks are dispatched at the same rate.

## Non Goals
A few cases which are somewhat related to this area but not necessary to address in this proposal itself are outlined below:

* The current proposal has no impact on the rate of requestAnimationFrame callbacks dispatched on worker threads, for instance with an offscreen canvas.

* The web also lacks APIs for a developer to query the refresh rate capabilities of the user's physical display. The most common way to infer this is by observing the rate of requestAnimationFrame callbacks. It seems ergonomic to provide APIs to explicitly query this info, similar to other [screen](https://developer.mozilla.org/en-US/docs/Web/API/Window/screen) properties.

## References
The API proposed here draws heavily from similar APIs on other platforms including the [preferredFramesPerSecond](https://developer.apple.com/documentation/quartzcore/cadisplaylink/1648421-preferredframespersecond) property on Mac and [setFrameRate](https://developer.android.com/ndk/reference/group/a-native-window#anativewindow_setframerate) API on Android.
