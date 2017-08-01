---
title: Dcard Web Intern 題目 - Card
date: 2017-05-06 18:00:15
tags:
  - JavaScript
---

突然有一天看到Dcard有在徵暑假實習生，往下滑一滑居然看到要寫小作業，看了看題目覺得還蠻有趣的，就慢慢寫了幾題，裡面有前端也有後端，最後一題還是演算法，看都看不懂所以果斷放棄哈哈！

這是第二題，題目是在說有6種情況（類似撲克牌，因為題目沒有寫得很清楚，所以不知道有沒有花色大小問題，我下面是忽略花色大小，只計算點數）
  - （同花順）Straight flush (five cards in sequence, all of the same suit)
  - （鐵支）Four of a kind (four cards of one rank and any other (unmatched) card)
  - （葫蘆）Full house (three matching cards of one rank and two matching cards of another rank)
  - （同花）Flush (five cards are of the same suit, but not in sequence)
  - （順子）Straight (five cards of sequential rank in at least two different suits)
  - （單隻）any unmatched card

## 起手式

因為是小作業，順便練練手感，所以嘗試使用TDD，所以會先寫測試，再來寫Code。
一開始先建立mock data，再開始寫測試。

<!--more-->

### Mock Deck

```js
function f(v, s) {
  return {
    value: v,
    suit: s
  }
}
const StraightFlush = [
  f(9, 1),
  f(10, 1),
  f(11, 1),
  f(12, 1),
  f(13, 1),
]

const FourKind = [
  f(3, 1),
  f(3, 2),
  f(3, 4),
  f(5, 1),
  f(3, 3),
]

const FullHouse = [
  f(4, 1),
  f(4, 2),
  f(4, 4),
  f(10, 1),
  f(10, 3),
]

const Flush = [
  f(5, 2),
  f(4, 2),
  f(6, 2),
  f(12, 2),
  f(10, 2),
]

const Straight = [
  f(9, 2),
  f(8, 4),
  f(6, 2),
  f(7, 3),
  f(10, 1),
]

const Unmatch = [
  f(9, 2),
  f(8, 3),
  f(10, 2),
  f(7, 3),
  f(10, 1),
]

const EmptyDeck = []

class DeckFactory {
  constructor() {
    this.resetPool()
  }
  createRandomDeck() {
    let deck = []
    const DECKNUM = 5
    for (let i = 1; i <= DECKNUM; i++) {
      deck.push(this.cardPool.splice(Math.floor(Math.random() * this.cardPool.length) - 1, 1)[0])
    }
    return deck
  }
  resetPool() {
    const SUITNUM = 4, CARDNUM = 13
    this.cardPool = []
    for (let i = 1; i <= SUITNUM; i++) {
      for (let j = 1; j <= CARDNUM; j++) {
        this.cardPool.push({
          value: j,
          suit: i
        })
      }
    }
  }
}

module.exports = {
  StraightFlush,
  FourKind,
  FullHouse,
  Flush,
  EmptyDeck,
  Straight,
  Unmatch,
  DeckFactory,
}
```

就只是單純模擬牌型，所以應該不太有什麼需要解釋的。最後的DeckFactory，是用來隨機模擬牌組。

### Test

預計會有一個牌組分析類別（DeckCompare）可以分析牌組或是比較大小。
然後分析每個牌組是否正確

