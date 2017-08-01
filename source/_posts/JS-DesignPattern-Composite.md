---
title: JavaScript 設計模型 - Composite
date: 2017-04-26 10:51:38
tags:
  - JavaScript
  - TypeScript
  - DesignPattern
  - js
---

原本是想要寫Decorator，但是發現他有點Composite Pattern的基礎，所以就想說先寫這個Pattern！
這個Pattern，可以使我們的Code能夠更組件化！能夠大大提升組件的覆用性，現在許多的前端框架也主打擁有Component，就像是樂高積木一樣，組起來就好！

## 起手式

『Composition over inheritance』雖然這句話是在物件導向比較適用，因為JS是FP不是OO阿！但是我們還是希望寫出覆用性高、耦合低和可維護性的Code，所以我們可以借鏡一下！

這個Pattern可以把他想像成資料夾與檔案的關係，檔案是Primitive Component、資料夾是Composite Component，資料夾裡面可以有檔案也可以有另外一個資料夾！有點『詩中有畫，畫中有詩』的感覺

<!--more-->

### 定義Interface
```ts

interface Component {
    readonly _name: String
    _level: number
    name: String
    operation(): void
    updateLevel(n: number): void
}

```

# 先貼上剩下的程式碼，過幾天再打詳解，最近太忙了，有點拖稿

```ts

class Directory implements Component {
    readonly _name: String
    _level: number = 0
    private directory: Component[] = []

    constructor(name: String) {
        this._name = name
    }
    get name() {
        return this._name
    }

    addComponent(c: Component) {
        c.updateLevel(this._level + 1)
        this.directory.push(c)
        this.directory.sort((a, b) => {
            if (typeof a === Directory.name) {
                return a.name < b.name? -1 : 1
            } else {
                return 1
            }
        })
    }
    operation() {
        // console.log(`${this._name}'s length -> ${this.directory.size} `)
        console.log(`${'\t'.repeat(this._level)}dir-> ${this._name}'s size -> ${this.directory.length}`)
        this.directory.map((c) => {
            c.operation()
        })
    }
    updateLevel(n: number) {
        this._level += n
        this.directory.forEach(c => c.updateLevel(this._level))
    }
}

class Filee implements Component {
    readonly _name
    _level: number = 0
    constructor(name: String) {
        this._name = name
    }
    get name() {
        return this._name
    }

    operation() {
        // process.stdout.write('\t')
        console.log(`${'\t'.repeat(this._level)}file -> ${this._name}`)
    }
    updateLevel(n: number) {
        this._level += n
    }
}

class Client1 {
    dir = new Directory('main')
    constructor() {
        const a = new Directory('a'),
                b = new Directory('b'),
                e = new Directory('e'),
                c = new Filee('c'),
                f = new Filee('f'),
                d = new Filee('d')
        a.addComponent(d)
        a.addComponent(b)
        this.dir.addComponent(a)
        this.dir.addComponent(c)
        b.addComponent(f)
        this.dir.addComponent(e)
    }

    start() {
        this.dir.operation()
    }
}

const c = new Client1()
c.start()

```
