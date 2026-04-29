# ex02 — Abstract class

---

## このプログラムは何？

**Animal を「実体化できない抽象クラス」に変える** プログラムです。

ex01 までは `Animal a;` のように Animal そのものを作れました。
でもよく考えると「動物」という概念は犬でも猫でもない **抽象的な存在** です。
実際に鳴ける「動物一般」って存在しませんよね？

この exercise では `makeSound()` に **`= 0`** を付けて、
`Animal` を **設計図だけのクラス（抽象クラス）** に変えます。

```
ex01 まで:
  Animal a;         → OK (作れる)
  new Animal();     → OK

ex02 から:
  Animal a;         → コンパイルエラー!
  new Animal();     → コンパイルエラー!
  Animal* a =
    new Dog();      → OK (ポインタは可)
```

変更は **Animal.hpp の1行だけ**。でも影響は大きい。

---

## 1. このexerciseで学ぶこと

- **純粋仮想関数 (`= 0`)**: 実装を持たない関数
- **抽象クラス**: 純粋仮想関数を1つでも持つクラス
- **インスタンス化できない**: `Animal a;` がコンパイルエラー
- **コンストラクタは必要**: 派生クラスから呼ばれる
- **ポインタや参照は使える**: 実体の生成だけが禁止
- **派生クラスで全て実装しないと派生も抽象になる**

---

## 2. 新しい概念の解説

### 純粋仮想関数って何？

**「= 0 を付けた virtual 関数。実装を持たない」** ものです。

```cpp
class Animal {
public:
    // virtual あり + = 0 が付いている
    // → 純粋仮想関数
    virtual void makeSound() const = 0;
};
```

- **実装を書かない**（Animal.cpp に `makeSound` の定義を書かない）
- **派生クラスで必ず実装しなければならない**
- **この関数を持つクラスは「抽象クラス」になる**

### `= 0` って何を意味するの？

`= 0` は **「この関数は未実装。派生で必ず実装してね」というマーク** です。

```cpp
// 通常の virtual 関数
virtual void makeSound() const;
// → 基底に実装あり、派生は上書きしてもOK

// 純粋仮想関数
virtual void makeSound() const = 0;
// → 基底に実装なし、派生は必ず上書きが必要
// → このクラスを抽象クラスにする
```

!!! info "構文の注意"
    `= 0` は「0 を代入する」意味ではありません。
    C++ の特別な構文で「この仮想関数は未実装」というマーカーです。
    覚えるしかないポイントです。

### 抽象クラスって何？

**「純粋仮想関数を1つでも持つクラス」** のことです。

```cpp
class Animal {
    // 純粋仮想関数が1つでもあれば
    // このクラスは抽象クラス
    virtual void makeSound() const = 0;
};
```

**抽象クラスの特徴**:

- **インスタンス化できない** （`Animal a;` → エラー）
- **ポインタや参照としては使える** （`Animal*` / `Animal&` はOK）
- **コンストラクタは必要** （派生から呼ばれる）
- **純粋仮想関数以外は普通の実装を持てる**

### なぜインスタンス化できないの？

`Animal a;` と書いたら、`a.makeSound()` が呼べてしまいます。
でも makeSound は未実装 → 呼んだらクラッシュ。
それを防ぐために C++ は **コンパイル時にエラー** にします。

```cpp
// もし Animal がインスタンス化できたら...
Animal a;
a.makeSound();   // 何を呼べばいい?
// → makeSound は = 0 で実装なし!
// → クラッシュ確定 だから最初から禁止
```

### でもポインタは使える

```cpp
// NG: Animal そのものは作れない
Animal a;              // コンパイルエラー
new Animal();          // コンパイルエラー

// OK: ポインタや参照は作れる
Animal* a = new Dog(); // 中身は Dog → OK
Animal& ref = *a;      // 参照もOK
Animal* arr[4];        // 配列もOK
```

ポインタや参照は「実際の Animal を指しているわけじゃない」ので、
中身が Dog などの具象クラスならOK。

