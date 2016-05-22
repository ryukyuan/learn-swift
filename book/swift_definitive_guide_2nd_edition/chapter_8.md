# 8章

## 8.1 クラス定義

### クラスメソッドとクラスプロパティ

* 構造体と列挙型にはタイプメソッドとタイププロパティ、つまり、その型に関係する手続きや値を定義することができた
* クラスも同様にタイプメソッドとタイププロパティを定義できる。定義の先導にstaticと記述する。
* 詳細はList8-4を参照。

* classから記述しはじめるメソッドとプロパティを定義できる。
* これらはクラスメソッド、クラスプロパティと呼ぶ

* クラスメソッド、クラスプロパティはサブクラスで定義を上書きできるが、タイプメソッド、タイププロパティはそれができない。
* また、クラスプロパティは格納型は定義できず、計算型に限定されている点もタイププロパティと異なる

### 整理

* 構造体、列挙型、クラスのすべてについて、タイプメソッドとタイププロパティが定義できる。
* これらの定義はstaticから書き始めるもので、格納型プロパティも定義可能。
* ただし、クラスにおいては、サブクラスで定義を上書きできません。

* クラス定義の場合のみ、クラスメソッドとクラスプロパティが定義できる。
* これらの定義はclassから書き始めるもので、格納型プロパティは定義できない。
* また、サブクラスで定義を上書きできる。

## 8.2 イニシャライザ

### クラス継承の例題

```
class DayOfMonth : CustomStringConvertible {
  var month, day: int
  init(month:Int, day:Int) {
    self.month = month; self.day = day
  }
  var description: String {
    return DayOfMonth.towDigits(month)
      + "/" + DayOfMonth.towDigits(day)
  }
  class func towDigits(n:Int) -> String {
    let i = n % 100
    if (i < 10) { return "0\(i)" }
    return "\(i)"
  }
}
```

* ベースクラスとして使うDayOfMonth
* 月と日で日付を表すクラスで、イニシャライザはごく普通のもの
* print表示ができるようにプロトコルCustomStringConvertibleを採用し、プロパティ descriptionを定義
* twoDigitsは整数を2桁で表すもの

* DayOfMonthを継承して、西暦年と曜日を追加したクラスDateを示す

```
func dayOfWeek(var m:Int, let d:Int, var year y:Int) -> Int {
  /* 省略：ツェラーの公式 */
}

class Date : DayOfMonth {
  var year: Int
  var dow: int
  init(_ y:Int, _ m:Int, _ d:Int) {
    year = y
    dow = dayOfWeek(m, d, year:y)
    super.init(month:m, day:d)
  }
}
```

* イニシャライザでは新しく追加したインスタンスヘンスを初期化してからスーパークラスのイニシャライザを呼ぶ
* 曜日を計算するのに以前にも登場したツェラーの公式を使う

* 最後にDateをさらに継承して、曜日を文字列で持つようにしたDateWクラスを示す

```
class DateW: Date {
  static let namesOfDay = [
    "Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat" ]
    var dweek = String()  // ここに注意
    override init(_ y:Int, _ m:Int, _ d:Int) { // overrideが必要
      super.init(y, m, d)
      dweek = DateW,nameOfDay[dow]
    }
    convenience init(_ m:Int, _ d:Int, year:Int = 2016) {
      self.init(year, m, d)
    }
    override var day:Int {
      didSet {
        dow = dayOfWeek(month, day, year:year)
        dweek = DateW.namesOfDay[dow] // 曜日の計算をやり直す
      }
    }
    override var description: String {
      return "\(year)/" + super.description + "(\(dweek))"
    }
}
```

* 最初のイニシャライザは指定イニシャライザだが、スーパークラスと同じ引数を使うためoverrideの指定が必要
* 次のイニシャライザは簡易イニシャライザで、西暦年を省略できるようにしている
* 次にプロパティオブサーバーを定義する。日付が変更された場合に曜日の計算をやり直すために定義
* プロパティオブサーバー自体の定義はスーパークラスにはないが、スーパークラスのdayの定義を変更することになるのでoverrideの指定が必要

* クラスDateWで簡易イニシャライザが呼ばれると、このイニシャライザは何もせずに同じクラスの指定イニシャ者ライザにデリゲートするだけ
* initの前にselfをつけないとコンパイルエラーになる

* dweekを初期化しているためスーパークラスのイニシャライザが動作する時に不定値とは見なされない
* 「指定イニシャライザは、スーパークラスに初期化を任せる前に、そのクラスで追加された変数、定数の初期化を済ませていなければならない」のがルールだが初期化しているためエラーとならない

