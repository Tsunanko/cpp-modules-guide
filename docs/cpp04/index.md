# cpp04 モジュール概要

---

## このモジュールは何？

cpp04 は **「ポリモーフィズム（多態性）を学ぶモジュール」** です。

ひとことで言うと:

> **同じ関数を呼ぶだけで、相手に合わせて違う動きをしてくれる仕組み**

例えば「鳴いて」と命令すると、犬なら「ワン」猫なら「ニャー」と鳴く。
呼び出す側は **「相手が犬か猫か」を気にせず** 「鳴いて」と言うだけ。

cpp03 の継承を一歩進めて、**基底クラスのポインタから派生クラスの関数を呼ぶ** 技術を学びます。
最終課題では 7 つのクラスを連携させて、メモリの所有権管理まで行います。

---

## このモジュールで学ぶこと

- **ポリモーフィズム**: 同じ呼び出しで違う動きをする仕組み
- **`virtual`**: 「この関数は子クラスで上書きされるかも」マーク
- **動的ディスパッチ**: 実行時に「本当の型」の関数を呼ぶ仕組み
- **vtable**: `virtual` の裏側で動く表（関数ポインタの配列）
- **抽象クラス**: 実体を作れない「設計図だけ」のクラス
- **インターフェース**: 中身が全部「派生クラスで実装してね」のクラス
- **ディープコピー**: ポインタの先までちゃんとコピーする方法
- **所有権（ownership）**: 「誰が new して誰が delete するか」の管理

---

## 新しい概念の解説

### ポリモーフィズムって何？

**「同じ関数呼び出しで、オブジェクトの型によって違う動作をすること」** です。

```
命令する側:  animal->makeSound();   ← 同じ呼び出し
              |
              +--- 中身が Dog なら → "Woof!"
              |
              +--- 中身が Cat なら → "Meow!"
```

呼ぶ側は `Animal*` を使うだけ。中身が Dog か Cat かは気にしない。
「動物に鳴いて」と命令すれば、相手が犬なら犬の鳴き方で、猫なら猫の鳴き方で鳴く。

!!! tip "身近な例"
    リモコンの「電源ボタン」みたいなもの。
    テレビに向ければテレビが点き、エアコンに向ければエアコンが点く。
    **ボタンは同じ、動作は相手によって変わる**。これがポリモーフィズム。

### `virtual` って何？

**「この関数は子クラスで上書きできる」というマーク** です。

```cpp
class Animal {
public:
    // virtual を付けると「子クラスで上書きOK」
    virtual void makeSound() const;
};

class Dog : public Animal {
public:
    // Animal::makeSound を上書き
    void makeSound() const;
};
```

`virtual` を付けると **動的ディスパッチ** が有効になります。
`virtual` がないと **静的バインディング** になります（後述）。

### 動的ディスパッチって何？

**「実行時に相手の本当の型を見て、その型の関数を呼ぶ」** 仕組みです。

```cpp
Animal* a = new Dog();
a->makeSound();
// a の型は Animal* だが、中身は Dog
// 動的ディスパッチ: 「本当の型 = Dog」と判断
//                    → Dog::makeSound() を呼ぶ
// 結果: "Woof!"
```

これに対して **静的バインディング** は「ポインタの型だけを見て決める」方式。

```
動的ディスパッチ (virtual あり):
  ポインタの型  → Animal*
  中身の型      → Dog
  呼ばれる関数  → Dog::makeSound() ← 中身を見る!

静的バインディング (virtual なし):
  ポインタの型  → Animal*
  中身の型      → Dog
  呼ばれる関数  → Animal::makeSound() ← 型しか見ない
```

### vtable って何？

**`virtual` の裏側で動いている「関数ポインタの表」** です。
日本語では「仮想関数テーブル」と呼びます。

```
Dog オブジェクト:
  ┌────────────────┐
  │ vptr ──────────┼──→ Dog の vtable
  │ _type = "Dog"  │       ┌─────────────────────┐
  └────────────────┘       │ makeSound → Dog版    │
                           │ ~Dog      → Dog版    │
                           └─────────────────────┘
```

- 各オブジェクトは **vptr（vtable へのポインタ）** を持つ
- `virtual` 関数を呼ぶと、vtable をたどって実際の関数を見つける
- これがあるから「中身の型に応じた関数」を呼べる

!!! info "普段は意識しなくてOK"
    vtable はコンパイラが自動で作ってくれます。
    私たちは `virtual` を書くだけで、裏の仕組みは意識しなくていい。
    でも「動的ディスパッチがなぜ動くのか？」の答えが vtable です。

### 抽象クラスって何？

**「実体を作れない、設計図だけのクラス」** です。

```cpp
class Animal {
public:
    // 「= 0」が付くと「純粋仮想関数」
    // 実装を持たず、派生クラスで必ず実装が必要
    virtual void makeSound() const = 0;
};

// Animal a;           ← コンパイルエラー！
// new Animal();       ← コンパイルエラー！
// Animal* a = new Dog();  ← OK（Dog は実装済み）
```

