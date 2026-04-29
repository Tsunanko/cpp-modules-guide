# ex00 — Polymorphism

---

## このプログラムは何？

**犬と猫に「鳴いて」と命令するプログラム** です。

ポイントは、命令する側が **「相手が犬か猫か」を気にせず**
`Animal*` ポインタ経由で `makeSound()` を呼ぶこと。
すると相手が犬なら「ワン！」、猫なら「ニャー！」と鳴きます。

```
Animal* a = new Dog();
a->makeSound();   → "Woof! Woof!"  (犬の鳴き方)

Animal* b = new Cat();
b->makeSound();   → "Meow! Meow!"  (猫の鳴き方)
```

これが **ポリモーフィズム（多態性）** です。

さらに、**`virtual` を付けない場合** にどうなるかを `WrongAnimal` / `WrongCat` で確認します。

---

## 1. このexerciseで学ぶこと

- **`virtual` 関数**: 子クラスで上書き可能にするマーク
- **動的ディスパッチ**: 実行時に「本当の型」の関数を呼ぶ
- **静的バインディング**: コンパイル時にポインタの型で決まる（virtual なし）
- **`virtual` デストラクタ**: 基底ポインタ経由 delete で派生も破棄される
- **コンストラクタ/デストラクタの呼び出し順序**

---

## 2. 新しい概念の解説

### `virtual` 関数って何？

**「子クラスで上書きできる関数」というマーク** です。

```cpp
class Animal {
public:
    // virtual を付ける
    virtual void makeSound() const;
};

class Dog : public Animal {
public:
    // Animal::makeSound を上書き
    void makeSound() const;
};
```

`virtual` を付けるだけで、**動的ディスパッチ** が有効になります。

### virtual なしだとどうなる？（静的バインディング）

**「ポインタの型だけを見て関数を決める」** 方式になります。

```cpp
class Animal {
    // virtual なし
    void makeSound() const;
};

class Dog : public Animal {
    void makeSound() const;
};

Animal* a = new Dog();
a->makeSound();
// ポインタの型は Animal*
//   → Animal::makeSound() が呼ばれる
//   → 中身が Dog でも関係ない!
```

これは意図した動きじゃないことが多い。
だから C++ では「子クラスで上書きする関数」には **必ず virtual を付ける** のが基本です。

### virtual ありだとどうなる？（動的ディスパッチ）

**「実行時に中身の型を見て、その型の関数を呼ぶ」** 方式になります。

```cpp
class Animal {
    virtual void makeSound() const;  // virtual あり
};

class Dog : public Animal {
    virtual void makeSound() const;  // 上書き
};

Animal* a = new Dog();
a->makeSound();
// ポインタの型は Animal* でも
// 中身が Dog なので → Dog::makeSound() が呼ばれる!
```

### 動的ディスパッチの裏側: vtable

`virtual` を使うと、コンパイラは自動的に **vtable（仮想関数テーブル）** を作ります。

```
Dog オブジェクトのメモリ:
  ┌───────────────┐
  │ vptr ─────────┼──→ Dog の vtable
  │ _type="Dog"   │       ┌───────────────┐
  └───────────────┘       │ makeSound →   │
                          │   Dog::makeSound
                          │ ~Dog      →   │
                          │   Dog::~Dog   │
                          └───────────────┘

a->makeSound() の時:
  1. a のオブジェクトを見る
  2. vptr をたどって vtable を取得
  3. vtable の「makeSound」スロットの関数を呼ぶ
  4. → Dog::makeSound が呼ばれる
```

!!! info "普段は意識しなくていい"
    これはあくまで裏の仕組み。私たちは `virtual` を書くだけでOK。
    ただし「なぜ動的ディスパッチが動くの？」の答えは vtable です。

### `virtual` デストラクタって何？

**`virtual ~Animal()` のように、デストラクタに virtual を付ける** ことです。

```cpp
class Animal {
public:
    virtual ~Animal();   // virtual あり
};

Animal* a = new Dog();
delete a;
// virtual なので →  Dog::~Dog() が呼ばれる
//               →   Animal::~Animal() が呼ばれる
// 両方呼ばれるので Dog 固有のリソースも解放される
```

