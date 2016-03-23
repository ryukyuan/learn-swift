
# 6章

## 6.2 列挙型

### シンプルな列挙型

* 特定の意味を持つ整定数の集合を作成し、１つの型として扱うことができる列挙型
* Swiftでは、それ独自のメソッドを定義できるなど、かなり拡張されている

```
enum Direction {
    case Up
    case Down
    case Right
    case Left
}
```

* UpやDownなどは列挙型のメンバ（member）と呼ぶ
* メンバは定数名のため、大文字開始のキャメルケースで記述する

* 次のようにも記述できる

```
enum Direction {
    case Up, Down, Right, Left
}
```

* 宣言したメンバは変数や定数に代入できる
* メンバ名はグローバルな名前ではないため、列挙型の名前と「.」を使って記述する
* ただし、使われる列挙型が明らかな場合には列挙型名を省略し、「.」だけを先頭に付けたき方が使える
* メンバ名は単独で使われることはないため、他の列挙型のメンバ名と重複しても問題ない

```
let d = Direction.Up
var x : Direction = .Right
d == x // false. 違う値
```

* 上記の方法で宣言されたメンバはそれぞれ異なる値が割り当てられる
* そのため相互に比較することができる
* これだけでも列挙型として使うこともできる
* 割り当てられている値は整数というわけではなく、「比較するとそれぞれ違うことがわかる」というものだけである


### メソッドの定義

* Swiftでは列挙型の定義にメソッドを含めることができる

```
enum Direction {
    case Up, Down, Right, Left
    func clockwise() -> Direction {
        switch self {
            case .Up : return Right    // 90°回転した方向
            case .Down : return Left
            case .Right : return Down
            case .Left : return Up
        }
    }
}
```

* それぞれの方向から時計回りで90°回転した時の方向を返すclockwise()

```
let d = Direction.Up
d.clockwise() == Direction.Down  // false
d.clockwise().clockwies() = Direction.Down // true
```

*  一般にはDirection.Upと各が、型が判明しているときは「.Up」と省略でき、「Up」のようにメンバ名だけで記述できるのは同じ列挙型の定義内だけである


### 値型の列挙型

* Swiftの列挙型には、全メンバが同じデータ型の何らかの値を持つ「値型」、それぞれのメンバが異なる構造を持ち、インスタンスごとに値を変えることができる「共有型」がある

* 値型（value type）の列挙型では、すべてのメンバが同じ型の値を持つように定義できるが、メンバの値は互いに異なっている必要がある
* メンバに共通するデータ型を、その列挙型の実体型（raw type）と呼ぶ
* 各メンバに割り当てられた値を実体値（raw value）と呼ぶ

* 新しく定義する型名の後に、「:」と実体型を記述する
* それぞれのメンバには値を記述することができるが、ここで記述できるは整数、実数、Bool値、また文字列リテラルだけである

```
enum 型名 : 実体型 {
    case メンバ名 = リテラル
    case メンバ名 = リテラル
}
```

























