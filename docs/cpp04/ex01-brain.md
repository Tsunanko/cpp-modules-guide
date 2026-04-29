# ex01 — I don't want to set the world on fire

---

## このプログラムは何？

**犬と猫に「脳」を持たせるプログラム** です。

ex00 の Dog/Cat にヒープ確保のメンバ `Brain*` を追加します。
ここでのポイントは、**Dog をコピーしたときに脳も別々に用意する** こと。
もし脳が共有されていたら、片方の Dog を消した瞬間にもう片方の脳も消える大事故になります。

```
ダメな例 (シャローコピー):
  Dog A ────→ Brain [0x100]
  Dog B ────→ Brain [0x100]  ← 同じ脳を共有!
  → Dog A が死ぬ → 脳も delete される
  → Dog B の脳がダングリングポインタに

良い例 (ディープコピー):
  Dog A ────→ Brain [0x100]
  Dog B ────→ Brain [0x200]  ← 別々の脳
  → それぞれ独立して delete できる
```

これを **ディープコピー** で解決します。

---

## 1. このexerciseで学ぶこと

- **Brain クラス**: ヒープに確保するメンバクラス
- **シャローコピー**: ポインタだけコピー（ダメな例）
- **ディープコピー**: ポインタの先までコピー（正解）
- **コピーコンストラクタ** でのディープコピー実装
- **代入演算子** でのディープコピー実装
- **自己代入チェック**: `a = a;` の事故を防ぐ
- **virtual デストラクタの真価**: `delete _brain` が呼ばれる

---

## 2. 新しい概念の解説

### シャローコピーって何？

**「ポインタの値だけをコピーすること」** です。
2つのオブジェクトが **同じ中身** を指してしまいます。

```cpp
class Dog {
public:
    Brain* _brain;
};

Dog A;
A._brain = new Brain();   // 脳を作る

Dog B = A;   // コピー
// デフォルトのコピー = シャローコピー
// B._brain = A._brain;  (ポインタの値だけ)
// → A も B も同じ Brain を指す
```

```
メモリの様子:

  Dog A            Dog B
  ┌─────┐         ┌─────┐
  │_brain─┐       │_brain─┐
  └─────┘ │       └─────┘ │
          ├───────────────┤
          │               │
          ▼               ▼
       ┌──────────────┐
       │    Brain     │  ← 同じ Brain を 2つが共有!
       │  ideas[100]  │
       └──────────────┘
```

### シャローコピーの何が問題？（二重 delete）

2つのオブジェクトが同じ Brain を指しているので、
**両方が死ぬ時に同じ Brain を2回 delete** してしまいます。

```cpp
Dog A;          // A._brain = new Brain()
{
    Dog B = A;  // シャローコピー: B._brain = A._brain
}   // B のスコープ終了
    // → B のデストラクタ: delete _brain
    // Brain が解放される

// A._brain は ダングリングポインタ (もう存在しない)
A.getBrain()->ideas[0] = "oops";  // 未定義動作!

// スコープ終了
// → A のデストラクタ: delete _brain
// 同じ Brain を二重 delete → クラッシュ!
```

!!! danger "二重 free / 二重 delete"
    同じメモリ領域を2回 `free` / `delete` すると、
    **未定義動作**（クラッシュ、メモリ破壊、ランダムバグ）が起きます。
    シャローコピーの典型的な失敗です。

### ディープコピーって何？

**「ポインタの先までちゃんと別途コピーすること」** です。
2つのオブジェクトは **独立した中身** を持ちます。

```cpp
Dog::Dog(const Dog& other)
    : Animal(other),
      // 新しい Brain を new で作り、
      // other._brain の中身をコピー
      _brain(new Brain(*other._brain))
{
}
```

```
メモリの様子:

  Dog A            Dog B
  ┌─────┐         ┌─────┐
  │_brain─┐       │_brain─┐
  └─────┘ │       └─────┘ │
          ▼               ▼
       ┌────────┐      ┌────────┐
       │ Brain  │      │ Brain  │  ← 別々!
       │ ideas  │      │ ideas  │
       └────────┘      └────────┘
```

### ディープコピーの実装方法

**コピーコンストラクタ** と **代入演算子** の2箇所で必要です。

=== "コピーコンストラクタ"

    ```cpp
    Dog::Dog(const Dog& other)
        : Animal(other),  // 基底をコピー
          _brain(
            new Brain(*other._brain)
          )  // Brain を新規に new
    {
    }
    ```