```js
var chai = require('chai'),
    assert = chai.assert

var DeckCompare = require('./../index'), // 引入主程式
    DeckConfig = require('./specialDeck') // 引入模擬牌組

describe('DeckCompare', () => {
  const DC = new DeckCompare()
  describe('constructor', () => {
    it('should return Object', (done) => {
      assert.typeOf(DC, 'object')
      done()
    })
    it('All should return Function ', (done) => {
      assert.typeOf(DC.analysis, 'function')
      assert.typeOf(DC.compare, 'function')
      done()
    })
  })
  describe('#analysis()', () => {
    it('StraightFlush', (done) => {
      assert.equal(DC.analysis(DeckConfig.StraightFlush), 'StraightFlush')
      done()
    })
    it('FourKind', (done) => {
      assert.equal(DC.analysis(DeckConfig.FourKind), 'FourKind')
      done()
    })
    it('FullHouse', (done) => {
      assert.equal(DC.analysis(DeckConfig.FullHouse), 'FullHouse')
      done()
    })
    it('Flush', (done) => {
      assert.equal(DC.analysis(DeckConfig.Flush), 'Flush')
      done()
    })
    it('Straight', (done) => {
      assert.equal(DC.analysis(DeckConfig.Straight), 'Straight')
      done()
    })
    it('Unmatch', (done) => {
      assert.equal(DC.analysis(DeckConfig.Unmatch), 'Unmatch')
      done()
    })
    it('EmptyDeck', (done) => {
      try {
        assert.typeOf(DC.analysis(DeckConfig.EmptyDeck), 'error')
      } catch (e) {
        done()
      }
    })
  })
  describe('#compare()', () => {
    it('StraightFlush > FourKind', (done) => {
      assert.equal(DC.compare(DeckConfig.StraightFlush, DeckConfig.FourKind), 1)
      assert.equal(DC.compare(DeckConfig.FourKind, DeckConfig.StraightFlush), 2)
      assert.equal(DC.compare(DeckConfig.StraightFlush, DeckConfig.StraightFlush), 3)
      done()
    })
    it('FourKind > FullHouse', (done) => {
      assert.equal(DC.compare(DeckConfig.FourKind, DeckConfig.FullHouse), 1)
      assert.equal(DC.compare(DeckConfig.FullHouse, DeckConfig.FourKind), 2)
      assert.equal(DC.compare(DeckConfig.FourKind, DeckConfig.FourKind), 3)
      done()
    })
    it('FullHouse > Flush', (done) => {
      assert.equal(DC.compare(DeckConfig.FullHouse, DeckConfig.Flush), 1)
      assert.equal(DC.compare(DeckConfig.Flush, DeckConfig.FullHouse), 2)
      assert.equal(DC.compare(DeckConfig.FullHouse, DeckConfig.FullHouse), 3)
      done()
    })

    // 隨機牌組要用肉眼判斷，所以會列印出牌組和比較結果
    // 有一個notEqual是用來跟同花順比較的，因為mock deck裡面的同花順是最大（9-13同花順）
    // 所以絕對不可能會有第二副牌大於他的選項
    // 所以notEqual 2
    it('RandomDeck Test', (done) => {
      const DeckFactory = new DeckConfig.DeckFactory()
      const deck1 = DeckFactory.createRandomDeck(),
            deck2 = DeckFactory.createRandomDeck()
      console.log(deck1)
      console.log(deck2)
      console.log(DC.compare(deck1, deck2))
      assert.notEqual(DC.compare(DeckConfig.StraightFlush, deck1), 2)
      done()
    })
  })
})
```

## 開始動手做

我們先照著測試的內容把主程式exports出去！
```js
class DeckCompare {
  constructor() {

  }
  analysis(d) {
    return 'Deck class'
  }
  compare(d1, d2) {
    return 1 | 2 | 3
  }
}

module.exports = DeckCompare
```

這個題目可以運用之前的CoR Pattern來實作，這樣以後可隨時隨地更改、擴充Handler功能，也可以更改牌型大小的順序！

Base Handler
```js
class Handler {
  constructor() {
    this.value = 0
  }
  setNext(next) {
    //  設置下一個Handler
    next._setValue(this.value + 1)
    this.next = next
    return this.next
  }
  doNext(params) {
    // 呼叫next
    if (this.next) {
      return this.next.handle(params)
    } else {
      return `deck unprocess`
    }
  }
  handle(params) {
    // 處理
    if (params.length !== 5) {
      // 5張牌
      throw new Error('deck length error!')
    }

    //排序過後會使我們的排比較好處理
    params.sort((a, b) => a.value - b.value) //sort deck
    return this.doNext(params)
  }
  _setValue(v) {
    //此value用來判斷順序，數字越大代表越深層，代表牌型越小。
    this.value = v
  }
}
```

### StraightFlush - 同花順
同時都是同樣花色並且是順子！

```js
class StraightFlushHandler extends Handler {
  handle(params) {

    const sameSuit = params.every((card, index, array) => {
      return card.suit === array[0].suit
    })
    // 判斷同樣花色
    // 在這裡可以同時判斷順子 同花
    // 可以直接return 但是我這裡沒有做 只有單純判斷是否為同花順
    if (sameSuit) {
      // 因為有排序過後 所以可以直接every去做檢查
      const sequence = params.every((card, index, array) => {
        // 當前與下一張看是不是 差1
        return card.value === array[0].value + index
      })
      if (sequence) {
        const sum = params.reduce((p, c) => p + c.value, 0)
        return `${this.value + sum * 0.01} - StraightFlush`
      } else {
        return this.doNext(params)
      }
    } else {
      return this.doNext(params)
    }
  }
}
```

### FourKind - 鐵支
4張相同數字搭配一張雜牌。