- **純粋仮想関数 (`= 0`)** を1つでも持つクラスが抽象クラス
- 抽象クラスは **直接インスタンス化できない**
- でも **ポインタや参照としては使える**（上の `Animal*` の例）
- 派生クラスが全ての純粋仮想関数を実装すると、その派生は具象クラス（インスタンス化可能）

### インターフェースって何？

**「全メソッドが純粋仮想関数な抽象クラス」** です。
データメンバ（変数）を持たず、**「何ができるか」だけを定義する契約** です。

```cpp
// 名前の先頭に "I" を付ける慣習（I = Interface）
class ICharacter {
public:
    virtual ~ICharacter() {}  // デストラクタだけ空実装
    virtual std::string const& getName() const = 0;
    virtual void equip(AMateria* m) = 0;
    virtual void unequip(int idx) = 0;
    virtual void use(int idx, ICharacter& target) = 0;
    // データメンバなし！ 全部 = 0 ！
};
```

- **名前の先頭に `I`** を付ける（`ICharacter`, `IMateriaSource`）
- **全メソッドが `= 0`**
- **データメンバなし**
- **virtual デストラクタは必須**（実装は `{}` でOK）

### 抽象クラス vs インターフェースの違い

| | 抽象クラス (A-prefix) | インターフェース (I-prefix) |
|---|---|---|
| 純粋仮想関数 | 1つ以上 | 全メソッド |
| 通常の実装メソッド | 持てる | なし |
| データメンバ | 持てる | なし |
| コンストラクタ | 必要 | 不要 |
| OCF | 必要 | virtual デストラクタだけ |
| 例 | `AMateria` | `ICharacter`, `IMateriaSource` |

---

## 4 つの exercise

| # | 名前 | 難度 | 所要時間 | 主要トピック |
|---|---|:---:|:---:|---|
| [ex00](ex00-polymorphism.md) | Polymorphism | ★★ | 1〜2時間 | virtual 関数、動的ディスパッチ |
| [ex01](ex01-brain.md) | Brain | ★★★ | 2〜3時間 | ディープコピー、new/delete |
| [ex02](ex02-abstract.md) | Abstract class | ★★ | 30分 | 純粋仮想関数、抽象クラス |
| [ex03](ex03-materia.md) | Interface & recap | ★★★★★ | 5〜8時間 | インターフェース、所有権管理 |

!!! warning "ex03 は cpp00〜04 で最も難しい"
    7 つのクラスを正しいメモリ管理と共に設計する必要があります。
    十分な時間を確保してください。

---

## 学ぶ概念マップ

```
                          cpp04
                            |
         +------------------+------------------+
         |                  |                  |
        ex00               ex01              ex02
    Polymorphism        Deep Copy          Abstract
         |                  |                  |
     +---+---+         +----+----+         Animal = 0
     |       |         |         |            |
   Animal  WrongAnimal Brain   Dog/Cat    new Animal()
  (virtual)(非virtual) (new)   (Brain*)   = エラー
     |       |         |         |
    Dog/Cat WrongCat  ideas[100] ディープ
  (override)(static)             コピー
     |       |
    動的    静的
 ディスパッチ バインディング

                          ex03
                        Interface
                            |
              +-------------+-------------+
              |             |             |
         ICharacter   IMateriaSource   AMateria
         (interface)   (interface)    (abstract)
              |             |             |
          Character    MateriaSource   Ice / Cure
          (equip/      (learn/        (clone)
           unequip)     create)
```

---

## 共通ルール

- コンパイル: `c++ -Wall -Wextra -Werror -std=c++98`
- `using namespace std;` は **禁止**
- `printf` / `malloc` / `free` は **使用禁止**
- ヘッダに関数実装を書かない（テンプレートを除く）
- `friend` キーワードは禁止

---

## 重要: OCF チェックの特別ルール

!!! danger "OCF がないクラスがあれば全 exercise 0 点"
    **インターフェース以外の全てのクラスが OCF（Orthodox Canonical Form）を
    満たしていなければ、採点されません。**

    OCF = 次の4つが揃っていること:

    1. デフォルトコンストラクタ
    2. コピーコンストラクタ
    3. 代入演算子オーバーロード
    4. デストラクタ

    **インターフェース（`ICharacter`, `IMateriaSource`）は OCF 不要**。
    状態を持たないため、virtual デストラクタだけあればOK。

    それ以外（`Animal`, `Dog`, `Cat`, `Brain`, `AMateria`, `Ice`, `Cure`,
    `Character`, `MateriaSource`）は **全て OCF 必須**。

---

## 評価（defense）の大まかな流れ

1. **OCF チェック** — インターフェース以外の全クラスに4関数があるか
2. `make` がクリーンビルドできるか
3. `virtual` キーワードの使い方が正しいか
4. ディープコピーが正しく動作するか（ex01）
5. メモリリークがないか（特に ex03）
6. 口頭質問: vtable の仕組み、abstract vs interface の違い、所有権の説明

---

## まず始める

[ex00 Polymorphism](ex00-polymorphism.md) から順番に進めましょう。