### コンストラクタは必要？（はい、必要）

抽象クラスでも **コンストラクタは必須** です。
なぜなら **派生クラスが構築される時に呼ばれる** から。

```cpp
class Animal {
    // 抽象クラスでもコンストラクタ必須
    Animal() : _type("Animal") {}
};

class Dog : public Animal {
    Dog() {
        // まず Animal::Animal() が呼ばれてから
        // Dog::Dog() が呼ばれる
        _type = "Dog";
    }
};

new Dog();
// 1. Animal::Animal() が呼ばれる ←必要!
// 2. Brain::Brain() が呼ばれる
// 3. Dog::Dog() が呼ばれる
```

### 抽象クラスが理にかなう理由

「Animal」という概念を考えてみてください:

- 動物一般に **共通の鳴き声** はない
- だから `Animal::makeSound()` の実装は **意味がない**
- むしろ **実装を忘れた時のバグを隠してしまう**

`= 0` で抽象クラスにすると:

- `Animal` を直接使おうとすると **コンパイルエラー** → バグ防止
- 派生が `makeSound` を実装し忘れると、その派生も抽象 → **コンパイルエラー**
- **設計の意図が言語レベルで保証される**

### 派生が全て実装しないと、派生も抽象クラス

```cpp
class Animal {
    virtual void makeSound() const = 0;
    virtual void sleep() const = 0;
};

class Dog : public Animal {
    virtual void makeSound() const {
        // 実装あり
    }
    // sleep() を実装し忘れた!
};

Dog d;   // コンパイルエラー!
// Dog も抽象クラスのまま
// (sleep が未実装だから)
```

派生クラスが **純粋仮想関数を1つでも実装し忘れる** と、
その派生もまだ抽象クラスのままになります。

---

## 3. 課題仕様

ex01 からの **変更はたった1行** です。

### Animal.hpp の変更

```diff
- virtual void makeSound(void) const;
+ virtual void makeSound(void) const = 0;
```

### Animal.cpp の変更

`makeSound` の定義を **削除** する。

### Dog / Cat の変更

**なし**。ex01 と同じ（既に `makeSound` を上書きしているので具象クラス）。

### main.cpp で確認

- `Animal a;` と書くとコンパイルエラーになることを確認
- `new Animal()` も同様
- `Animal*` / `Dog` / `Cat` は使える

---

## 4. 実行例

```console
$ make && ./abstract
=== Create Animals ===
Animal default constructor called
Brain default constructor called
Dog default constructor called
Animal default constructor called
Brain default constructor called
Cat default constructor called

Dog: Woof! Woof!
Cat: Meow! Meow!

=== Delete Animals ===
Dog destructor called
Brain destructor called
Animal destructor called
Cat destructor called
Brain destructor called
Animal destructor called
```

### コンパイルエラーの確認

`main.cpp` に以下を書き加えてみる:

```cpp
Animal test;   // このコメントを外すと...
```

```
error: variable type 'Animal' is an abstract class
note: unimplemented pure virtual method 'makeSound'
      in 'Animal'
```

**抽象クラスはインスタンス化できない** のがコンパイラに強制されます。

---

## 5. C と C++ の比較

C には「抽象クラス」に直接対応する仕組みがありません。
関数ポインタを `NULL` にして「未実装」を表す程度です。

!!! info "純粋仮想 (= 0) = 「関数ポインタ NULL をエラーにする」の言語版"
    C では:

    1. 構造体の関数ポインタを NULL で作る
    2. 子側で「必ず設定しなきゃ」とルール化したい
    3. でも言語が強制してくれない → **実行時に NULL 呼び出しで segfault**

    C++ の `= 0` は:

    1. 「この関数は未実装」を言語で宣言
    2. 子クラスで必ず上書き実装しないとエラー
    3. **コンパイル時に止める** → 安全