=== "代入演算子"

    ```cpp
    Dog& Dog::operator=(const Dog& other) {
        // 自己代入チェック
        if (this != &other) {
            // 基底部分をコピー
            Animal::operator=(other);
            // 古い Brain を削除 (リーク防止)
            delete _brain;
            // 新しい Brain を new
            _brain = new Brain(*other._brain);
        }
        return *this;
    }
    ```

### 自己代入チェックって何？

**`a = a;` のような自分への代入を検出するガード** です。

```cpp
Dog& Dog::operator=(const Dog& other) {
    // other が自分自身かチェック
    if (this != &other) {
        // 違う相手の時だけ処理する
        delete _brain;
        _brain = new Brain(*other._brain);
    }
    return *this;
}
```

なぜ必要？ チェックを忘れると **自分を破壊** してしまいます:

```cpp
// チェックなしの場合 (ダメな例):
Dog& Dog::operator=(const Dog& other) {
    delete _brain;
    _brain = new Brain(*other._brain);
    // ↑ other が自分自身だったら?
    //   other._brain は今 delete したばかり
    //   → delete 済みのメモリを読む → 未定義動作!
    return *this;
}

Dog a;
a = a;   // これで事故が起きる!
```

### `new Brain(*other._brain)` を詳しく見る

この1行がディープコピーの核心です。

```cpp
new Brain(*other._brain)
```

分解すると:

```
other._brain     → Brain* (ポインタ)
*other._brain    → Brain (ポインタの中身を取り出す)
Brain(...)       → Brain のコピーコンストラクタ
new Brain(...)   → ヒープに新しい Brain を作る

つまり:
  1. other._brain が指す Brain の中身を読む
  2. その中身をコピーして新しい Brain をヒープに作る
  3. 新しい Brain へのポインタを返す
```

---

## 3. 課題仕様

| クラス | メンバ / 関数 | 説明 |
|---|---|---|
| **Brain** | `std::string ideas[100]` | 100個のアイデア |
| | OCF 4関数 | ディープコピー対応 |
| **Dog** | `private: Brain* _brain` | ヒープの Brain |
| | コンストラクタ | `new Brain()` |
| | コピーコンストラクタ | `new Brain(*other._brain)` |
| | 代入演算子 | delete 旧 → new 新 |
| | デストラクタ | `delete _brain` |
| **Cat** | Dog と同じ構造 | |
| **Animal** | ex00 と同じ | virtual デストラクタ必須 |

追加テスト: Dog をコピーし、片方の Brain を変更してもう片方が影響を受けないことを確認。

---

## 4. 実行例

```console
$ make && ./brain
=== Create Animal Array ===
Animal default constructor called
Brain default constructor called
Dog default constructor called
... (Dog/Cat が交互に生成)

Dog: Woof! Woof!
Cat: Meow! Meow!
... (virtual で正しく動作)

=== Delete Animal Array ===
Dog destructor called
Brain destructor called    ← _brain が解放される
Animal destructor called
... (全て順次破棄)

=== Deep Copy Test ===
Original idea[0]: Chase the ball
Copy idea[0]: Chase the ball
After modifying copy:
Original idea[0]: Chase the ball    ← 変わっていない!
Copy idea[0]: Sleep all day         ← copy だけ変わった

=== Assignment Operator Deep Copy ===
Original idea[0]: Chase the ball
Assigned idea[0]: Dig a hole

=== Cleanup ===
(スタック上の Dog 自動で破棄)
```

**注目**: copy の Brain を変更しても original の Brain は変わらない
→ **ディープコピーが効いている証拠**。

---

## 5. C と C++ の比較

!!! info "ディープコピー = C の「自分で別 malloc して中身コピー」の自動化"
    C でヒープメンバを持つ構造体をコピーするには:

    1. `malloc` で新しいメモリを確保
    2. `memcpy` で中身をコピー
    3. コピー関数 (`dog_copy` など) を **自分で** 書く

    C++ の **コピーコンストラクタ** は、この 3 ステップを
    「コピーするたびに呼ばれる関数」に **言語機能として** 組み込んだものです。

