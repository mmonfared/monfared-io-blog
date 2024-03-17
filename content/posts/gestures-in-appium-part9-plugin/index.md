+++
title = 'Gestures in Appium - Part 9- Plugin'
date = 2024-03-14T08:00:00+01:00
draft = false
tags = [
    "appium",
    "gestures",
]
categories = [
    "Mobile Automation"
]
series = ["Appium Gestures"]
+++

<p align="center" width="100%">
    <img src="./images/appium-gestures-9-banner.png" style="width: 100%" title="Gestures in Appium - Part 9- Plugin">
</p>

## Overview

This is the ninth part of the complete Gestures guide in the Appium series. 

There is a plugin called [appium-gestures-plugin](https://github.com/AppiumTestDistribution/appium-gestures-plugin) from the **AppiumTestDistribution** team, who are famous for their useful plugins. So, I thought it is worth talking a bit about it here in this series.

[The repository](https://github.com/AppiumTestDistribution/appium-gestures-plugin) itself is the best place to follow the project, but this post is a sample in Python which was not included in their docs. In short, this plugin supports the following gestures as of the time of writing this blog post:

- Swipe Left, right, up and down
- scrollElementIntoView
- Drag and Drop
- Double Tap
- Long Press

To install it, run the following command in your terminal:

```shell
appium plugin install --source=npm appium-gestures-plugin
```
Then run the appium server including the `--use-plugins=gestures` option, which indicates that we want to run the server using an installed plugin named `gestures`:

```shell
appium --use-plugins=gestures
```

To use this plugin, we should run commands like we did with the W3C `Mobile Gestures Commands` so far. The following are the commands we can use:

- Swipe Left, right, up and down > `gesture: swipe`
- scrollElementIntoView > `gesture: scrollElementIntoView`
- Drag and Drop > `gesture: dragAndDrop`
- Double Tap > `gesture: doubleTap`
- Long Press > `gesture: longPress`

Now let's do some demos:

## Scroll Element Into View (Search) using Gestures Plugin

```python
from appium import webdriver
from appium.webdriver.common.appiumby import AppiumBy
from appium.options.android import UiAutomator2Options

import desired_caps # Explained in Part 1

appium_options = UiAutomator2Options().load_capabilities(desired_caps.apidemos)
driver = webdriver.Remote(appium_server, options=appium_options)
driver.find_element(by=AppiumBy.ACCESSIBILITY_ID, value='Views').click()
list_view = driver.find_element(by=AppiumBy.ID, value='android:id/list')
driver.execute_script('gesture: scrollElementIntoView',
                      {'scrollableView': list_view.id, 'strategy': 'accessibility id', 'selector': 'Picker',
                       'percentage': 50, 'direction': 'up', 'maxCount': 3})
```

Let's take a look at the arguments:

- `scrollableView`: This is the same scroll bounding area we had so far. Note that you should pass the session's `id` property of the element which is accessible via `elementObject.id `
- `strategy` and `locator `: These are a pair used to find the element we want to scroll into view (want to search for it)
- `percentage`: This indicates how much of the scroll bounding area we want to use to perform the gesture.
- `direction`: This can be `up `, `down `, `left` and `right`
- `maxCount`: This specifies how many times the driver is allowed to perform the gesture to find the element.

## Drag and Drop using Gestures Plugin

```python
from appium import webdriver
from appium.webdriver.common.appiumby import AppiumBy
from appium.options.android import UiAutomator2Options

import desired_caps # Explained in Part 1

appium_options = UiAutomator2Options().load_capabilities(desired_caps.apidemos)
driver = webdriver.Remote(appium_server, options=appium_options)
driver.find_element(by=AppiumBy.ACCESSIBILITY_ID, value='Views').click()
driver.find_element(by=AppiumBy.ACCESSIBILITY_ID, value='Drag and Drop').click()

el1 = driver.find_element(by=AppiumBy.ID, value='io.appium.android.apis:id/drag_dot_1')
el2 = driver.find_element(by=AppiumBy.ID, value='io.appium.android.apis:id/drag_dot_2')

driver.execute_script('gesture: dragAndDrop', {
    'sourceId': el1.id,
    'destinationId': el2.id,
})
```
The `gesture: dragAndDrop` the command simply accepts `sourceId` and `destinationId` which are the session's id properties of the starting (draggable) and ending (droppable) elements.

Thank you for taking the time to read. If you enjoyed the post, please leave your reactions, comments, and questions. Your feedback is greatly appreciated!

In the next article, we will see how we can perform gestures in Mobile Web Browsers.

*Previous*: [Part 8 - Zoom (Pinch)](https://blog.monfared.io/posts/gestures-in-appium-part8-zoom-pinch)

*Next*: [Part 10 - Perform gestures in Mobile Web Browsers](https://blog.monfared.io/posts/gestures-in-appium-part10-perform-gestures-in-mobile-web-browsers)

Follow me on LinkedIn: https://www.linkedin.com/in/mohammad-monfared/

Happy testing ✌️

