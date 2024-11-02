# eventBus.js

> 在微信小程序中实现发布/订阅模式可以通过创建一个简单的事件管理器来实现。这个事件管理器将负责注册事件监听器、触发事件以及移除监听器。下面是一个具体的实现示例：

## 1. 创建事件管理器

首先，我们创建一个单独的文件 `eventBus.js` 来管理事件的发布和订阅。

```javascript
// eventBus.js
class EventBus {
  constructor() {
    this.events = {};
  }

  // 订阅事件
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }

  // 触发事件
  emit(eventName, ...args) {
    const callbacks = this.events[eventName];
    if (callbacks) {
      callbacks.forEach((callback) => callback(...args));
    }
  }

  // 移除事件监听器
  off(eventName, callback) {
    if (this.events[eventName]) {
      this.events[eventName] = this.events[eventName].filter(
        (cb) => cb !== callback
      );
    }
  }
}

const eventBus = new EventBus();

export default eventBus;
```

## 2. 在 `app.js` 中使用事件管理器

在 `app.js` 中，我们可以使用事件管理器来触发用户信息更新的事件。

```javascript
// app.js
import eventBus from "./utils/eventBus";

App({
  globalData: {
    userInfo: null,
  },
  onLaunch: function () {
    // 模拟异步请求用户信息
    wx.request({
      url: "your-api-url",
      method: "GET",
      success: (res) => {
        this.globalData.userInfo = res.data; // 更新全局数据
        eventBus.emit("userInfoUpdated", res.data); // 触发用户信息更新事件
      },
      fail: () => {
        // 错误处理
      },
    });
  },
});
```

## 3. 在页面中订阅事件

在页面中，我们可以订阅 `userInfoUpdated` 事件，并在事件触发时更新页面数据。

```javascript
// pages/index/index.js
import eventBus from "../../utils/eventBus";

Page({
  data: {
    userInfo: null,
  },
  onLoad: function () {
    const app = getApp();
    this.setData({ userInfo: app.globalData.userInfo }); // 初始化数据
    this.subscribeToUserInfoUpdate();
  },
  subscribeToUserInfoUpdate: function () {
    eventBus.on("userInfoUpdated", (userInfo) => {
      this.setData({ userInfo });
    });
  },
  onUnload: function () {
    // 移除事件监听器，防止内存泄漏
    eventBus.off("userInfoUpdated", this.handleUserInfoUpdate);
  },
  handleUserInfoUpdate: function (userInfo) {
    this.setData({ userInfo });
  },
});
```

## 解释

1. **事件管理器 (`eventBus.js`)**：

   - `on(eventName, callback)`：注册一个事件监听器。
   - `emit(eventName, ...args)`：触发一个事件，所有注册的回调函数都会被执行。
   - `off(eventName, callback)`：移除一个事件监听器。

2. **`app.js`**：

   - 在 `onLaunch` 中发起请求获取用户信息。
   - 当请求成功时，更新 `globalData` 并触发 `userInfoUpdated` 事件。

3. **页面 (`index.js`)**：
   - 在 `onLoad` 中初始化用户信息。
   - 订阅 `userInfoUpdated` 事件，并在事件触发时更新页面数据。
   - 在 `onUnload` 中移除事件监听器，防止内存泄漏。

通过这种方式，你可以实现一个简单的发布/订阅模式，使得不同页面之间能够高效地通信和同步数据。

## ⚠️ 注意

务必记得在离开页面的时候，`移出事件监听器`，不然不只是什么性能问题，而是`会触发所有未移出的回调方法`。

可以参考下面这段伪代码：

```js
// chat.js - 这是需要监听的页面逻辑

/**
 * 生命周期函数--监听页面加载
 */
onLoad() {
  // 订阅消息通知
  this.initEventBus();
},

/**
 * 生命周期函数--监听页面卸载
 */
onUnload() {
  // 取消订阅
  this.closeEventBus();
},


/** 订阅消息通知 */
initEventBus() {
  // 监听聊天好友消息
  eventBus.on("chatMsgResponse", this.chatMsgResponseCb);
},

/** 取消订阅 */
closeEventBus() {
  // 监听聊天好友消息
  eventBus.off("chatMsgResponse", this.chatMsgResponseCb);
},

/** 监听事件的回调函数 */
chatMsgResponseCb(response) {
  console.log('[chatMsgResponseCb] 当前聊天中的好友消息 response:', response);
  // todo ...
},

```
