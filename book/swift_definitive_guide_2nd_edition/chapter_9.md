# 9.1 参照型とARC

## ARCによるメモリ管理

* Swiftのメモリ管理はインスタンスが変数から参照されたり、されなくなったりすタイミングでリファレンスカウントの値を増減させることで行う
* このメモリ管理方式をARC（Automatic Reference Counting）と呼ぶ
* Objective-Cでは従来、プログラムがリファレンスカウンタの操作を行う方式を利用してきたが、これを自動化したのがARC
* ガーベージコレクションとは異なり、インスタンスが不要になるポイントが実行ドード上でわかっている
* このため処理は高速でメモリの解放に伴う負荷の集中も起きなくなる

```
func temp(entry:[String:Person]) { // 辞書型データを引数とする
    var list = [[String:Person]]()
    list.append(entry)
    print("\(list.count)個のエントリ)
}

temp( ["Natsuki: Person(name:"夏希, age:18] )
```

```
１個のエントリ
夏希: deinit
```

* temp関数が終了する変数listが使われなくなるので、格納されたデータも解放される
* 実行するとtemp関数の実行終了とともにPersonのインスタンスが解放される


## 値型データの共有とコピー

* 値型のデータについて
* C言語の場合配列を関数で処理する場合は、引数として配列の先頭のポインタを渡して無駄なコピーを防ぐ
* SwiftではCopy-On-Writeの方式で配列などの値型のデータを受渡している
* 最初に配列のポインタだけ私、データに変更がないかぎりは１つのデータを共有して参照し、元のデータ側、あるいは渡した先で書き込みが発生した時点でデータの複製を作成する
* 「書き換え時にコピーする」、コピーオンライト (Copy-On-Write) の基本的な形態

* List9-2では、変数に書き込むか書き込まないかで実行時間を比較したところ、書き込みがあるほうが明らかに実行速度に差がある事が分かる例
* 

# 9.3 オプションチェーン

## オプショナル型の値を連続利用する場合

* オプショナル型は、プロパティやメソッド呼び出しの返り値など、さまざまな場所で使われている
* 値がnilの可能性がある場合には必ずオプショナル型が使われている
* 弱い参照ではクラスのインスタンスを自動的に開放される目的で使われるが、常にオプショナル型とセットで指定される

* オプショナル型を開示して利用する場合、nilなのか何らかのインスタンスを参照しているかチェックが必要になる
* Swiftではオプショナル束縛公文やnil合体演算子のようにコンパクトかつ効果的な記述方法をいくつか提供している
* オプショナルチェーンもそのような記法の１つ

```
class Student {
  let name: String // 名前
  weak var club: Club // 弱い参照
  init(name:String) { self.name = name }
}

class Teacher {
  let name: String // 名前
  var subject: String? = nil // 担当教科
  init(name:String) { self.name = name }
}

class Club {
  let name:String // 名前
  weak var teacher: Teacher? // 弱い参照型
  var budget = 0 // 予算
  var members = [Student]()

  init(name:String) { self.name = name }

  func add(p:Student) {
    members.append(p)
    p.club = self
  }

  func legal() -> Bool {
    return members.count >= 5 && teacher != nil
  }
}
```

* Student型のオプショナル型変数whoがあった時、この変数が学生のインスタンスを参照していて、その学生がクラブに所属している場合、そのクラブに顧問の先生がいればその先生の名前を表示する、というプログラムを考える

* 以下の場合、変数whoがnilの場合や、学生がクラブに入っていない場合、あるいは顧問がいない場合に実行時エラーになる

```
print(who!.club!.teacher!.name)
```

* nilかどうかをif分で判断していくと以下のように書ける

```
if who != nui {
  if who!.club != nil {
    if who!.club!.teacher != nil {
      print(who!.club!.teacher!.name)
    }
  }
}
```

* オプショナル束縛構文を用いると開示していの「!」がなくなり、見た目の複雑さは少なくなるが、新たに定数の意味を把握して追跡しなければならなくなる

```
if let w  = who {
  if let club = w.club {
    if let tc = club.teacher {
      print(tc.name)
    }
  }
}
```

## オプショナルチェーンとは

* 以下のように繰り返しインスタンスを得て、さらにそのインスタンスのプロパティを参照して・・・という場合がある
* その判定の度にif分で判定を行うと記述が大変面倒になる
* このときに役立つのがオプショナルチェーン（optional chaining)
* 上の例は以下のようにコンパクトに記述できる

```
if let name = who?.club?.teacher?.name {
  print(name)
}
```

* clubで変数を得たい場合は、if分をたくさん並べるしかないが、目的のprintのみであれば上記の記述ようにコンパクトな記述は役立つ

* オプショナルチェーンはオプショナル型が利用される場面で幅広く利用することができる
* Swiftの辞書データはキーを使って値にアクセスする際、オプショナル型で値が返される
* この場合にもオプショナルチェーンを利用することができる

```
var studentCouncil = [String:Student]() // 生徒会の役員
studentCouncil["会長"] = Student(name:"日高")
```

* ここで常任委員という役職は空席であるとする
* 誰かが就いている場合にはその名前を表示したいという場合、「?」を使って次のように記述できる

```
if let name = studentCouncil["常任委員"]?.name {
  print("常任委員： " + name)
}
```
