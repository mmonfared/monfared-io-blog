+++
title = 'Gestures in Appium - Part 1 - History | Element/Display Rectangular | W3C ActionsAPI | Setup'
date = 2024-01-14T07:07:07+01:00
draft = false
+++
## What is Appium?

[Appium](https://appium.io/docs/en/2.5/intro/) is an open-source project and ecosystem designed to facilitate **UI automation** for a wide range of app platforms, including:
- **Mobile**: iOS, Android, Tizen
- **Browser**: Chrome, Firefox, Safari
- **Desktop**: macOS, Windows
- **TV**: Roku, tvOS, Android TV, Samsung, and more.

In other words, Appium does the same as selenium but for mobile applications! With the release of **Appium 2.0**, it aims to achieve the following primary goals:
1. **Make platform-specific automation capabilities** available under a **cross-platform, standard API (Compatible with W3C WebDriver protocol).**
2. **Allow easy access** to this API from **any programming language.**
3. Provide tools for **convenient community development of Appium extensions.**

<div style="display: block; margin: 0 auto; max-width: 100%;">

![Appium Architecture](/images/1-appium-arch.webp "Appium Architecture")

</div>

**Appium** leverages the **WebDriver specification** as its API (Including groundbreaking [**Actions API**](https://w3c.github.io/webdriver/#actions) for its Gestures actions). This choice was influenced by Selenium, a long-standing pioneer in UI automation for web browsers. Selenium's stable API for browser automation served as a solid foundation, and Appium extended it to support mobile apps (iOS and Android). By adopting the WebDriver spec, Appium ensures a unified approach to automation across platforms.

- While user interactions on websites and native mobile apps differ, the commonalities in software UIs allow the WebDriver spec to map effectively to any platform. Appium's goal is to provide a consistent experience for developers and testers, regardless of the underlying technology.

In this series of articles, I will explain in detail how to use Appium to perform mobile gestures like zoom, scroll, swipe, drag and drop, and more, using the latest techniques and APIs.

- [**W3C Actions API**](https://w3c.github.io/webdriver/#actions)
- [**W3C Mobile Gestures Actions**](https://github.com/appium/appium-uiautomator2-driver/blob/master/docs/android-mobile-gestures.md)
- [**UiScrollable Class**](https://developer.android.com/reference/androidx/test/uiautomator/UiScrollable) (Deprecated)
- [**Appium Gestures Plugin**](https://github.com/AppiumTestDistribution/appium-gestures-plugin)
- **TouchAction | MultiAction** (Deprecated, but it is worth mentioning since a lot of projects still use it)

You will not find all of them in one place except here! So let's get started 🚀

![Some mobile gestures](/images/1-gestures.gif "Some mobile gestures")

I'll show how to do the gestures in "Android "and in later articles, I will write about them in iOS. 

**Git Repository**: All the codes written in this series of articles are pushed to the following repository, so feel free to use it as a reference: [automationcamp-appium Github Repo](https://github.com/mmonfared/automationcamp-appium/blob/master/Session7_Gestures.py)

## Install and setup project

If you haven't installed Appium yet, please follow this article. I explained step by step how to setup an appium project. For the demo, we will use appium official ApiDemos application which you can find in the above repository `APK` folder or directly from appium repo. Now:

1. Create a Python file like `appium_gestures.py` (test file)
2. Import all the following classes/libraries. (Look at the comments)

```python
# Main appium driver
from appium import webdriver

# It is used to locate elements
from appium.webdriver.common.appiumby import AppiumBy

# It is used to create a UiAutomator2 driver instance
from appium.options.android import UiAutomator2Options

# W3C compatible gestures helper librart created to update driver's default gestures
from appium.webdriver.extensions.action_helpers import ActionHelpers

# W3C compatible libraries to create any gestures rather than driver's default gestures
from selenium.webdriver.common.actions.action_builder import ActionBuilder
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.actions.mouse_button import MouseButton
from selenium.webdriver.common.actions.pointer_input import PointerInput
from selenium.webdriver.common.actions import interaction

# Old touch action libraries (Deprecated and will remove soon)
from appium.webdriver.common.touch_action import TouchAction
from appium.webdriver.common.multi_action import MultiAction

# It is used to demo purposes. 
from time import sleep
```
3. Set the appium server URL globally in the file:

```python
appium_server = "http://127.0.0.1:4723"
```
4. Create a file named `desired_caps.py` in the project root and add the following dictionaries in it:

```python
apidemos = {
    "platformName": "Android",
    "appium:options": {
        "appPackage": "io.appium.android.apis",
        "appActivity": ".ApiDemos",
        ":automationName": "UiAutomator2"
    }
}

contacts = {
    "appium:appPackage": "com.android.contacts",
    "appium:appActivity": ".activities.PeopleActivity",
    "platformName": "Android",
    "appium:automationName": "UiAutomator2"
}

maps = {
    "appium:appPackage": "com.google.android.apps.maps",
    "appium:appActivity": "com.google.android.maps.MapsActivity",
    "platformName": "Android",
    "appium:automationName": "UiAutomator2"
}

chrome = {
    "platformName": "Android",
    "browserName": "Chrome",
    "appium:options": {
        "automationName": "UiAutomator2",
        # "chromedriverExecutable": "C:/chromedriver.exe"
    }
}
```
then add the following import statement to the test file.

```python
import desired_caps
```
## Get window and element rectangular / size
Doing precise gestures on the display needs interaction  with coordination and calculations related to the positions of the elements or display size. 
Before diving into gestures we should be aware of them.

![Element Rectangular](/images/1-element-rect.webp "Element Rectangular")

### Display Rectangular:

In the figure above, the black box represents the Display, which is currently in landscape view. We measure the **Size** of the Display in pixels (Height/Width), for example, **800px*1440px**. Note that this is not the same as the device dimensions, which are measured in inches. The screen size of applications and the display resolution are determined based on display density. Additionally, the app's window may not be entirely maximized, so it could be located anywhere on the screen. The **(x,y)** coordinates of the top-left corner of this window are referred to as the **Location** or **Coordination**.

The following command returns the rectangle of a window, including its X and Y coordinates, as well as its height and width.

```python
window_rect = driver.get_window_rect() # {'height': '800', 'width: '1440', 'x': 100, 'y': 200}
```
Also, if we just want the **Window Size**, we can use the following command:

```python
window_rect = driver.get_window_size() # {'height': '800', 'width: '1440'}
```
### Element Rectangular:
Each element within a window has its own rectangular properties, including its **Size** and **Location** on the page. The size of an element is defined in terms of its **Height** and **Width** in pixels, while its location is determined by measuring the distance from the top-left corner of the screen. To obtain these values, we can use the following method:

```python
# First we find the element and store its object in a value
el = driver.find_element(by=AppiumBy.ACCESSIBILITY_ID, value='Button')
# Then the "rect" and "location" are what we need:
print(el.rect) # {'height': '100', 'width: '150', 'x': 840, 'y': 680}
print(el.location) # {'x': 840, 'y': 680}
```
### Calculate the center of an element:
If we want to execute a gesture on an element object, we can pass the element object to the relevant command. The driver will automatically perform the gesture at the center of the element. However, if we want to pass the coordinates of the element instead and perform the action on the center, we need to calculate its center point first. This is often necessary as the `location` attribute returns the coordinates of the element's top-left corner, and we need to determine the center point ourselves. The center point is shown in purple as (x,y) in the above figure.

The following code can be used to do it:

```python
el = driver.find_element(by=AppiumBy.ACCESSIBILITY_ID, value='Button')
center_x = el.rect['x'] + el.rect['width']/2
center_y = el.rect['y'] + el.rect['height']/2
```

Ok, let's do the very first gesture which is tapping!

## W3C Actions API

Let's take a quick look at the libraries and history of gesture commands. Selenium has its own libraries for mouse actions called "TouchActions" and "ActionChains". While it can handle almost all mouse actions on the web, it may not be completely useful for mobile gesture actions. Therefore, the Appium team had to develop libraries called "TouchAction" and "MultiAction". Let's take a closer look at the comments written by the Appium team at the top of the TouchAction class.

![touch-action.py comments](/images/1-touch-action-comment.webp "touch-action.py comments")

It worked for many years, but in 2018, WebDriver became a W3C standard and introduced the Actions API in section 15 of its specifications. Selenium and its sub-projects, including Appium, must follow these specifications.

![W3C Webdriver Protocol - The Actions API ](/images/1-w3c-webdriver-protocol.webp "W3C Webdriver Protocol -The Actions API")

By introducing the concepts of Multiple Pointer Inputs and Ticks, it became possible to create almost any gesture by simulating parallel sequences. For instance, you can make a zoom gesture by moving two fingers in opposite directions. You can even draw on the screen! The Appium team explained this concept very well in [an article](https://appiumpro.com/editions/29-automating-complex-gestures-with-the-w3c-actions-api), which you can read to get a better understanding.

![AppiumPro Blog - FingerPaint ](/images/1-appiumpro-face-gestures-post.gif "Source: https://appiumpro.com/editions/29-automating-complex-gestures-with-the-w3c-actions-api")

## W3C Mobile Gestures Commands
We know that in Selenium/Appium, JavaScript commands can be executed using `driver.execute_script()` , so it opens the door to leveraging the power of JavaScript to perform complex tasks like gestures in the page or app. Thanks to the Appium team, we have [a range of commands](https://github.com/appium/appium-uiautomator2-driver/blob/master/docs/android-mobile-gestures.md) implemented to perform gestures and they work well in most cases.

```javascript
mobile: dragGesture
mobile: flingGesture
mobile: doubleClickGesture
mobile: clickGesture
mobile: longClickGesture
mobile: pinchCloseGesture
mobile: pinchOpenGesture
mobile: swipeGesture
mobile: scrollGesture
```
A notable advantage of using these commands is their multi-platform compatibility. You can use the same commands with slight change in syntax for both iOS and Android, eliminating the need for driver-specific commands. (`UiAutmator2` driver in android and `XCUITest` in iOS ).

So Selenium has implemented W3C actions and updated the `ActionChains` class, which allows you to use the same class for both Appium and Selenium. You can create your own gestures using this class. Also, there are some official helper classes available for popular gestures using W3C Actions such as `driver.swipe()` and `driver.drag_and_drop()` or `Mobile Gestures Commands` in the latest version of the Appium Python client. We will implement each gesture using all available options in the following articles! 

Thank you for taking the time to read. If you enjoyed the post, please leave your comments, questions, and reactions. Your feedback is greatly appreciated!

In the next article, we will cover **Tap**, **Multi-Finger Tap**, and **Double-Tap** gestures.

Follow me on LinkedIn: 
https://www.linkedin.com/in/mohammad-monfared/

Happy testing ✌️