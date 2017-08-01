---
title: '不需要JQuery #1'
date: 2017-04-16 14:56:50
tags:
  - jquery
  - JavaScript
---
隨著時代的推進，瀏覽器的進步神速，網頁的崛起。在這個愈來愈講究使用者體驗(UX)的時代，效能是一大重點，以前撰寫網頁的時候常常會使用JQuery，來控制元素，但是現在已經不同了，反而愈來愈追求原生的JS。
  [Vanilla JS——世界上最轻量的JavaScript框架（没有之一）](https://segmentfault.com/a/1190000000355277)

  ----------
<!--more-->
## Selector
元素選擇器

### JQuery
```javascript
$('.class #id')
```

### Vanilla JS
```javascript
document.body.querySelector('.class #id')
document.body.querySelectorAll('.class #id')

document.body.getElementById('id')
document.body.getElementsByTagName('tagName')
document.body.getElementsByClassName('class')
```
querySelector 與 getElement 最大差別就是前者無法抓取動態產生之元素，後者可以。
詳情可見 >
  [Fw: [心得] 都2017年了  學學用原生JS來操作DOM吧](https://www.ptt.cc/bbs/Web_Design/M.1491563726.A.508.html)

  ----------

## Event
事件

### JQuery
```javascript
$('dom').click((e) => {

})
```

### Vanilla JS
```javascript
document.body.querySelector('dom').addEventListener('click', (e) => {

})
```

----------

## Animate
動畫

### JQuery
```javascript
$('dom').animate({
  scrollTop: 0
}, 500)
```

### Vanilla JS
```javascript
const si = setInterval(() => {
  if (document.body.scrollTop <= 10) {
    document.body.scrollTop = 0
    clearInterval(si)
    return
  }
  document.body.scrollTop -= 10
}, 5)

// 下方效能較佳

window.requestAnimationFrame(function sc () {
  document.body.scrollTop -= document.body.scrollTop / 10
  if (document.body.scrollTop > 5) {
    window.requestAnimationFrame(sc)
  } else {
    document.body.scrollTop = 0
  }
})
```
