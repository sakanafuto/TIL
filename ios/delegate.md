# delegate

```swift
import UIKit

protocol ButtonDelegate {
    func audio()
    func background()
}

class Button {
    var delegate: ButtonDelegate? = nil

    func click() {
        print("pushed")
        if let dg = self.delegate {
            dg.audio()
            dg.background()
        } else {
            print("do nothing")
        }
    }
}

class Button1: ButtonDelegate {
    func audio() {
        print("beep1")
    }
    func background() {
        print("background1")
    }
}

class Button2: ButtonDelegate {
    func audio() {
        print("beep2")
    }
    func background() {
        print("background2")
    }
}

let button = Button()

let button1 = Button1()
let button2 = Button2()

button.delegate = button1
button.click()

button.delegate = button2
button.click()
```

結果

```shell
pushed
beep1
background1
pushed
beep2
background2
```

## 参考

- [Swift における Delegate とは何か、なぜ使うのか](https://qiita.com/st43/items/9f9990d76cefa1909ef4)
- [プロトコルとデリゲートのとても簡単なサンプルについて](https://qiita.com/mochizukikotaro/items/a5bc60d92aa2d6fe52ca)
- [【Swift 入門】難解なデリゲート(delegate)の使い方を理解しよう！](https://www.sejuku.net/blog/33867)
