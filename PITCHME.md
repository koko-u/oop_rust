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

@ol
- オブジェクト指向とは
- トレイトオブジェクト
- オブジェクト指向的実装例
@olend

---

- @css[text-white](オブジェクト指向とは)
- @css[text-gray](トレイトオブジェクト)
- @css[text-gray](オブジェクト指向的実装例)


---
## オブジェクト指向とは

次のような機能を持つプログラミング言語を一般的にオブジェクト指向言語と呼ぶ（ただし諸説あり）

@ul
- オブジェクト
- カプセル化
- 継承
@ulend

+++
### オブジェクト

Rust では `struct` または `enum` がオブジェクトに相当する。

オブジェクト指向的には、データとその振る舞いを一つにまとめて「オブジェクト」と呼ぶ

@snap[south span-100]
@box[text-gray](後述のトレイトオブジェクトと紛らわしいので、「オブジェクト」という用語はあまり使われない？)
@snapend

+++
### カプセル化

ライブラリを利用する時に「余計なこと」を考えなくても済むようにするための機能

Rust では公開（`pub`）かそうでないかをアイテムに指定することで、この機能を実現している

@snap[south span-100]
@box[text-gray](`pub(crate)`とか`pub(self)`についてはわからん)
@snapend

+++
### 継承

Rust にはない。
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