=== "C の書き方（実行時エラー）"

    ```c
    /* printf 用 */
    #include <stdio.h>
    /* strcpy 用 */
    #include <string.h>

    /* 関数ポインタ付き構造体 */
    typedef struct s_animal {
        char type[32];
        /* 関数ポインタ (未実装なら NULL) */
        void (*makeSound)(void);
    } t_animal;

    int main(void) {
        /* Animal を作る */
        t_animal a;
        /* 型名をコピー (手動) */
        strcpy(a.type, "Animal");
        /* 関数ポインタを NULL に */
        /* ← C++の = 0 に相当する意図 */
        /* でも言語が強制してくれない */
        a.makeSound = NULL;

        /* コンパイルは通ってしまう! */
        /* 実行時にチェックが必要 */
        if (a.makeSound)
            /* 関数ポインタ経由で呼ぶ */
            a.makeSound();
        else
            /* NULL なら自力でハンドリング */
            /* 忘れると segfault */
            printf("unimplemented!\n");

        return 0;
    }
    ```

=== "C++ の書き方（コンパイル時エラー）"

    ```cpp
    // cout 用
    #include <iostream>

    // ── 抽象クラス ──
    class Animal {
    public:
        // virtual デストラクタ (必須)
        virtual ~Animal() {}
        // ── 純粋仮想関数 ──
        // = 0 が「未実装」マーカー
        // Cの「関数ポインタ NULL」を
        // 言語機能にしたもの
        // 派生で必ず実装しないと
        // そちらも抽象クラスに
        virtual void makeSound(
            void
        ) const = 0;
    };

    int main(void) {
        // ↓ これはコンパイルエラー!
        // Animal a;
        //
        // error: variable type 'Animal'
        //   is an abstract class
        //
        // C の NULL チェック忘れと違い
        // 実行せずに発覚するので安全
        return 0;
    }
    ```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|---|---|
| 関数ポインタを `NULL` | `virtual void f() = 0;` | 「未実装」を言語で表現 |
| 実行時エラー | コンパイル時エラー | 早期にバグ検出 |
| 自制でインスタンス化を避ける | コンパイラが禁止 | 強制力がある |
| C には相当機能なし | 抽象クラス | OOP の基本概念 |

!!! info "なぜ C++ の方が優れているか"
    C の関数ポインタ `NULL` は実行時にならないと問題が発覚しません。
    C++ の `= 0` は **コンパイル時にバグを止めてくれる** ので、
    「うっかり Animal を直接使った」というミスを事前に防げます。

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
new Dog() / new Cat()
  |  Animal のコンストラクタも呼ばれる
  |  (抽象クラスでも派生経由で構築される)
  v
Animal* 経由で makeSound()
  |  = 0 だが派生 (Dog/Cat) が実装済み
  |  → 動的ディスパッチで Dog/Cat のが呼ばれる
  v
delete (Animal* → Dog/Cat)
  |  virtual デストラクタで
  |  派生 → 基底 の順で破棄される
  v
終了

(試しに)
  Animal a;
  → コンパイルエラー!
```

### Animal.hpp（`= 0` を追加）

```cpp title="Animal.hpp" linenums="1"
#ifndef ANIMAL_HPP
#define ANIMAL_HPP

#include <string>
#include <iostream>

// 抽象クラス: 1つ以上の純粋仮想関数 (= 0)
//             を持つクラス
// Animal 単体では new Animal() 不可
// 基底としてのみ使用される
class Animal {
protected:
    std::string _type;

public:
    // OCF 4 点 (抽象クラスでも必要)
    Animal(void);
    Animal(const Animal& other);
    Animal& operator=(const Animal& other);
    // virtual デストラクタ必須
    virtual ~Animal(void);

    // ↓ = 0 を追加!
    // これで純粋仮想関数になり
    // Animal は抽象クラスに変わる
    virtual void makeSound(
        void
    ) const = 0;
    std::string getType(void) const;
};

#endif
```

!!! info "OCF は抽象クラスでも必要"
    抽象クラス自体がインスタンス化されなくても、
    **派生クラスの構築/コピー時に基底の4関数が呼ばれる** ため、
    OCF はちゃんと書く必要があります。

### Animal.cpp（`makeSound` を削除）

```cpp title="Animal.cpp" linenums="1"
#include "Animal.hpp"