=== "C の書き方"

    ```c
    /* printf / fprintf 用 */
    #include <stdio.h>
    /* malloc / free 用 */
    #include <stdlib.h>
    /* strcpy / memcpy 用 */
    #include <string.h>

    /* ── Brain 構造体 ── */
    /* 100 個のアイデアを持つ */
    typedef struct s_brain {
        char ideas[100][256];
    } t_brain;

    /* ── Dog 構造体 ── */
    /* Brain* をヒープで持つ */
    typedef struct s_dog {
        char type[32];
        /* ヒープ上の Brain を指す */
        t_brain *brain;
    } t_dog;

    /* ── Dog をヒープに作る ── */
    t_dog *dog_new(void) {
        /* Dog 本体を確保 */
        t_dog *d = malloc(sizeof(t_dog));
        /* 型名をコピー (手動) */
        strcpy(d->type, "Dog");
        /* さらに Brain もヒープに確保 */
        /* C++ ならコンストラクタで new */
        d->brain = malloc(sizeof(t_brain));
        return d;
    }

    /* ── ディープコピー関数 ── */
    /* C++ のコピーコンストラクタに相当 */
    /* C では「自分で書く」必要がある */
    t_dog *dog_copy(t_dog *src) {
        /* 新しい Dog をヒープに */
        t_dog *d = malloc(sizeof(t_dog));
        /* 型名をコピー */
        strcpy(d->type, src->type);
        /* シャローコピーではなく */
        /* brain を別途 malloc する */
        /* ← ここがディープコピーの核心 */
        d->brain = malloc(sizeof(t_brain));
        /* 中身を memcpy でバイト単位コピー */
        /* (C++ なら std::string が自動で */
        /*  賢くコピーしてくれる) */
        memcpy(
            d->brain, src->brain,
            sizeof(t_brain)
        );
        return d;
    }

    /* ── 後片付け ── */
    /* 順序が大事: brain → 本体 */
    /* C++ ならデストラクタが自動でやる */
    void dog_free(t_dog *d) {
        /* 先に Brain を解放 */
        free(d->brain);
        /* 次に Dog 本体 */
        free(d);
    }
    ```

=== "C++ の書き方"

    ```cpp
    // cout 用のヘッダ
    #include <iostream>
    // std::string 用のヘッダ
    #include <string>

    // ── Brain クラス ──
    class Brain {
    public:
        // std::string の配列 (100 個)
        // std::string 自身が
        // ディープコピーを自動でやる
        std::string ideas[100];
        // デフォルトコンストラクタ
        Brain() {}
        // コピーコンストラクタ:
        // C の「dog_copy」に相当する処理を
        // 言語機能で自動呼び出し
        Brain(const Brain& o) {
            // 各要素を順にコピー
            // string の = は中身までコピー
            for (int i = 0; i < 100; i++)
                ideas[i] = o.ideas[i];
        }
        // デストラクタ: 自動呼び出し
        ~Brain() {}
    };

    // ── Dog クラス ──
    // : public Animal = Animal を継承
    class Dog : public Animal {
    private:
        // ヒープ上の Brain を指す
        // Cの t_dog.brain と同じ役割
        Brain* _brain;
    public:
        // コンストラクタ:
        // 初期化子リストで new Brain()
        // → C の malloc(sizeof(t_brain)) と
        //   init を合わせたもの
        Dog() : _brain(new Brain()) {
            _type = "Dog";
        }
        // ── コピーコンストラクタ ──
        // Cの dog_copy の自動化版
        Dog(const Dog& o)
            // 基底 Animal もコピー
            : Animal(o),
              // 新しい Brain を new で作り
              // o._brain の中身をコピー
              // = C の malloc + memcpy の合体
              _brain(new Brain(*o._brain))
        {}
        // ── 代入演算子 ──
        Dog& operator=(const Dog& o) {
            // 自己代入チェック (a=a の事故防止)
            if (this != &o) {
                // 基底部分の代入
                Animal::operator=(o);
                // 旧 Brain を解放
                // (忘れるとリーク!)
                delete _brain;
                // 新 Brain をディープコピー
                _brain = new Brain(*o._brain);
            }
            return *this;
        }
        // ── デストラクタ ──
        // 自動で呼ばれる (C の dog_free 不要)
        // virtual: 基底* delete でも呼ばれる
        virtual ~Dog() {
            // 所有している Brain を解放
            delete _brain;
        }
    };
    ```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|---|---|
