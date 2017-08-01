---
title: JavaScript 設計模型 - Chain of Responsibility
date: 2017-04-25 16:07:37
tags:
  - JavaScript
  - TypeScript
  - DesignPattern
  - js
---

由於室友的物件導向軟體開發正在重修，想說有空可以去旁聽，順便複習一下以前所學的，正好正在教Chain of Responsibility，最近又因為正在學Angular，所以剛好有接觸TypeScript，所以就想說能不能應用上去，寫一寫發現真的還蠻有趣的，所以以後應該會慢慢地去實作其他的Pattern，順便讓我更增強自己的維護性。手上也有一本*JavaScript Design Pattern*這本書，所以這正是一個不錯的時機啊！

### 起手式

首先要先了解CoR這個Pattern是想要改善什麼東西！
想像一下，假如你需要去Handle一件事情時，他有很多種情況，那我們會怎麼寫呢？第一個直覺當然是if/switch，這是正常的，但是當可能處理事項有10/20/50種可能，那麼寫出來的code可能會跟阿婆的臭襪子一樣長，這就像是JS的Callback Hell，到後面你回頭要修改或是維護時，這時你會發現已經太晚了！所以我們可以把它拆解開來，將每個Handler拆開，將耦合降低，每個Handler只需要處理自己的能力範圍，如不能處理，就在繼續丟給下一個，一直丟到最後一個，這將會使程式更Flexible！

舉例：販賣機找零錢，假設找零為97元，這樣需要找 50\*1 + 10\*4 + 5\*1 + 1\*2

<!--more-->

### 定義Interface

首先我們要定義一個規格出來，這一才能統一。

```ts
interface Handler {
    next: Handler;
    handle(params: any): void;
    doNext?(params: any): void; //unnecessary
}
```

定義好之後我們可以實作Concrete Handler。

```ts
class FiftyHandler implements Handler {
    next: Handler;
    constructor (h: Handler) {
        this.next = h;
    }
    doNext(params: any): void {
        if (this.next) {
            this.next.handle(params);
        }
    }
    handle(params: any): void {
        if (+params % 50 !== 0) {
            console.log(`50元 -> ${Math.floor(+params / 50)} 個`);
        }
        this.doNext(+params % 50);
    }
}

class TenHandler implements Handler {
    next: Handler;
    constructor (h: Handler) {
        this.next = h;
    }
    doNext(params: any): void {
        if (this.next) {
            this.next.handle(params);
        }
    }
    handle(params: any): void {
        if (+params % 10 !== 0) {
            console.log(`10元 -> ${Math.floor(+params / 10)} 個`);
        }
        this.doNext(+params % 10);
    }
}

class FiveHandler implements Handler {
    next: Handler;
    constructor (h: Handler) {
        this.next = h;
    }
    doNext(params: any): void {
        if (this.next) {
            this.next.handle(params);
        }
    }
    handle(params: any): void {
        if (+params % 10 !== 0) {
            console.log(`5元 -> ${Math.floor(+params / 5)} 個`);
        }
        this.doNext(+params % 5);
    }
}

class OneHandler implements Handler {
    next: Handler;
    constructor (h: Handler) {
        this.next = h;
    }
    handle(params: any): void {
        if (+params % 10 !== 0) {
            console.log(`1元 -> ${+params} 個`);
        }
        console.log('Done!');
    }
}
```

需要擴充處理器，只需要時作出Handle，再將它加入到Chain裡面就可以了，不是每次都加上一大串的if/switch！
Ex.如果要再多一個百元找零，這樣就只要再新增一個HundredHandler就可以啦！

接下來是模擬Client使用這些Handler，來處理事情。

```ts
class Client {
    n: number;
    handler: Handler;
    constructor(n: number){
        this.n = n;
        this.handler =
        new FiftyHandler(
            new TenHandler(
                new FiveHandler(
                    new OneHandler(null)
                    )));
    }

    start() :void {
        this.handler.handle(this.n);
    }
}

const c1 = new Client(97);
c1.start();
```

### Output

```
50元 -> 1 個
10元 -> 4 個
5元 -> 1 個
1元 -> 2 個
Done!
```

大功告成，完成了簡易販賣機，擁有擴充性的Code！

### V2

假如覺得一層包一層這樣不好維護，也可以加一個setNext的Function！下面是Sample Code。

```ts
interface Handler {
    next: Handler;
    handle(params: any): void;
    doNext?(params: any): void; //unnecessary
    setNext(n: Handler): Handler;
}

class ConcreteHandler implements Handler {
    next: Handler;
    constructor() {
        this.next = null;
    }
    handle(params: any) {
        if (this.next) {
            this.next.handle(params);
        }
    }
    doNext(params: any) {
        if (this.next) {
            this.next.handle(params);
        } else {
            console.log(`${params} unprocess!`);

        }
    }
    setNext(n: Handler): Handler {
        this.next = n;
        return this.next;
    }
}

class FiftyHandler extends ConcreteHandler {
    handle(params: any): void {
        if (+params % 50 !== 0) {
            console.log(`50元 -> ${Math.floor(+params / 50)} 個`);
        }
        this.doNext(+params % 50);
    }
}

class TenHandler extends ConcreteHandler {
    handle(params: any): void {
        if (+params % 10 !== 0) {
            console.log(`10元 -> ${Math.floor(+params / 10)} 個`);
        }
        this.doNext(+params % 10);
    }
}

class FiveHandler extends ConcreteHandler {
    handle(params: any): void {
        if (+params % 10 !== 0) {
            console.log(`5元 -> ${Math.floor(+params / 5)} 個`);
        }
        this.doNext(+params % 5);
    }
}

class OneHandler  extends ConcreteHandler {
    handle(params: any): void {
        if (+params % 10 !== 0) {
            console.log(`1元 -> ${+params} 個`);
        }
        console.log('Done!');
    }
}

class Client {
    n: number;
    handler: Handler = new ConcreteHandler();
    constructor(n: number){
        this.n = n;
        const fifty = new FiftyHandler(),
              ten = new TenHandler(),
              five = new FiveHandler(),
              one = new OneHandler();
        this.handler.setNext(fifty).setNext(ten).setNext(five).setNext(one);
        //seq : fifty => ten => five => one
    }

    start() :void {
        this.handler.handle(this.n);
    }
}

const c1 = new Client(97);
c1.start();

```

### Output
```
50元 -> 1 個
10元 -> 4 個
5元 -> 1 個
1元 -> 2 個
Done!
```