Animal::Animal(void) : _type("Animal") {
    std::cout
        << "Animal default constructor called"
        << std::endl;
}

Animal::Animal(const Animal& other)
    : _type(other._type) {
    std::cout
        << "Animal copy constructor called"
        << std::endl;
}

Animal& Animal::operator=(
    const Animal& other
) {
    std::cout
        << "Animal assignment operator called"
        << std::endl;
    if (this != &other)
        _type = other._type;
    return *this;
}

Animal::~Animal(void) {
    std::cout
        << "Animal destructor called"
        << std::endl;
}

// ↓ makeSound の定義を削除!
// = 0 なので実装不要
// (書くこともできるが普通は書かない)

std::string Animal::getType(void) const {
    return _type;
}
```

### Dog / Cat（変更なし）

ex01 と完全に同じです。
ただし、**Dog/Cat が `makeSound` を上書きしているから具象クラスになる** という理解が重要。

```cpp title="Dog.hpp (抜粋)"
class Dog : public Animal {
public:
    // ... OCF ...
    // makeSound を実装 → 具象クラスに
    virtual void makeSound(void) const;
};
```

もし Dog が `makeSound` を実装し忘れたら **Dog も抽象クラス** になり、
`new Dog()` もコンパイルエラーになります。

### main.cpp（抽象クラスの使用）

```cpp title="main.cpp" linenums="1"
#include "Animal.hpp"
#include "Dog.hpp"
#include "Cat.hpp"

int main(void) {
    // Animal test;  ← コンパイルエラー
    //   抽象クラスはインスタンス化不可
    //   コメントアウトしておく

    std::cout << "=== Create Animals ==="
              << std::endl;

    // Animal* は OK
    // 中身は Dog なので具象 → 作れる
    const Animal* dog = new Dog();
    const Animal* cat = new Cat();

    std::cout << dog->getType() << ": ";
    dog->makeSound();  // Dog::makeSound
    std::cout << cat->getType() << ": ";
    cat->makeSound();  // Cat::makeSound

    // virtual デストラクタで
    // 派生→基底の順に破棄される
    delete dog;
    delete cat;

    // Animal* の配列もOK
    const int size = 4;
    Animal* animals[size];
    for (int i = 0; i < size / 2; i++)
        animals[i] = new Dog();
    for (int i = size / 2; i < size; i++)
        animals[i] = new Cat();

    for (int i = 0; i < size; i++)
        delete animals[i];

    return 0;
}
```

### 使える使えないまとめ

| 書き方 | OK? | 理由 |
|---|---|---|
| `Animal a;` | NG | 抽象クラスのインスタンス化 |
| `new Animal();` | NG | 同上 |
| `Animal* a = new Dog();` | OK | ポインタは可、中身は具象 |
| `Animal& ref = *dog;` | OK | 参照も可 |
| `Animal arr[4];` | NG | Animal 実体の配列は不可 |
| `Animal* arr[4];` | OK | ポインタの配列は可 |

### コンストラクタの呼び出し順

```
new Dog() の場合:
  1. Animal::Animal()  ← 抽象だが呼ばれる
  2. Brain::Brain()
  3. Dog::Dog()

delete (Animal* → Dog):
  1. Dog::~Dog()       → delete _brain
  2. Brain::~Brain()
  3. Animal::~Animal()
