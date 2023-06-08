---
title: React Navigation在特定页面隐藏tab菜单栏
categories: 
  - 蜻蜓点水
  - React
tags: 
  - React
  - React-Navigation
  - Tab
date: 2021-06-28 16:55:51
---

## React Navigation在特定页面隐藏tab菜单栏

在基于tab的导航应用中，有时候我们需要在某些页面隐藏tab菜单栏。假设这里有5个页面：

- Home
- Feed
- Notifications
- Profile
- Settings

<!-- more -->

App对应的导航逻辑大致是这样的:

```jsx
function HomeStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={Home} />
      <Stack.Screen name="Profile" component={Profile} />
      <Stack.Screen name="Settings" component={Settings} />
    </Stack.Navigator>
  );
}

function App() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeStack} />
      <Tab.Screen name="Feed" component={Feed} />
      <Tab.Screen name="Notifications" component={Notifications} />
    </Tab.Navigator>
  );
}
```

在现有结构中，当我们导航到Profile和Settings页面时，仍然可以在页面底部看到tab菜单栏。

但是我们的设计是只在Home、Feed和Notifications页面显示tab菜单栏，而Profile和Settings页面时却不需要显示tab，我们需要改变现有的导航结构。最简单的方法是将tab导航器放在导航栈的第一个页面内部，而不是将导航栈放在tab里边，像下边这样：

```jsx
function HomeTabs() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={Home} />
      <Tab.Screen name="Feed" component={Feed} />
      <Tab.Screen name="Notifications" component={Notifications} />
    </Tab.Navigator>
  );
}

function App() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeTabs} />
      <Stack.Screen name="Profile" component={Profile} />
      <Stack.Screen name="Settings" component={Settings} />
    </Stack.Navigator>
  );
}
```

重新组织过导航结构以后，现在，当我们导航到Profile和Settings页面时，就看不到tab菜单栏了。

有一些导航器，比如[底部tab导航器](https://reactnavigation.org/docs/bottom-tab-navigator)，也会有一个tabBarVisible的选项，可以用来隐藏tab栏。然而，我们不推荐使用这种方法，因为在导航过程中使用改变tab栏的显示和隐藏会影响导航栈的动画切换，有时还可能导致其它一些想不到的错误。
