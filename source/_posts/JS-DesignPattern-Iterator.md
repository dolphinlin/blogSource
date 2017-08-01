---
title: JavaScript 設計模型 - Iterator
date: 2017-04-29 23:03:15
tags:
  - JavaScript
  - TypeScript
  - DesignPattern
  - js
---

Iterator Pattern是一個很重要也很簡單的Pattern：迭代器！
我們可以提供一個統一入口的迭代器，Client只需要知道有哪些方法，或是有哪些Concrete Iterator，並不需要知道他們底層如何實作！現在就讓我們來開始吧！

### 起手式
Iterator最主要的東西就是兩個：hasNext、next。要讓Client知道是否還有下一個，和切換到下一個！
<!--more-->


### 定義Interface

```ts
interface IteratorInterface {
    index: number
    dataStorage: any

    hasNext(): boolean
    next(): any
    addItem(item: any): void
}
```

### 實作介面
下面的範例我將會使用Map、Array這兩個常見的介面實作。

```ts
// array
class iterator1 implements IteratorInterface {
    index: number
    dataStorage: any[]

    constructor() {
        this.index = 0
        this.dataStorage = []
    }

    hasNext(): boolean {
        return this.dataStorage.length > this.index
    }

    next(): any {
        return this.dataStorage[this.index ++]
    }

    addItem(item: any): void {
        this.dataStorage.push(item)
    }
}
```

```ts
// map
class iterator2 implements IteratorInterface {
    index: number
    dataStorage: Map<number, any>

    constructor() {
        this.index = 0
        this.dataStorage = new Map<number, any>()
    }

    hasNext(): boolean {
        return this.dataStorage.get(this.index) != undefined
    }
    next(): any {
        return this.dataStorage.get(this.index ++)
    }
    addItem(item: any): void {
        this.dataStorage.set(this.dataStorage.size, item)
    }
}
```

### Client
我沒有實作一個Client，所以我是直接new一個類別出來直接使用！

```ts
const i = new iterator1()

i.addItem(123)
i.addItem(456)
i.addItem('dolphin')

while(i.hasNext()){
    console.log(i.next())
}

console.log(`====================`)

const i2 = new iterator2()

i2.addItem(123)
i2.addItem(456)
i2.addItem('dolphin')

while(i2.hasNext()){
    console.log(i2.next())
}
```

### 結論
會發現Iterator 1號 2號的結果都是一樣的！他們都只需要讓Client知道有hasNext、next就好，底層的實作不需要讓他們知道！