### なぜ virtual デストラクタが必須なのか？

**virtual デストラクタがないと、派生クラスのリソースがリークする** からです。

```cpp
class Animal {
    ~Animal();   // virtual なし！
};

class Dog : public Animal {
    Brain* _brain;   // 派生がヒープメンバを持つ
    ~Dog() { delete _brain; }
};

Animal* a = new Dog();
delete a;
// virtual がない → Animal::~Animal() だけ呼ばれる
//                   Dog::~Dog() が呼ばれない
//                   → _brain がリーク!
//                   → 未定義動作
```

!!! danger "鉄則: 基底クラスのデストラクタは必ず virtual"
    継承される可能性のあるクラスは、**必ず virtual デストラクタを宣言** しましょう。
    これを忘れるとメモリリークや未定義動作が起きます。

---

## 3. 課題仕様

| クラス | メンバ / 関数 | 説明 |
|---|---|---|
| **Animal** | `protected: _type` | 型名（文字列） |
| | `virtual makeSound()` | 基底の鳴き声 |
| | `virtual ~Animal()` | virtual デストラクタ |
| | `getType()` | `_type` を返す |
| **Dog** | `Animal` を継承 | `_type = "Dog"` |
| | `virtual makeSound()` | `"Woof! Woof!"` |
| **Cat** | `Animal` を継承 | `_type = "Cat"` |
| | `virtual makeSound()` | `"Meow! Meow!"` |
| **WrongAnimal** | Animal と同構造 | ただし `makeSound` は **virtual なし** |
| **WrongCat** | `WrongAnimal` を継承 | `WrongAnimal*` 経由では上書きされない |

全クラスに **OCF（4関数）** が必須。

---

## 4. 実行例

```console
$ make && ./animal
=== Polymorphism Test ===
Animal default constructor called
Animal default constructor called
Dog default constructor called
Animal default constructor called
Cat default constructor called

Type of j: Dog
Type of i: Cat
Meow! Meow!            ← Animal* i だが Cat の鳴き方 (virtual)
Woof! Woof!            ← Animal* j だが Dog の鳴き方 (virtual)
... (generic animal sound)

Animal destructor called
Dog destructor called
Animal destructor called
Cat destructor called
Animal destructor called

=== Wrong Polymorphism Test ===
Type of wrongCat: WrongCat
... (wrong generic animal sound)   ← WrongAnimal* なので基底が呼ばれる
... (wrong generic animal sound)
Direct WrongCat call: Meow! Meow! (WrongCat)   ← 型を WrongCat* にすればOK
```

**注目**:

- `Animal*` 経由 → **Dog/Cat の鳴き方**（virtual のおかげ）
- `WrongAnimal*` 経由 → **WrongAnimal の鳴き方**（virtual なしのため）
- `WrongCat*` 型で直接呼ぶ → WrongCat の鳴き方（ポインタの型で決まる）

---

## 5. C と C++ の比較

C で同じことをやろうとすると **関数ポインタテーブルを手動で作る** 必要があります。
実は `virtual` はこの「手動 vtable」を **言語機能にしたもの** です。

!!! info "virtual 関数は「関数ポインタを構造体に入れる」の自動化"
    C で多態性を作るには:

    1. 構造体に関数ポインタを仕込む
    2. 派生の型ごとに関数ポインタを差し替える
    3. 呼び出しは `obj->func(obj)` のように書く

    C++ の `virtual` はこの 3 ステップを **全てコンパイラが自動で** やってくれる機能です。

