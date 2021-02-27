# React路由实现原理

## 路由两种实现方式

- HashRouter：利用hash实现路由切换
- BrowserRouter：通过H5 API实现路由的切换

### HashRouter

通过监听`hashchange`事件触发变更

```javascript
window.onload = function() {
  window.addEventListener("hashchange", function() {
    console.log(window.location.hash.slice(1))
  })
}
```

### BrowserRouter

H5里面有关于history变更方面的API `window.history`

history内部其实是一个栈，并且有一个指针指向当前所展示的页面也URL，而每次push进来的URL为进栈，指针会指向最新的URL。如果用back或者forward或者go函数，就是改变指针指向的URL。

`putstate` 更改URL

`replaceState` 替换URL

`onpopstate` 一旦指针发生改变会出发该事件

