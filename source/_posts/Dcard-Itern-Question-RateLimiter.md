---
title: Dcard Web Intern 題目 - Rate Limiter
date: 2017-05-10 15:39:46
tags:
  - JavaScript
---

題目如下

---

Dcard 每天午夜都有大量使用者湧入抽卡,為了不讓伺服器過載,請設計一個 middleware:

- 限制每小時來自同一個 IP 的請求數量不得超過 1000
- 在 response headers 中加入剩餘的請求數量 (X-RateLimit-Remaining) 以及 rate limit
- 歸零的時間 (X-RateLimit-Reset)
- 如果超過限制的話就回傳 429 (Too Many Requests)
- 可以使用各種資料庫達成

---

這題我是使用Koa@2來實作，順便當作練習Koa，之前雖然寫過ExpressJS，但是Koa跟Express有點不太一樣，可是轉換起來應該還是可以無縫接軌！

## 起手式
首先就先簡單的引入yarn(npm)來安裝koa！我是用yarn，所以下面範例都是用yarn來當做範例！

整個middleware流程大概是
routing -> get ip -> query database                               -> return res and set headers
                  ==> null -> add a new data row
                  ==> validity period -> update data
                  ==> expired -> update
                  ==> validity period && too many req-> reject request

```bash
take middleware
# take = (mkdir middleware & cd middleware)
yarn init
```

<!--more-->

## 引入Koa並測試

```js
const Koa = require('koa'),
      app = new Koa()

app.use(ctx => {
  ctx.body = 'Hello world'
})
app.listen(3000) //listen port
```

### 執行伺服器

```bash
node app.js
# go to localhost:3000
```

在瀏覽器正常來說會看到 'Hello world'!

## 開始編寫Middleware
新建一個`middleware.js`，必且在主js引入後使用它。

### Middleware.js

```js
module.exports = function (options = {}) {
  return async function (ctx, next) {
    console.log('middleware test log')

    await next()
  }
}
```

### App.js

```js
const Koa = require('koa'),
      app = new Koa(),
      RateLimit = require('./middleware')

app.use(RateLimit())
app.use(ctx => {
  ctx.body = 'Hello world'
})

app.listen(3000)
```

### node app.js
執行`node app.js`後，到瀏覽器觀看網站，會得到一樣的`Hello world`，但是不一樣的是回去看Terminal，會發現多了一條`middleware test log`，因為我們有`app.use(RateLimit())`，只要經過所有的路由都會先經過這個middleware。

## 引入資料庫
題目上面說任何資料庫都可以使用，這樣我這邊是用MySql，所以下面會教如何使用。

### Install Mysql

```bash
yarn add mysql
```

### middleware.js

這邊會在載入的時候會需要一些options，所以我們需要在function params裡面加入一些參數，並且使用es6解構來解構Params。
```js
const mysql = require('mysql')

module.exports = function (options = {}) {
  const {host, user, password, database} = options //解構Params
  const RATELIMIT_TIME = options.timeout || 2400 * 1000 //設定timeout時間 單位：ms

  const connection = mysql.createConnection({
    host,
    user,
    password,
    database,
  }) // 建立mysql連線

  function QueryData(ip) {
    // return Promise
    return new Promise((resolve, reject) => {
      const cmd = 'SELECT * FROM ratelimit WHERE ip = ? LIMIT 1;'

      // execute sql
      // see -> https://github.com/mysqljs/mysql#escaping-query-values
      // 問號欄位對應陣列順序
      connection.query(cmd, [ip], (err, row) => {
        if (err) {
          reject(err)
        } else {
          resolve(row)
        }
      })
    })
  }


  function CreateNewData(ip) {
    return new Promise((resolve, reject) => {
      const cmd = 'INSERT INTO ratelimit (ip, remaining, reset) VALUES (?, 999, ?);'

      connection.query(cmd, [ip, Date.now() + RATELIMIT_TIME], (err, row) => {
        if (err) {
          reject(err)
        } else {
          resolve(row)
        }
      })
    })
  }

  function UpdateIPData(ip) {
    return new Promise((resolve, reject) => {
      const cmd = 'UPDATE ratelimit SET remaining = 999, reset = ? WHERE ip = ?;'

      connection.query(cmd, [Date.now() + RATELIMIT_TIME, ip], (err, row) => {
        if (err) {
          reject(err)
        } else {
          resolve(row)
        }
      })
    })
  }

  function UpdateIPReamining(ip, n) {
    return new Promise((resolve, reject) => {
      const cmd = 'UPDATE ratelimit SET remaining = ? WHERE ip = ?;'

      connection.query(cmd, [n, ip], (err, row) => {
        if (err) {
          reject(err)
        } else {
          resolve(row)
        }
      })
    })
  }

  return async function (ctx, next) {

    await next()
  }
}
```

# 未完待續！
