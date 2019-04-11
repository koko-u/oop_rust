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

1. @css[text-gray](オブジェクト指向とは)
2. @css[text-white](トレイトオブジェクト)
3. @css[text-gray](オブジェクト指向的実装例)

---
## トレイトオブジェクト

#### 動機

われわれが継承で欲しかったのは、コードの共有ではなく、ポリモーフィズムだ。

- 継承によるコードの共有は密結合を招いて、歓迎されない|
- インターフェースを切り出して、疎結合を実現したい|

+++

@quote[ポリモーフィズム（英: Polymorphism）とは、プログラミング言語の型システムの性質を表すもので、プログラミング言語の各要素（定数、変数、式、オブジェクト、関数、メソッドなど）についてそれらが複数の型に属することを許すという性質を指す。](Wikipediaより)

---
### トレイトオブジェクトとは

`Draw`トレイトに対して、`&dyn Draw` または `Box<dyn Draw>`をトレイトオブジェクトを呼ぶ。

これらはポインタで、そのアドレスの指す先は`Draw`トレイトを実装する任意の型(`struct`, `enum`)となる。

ポインタの指す先が任意の型でありうるので、`str`と同様に`dyn Draw`は直接型として指定することはできない。

---
### 例（Draw提供側）

```rust
pub trait Draw {
    fn draw(&self);
}
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>
}
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```
@[1-3](普通のトレイトの定義)
@[4-6](`Box<dyn Drow>`を要素にもつベクトルをもつ構造体)
@[7-12](`Draw`を実装している型が何であっても動作する)

---
### C++ とほとんど同じ

- `Draw` は抽象メソッド`draw`を持つクラス 
- 任意の `Draw` のサブタイプを格納する `vector` を作るために `vector<unique_ptr<Draw>>`を用意する

---
### 例（Draw利用側）

```rust
pub struct Button {
    // ...
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

@[1-14](利用側が任意に`Draw`を実装する)
@[15-23](実際の型からトレイトオブジェクトを作成する)

---
### ジェネリックスとの違い（１）

先の`Screen`をジェネリックスを用いて定義したとする

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>
}
impl<T> Screen<T> where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

+++
### ジェネリックスとの違い（２）

クライアントが困る

```rust
fn main() {
    let screen = Screen {
        components: vec![
            SelectBox { ... }
            Button { ... }
        ]
    };
    screen.run();
}
```
![Compile Error](assests/images/compile_error01.png)

---
## オブジェクト安全

トレイトオブジェクトは任意のトレイトに対して作成できるわけではない。

トレイトが定義する関数が次の条件を満たす必要がある（十分ではないが、実用上は十分）

- 戻り値が`Self`でない|
- ジェネリックパラメータを持たない|

+++
### 戻り値が`Self`でない

`Self`とは、トレイトを実装している具体型を表している。

トレイトオブジェクトは具体的な型がなにかによらずにそのポインタを格納する。

同時に、トレイトが定義する関数の具体的な実装へのポインタも格納する。

このとき、`Self`が戻り値として含まれる関数は、トレイトを実装する具体的な型が決まるまでシグネチャが決まらない。

+++
### ジェネリックパラメータを持たない

同じく、ジェネリックパラメータも「トレイトだけある」状態では、その型パラメータが決まらず、トレイトオブジェクトを作ることができない。

トレイトの Associated Type （訳わからん）については、トレイトオブジェクトを作成するときに `&dyn Draw<Item=i32>`のような記法をとる。

+++
### オブジェクト安全でない例

`Clone`トレイトを考えると、これは`Self`を戻り値としており、オブジェクト安全ではない

```rust
pub trait Clone {
    fn clone(&self) -> Self
}
```

Java との対比

```java
class Object {
    // ...
protected Object clone() { /* ... */ }
}
```
@[3](誰しもが、cloneの戻り値の型がObjectで、その実装や使い方に注意が必要で苦労したはず)