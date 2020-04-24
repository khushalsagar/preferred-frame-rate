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
window.setPreferredFrameRate(30, "lower");
window.clearPreferredFrameRate();
```

The API allows the developer to provide 2 parameters to indicate their frame rate preference:

* The first is an int specifying the preferred frame rate value in frames per second. This value is capped at the maximum refresh rate supported by the display. There should likely be a minimum acceptable value but that is TBD at this point.

* The second parameter is a string with 2 possible values: "lower" or "higher". It specifies whether the browser should pick a lower or higher value than the provided rate if using the exact value is not possible. This is needed because the actual frame rate chosen by the browser could depend on multiple factors including the refresh rates supported by the physical display and the preferred frame rate of other content sources animating onscreen. This means that it may not be optimal to animate the page content at the exact provided rate and it needs to be adjusted.

* The clearPreferredFrameRate API allows the developer to remove this setting, which is equivalent to never having called setPreferredFrameRate.

Once a preference is specified, the browser will provide [rendering opportunities](https://html.spec.whatwg.org/multipage/webappapis.html#rendering-opportunity) to the page at a rate as close as possible to the specified rate, which includes dispatch of requestAnimationFrame callbacks.

## Threaded Animations
While the primary use-case of this API is to specify the rate of updates for animations driven completely by script (requestAnimationFrame/timers), its important to clarify the impact of this setting on animations which can be updated outside the [window event loop](https://html.spec.whatwg.org/multipage/webappapis.html#concept-agent-event-loop). This broadly includes declarative animations such as a transform [css animation](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations), animated images, videos and browser initiated animations in response to user gestures such as scrolling, pinch-zoom, double tap zoom etc. The browser may perform these on a different thread for performance reasons.

The developer intent with using this API may or may not be to throttle these threaded animations, particularly the ones triggered in response to user input. Ideally this should be a preference that the developer can specify but its unclear how it should be exposed by the API given that threaded animations are largely a browser implementation detail. One thing to note is that if threaded animations are not throttled (whether through developer opt-in or otherwise) the behaviour for whether an animation is throttled or not could be inconsistent across browsers.

## Browsing Contexts
The API is a no-op if it is called from a window object which does not belong to the top level browsing context, to ensure that an embeded iframe can not change the frame rate for the page embedding it. The setting will also apply to all nested browsing contexts for this top level context.

Ideally the above restrictions shouldn't be necessary but this requires the ability to independently run rendering lifecyle updates for documents in nested browsing contexts. Currently in chrome this is only possible with cross-process iframes and its not clear whether its possible to support this in other browsers at all.

## Non Goals
A few cases which are somewhat related to this area but not necessary to address in this proposal itself are outlined below:

* The current proposal has no impact on the rate of requestAnimationFrame callbacks dispatched on worker threads, for instance with an offscreen canvas.

* The web also lacks APIs for a developer to query the refresh rate capabilities of the user's physical display. The most common way to infer this is by observing the rate of requestAnimationFrame callbacks. It seems ergonomic to provide APIs to explicitly query this info, similar to other [screen](https://developer.mozilla.org/en-US/docs/Web/API/Window/screen) properties.

## References
The API proposed here draws heavily from similar APIs on other platforms including the [preferredFramesPerSecond](https://developer.apple.com/documentation/quartzcore/cadisplaylink/1648421-preferredframespersecond) property on Mac and [setFrameRate](https://developer.android.com/ndk/reference/group/a-native-window#anativewindow_setframerate) API on Android.