| `malloc` + 手動コピー関数 | コピーコンストラクタ | 言語が「コピー時の処理」をフックしてくれる |
| `memcpy` で中身コピー | `std::string` が自動でコピー | 文字列がオブジェクトで賢い |
| `free` の順序に注意 | デストラクタで `delete` | RAII で自動解放 |
| 二重 free が起きやすい | 自己代入チェックで防ぐ | 言語機能で安全性 |

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
new Dog() → _brain = new Brain()
  |  基底 (Animal) → Brain → Dog の順
  v
Dog をコピー
  |  コピーコンストラクタ呼び出し
  |  _brain = new Brain(*other._brain)
  |  → 別の Brain がヒープに作られる
  v
copy の brain を変更
  |  original には影響なし (独立)
  v
assigned = original (代入)
  |  operator= で
  |  delete 旧 _brain
  |  new Brain(*other._brain) で新規作成
  v
スコープ終了
  |  デストラクタが派生→基底の順で呼ばれる
  |  _brain が delete される
  v
終了
```

### Brain.hpp

```cpp title="Brain.hpp" linenums="1"
#ifndef BRAIN_HPP
#define BRAIN_HPP

#include <string>
#include <iostream>

class Brain {
public:
    // 100個のアイデアを保持
    // std::string は自身で
    // ディープコピーを行ってくれる
    std::string ideas[100];

    // OCF 4 点セット
    Brain(void);
    Brain(const Brain& other);
    Brain& operator=(const Brain& other);
    // 継承されないので virtual 不要
    ~Brain(void);
};

#endif
```

### Brain.cpp

```cpp title="Brain.cpp" linenums="1"
#include "Brain.hpp"

Brain::Brain(void) {
    std::cout
        << "Brain default constructor called"
        << std::endl;
}

// コピーコンストラクタ
// 各要素を順にコピー
// std::string の = 演算子が
// 自動で中身までコピーしてくれる
Brain::Brain(const Brain& other) {
    std::cout
        << "Brain copy constructor called"
        << std::endl;
    for (int i = 0; i < 100; i++)
        ideas[i] = other.ideas[i];
}

// 代入演算子
Brain& Brain::operator=(
    const Brain& other
) {
    std::cout
        << "Brain assignment operator called"
        << std::endl;
    // 自己代入チェック
    if (this != &other) {
        for (int i = 0; i < 100; i++)
            ideas[i] = other.ideas[i];
    }
    return *this;
}

Brain::~Brain(void) {
    std::cout
        << "Brain destructor called"
        << std::endl;
}
```

### Dog.hpp（Brain* メンバの追加）

```cpp title="Dog.hpp" linenums="1"
#ifndef DOG_HPP
#define DOG_HPP

#include "Animal.hpp"
#include "Brain.hpp"

class Dog : public Animal {
private:
    // ヒープ上の Brain へのポインタ
    // Dog が所有する
    //   コンストラクタで new
    //   デストラクタで delete
    Brain* _brain;

public:
    Dog(void);
    Dog(const Dog& other);
    Dog& operator=(const Dog& other);
    virtual ~Dog(void);

    virtual void makeSound(void) const;
    // テスト用: Brain へのアクセス
    Brain* getBrain(void) const;
};

#endif
```

### Dog.cpp（ディープコピー実装）

```cpp title="Dog.cpp" linenums="1"
#include "Dog.hpp"

// コンストラクタ
// 初期化子リストで new Brain()
// → Dog が生まれた瞬間に Brain も確保
Dog::Dog(void) : _brain(new Brain()) {
    std::cout
        << "Dog default constructor called"
        << std::endl;
    _type = "Dog";
}

// ディープコピー版コピーコンストラクタ
// ここが ex01 最大の学びポイント
Dog::Dog(const Dog& other)
    : Animal(other),                  // (1)
      _brain(new Brain(*other._brain))// (2)
{
    std::cout
        << "Dog copy constructor called"
        << std::endl;
}
// (1) 基底 Animal のコピーコンストラクタ呼び出し
//     _type などがコピーされる
// (2) ディープコピーの核心
//     other._brain は Brain*
//     *other._brain は Brain (実体)
//     new Brain(...) で
//     別ヒープに新しい Brain を作る

