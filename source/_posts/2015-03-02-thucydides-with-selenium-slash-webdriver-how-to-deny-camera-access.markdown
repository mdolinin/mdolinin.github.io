---
layout: post
title: "Thucydides with Selenium/Webdriver: How to deny camera access?"
date: 2015-03-02 23:13:36 +0200
comments: true
categories: 
- Thucydides 
- Selenium 
- Webdriver 
---
If you have web application that needs access to media devices, such as web camera,
and try to use Selenium/Webdriver for test automation, you will find that browser shows you own promt.
In this promt the user is asked to allow or deny access to camera.

To solve this issue you need to add extra parameter when start browser.
Here is an example for Firefox and Chrome browsers for Thucydides framework,
but you can use those options directly with Webdriver.
```properties thucydides.properties
	firefox.preferences=media.navigator.permission.disabled=true
	chrome.switches=--use-fake-device-for-media-stream,--use-fake-ui-for-media-stream
``` 
I hope this will be usefull for someone.