* この例でdweekは定数にはできない
* 定数は初期値を指定しておくか、スーパークラスのイニシャライザを呼び出す前に指定イニシャライザで値を設定する必要がある


### 初期化は結局どうすればよいのか

#### すべての場合
* プロパティの値がすべて設定されるまで、自らのインスタンスメソッドは利用できない
* selfを値として使うこともできない
* プロパティに初期値が設定されている場合、イニシャライザで値を設定しなおさなくても構わない

#### スーパークラスがない場合
* イニシャライザではプロパティの値を適切に設定できれば問題ない

#### スーパークラスの有無によらず、同じクラスの別のイニシャライザを利用する場合
* この場合、自身は簡易イニシャライザになる
* プロパティの変数に独自の値を設定できるのはイニシャライザの呼び出しの後
* 定数には値は設定できない

#### スーパークラスがあり、その指定イニシャライザを呼び出す場合
* 自身も指定イニシャライザになる
* スーパークラスのイニシャライザを呼び出す前に、サブクラスで新たに追加したプロパティの値を決定する
* イニシャライザを呼び出した後、プロパティの変数には値を設定しなおすこともできる

#### 初期化が一通り済んだ後で、改めてプロパティの初期値を決めたいという場合
* 後述する遅延格納型プロパティが役立つかもしれない
* ただし、定数には適用できない

## 8.3  継承とサブクラス定義

### 継承とプロパティオブザーバ

* クラスはプロパティオブザーバを定義し、継承できる
* サブクラスではスーパークラスで定義したプロパティを監視することができる
* 監視対象のプロパティは格納型プロパティだけではなく、計算型プロパティも可能
* ただし、読み取り専用のプロパティは監視できない
* 同じクラスで定義されている計算型プロパティのset節を監視することもできない

* プロパティオブザーバはサブクラスの定義でスーパクラスの定義が上書きされ動作するわけではない
* サブクラスのオブザーバもスーパークラスのオブザーバも順番に呼び出される

```
class Propa {
	var attr = 0
}

class Propb : Propa {
	override var attr : Int {
		willSet{ print("B: will set") }
		didSet{ print("B: did set") }
	}
}

class PropGamma : PropB {
	override var attr : Int {
		willSet{ print("G: will set") }
		didSet{ print("G: did set") }
	}
}
```

```
var g = PropGanmma()
g.attr = 1
```

```
G: will set
B: will set
B: did set
G: did set
```

### 継承をさせない指定

* 継承をして機能の追加、変更をさせないためにはfinalという修飾語を用いる
* キーワードstaticを使って定義するクラスのタイププロパティは、finakの宣言されたクラスプロパティと同じ扱いのよう

## 8.4 解放時処理

### デイニシャライザを使った例

* ファイルからテキストを読み込むクラスを作成し、デイイニシャライザの例を示す

* 指定したファイルから文字列を読み込み特定の文字列が読み込まれたらデイニシャライザを処理する

```
class ReadWord {
    var f@: UnsafeMutablePinter<File> = nil
    init?(open:String) { // 失敗のあるイニシャライザ
        fp = fopen(open, "r") // fpはC言語のFILE*型
        if fp == nil {return nil}
    }
    deinit { // デイニシャライザ
        print("[[deinit]]")
        self.close() // 動作確認のために文字列を表示する
    }
    func close() {
        if fp != nil { // オープンされていればクローズする
            fclose(fp)
            fp = nil
        }
    }
    func nextWord() -> String? {
        var ch:Int
        repeat {
            ch = Int( fgetc(fp) ) // 1バイト読み込み
            if cf < 0 { // EOFの場合にnilを返す
                return nil
            }
        } while ch <= 0x20 // ASCIIの空白（改行やタブ）を読み飛ばす
        var s = String(UnicodeScalar(ch)) // 1文字だけの文字列で初期化する
        while true { // 非空白文字の列を作る
            ch = Int( fgetc(fp) )
            if ch <= 0x20 {
                break
            }
            s.append(UnicodeScalar(ch))
        }
        return s
    }
}
```

```
func readIt() {
    if let reader = ReadWord(open:"text.txt") { // オープンできれば
        while let w = reader.nextWord() { // オプショナル束縛構文
            if == "Q!" {
                return
            }
            print(w, terminator:" ") // 文字列を表示
        }
        print("(EOF)")
        reader.close() // 処理を終了
    }
}
```


* 実行するとインスタンスが解放されるタイミングでデイニシャライザが動作し、「後始末」ができていることが確認できる

