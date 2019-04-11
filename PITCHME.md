---
# Rust Book 勉強会 \#8

@size[smaller](Rust のオブジェクト指向的な機能)

---
## おことわり
<hr/>

- Javaを引き合いに説明する場合が多々あります
- ...

---
## もくじ
<hr/>

1. オブジェクト指向とは
2. トレイトオブジェクト
3. オブジェクト指向的実装例

---

1. @css[text-white](オブジェクト指向とは)
2. @css[text-gray](トレイトオブジェクト)
3. @css[text-gray](オブジェクト指向的実装例)


---
## オブジェクト指向とは

次のような機能を持つプログラミング言語を一般的にオブジェクト指向言語と呼ぶ（ただし諸説あり）

@ul
- オブジェクト
  - データとその振る舞いを一つにまとめて扱うことができる
- カプセル化
  - データやメソッドを隠蔽して抽象度を上げることができる
- 継承
  - オブジェクトからデータと振る舞いを引き継いだオブジェクトを作成できる
@ulend

+++
### Rust はオブジェクトの機能を持っているか？

`struct` や `enum` がある。

impl ブロックで、構造体や列挙体に振るまいを追加している、とみなすことができる。

@snap[south span-100 fragment]
@box[text-gray](後述のトレイトオブジェクトと紛らわしいので、「オブジェクト」という用語はあまり使われない？)
@snapend

+++
### Rust はカプセル化の機能を持っているか？

`pub` がある。

アイテム（モジュール、構造体、関数 など）に対して、公開・非公開を選択することで、APIとして必要な機能を外部に公開することができる。

@snap[south span-100 fragment]
@box[text-gray](`pub(crate)`とか`pub(self)`についてはわからん)
@snapend

+++
### Rust は継承の機能をもっているか？

ない

---

- @css[text-gray](オブジェクト指向とは)
- @css[text-white](トレイトオブジェクト)
- @css[text-gray](オブジェクト指向的実装例)

---
## トレイトオブジェクト

#### 動機

われわれが継承で欲しかったのは、コードの共有ではなく、ポリモーフィズムだったのだ

+++

@quote[ポリモーフィズム（英: Polymorphism）とは、プログラミング言語の型システムの性質を表すもので、プログラミング言語の各要素（定数、変数、式、オブジェクト、関数、メソッドなど）についてそれらが複数の型に属することを許すという性質を指す。](Wikipediaより)

---
### 例

第8章で、複数のタイプ（整数、浮動小数点数、文字列）を含む`Vec`を考えた。

複数のタイプが事前にいくつあるかわからない場合がある。
ここでは例として、`Draw`トレイトを実装する任意の型を格納できるような`Vec`がほしいとする。

---
### 利用側のコード

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}
impl Draw for Button {
    fn draw(&self) {
        // ...
    }
}
pub struct SelectBox {
    // ...
}
impl Draw for SelectBox {
    // ...
}
fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox { ... })
            Box::new(Button { ... })
        ]
    };
    screen.run();
}
```

@[1-16](Drawトレイトと独立してButton, SelectBox を作成する)
@[17-27](components に Box&lt;dyn Draw&gt;を格納)
