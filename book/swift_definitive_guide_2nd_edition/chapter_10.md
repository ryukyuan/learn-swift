# 10章 プロトコル

## 10.2 プロトコル型

### 型としてのプロtコル

* プロトコルはプログラム内で型として使える

```
func printNames(list:[Personal]) {
  for p in list {
    print(p.name + ": " + p.saySomething())
  }
}
```

* プロトコルPersonalに適合するインスタンスを要素とする配列listを引数として、内容の一覧を表示する

* ただし全てのプロトコルが方して使えるわけではない
* typealiasによる付属型の指定やSelfを使って宣言されたプロトコルは型として使えない

### プロトコロの継承

* 新しいプロトコロを宣言する際に別のプロトコルを軽装することができる

```
enum Sex { case Male, Female } // 性別を表す列挙型
protocol HealthInfo : Personal { // プロトコルを継承する
  var weight: Double { get set }  // 読み書き可能なプロパティ
  var height: Double { get set }
  var sex: Sex? { get }
}
```

* プロトコルは、必要に応じて複数個を採用してクラスや構造体を定義できる
* ただし、複数個のプロトコルで同盟のプロパティが異なる型で定義されている場合など、同時に採用できないプロトコルもある

### プロトコルの合成

* 複数のプロトコルで宣言しているインタフェースを和集合のように合わせ持つプロトコルを考えることができます

* 2つのプロトコルのインタフェースを持つ新しいプロトコル
```
protocol NewProtocol : プロトコル1, プロトコル2 {}
```

* 複数のプロトコルの和集合としての型を指定する方法
```
protocol<プロトコル1, プロトコル2>
```

* 以下の2つの記述は同じ意味になる
```
var teller: NewProtocol
var teller: protocol<プロトコル1, プロトコル2>
```

* ただしprotocol<>を使う記法はその箇所で一時的にのみ仕様する記法
* 頻繁に使う型であれば名前のついたプロトコルとして定義したほうがよい
* またクラスや構造体の宣言の先頭にも記述できない


## 10.3 プロトコルと付属型

### ネスト型とプロトコル

* プロトコルでも、型定義の内部で使われるネスト型に関して宣言を記述しておくことができる

```
protocol VectorType { // 2次元ベクトル型
  typealias Element  // 付属型の宣言
  var x : Element { get set }
  var y : Element { get set }
}
```

* 上の例ではVectorTypeの内部ではプロパティxとyの型がElementであるということが記述されている

* プロトコル内でtypealiasを使って宣言される付属型はジェネリクスの機能の1つ
* 型の定義にプロトコルが採用された時、その型で実際に使われる型にマッチし、プロトコルの記述を実際の定義に置き換える働きがある

```
struct VectorFloat : VectorType {
  var x, y: Float // 具体的な型を指定
  func convToDouble() -> VectorDouble {
    return VectorDouble(
       x: VectorDouble.Element(x), // 型として利用可能
       y: VectorDouble.Element(y))
  }
}
struct VectorDouble : VectorType, CustomStringCovertible {
  var x, y: Double
  var description: String { return "[\(x), \(y)]" }
}
struct VectorGrade : VectorType {
  enum Element { case A, B, C, D, X } // ネスト型を定義
  var x, y : Element
}
```

* 上の例ではVectorTypeを使って、Float型を値として持つ構造体VectorFloatと、Double型を値とする構造体VectorDoubleを定義している
* どちらの定義も自身の付属型のElementを直接には定義していない
* 一方、構造体VectorGradeの定義のように、付属型を改めてネスト型として定義したり、typealiasを使って型の別名を指定することができる

```
var a = VectorFloat(x: 10.0, y: 12.0)
let b = a.convToDouble()
print(b) // [10.0, 12.0] と表示される
```

* VectorDouble型で定義したdescriptionの形式で出力が行われている
* VectorFloat型からVecotrDouble型を作りだせていることがわかる

（最後に）
* このような付属型を持つプロトコルは変数などの型として利用することができない
* プロパティが返す型はそれぞれに異なっているため、共通のデータ型を扱うインターフェースとして機能しない