// 代入演算子もディープコピー
Dog& Dog::operator=(const Dog& other) {
    std::cout
        << "Dog assignment operator called"
        << std::endl;
    // 自己代入チェック: a = a; の事故を防ぐ
    // これがないと自分の _brain を
    // 消してから参照してしまう
    if (this != &other) {
        // 基底部分の代入を明示呼び出し
        Animal::operator=(other);
        // 旧 Brain を解放 (リーク防止)
        delete _brain;
        // 新しい Brain をディープコピーで作成
        _brain = new Brain(*other._brain);
    }
    return *this;
}

// デストラクタで _brain を解放
// virtual デストラクタなので
// Animal* 経由でも呼ばれる
Dog::~Dog(void) {
    std::cout
        << "Dog destructor called"
        << std::endl;
    delete _brain;
}

void Dog::makeSound(void) const {
    std::cout << "Woof! Woof!" << std::endl;
}

Brain* Dog::getBrain(void) const {
    return _brain;
}
```

Cat.cpp もほぼ同じ。違いは `_type = "Cat"` と `makeSound` の中身だけ。

!!! info "なぜ `delete _brain;` でいいのか？"
    `_brain` は `new Brain()` で作ったので、`delete _brain` で解放できます。
    さらに Brain のデストラクタが自動で呼ばれ、Brain 内部の `std::string` も
    自動で解放されます（RAII）。

### main.cpp（ディープコピー検証）

```cpp title="main.cpp (ディープコピーテスト)" linenums="1"
// Original を作成
Dog original;
original.getBrain()->ideas[0]
    = "Chase the ball";

// コピーコンストラクタでコピー
// → ディープコピー
Dog copy(original);

// 両方とも "Chase the ball"
std::cout << "Original: "
    << original.getBrain()->ideas[0]
    << std::endl;
std::cout << "Copy: "
    << copy.getBrain()->ideas[0]
    << std::endl;

// copy の Brain を変更
copy.getBrain()->ideas[0]
    = "Sleep all day";

// ディープコピーの証明
// original は変わっていない!
std::cout << "After modify:" << std::endl;
std::cout << "Original: "
    << original.getBrain()->ideas[0]
    << std::endl;   // "Chase the ball"
std::cout << "Copy: "
    << copy.getBrain()->ideas[0]
    << std::endl;   // "Sleep all day"
```

### virtual デストラクタの真価

```cpp title="main.cpp (配列テスト)" linenums="1"
Animal* animals[6];
for (int i = 0; i < 3; i++)
    animals[i] = new Dog();
for (int i = 3; i < 6; i++)
    animals[i] = new Cat();

// Animal* 経由で delete
// virtual デストラクタが効くので
// Dog::~Dog / Cat::~Cat が呼ばれ、
// 中の _brain も解放される
for (int i = 0; i < 6; i++)
    delete animals[i];
