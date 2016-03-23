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

* 実態数が整数の場合、リテラで値を指定しなかったメンバは、１つ前のメンバの値に１を加えたものになる
* 先頭のリテラルに値がない場合は値は0となる
* メンバの中に同じ値を持つものがないようにしなければならない
* 実体型が文字列で、リテラルを指定しない場合、メンバの文字列が実体型の値になる
* 整数、文字列以外の型が実体型の場合、各メンバに対して必ずリテラルを記述し、それぞれが異なるようにする

```
enum Direction : Int {
   case Up = 0, Down, Right, Left
}
```

* 最初のUpの実体値を0にしたので、それ以降のDown、Right、Leftはそれぞれ1、2、3を値に持つことになる
* それぞれが持つ実体値を取り出すためにrawValueというプロパティを使うことができる

```
let a = Direction.Right
let i = a.rawValue // i = 2 (Int)
let k = Direction.Down.rawValue // k = 1 (Int)
```

* 実体型の値からそれに対応する列挙型のインスタンスを得るには、rawValueというキーワードの引数を１つもつイニシャライザを使う
* 実体型の値に対応する列挙型がない場合は、列挙型のオプショナル型になる

```
let b : Direction? = Direction(rawValue:3)
b! == Dreciton.Left // true
if let c = Direction(rawValue:2) { // オプショナル束縛構文
    print("OK \(c.rawValue)") // "OK 2"
}
```

* 値型の列挙型を用いてDirecitonの列挙型を書き直す

```
enum Direction : Int {
    case Up = 0, Right, Down, Left
    func clockwise() -> Direction {
        let t = (self.rawValue + 1) % 4 // selfはインスタンス自体
        return Direction(rawValue:t)! // nilにはならない
    }
}
```

### 列挙型に対するメソッドとプロパティ

* 列挙型に定義できるプロパティは計算型のプロパティのみ

```
enum Direction : Int {
    case Up = 0, Right, Down, Left
/* ... 中略 ... */
    var horizontal : Bool {
        switch self {
        case Right, Left: return true
        default: return false
        }
    }
    mutating func turnBack() {
        self = Direction(rawValue:((self.rawValue + 2) % 4))!
    }
}
```

* 値が右か左かを表す計算型プロパティのhorizontalを定義
* 値の正反対方向に反転するメソッドturnVackをmutating属性を持つメソッドとして定義することで列挙型自体の値を変更している

```
var d = Direction.Left
print(d.rawValue) // 3が出力される
d.turnBack
print(d.rawValue) // 1が出力される（3の反対方向）
print(d.horizontal) // trueが出力される
```


### 列挙型のタイプメソッド

* 列挙型にも、構造体と同様にタイプメソッド、タイププロパティ、イニシャライザを定義することができる
* タイプメソッドとタイププロパティは、先頭にキーワードstaticを記述する
* イニシャライザではselfに何を代入するかを定義することになる

```
enum Direction : Int {
    case Up, Right, Down. Left
    static var defaultDirection = Direction.Up
    init() {
        self = Direction.defaultDirection
    }
    static func arrow(d:Direction) -> String {
        return["↑", "→", "↓", "←"][d.rawValue]
    }
}
```

* イニシャライザinit()を使ってインスタンスを静止したあと、Direction.defaultValueを使い、値を初期化している
* タイプメソッドとしてインスタンスに対応する向きを持つ矢印を返すarrowというメソッドを定義している

```
Direction.defaultDireciton = .Right // 右向きを初期値にする
var e = Dirreciont() // イニシャライザを使う
print(Direciont.arrow(e)) // "→"を出力
```






