=== "C の書き方（手動 vtable）"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>
    /* strcpy 用のヘッダ */
    #include <string.h>

    /* ── 構造体に関数ポインタを持たせる ── */
    /* これが手動の vtable */
    typedef struct s_animal {
        /* 型名を格納 */
        char type[32];
        /* 関数ポインタ (手動 vtable) */
        /* 派生ごとに差し替える */
        void (*makeSound)(void);
    } t_animal;

    /* Dog 用の鳴き声関数 */
    /* 後で関数ポインタに代入する */
    void dog_sound(void) {
        printf("Woof! Woof!\n");
    }

    /* Cat 用の鳴き声関数 */
    void cat_sound(void) {
        printf("Meow! Meow!\n");
    }

    int main(void) {
        /* Dog を作る */
        t_animal dog;
        /* 型名をコピー (Cは手動) */
        strcpy(dog.type, "Dog");
        /* 関数ポインタを手動で Dog 用に設定 */
        /* ← C++ では virtual が自動でやる */
        dog.makeSound = dog_sound;

        /* Cat も同じように手動で組む */
        t_animal cat;
        strcpy(cat.type, "Cat");
        cat.makeSound = cat_sound;

        /* 関数ポインタ経由で呼び出し */
        /* 呼び出し側は「何型か」を気にしない */
        dog.makeSound();   /* "Woof! Woof!" */
        cat.makeSound();   /* "Meow! Meow!" */
        return 0;
    }
    ```

=== "C++ の書き方（virtual で自動）"

    ```cpp
    // cout 用のヘッダ
    #include <iostream>
    // std::string 用のヘッダ
    #include <string>

    // ── 基底クラス ──
    class Animal {
    protected:
        // 型名 (派生から書き換えられる)
        std::string _type;
    public:
        // : _type("Animal") は初期化子リスト
        Animal() : _type("Animal") {}
        // virtual デストラクタ:
        // 基底* delete で派生の
        // デストラクタも呼ぶため必須
        virtual ~Animal() {}
        // virtual = 「派生で上書きOK」マーク
        // Cでは構造体に関数ポインタを
        // 入れる必要があったが、
        // virtual なら自動で vtable が作られる
        virtual void makeSound() const {
            std::cout
                << "... generic animal sound"
                << std::endl;
        }
    };

    // ── 派生: Dog ──
    // : public Animal = 「Animal を継承」
    class Dog : public Animal {
    public:
        // コンストラクタ本体で _type を上書き
        // (_type は protected なので派生OK)
        Dog() { _type = "Dog"; }
        // 基底の makeSound を上書き
        // C の「dog.makeSound=dog_sound」に相当
        // ただしこちらは自動で差し替わる
        virtual void makeSound() const {
            std::cout << "Woof! Woof!"
                      << std::endl;
        }
    };

    // ── 派生: Cat ──
    class Cat : public Animal {
    public:
        Cat() { _type = "Cat"; }
        // 上書き
        virtual void makeSound() const {
            std::cout << "Meow! Meow!"
                      << std::endl;
        }
    };

    int main(void) {
        // Animal* 型に Dog/Cat を入れる
        // 呼び出し側は「何型か」を気にしない
        Animal* a = new Dog();
        Animal* b = new Cat();
        // virtual で動的ディスパッチ:
        // → 実行時に vtable を見て
        //   本当の型の関数が呼ばれる
        a->makeSound();   // "Woof! Woof!"
        b->makeSound();   // "Meow! Meow!"
        // virtual デストラクタなので
        // Dog::~Dog / Cat::~Cat も呼ばれる
        delete a;
        delete b;
        return 0;
    }
    ```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|---|---|
| 関数ポインタを手動設定 | `virtual` を付けるだけ | コンパイラが vtable を自動生成 |
| `struct` に関数ポインタ | `class` にメソッド | OOP の表現がネイティブ |
| 派生の概念なし | `: public Animal` | 継承の構文あり |
| 手動キャストで型変換 | 基底ポインタから自動 | 安全な多態性 |

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
new Animal() / new Dog() / new Cat()
  |  → コンストラクタは 基底 → 派生 の順
  v
Animal* 経由で getType() / makeSound()
  |  → virtual なので動的ディスパッチ
  |     Dog なら Dog::makeSound
  |     Cat なら Cat::makeSound
  v
delete 各オブジェクト
  |  → virtual デストラクタなので
  |     派生 → 基底 の順に呼ばれる
  v
=== Wrong Polymorphism Test ===
new WrongAnimal() / new WrongCat()
  |
  v
WrongAnimal* 経由で makeSound()
  |  → virtual なしなので静的バインディング
  |     中身が WrongCat でも
  |     WrongAnimal::makeSound が呼ばれる
  v
WrongCat* で直接呼び出し
  |  → ポインタの型が WrongCat*
  |     → WrongCat::makeSound が呼ばれる
  v
終了
```

