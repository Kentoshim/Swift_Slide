---
marp: true
theme: gaia
---

# Swift

1. Swiftの特徴
2. 型について
3. 変数宣言
4. 関数の定義
5. オプショナルについて
6. enum
7. パターンマッチ
8. closure
9. delegate
10. Protocol
11. extension

---

# 1.Swiftの特徴

- 値型を中心としたマルチパラダイム言語
  - classでできることは大体structでできる
  - 関数型言語の影響を受けたシンタックス、機能
- 柔軟な表現が可能
  - 外部引数名、タイプエイリアス
- 他にもいろいろ
- ***それらを台無しにするUIKitとXcodeの残念さ***

---
# 2.型について
- String
  - 文字列
- Int
  - 整数
この辺は割愛します

---
- struct
  - 構造体(C#にもひっそりとある値型のクラスみたいなやつ)
  - classとの違いは値型であることってくらいの説明しかされてないような気がします
    - 最も大きな違いは継承が不可能な点ですがSwiftではProtocolやextensionでうまいことやってます
  - Swiftではよく出てくる
---
- Tuple
  - C#にも最近追加された
  - `(Int, String)`のように複数の値（型が異なってもいい）を格納できる
  - 複数の値を戻り値にしたいとか、匿名型使うほどじゃない時に使います

---

# 3.変数宣言

## 変数
```swift
let hoge: String
var hoge: String
```
- `let`: 再代入不可能な定数宣言のキーワード
- `var`: 再代入可能な変数宣言のキーワード
- `hoge`: 変数名
- `:String`: 型名

---

- Swiftにおいて基本の変数宣言は`let`です
  - 関数型にかぶれた言語なので
    - 関数型では変数への再代入は基本できません
- なるべく再代入を行わないようにすることが奨励されている気がします
  - Objctive-Cと連携する部分ではグダグダですが

---

## 変数宣言のバリエーション
```Swift
let hoge: String = "hoge" // 初期値を与えるパターン
let hoge = "hoge" // 初期値を与えて型名を省略するパターン
var hoge: String { String(12) } // クロージャ(後述)の結果を返すパターン
```
型推論が効くので初期値を与える場合は型名を省略して構いません、
クロージャの結果を渡す場合は書いた方がいいでしょう。
初期値を与えない場合は型名を省略することはできません。

---

# 4.関数の定義
基本型
```swift
func hoge(hogeName name: String) -> String {
    return "hello" + name
}
```
- `func`: 関数定義のキーワード
- `hoge`: 関数名
- `hogeName`: **外部引数名**
- `name`: **内部引数名**
- `String`: 引数の型
- `-> String`: 戻り値の型

---

## 関数定義のバリエーション
```swift
func hoge(name: String) -> String {} // 引数名を一つにすると外部、内部引数名が同じになります
func hoge(_ name: String) -> String {} // 外部引数名に"_"を使うことで呼び出し時に外部引数名を省略できます
func hoge() {} // 戻り値がない場合は"-> Void"を省略できます
```
外部引数名を`_`で省略すると`hoge("麻生太郎")`とC#みたいな呼び出し方ができますが
  
  
***外部引数名を無闇に省略したものは地獄に落ちます***
  
---

## 外部引数名、内部引数名について
関数を呼び出す側と関数の内部では同じ値であっても意味合いが異なります、
例えば引数でファイルの保存先を決定する関数があったとします
```swift
func saveFile(to directory: URL) {
    FileManager.default.createFile(atPath: directory ~~~
}
```
内部ではファイルの保存先を`directory`で表現しています。
呼び出す側は`saveFile(to: savepath)`と呼び出すことができます。
これがファイルを保存する関数で、保存先を引数に求めていることがわかりやすいかと思います。
特にオーバーロードをするときに外部引数名で用途を切り替えられるので便利です。

---

# 5.オプショナルについて

JavaやC#(オプションの機能を有効にしない場合)は*Null不安全*な言語です
あらゆるオブジェクトがnullでないことは静的に保証されません。
  
Swiftでは基本的な型(String,Int等)にnil(Swiftではnullじゃない)を代入することはできません
nilが入る可能性のある値を表現する物がOptional型です。

---
## 変数として宣言
```swift
let hoge: String?
let hoge: String!
```
型名の後ろに`?`をつければふつうのOptionalです
このままではStringとして扱うことはできないので後述するOptionalを*unwrap(重要)*する方法を使って値を取り出します。

型名の後ろに`!`をつけると暗黙的に*force unwrap(後述)*されるOptionalです
nilを代入でき、特別なことをしなくてもStringとして扱えますが,
値がnilのときに値を評価すると強制終了します。

---
## guard let
```swift
func greet(name: String?) {
    guard let name = name else {
        print("名前を入力してください")
        return 
        }
    print("hello \(name)")
}
```
guard節のようにunwrapします。
もし値があるなら下の処理に移り、nilなら`else`節の中を実行します。
`else`節では必ず`return`か`throw`する必要があります。

---
## if let
```swift
func greet(name: String?) {
    if let name = name {
        print("hello \(name)")
    } else { 
        print("名前を入力してください")
    }
}
```
if文のようにunwrapします。
値があるなら直後のブロックを処理し、なければ`else`節を実行します。
elseは省略可能です。
ifの直後のブロックを抜けるとunwrapした結果は使えません。

---

## どっち使えばいいの
どっちも使います
`guard let`ではunwrapした結果後ろで使えて、
nilだった場合は即座にスコープを抜けるので、
本筋の処理を行う前に抜けてしまいたい時に使います。

`if let`では直後にしか値を使えませんが、
nilだった場合もスコープを抜ける必要はないので、
nilだった場合の処理もある場合はこちらを使うことが多いです。

---

## 他にもある
```swift
struct Hoge {
    let name = "hoge"
    let age = 12
}

let hoge: Hoge?
```
- `Optional Chaining`: ↑のような時に`hoge`の`name`の値を取得したい場合
  - `let name = hoge?.name` このように書くことができます。
  - この場合の`name`の型は`String?`です

---
## まだある

- `Optional Binding`: Optionalの値を評価するとき、nilだった場合には別の値を使いたい場面があります
- 例えばユーザー名が未入力の場合
  - `let name = user.name ?? "名前を入力してください"`
  - `Optional ?? nilだった場合の値`と書くことで値がある場合とない場合で使い分けることができます

---

## もっとある
```swift
let name: String?

name = nil
greet(name: name!)
```
- `force unwrap`: Optionalに`!`をつけることで強制的にunwrapします
  - 上の例だと`name!`は単なる`String`と同じように扱われます。

---

## どれ使えばいいの
どれも使いますが`Optional Binding`と`force unwrap`を軽率に使うものは
***地獄に落ちます***

特に`force unwrap`はプログラムの安全性を損なう危険があるので注意しましょう。
