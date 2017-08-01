---
title: Koa與Socket實作
date: 2017-05-12 19:04:27
tags:
  - JavaScript
  - Koa
  - Socket
---

最近快要畢業了，想要找個公司能夠有個能讓好好充實自己的地方，但是有兵役問題，很多公司無法接受，也有投一些實習，可是也都沒有上，就突然有感而發覺得似乎是自己能力太不足夠，所以我一定要再更奮發向上！廢話不多說，我們來進入主題～

昨天在聽讀書會的時候，主題是node design pattern，強調使用Stream來作分流、分支，後來就突然想到之前一直很想學如何做一個聊天室，所以就慢慢的實作，就把實踐的過程中一步一步記錄下來。

## 起手式

這個實作會比較簡單，就是簡單地利用Koa建立小型的伺服器，並且導入Koa-Router可以來處理路由，這個範例沒有做的很精細，只是稍微簡單的實作而已，假如需要更完整的功能可以去看[官方文件](https://socket.io/docs/)，上面的功能有更詳細得解說。

<!--more-->

## Koa-Server
用`yarn init`建立package.json之後再來引入需要的套件：`koa`、`koa-router`

### server.js

```js
const Koa = require('koa'),
      app = new Koa(),
      router = require('koa-router')()

const fs = require('fs') // 這個是拿來處理檔案的，node.js內建，所以不需要安裝！

router
  .get('/', async (ctx, next) => {
    ctx.body = 'Hello, World!'
    await next()
  })
  .get('/socket', async (ctx, next) => {
    ctx.body = 'Hello, Socket'
    await next()
  })

app.use(router.routes())

app.listen(3000, () => {
  console.log('listening 3000 port => http://localhost:3000/')
})

```
運行`node server.js`之後打開瀏覽器，到3000port的本機應該可以看到有`Hello, World!`，然後再到路由/socket，應該可以看到`Hello, Socket`，這樣就代表正常運作了！

### fs ReadFile

現在我們要從伺服器端渲染出html **(Server Side Render)** ，從伺服器主機位置讀取要的html，並且渲染到瀏覽器！

現在我們先來寫我們要的html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Socket Test</title>
</head>
<body>
  first Socket
  <input type="text" id="msg" value="">
  <button type="button" id="sendMsg">Msg</button>
</body>
</html>
```

現在我們再來修改server.js
```js
// 先做出一個ReadFile的Promise Function
function readFileAsync(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, (err, data) => {
      if (err) reject(err)

      resolve(data)
    })
  })
}

// 然後修改原本的路由
.get('/socket', async (ctx, next) => { // 一定要加入async這個關鍵字！

  // await 這個關鍵字會等待後面的Promise執行完取得結果後再繼續執行
  const data = await readFileAsync(__dirname + '/src/index.html')
  // 需要將readFile的結果轉成utf8，不然原本的資料型態是Buffer，沒有轉的話會變成直接下載檔案，而不是渲染。
  ctx.body = data.toString('utf8')

  await next()
})
```

現在重新執行`node server.js`，之後重新整理網頁應該會看到完整的頁面，這樣就代表有渲染成功了！

## socket.io
我們製作Socket監聽控制需要使用到一個package就是 **socket.io** ，他可以讓我們簡單的去實作Socket控制！

一開始就是一樣先安裝套件
```bash
yarn add socket.io
```

引入套件之後我們還需要去使用`http`這個官方的元件，去做網路監聽。

### server.js
安裝套件之後我們就來引入套件
完整的實作會是下面的code

```js
const Koa = require('koa'),
      app = new Koa(),
      router = require('koa-router')(),
      server = require('http').createServer(app.callback())
      // 這邊放入的app.callback()，如果不懂可以看官方的套件說明
      // https://github.com/socketio/socket.io#in-conjunction-with-koa

let io = require('socket.io')(server);

const fs = require('fs')

function readFileAsync(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, (err, data) => {
      if (err) reject(err)

      resolve(data)
    })
  })
}

router
  .get('/', async (ctx, next) => {
    ctx.body = 'Hello, World!'
    await next()
  })
  .get('/socket', async (ctx, next) => {
    const data = await readFileAsync(__dirname + '/src/index.html')
    ctx.body = data.toString('utf8')

    await next()
  })

io.on('connection', (socket) =>{
  console.log('a user connected');
  socket.on('message', (msg) => {
    // 使用io.emit 是對全部監聽者做廣播
    // socket.emit 是對單一用戶做廣播
    // 更詳情可以看官方文件
    io.emit('msg', {
      data: msg
    })
  })
})


app.use(router.routes())

server.listen(3000, function(){
  console.log('listening on *:3000')
})

```

### index.html

我們來修改一些功能，讓他可以接收與傳送訊息

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Socket Test</title>
</head>
<body>
  first Socket
  <input type="text" id="msg" value="">
  <button type="button" id="sendMsg">Msg</button>
  <div class="msgBox">
    <ul></ul>
  </div>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.0.1/socket.io.js"></script>
  <script>
    var socket = io.connect() // socket連線

    let btn = document.body.querySelector('#sendMsg'),
        msg = document.body.querySelector('#msg'),
        msgBox = document.body.querySelector('.msgBox > ul')
    console.log(msgBox)

    btn.addEventListener('click', (e) => {
      socket.emit('message', msg.value) // 對伺服器 emit訊息
    })
    msg.addEventListener('keydown', (e) => {
      if (e.keyCode === 13) { // 13 = Enter keyCode
        socket.emit('message', msg.value)
      }
    })
    socket.on('msg', (msg) => {
      // 當收到廣播時，要進行的處理
      let liDom = document.createElement('li'),
          textDom = document.createTextNode(msg.data)
      liDom.appendChild(textDom)

      msgBox.appendChild(liDom)
      console.log(msg)
    })
  </script>
</body>
</html>

```