```js
class FourKindHandler extends Handler {
  handle(params) {
    //first handle sorted

    // 比較 0 - 1~4
    const tmp1 = params.slice(0, 4).every((card, index, array) => {
      return card.value === array[index - 1 < 0 ? 0 : index - 1].value
    })

    // 比較 0~3 - 4
    const tmp2 = params.slice(1).every((card, index, array) => {
      return card.value === array[index - 1 < 0 ? 0 : index - 1].value
    })

    if ((tmp1 && (params[0].value !== params[4].value)) || (tmp2  && (params[0].value !== params[1].value))) {
      return `${this.value} - FourKind`
    } else {
      return this.doNext(params)
    }
  }
}
```

### FullHouse - 葫蘆
3張相同配2張相同。

```js
class FullHouseHandler extends Handler {
  handle(params) {
    //first handle sorted

    // 判斷 0~2相同 && 3.4 相同
    const tmp1 = params.slice(0, 3).every((card, index, array) => {
      return card.value === array[index - 1 < 0 ? 0 : index - 1].value
    }) && params[3].value === params[4].value

    // 判斷 0.1相同 && 2~4 相同
    const tmp2 = params.slice(2).every((card, index, array) => {
      return card.value === array[index - 1 < 0 ? 0 : index - 1].value
    }) && params[0].value === params[1].value

    if (tmp1 || tmp2) {
      return `${this.value} - FullHouse`
    } else {
      return this.doNext(params)
    }
  }
}
```

### Flush - 同花
5張牌都同一個花色

```js
class FlushHandler extends Handler {
  handle(params) {

    // 檢查是否所有花色都與第一張相同
    const sameSuit = params.every((card, index, array) => {
      return card.suit === array[0].suit
    })

    if (sameSuit) {
      return `${this.value} - Flush`
    } else {
      return this.doNext(params)
    }
  }
}
```

### Straight - 順子
所有牌都是有順序的！
（沒有檢查是不是同花順是因為前面第一個就handler就檢查過了）

```js
class StraightHandler extends Handler {
  handle(params) {
    //first handle sorted

    const sequence = params.every((card, index, array) => {
      return card.value === array[0].value + index
    })

    if (sequence) {
      return `${this.value + params[4].value * 0.01} - Straight`
    } else {
      return this.doNext(params)
    }
  }
}
```

### Unmatch - 雜牌
這個就是最後一個了，前面都沒有的就是這個了！

```js
class UnmatchHandler extends Handler {
  handle(params) {
    const max = params.reduce((a, b) => {
        return Math.max(a, b.value);
    }, 0)
    return `${this.value + max * 0.01} - Unmatch`
  }
}
```

### DeckCompare
所有的handler都做好了，這樣就可以開始把它們串起來了。

```js
class DeckCompare {
  constructor() {
    this.handler = new Handler()
    const h1 = new StraightFlushHandler(),
        h2 = new FourKindHandler(),
        h3 = new FullHouseHandler(),
        h4 = new FlushHandler(),
        h5 = new StraightHandler(),
        h = new UnmatchHandler()
        // u can change the handler sequence, and also add new handler
        // 在這邊可以更改順序或是新增想要的處理器

    this.handler.setNext(h1).setNext(h2).setNext(h3).setNext(h4).setNext(h5).setNext(h)
  }
  analysis(d) {
    // 分析牌組
    return this.handler.handle(d).split(' - ')[1]
  }
  compare(d1, d2) {
    // 比較牌組

    const d1value = this.handler.handle(d1),
          d2value = this.handler.handle(d2)


    const d1valueNum = +d1value.split(' - ')[0],
          d2valueNum = +d2value.split(' - ')[0]
    if (d1valueNum !== 0 && d2valueNum !== 0) {
      if (Math.floor(d1valueNum) < Math.floor(d2valueNum)) {
        return 1
      } else if (Math.floor(d1valueNum) === Math.floor(d2valueNum)) {
        if ((d1valueNum % 1) === (d2valueNum % 1)) return 3
        return (d1valueNum % 1) > (d2valueNum % 1) ? 1 : 2
      } else {
        return 2
      }
    } else {
      throw new Error('deck unprocess')
    }
  }
}
```

## 結論
雖然可以直接用switch做出來，但是就像我之前在介紹[CoR Pattern](https://dolphinlin.github.io/2017/04/25/JS-DesignPattern-COR/)的時候說的一樣，假如要再新增功能或是修改順序，那個程式將會變得很難讀，也沒有彈性！

假如有任何錯誤歡迎提出來，因為還是牛刀小試，多多少少都有Bug！哈哈哈