### Animal.hpp（基底クラス）

```cpp title="Animal.hpp" linenums="1"
#ifndef ANIMAL_HPP
#define ANIMAL_HPP

// std::string (C++ の文字列クラス) を使う
#include <string>
// std::cout / std::endl を使う
#include <iostream>

class Animal {
// protected: 派生クラスから見える
//   (public = 全員、private = 自分、
//    protected = 自分 + 子供)
protected:
    std::string _type;

public:
    // OCF 4 点セット
    Animal(void);
    Animal(const Animal& other);
    Animal& operator=(const Animal& other);
    // virtual デストラクタ: 必須!
    // これがないと基底* delete で
    // 派生のデストラクタが呼ばれない
    virtual ~Animal(void);

    // virtual: 子クラスで上書き可
    //   → 動的ディスパッチになる
    // 末尾 const: この関数は
    //   _type を変えないと約束
    virtual void makeSound(void) const;
    std::string getType(void) const;
};

#endif
```

!!! danger "virtual デストラクタを忘れない"
    基底クラスのデストラクタは **必ず virtual** にします。
    これを忘れると `Animal* → Dog` の delete で
    `Dog::~Dog` が呼ばれず、Dog のリソースがリークします。

### Animal.cpp

```cpp title="Animal.cpp" linenums="1"
#include "Animal.hpp"

// : _type("Animal") は初期化子リスト
// コンストラクタ本体 {} に入る前に
// _type を "Animal" で初期化
Animal::Animal(void) : _type("Animal") {
    std::cout
        << "Animal default constructor called"
        << std::endl;
}

// コピーコンストラクタ
// other の _type を自分にコピー
Animal::Animal(const Animal& other)
    : _type(other._type) {
    std::cout
        << "Animal copy constructor called"
        << std::endl;
}

// 代入演算子
Animal& Animal::operator=(
    const Animal& other
) {
    std::cout
        << "Animal assignment operator called"
        << std::endl;
    // 自己代入チェック: a = a; の事故を防ぐ
    if (this != &other)
        _type = other._type;
    // 連鎖代入 (a = b = c) のため *this を返す
    return *this;
}

Animal::~Animal(void) {
    std::cout
        << "Animal destructor called"
        << std::endl;
}

// virtual 関数のデフォルト実装
// 派生が上書きしない場合はこれが呼ばれる
void Animal::makeSound(void) const {
    std::cout
        << "... (generic animal sound)"
        << std::endl;
}

std::string Animal::getType(void) const {
    return _type;
}
```

### Dog.hpp / Dog.cpp（派生クラス）

```cpp title="Dog.hpp" linenums="1"
#ifndef DOG_HPP
#define DOG_HPP

#include "Animal.hpp"

// : public Animal = 「Animal を継承」
// Animal の public/protected メンバが
// Dog でも使える
class Dog : public Animal {
public:
    Dog(void);
    Dog(const Dog& other);
    Dog& operator=(const Dog& other);
    // 派生のデストラクタも virtual
    // (基底が virtual なら自動で virtual
    //  だが明示するのが分かりやすい)
    virtual ~Dog(void);

    // 基底の virtual 関数を上書き
    virtual void makeSound(void) const;
};

#endif
```

```cpp title="Dog.cpp" linenums="1"
#include "Dog.hpp"

Dog::Dog(void) {
    std::cout
        << "Dog default constructor called"
        << std::endl;
    // 基底の _type を書き換える
    // (_type は protected なので派生からOK)
    _type = "Dog";
}

// 基底のコピーコンストラクタを
// 初期化子リストで呼ぶ
// → Animal 部分がコピーされる
// 忘れると Animal 部分がデフォルト構築され
// _type が "Animal" になってしまう!
Dog::Dog(const Dog& other) : Animal(other) {
    std::cout
        << "Dog copy constructor called"
        << std::endl;
}

Dog& Dog::operator=(const Dog& other) {
    std::cout
        << "Dog assignment operator called"
        << std::endl;
    if (this != &other)
        // 基底の operator= を :: で呼ぶ
        // :: がないと無限再帰の恐れ
        Animal::operator=(other);
    return *this;
}

Dog::~Dog(void) {
    std::cout
        << "Dog destructor called"
        << std::endl;
}

// 上書きされた makeSound
// Animal* 経由でも virtual でこれが呼ばれる
void Dog::makeSound(void) const {
    std::cout << "Woof! Woof!"
              << std::endl;
}
```