```

!!! danger "virtual デストラクタを忘れるとリーク"
    もし `virtual ~Animal()` を忘れると、
    `delete animals[i]` は `Animal::~Animal` しか呼ばれず、
    `Dog::~Dog` が呼ばれない → **Brain が全部リーク**。

### 代入演算子の中の実行順序

```
Dog& Dog::operator=(const Dog& other)
{
  ┌─ if (this != &other)
  │
  │   Animal::operator=(other)
  │   → _type をコピー
  │
  │   delete _brain;
  │   → 今持っている古い Brain を破棄
  │     (リーク防止)
  │
  │   _brain = new Brain(*other._brain);
  │   → 新しい Brain を
  │     ディープコピーで作成
  └─
  return *this;
}
```

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Add a Brain class to your Dog and Cat. It must
    > contain an array of 100 std::string called ideas.
    > ... Make sure that deep copy is performed when
    > copying your Dog or Cat instances."

    ディープコピーが正しく実装されているかが評価の核心。

- [ ] `make` がエラー/警告なく通る
- [ ] Brain クラスに `std::string ideas[100]` がある
- [ ] Dog/Cat に `Brain*` メンバがある
- [ ] Dog をコピーして Brain を変更しても、元の Brain は変わらない
- [ ] 代入演算子でも同様のディープコピーが機能する
- [ ] メモリリークなし
- [ ] 配列 `Animal* animals[]` を正しく delete できる

---

## 8. テストチェックリスト

### ディープコピーの検証

- [ ] Dog をコピーコンストラクタでコピー
- [ ] コピーの Brain を変更 → original が変わらない
- [ ] Dog を代入演算子でコピー
- [ ] 代入版でも同様の独立性を確認
- [ ] Cat でも同様にテスト
- [ ] `getBrain()` のアドレスが original と copy で異なる

### メモリ管理

- [ ] Animal* 配列の delete で Brain が解放される
- [ ] `leaks` コマンドまたは valgrind でリークなし
- [ ] virtual デストラクタが効いている

### OCF 完全性

- [ ] Brain: 4関数
- [ ] Dog: 4関数（ディープコピー版）
- [ ] Cat: 4関数（ディープコピー版）
- [ ] Animal: 4関数

### 基本動作

- [ ] Dog/Cat 配列が正しく生成/破棄される
- [ ] `makeSound()` が virtual で動的ディスパッチ
- [ ] コンストラクタ/デストラクタのログが正しい順序

### 規約

- [ ] `malloc` / `free` / `printf` 不使用
- [ ] `using namespace std;` なし
- [ ] `_brain` は `private`

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|---|---|
| シャローコピーとは？ | ポインタの値だけコピーして中身を共有する。2つのオブジェクトが同じメモリを指す |
| シャローコピーの何が問題？ | 両方が破棄される時に同じメモリを2回 delete して未定義動作になる |
| ディープコピーとは？ | ポインタの先までちゃんと別途 new してコピーする。独立した2つのオブジェクトになる |
| ディープコピーの実装場所は？ | コピーコンストラクタと代入演算子の2箇所 |
| `new Brain(*other._brain)` の意味は？ | `*other._brain` で Brain の実体を取り出し、Brain のコピーコンストラクタでヒープに新規作成 |
| 自己代入チェックの目的は？ | `a = a;` の時に自分の _brain を delete してから参照する事故を防ぐ |
| 代入演算子で delete を忘れるとどうなる？ | 古い Brain がリークする |
| なぜ virtual デストラクタが必須？ | Animal* 経由 delete で Dog::~Dog が呼ばれないと _brain がリークする |
| RAII とは？ | Resource Acquisition Is Initialization。コンストラクタで取得、デストラクタで解放のパターン |
| Rule of Three とは？ | コピーコンストラクタ/代入演算子/デストラクタのどれか1つを書いたら残り2つも書くべき |

---

## 10. よくあるミス

!!! warning "シャローコピー（デフォルト）のまま使う"
    ```cpp
    // NG: コピーコンストラクタを書かない
    class Dog : public Animal {
        Brain* _brain;
    };
    // コンパイラが自動生成するのは
    // シャローコピー → 二重 delete
    ```

!!! warning "代入演算子で旧 Brain を delete し忘れる"
    ```cpp
    Dog& Dog::operator=(const Dog& other) {
        if (this != &other) {
            Animal::operator=(other);
            // delete _brain; が抜けてる!
            _brain = new Brain(*other._brain);
            // → 旧 Brain がリーク
        }
        return *this;
    }
    ```

!!! warning "自己代入チェックを忘れる"
    ```cpp
    Dog& Dog::operator=(const Dog& other) {
        // NG: 自己代入チェックなし
        Animal::operator=(other);
        delete _brain;
        // other == this だった場合
        // other._brain は今消したばかり
        _brain = new Brain(*other._brain);
        // → 未定義動作!
        return *this;
    }
    ```

!!! warning "コピーコンストラクタで基底を忘れる"
    ```cpp
    // NG: Animal(other) がない
    Dog::Dog(const Dog& other)
        : _brain(new Brain(*other._brain))
    {
        // Animal 部分がデフォルト構築
        // _type が "Animal" になる
    }

    // OK:
    Dog::Dog(const Dog& other)
        : Animal(other),  // 基底をコピー
          _brain(new Brain(*other._brain))
    {
    }
    ```

!!! warning "デストラクタで delete を忘れる"
    ```cpp
    Dog::~Dog(void) {
        // delete _brain; が抜けてる!
        // → _brain がリーク
    }
    ```

!!! warning "virtual デストラクタがない"
    ```cpp
    class Animal {
        ~Animal();  // virtual なし!
    };
    Animal* a = new Dog();
    delete a;
    // Dog::~Dog が呼ばれない
    // → _brain がリーク
    ```

---

## 11. 次の exercise へ

次の [ex02 Abstract class](ex02-abstract.md) では、
**Animal を抽象クラスに変更** します。

`virtual makeSound() const = 0;` と書くだけで、
`Animal a;` がコンパイルエラーになる仕組みを学びます。
