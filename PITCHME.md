---
# Rust Book 勉強会 \#8

@size[smaller](Rust のオブジェクト指向的な機能)

---
## おことわり
<hr/>

- オブジェクト指向については知っているものとします
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

次のような機能を持つプログラミング言語を一般的にオブジェクト指向言語と呼ぶ（諸説あります）

- オブジェクト
  - データと振る舞いを一つにまとめて扱える|
- カプセル化
  - データを隠蔽することで抽象度を上げる|
- 継承
  - オブジェクトから振る舞いを引き継げる|

+++
### Rust はオブジェクトの機能を持っているか？

`struct` や `enum` がある。

impl ブロックで、構造体や列挙体に振るまいを追加している、とみなすことができる。

@snap[south span-100 fragment]
@box[text-gray](後述のトレイトオブジェクトと紛らわしいので「オブジェクト」という用語はあまり使われない？)
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

@snap[south fragment]
@size[1.8em](ありません)
@snapend

---

1. @css[text-gray](オブジェクト指向とは)
2. @css[text-white](トレイトオブジェクト)
3. @css[text-gray](オブジェクト指向的実装例)

---
## トレイトオブジェクト

#### 動機

われわれが継承で欲しかったのは、コードの共有ではなく、ポリモーフィズムだ。

@ul
- 継承によるコードの共有は密結合を招いて、歓迎されない
- インターフェースを切り出して、疎結合を実現したい
@ulend

---
### トレイトオブジェクトとは

`Draw`トレイトに対して、`&dyn Draw` または `Box<dyn Draw>`をトレイトオブジェクトと呼ぶ。

これらはポインタで、そのアドレスの指す先は`Draw`トレイトを実装する任意の型(`struct`, `enum`)となる。

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
            Button { ... },
            SelectBox { ...},
        ]
    };
    screen.run();
}
```
![Compile Error](assets/images/compile_error01.png)

---
### オブジェクト安全

トレイトオブジェクトは任意のトレイトに対して作成できるわけではない。

トレイトが定義する関数が次の条件を満たす必要がある（十分ではないが、実用上は十分）

- 戻り値が Self でない|
- ジェネリックパラメータを持たない|

+++
#### 戻り値が`Self`でない

トレイトオブジェクトは「具体的な型を気にする必要がない」という機能であった。

一方で、`Self`が戻り値として含まれる関数は、具体的な型が明らかになる必要があるため、
トレイトオブジェクトとして適していない。

+++
#### ジェネリックパラメータを持たない

同じく、ジェネリックパラメータをもつ関数も型パラメータを与えることができず、
トレイトオブジェクトとして適していない。

ただし、トレイトの関連型(Associated Type)は、トレイトオブジェクトを
`&dyn Draw<Item=i32>`のように指定することで、関連型ごとにトレイトオブジェクトを作成できる。

+++
#### オブジェクト安全でない例

`Clone`トレイトを考えると、これは`Self`を戻り値としており、オブジェクト安全ではない

```rust
pub trait Clone {
    fn clone(&self) -> Self
}
```

@size[smaller](Java との対比)

```java
class Object {
    // ...
protected Object clone() { /* ... */ }
}
```
@[6](誰しも、cloneの戻り値の型がObjectで、その使い方に注意が必要で苦労したはず)

---

1. @css[text-gray](オブジェクト指向とは)
2. @css[text-gray](トレイトオブジェクト)
3. @css[text-white](オブジェクト指向的実装例)

---
## オブジェクト指向実装例

#### ステートパターン

![State Pattern](assets/images/state_pattern.png)

- コンテキストは内部状態（ステート）を持つ
- 状態はステートクラスのサブタイプである
- 状態に応じた振る舞いをサブタイプが実装する

---

### 写経してハマった所(ザ・素人)

1. `Post`構造体が、トレイトオブジェクトを`Option`型で包んでいる
2. `approve`メソッド等の引数が`self`では動かない
3. `State`トレイトが`approve`メソッド等に対してデフォルトの実装を持てない

---

#### 素人（１）

`Post`の`state`が`Option`に包まれている

```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}
```
@[2](直接Box&lt;dyn State&gt;ではだめなのか？)

+++
#### Option::take のちから

次のように定義したとすると

```rust
pub struct Post {
    state: Box<dyn State>,
    content: String,
}
```

`approve`などは次のような実装をしてしまう（私はした）

```rust
pub fn approve(&mut self) {
    self.state = self.state.approve();
}
```

+++
#### Option::take のちから

![Compile Error](assets/images/compile_error02.png)

`Post`が所有している`state`を一瞬足りとも`move`することはできない！

その分、`content`の実装が面倒くさくなっている

+++
#### Option::take のちから

```rust
pub fn approve(&mut self) {
    if let Some(state) = self.state.take() {
        self.state = Some(state.approve());
    }
}
```
@[2](Postが保持しているstateを取り出して、代わりにNoneで埋める)
@[3](取り出したstateから次の状態を求めて、穴埋めする)

+++
#### Option::take のちから

その代わり`content`の実装が回りくどくなっている

```rust
pub fn content(&self) -> &str {
    self.state.as_ref().unwrap().content(&self)
}
```
@[2](借用しているOption<Box<dyn State>>からBox<dyn State>を借用しなおしてから、State の contentを呼ぶ)

---
#### 素人（２）

`approve`メソッド等の引数が`self: Box<Self>`になっている理由がわからない

```rust
pub trait State {
    fn approve(self: Box<Self>) -> Box<dyn State>;
    //...
}
```

+++
#### `self`引数に渡されるのは Box<dyn State>ではない

試しに、`fn approve(self) -> Box<dyn State>`と宣言を変えてみる。

![Compile Error](assets/images/compile_error03.png)

ステータスを変化させない実装で問題が起こっている

+++
#### approve に Box<dyn State> をそのまま move したい

`Post`側の`approve`から`State`の`approve`は次のように呼ばれている

```rust
pub fn approve(&mut self) {
    if let Some(state) = self.state.take() {
        self.state = Some(state.approve());
    }
}
```
@[2](ここで得られる state は`Option`を剥がされた `Box<dyn State>`)
@[3](この approve の引数が self だと、自動的にBoxが指す先がselfに渡る)

---
#### 素人（３）

`approve`メソッドはだいたい`self`を返すので、デフォルトの実装としたい

```rust
pub trait State {
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
    //...
}
```

+++
#### コンパイルエラー

![Compile Error](assets/images/compile_error04.png)

@snap[south fragment]
@size[1.8em](わかりません)
@snapend