Cat.cpp もほぼ同じ。違いは `_type = "Cat"` と `makeSound` が `"Meow! Meow!"` なだけ。

### WrongAnimal.hpp（対比用）

```cpp title="WrongAnimal.hpp" linenums="1"
#ifndef WRONGANIMAL_HPP
#define WRONGANIMAL_HPP

#include <string>
#include <iostream>

class WrongAnimal {
protected:
    std::string _type;

public:
    WrongAnimal(void);
    WrongAnimal(const WrongAnimal& other);
    WrongAnimal& operator=(
        const WrongAnimal& other
    );
    // デストラクタは virtual
    // (基底* delete の安全のため)
    virtual ~WrongAnimal(void);

    // ← virtual が付いていない!
    // WrongAnimal* 経由で呼ぶと
    // 常にこの実装が呼ばれる
    void makeSound(void) const;
    std::string getType(void) const;
};

#endif
```

`WrongCat` は `WrongAnimal` を継承し、`makeSound` を独自に書きますが、
**virtual がないので WrongAnimal* 経由では上書きされません**。

### main.cpp（動的ディスパッチ vs 静的バインディング）

```cpp title="main.cpp (抜粋)" linenums="1"
// ── virtual あり (動的ディスパッチ) ──
const Animal* j = new Dog();
j->makeSound();
// → "Woof! Woof!"
// Animal* だが中身は Dog
// virtual なので Dog::makeSound が呼ばれる

// ── virtual なし (静的バインディング) ──
const WrongAnimal* wc = new WrongCat();
wc->makeSound();
// → "... (wrong generic animal sound)"
// WrongAnimal* なので
// WrongAnimal::makeSound が呼ばれる
// 中身が WrongCat でも関係ない!

// ── 型を WrongCat* にすれば直接呼べる ──
const WrongCat* direct = new WrongCat();
direct->makeSound();
// → "Meow! Meow! (WrongCat)"
// ポインタの型が WrongCat* なので
// WrongCat::makeSound が呼ばれる
```

### コンストラクタ/デストラクタの呼び出し順

```
new Dog() の場合:
  1. Animal::Animal()   ← 基底が先
  2. Dog::Dog()         ← 派生が後

delete (Animal* → Dog) の場合
(virtual デストラクタ):
  1. Dog::~Dog()        ← 派生が先
  2. Animal::~Animal()  ← 基底が後

  → 「組み立ては下から、片付けは上から」
```

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "We need animals. Implement the following classes:
    > Animal, Dog, Cat. Each one must have a type attribute,
    > ... When its makeSound() function is called,
    > it should output an appropriate sound."
    >
    > "Also implement a WrongAnimal class and a WrongCat
    > class that inherits from it. WrongCat::makeSound()
    > should have a different output than
    > WrongAnimal::makeSound(). Then compare them with
    > the Animal/Cat output."

    virtual の有無による動作の違いが評価ポイント。

- [ ] `make` がエラーも警告もなく通る
- [ ] Animal, Dog, Cat の OCF が揃っている
- [ ] WrongAnimal, WrongCat の OCF も揃っている
- [ ] Animal* 経由で Dog/Cat の makeSound が呼ばれる
- [ ] WrongAnimal* 経由では WrongAnimal の makeSound が呼ばれる
- [ ] virtual デストラクタが宣言されている
- [ ] メモリリークなし

---

## 8. テストチェックリスト

### virtual キーワード

- [ ] `Animal::makeSound` に `virtual` が付いている
- [ ] `Animal::~Animal` に `virtual` が付いている
- [ ] `Dog::makeSound` / `Cat::makeSound` が上書きされている
- [ ] `WrongAnimal::makeSound` には virtual が **付いていない**
- [ ] `WrongAnimal::~WrongAnimal` には virtual が付いている