```

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Modify your Animal class so that instantiating it
    > is impossible. Except for that, every existing
    > feature in Exercise 01 should remain."

    Animal が `= 0` で抽象クラスになっているかが評価の核心。

- [ ] `make` がエラー/警告なく通る
- [ ] `Animal::makeSound` に `= 0` が付いている
- [ ] `Animal a;` がコンパイルエラーになる
- [ ] `new Animal()` がコンパイルエラーになる
- [ ] Dog/Cat は正しく動作する（ex01 と同じ）
- [ ] メモリリークなし

---

## 8. テストチェックリスト

### 純粋仮想関数

- [ ] `Animal::makeSound` に `= 0` が付いている
- [ ] Animal.cpp に `makeSound` の定義がない
- [ ] `Animal a;` がコンパイルエラー（コメント確認）
- [ ] `new Animal()` がコンパイルエラー

### 基本動作

- [ ] `Animal*` ポインタで Dog/Cat を管理できる
- [ ] `makeSound()` が動的ディスパッチされる
- [ ] Dog/Cat の Brain ディープコピーが ex01 と同じく動く

### OCF

- [ ] Animal: 4関数（抽象クラスでも必要）
- [ ] Dog / Cat: 4関数（ディープコピー版）
- [ ] Brain: 4関数

### メモリ管理

- [ ] Animal* 配列の delete で Brain が解放される
- [ ] メモリリークなし

### 規約

- [ ] `malloc` / `free` / `printf` 不使用
- [ ] `using namespace std;` なし

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|---|---|
| 純粋仮想関数とは？ | `= 0` を付けた virtual 関数。実装を持たず、派生で必ず実装が必要 |
| 抽象クラスとは？ | 純粋仮想関数を1つでも持つクラス。インスタンス化できない |
| `Animal a;` と書くとどうなる？ | コンパイルエラー。抽象クラスはインスタンス化できない |
| 抽象クラスでもポインタは使える？ | はい。`Animal*` / `Animal&` は具象クラスを指す/参照する限りOK |
| 抽象クラスにコンストラクタは必要？ | 必要。派生クラスが構築される時に呼ばれる |
| 派生が純粋仮想関数を実装し忘れるとどうなる？ | その派生も抽象クラスになり、インスタンス化できなくなる |
| `= 0` の意味は？ | 「この関数は未実装」を表す C++ の特別な構文 |
| 抽象クラスと interface の違いは？ | interface は全メソッドが `= 0` + データなし。抽象クラスは一部 `= 0` + データあり |
| なぜ Animal を抽象にする？ | 「動物一般」は具体的な鳴き声を持たない。概念と実体を言語で区別するため |
| 純粋仮想関数でも実装を持てる？ | 持てる。ただし 42 では使わない |

---

## 10. よくあるミス

!!! warning "`= 0` を付け忘れる"
    ```cpp
    // NG: = 0 がない
    virtual void makeSound() const;
    // → 抽象クラスにならない
    // → Animal がインスタンス化できてしまう
    ```

!!! warning "Animal.cpp の makeSound を削除し忘れる"
    ```cpp
    // Animal.hpp
    virtual void makeSound() const = 0;

    // Animal.cpp
    void Animal::makeSound() const {
        // この定義は書かない方が普通
    }
    ```

!!! warning "抽象クラスに OCF を書かない"
    ```cpp
    // NG: 「抽象クラスだからコンストラクタ不要」
    //     と思ってしまう
    class Animal {
        virtual void makeSound() const = 0;
        // OCF を書かない
    };
    // → 42 では OCF 全て明示が必須
    ```

!!! warning "`= 0` と `{}` (空実装) を混同"
    ```cpp
    // 全く違う!
    virtual void f() const = 0;
    // → 純粋仮想 → 抽象クラスに

    virtual void f() const {}
    // → 空の実装あり → 抽象クラスではない
    ```

!!! warning "派生の `makeSound` を削除してしまう"
    ```cpp
    // Dog から makeSound を削除すると...
    class Dog : public Animal {
        // makeSound がない
    };
    new Dog();  // ← コンパイルエラー!
    // Dog も抽象クラスのまま
    ```

---

## 11. 次の exercise へ

次の [ex03 Interface & recap](ex03-materia.md) では、
**インターフェース** と **プロトタイプパターン** を学びます。

cpp00〜04 の集大成。**7つのクラス** を連携させて、
メモリの **所有権管理** までしっかり行う、最も難しい exercise です。
