# Preferred Frame Rate API Explained

## Problem Statement
A web developer currently has no way of indicating to the browser the rate at which they want to update the page content. The browser generally schedules rendering lifecyle updates at the same frequency as the display's default refresh rate, which is also the frequency at which requestAnimationFrame is dispatched. However, a page may wish to animate at a lower frame rate for multiple reasons : 

* If the app can not consistently produce frames at the display's refresh rate given the workload for each frame update.
* If reducing the update frequency for power efficieny is a better tradeoff for the user experience.

The common patterns developers can use to control the frequency of their content updates are:

* Use a timer to schedule the next animation update.
* Use requestAnimationFrame but skip rendering in callbacks at a fixed frequency.

The above methods can allow the application to control the rate of updates driven solely by script on the main event loop. However, it doesn't enable aligning the presentation of these updates by the browser to the same vsync as other fixed rate animations (such as video). An explicit hint from the app could allow the browser to align all such animations to the same vsync and reduce overall composition overhead.

## Proposed API
The API

1) An API to query the max refresh rate supported by the physical display and an event to observe changes to this property, if the window's associated display changes.

```javascript
// Maximum refresh rate supported by the physical display, in frames per second.
int maxRefreshRate = window.screen.maxRefreshRate;

window.addEventListener("maxRefreshRateChange", function() {
  ...
});
```

2) The fra