### 基本動作

- [ ] `Animal*` 経由で `Dog::makeSound` が呼ばれる
- [ ] `Animal*` 経由で `Cat::makeSound` が呼ばれる
- [ ] `Animal` 単体は "generic animal sound"
- [ ] `getType()` が正しい型名を返す

### WrongAnimal の対比

- [ ] `WrongAnimal*` 経由だと `WrongAnimal::makeSound`
- [ ] `WrongCat*` 型で直接呼ぶと `WrongCat::makeSound`

### コンストラクタ/デストラクタ

- [ ] 呼び出し順: 基底 → 派生（構築時）
- [ ] 呼び出し順: 派生 → 基底（破棄時）
- [ ] デストラクタにログ出力がある

### メモリ管理

- [ ] `leaks` コマンドまたは valgrind でリークなし

### 規約

- [ ] `malloc` / `free` / `printf` 不使用
- [ ] `using namespace std;` なし

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|---|---|
| ポリモーフィズムとは？ | 同じ関数呼び出しで、オブジェクトの型によって違う動作をすること |
| `virtual` を付けると何が起きる？ | 動的ディスパッチが有効になり、中身の型の関数が呼ばれる |
| virtual なしだとどうなる？ | 静的バインディングになり、ポインタの型の関数が呼ばれる |
| vtable とは？ | virtual 関数の関数ポインタの表。コンパイラが自動で生成する |
| なぜ virtual デストラクタが必要？ | 基底* 経由 delete で派生のデストラクタを呼ぶため。忘れると派生のリソースがリーク |
| コンストラクタの呼び出し順は？ | 基底 → 派生（組み立ては下から） |
| デストラクタの呼び出し順は？ | 派生 → 基底（片付けは上から） |
| WrongAnimal が示す問題は？ | virtual なしだとポインタの型で関数が決まり、期待する多態性が効かない |
| 基底のコピーコンストラクタを呼ぶ構文は？ | 初期化子リストで `: Animal(other)` と書く |
| 基底の operator= を呼ぶ構文は？ | `Animal::operator=(other)` と `::` 付きで呼ぶ |

---

## 10. よくあるミス

!!! warning "`virtual` を付け忘れる"
    ```cpp
    class Animal {
        void makeSound() const;  // virtual なし!
    };
    Animal* a = new Dog();
    a->makeSound();
    // → Animal::makeSound が呼ばれる
    // 期待した Dog の鳴き声にならない
    ```

!!! warning "virtual デストラクタを忘れる"
    ```cpp
    class Animal {
        ~Animal();   // virtual なし!
    };
    Animal* a = new Dog();
    delete a;
    // → Animal::~Animal のみ呼ばれる
    // Dog::~Dog が呼ばれない
    // → Dog のリソースがリーク
    ```

!!! warning "コピーコンストラクタで基底を忘れる"
    ```cpp
    // NG: Animal(other) がない
    Dog::Dog(const Dog& other) {
        // Animal 部分がデフォルト構築
        // _type が "Animal" になる
    }

    // OK: 基底のコピーコンストラクタを呼ぶ
    Dog::Dog(const Dog& other)
        : Animal(other) {
    }
    ```

!!! warning "operator= で基底を忘れる"
    ```cpp
    // NG: 基底部分がコピーされない
    Dog& Dog::operator=(const Dog& other) {
        if (this != &other) {
            // Animal::operator=(other); が抜けてる
        }
        return *this;
    }
    ```

!!! warning "operator= で `::` を忘れて無限再帰"
    ```cpp
    Dog& Dog::operator=(const Dog& other) {
        // NG: 自分自身を呼んでしまう!
        operator=(other);

        // OK: 基底のを :: 付きで呼ぶ
        Animal::operator=(other);
    }
    ```

---

## 11. 次の exercise へ

次の [ex01 Brain](ex01-brain.md) では、
**Dog/Cat にヒープメンバ `Brain*` を追加** します。

すると virtual デストラクタの真価が分かり、
**ディープコピー** の実装が必須になります。
