# ex03 — Interface & recap

---

## このプログラムは何？

**キャラクターがマテリア（魔法の素材）を装備して使うゲーム** です。

cpp00〜04 の集大成。**7つのクラス** を設計し、
インターフェース、プロトタイプパターン、所有権管理を一度に扱います。

```
ストーリー:
  "MateriaSource" という工房に
  Ice (氷) と Cure (回復) の原本を登録
     ↓
  "Character" (me) を作成
     ↓
  工房に "ice を作って" と注文
     → 原本を clone してマテリアを生産
     → me に装備させる
     ↓
  me が bob に対して
     → Ice を使う → "* shoots an ice bolt at bob *"
     → Cure を使う → "* heals bob's wounds *"
```

このプログラムの本当の難しさは **コード量** ではなく、
**「誰が new して、誰が delete するか」** を間違えないことです。

---

## 1. このexerciseで学ぶこと

- **インターフェース（I-prefix）**: 契約だけを定義するクラス
- **抽象クラス（A-prefix）**: 基底として機能を提供
- **プロトタイプパターン**: `clone()` で型を知らずにコピー
- **所有権管理（ownership）**: new / delete の責任者を明確に
- **前方宣言**: ヘッダの循環 include を避ける
- **OCF の徹底**: インターフェース以外の全クラスに4関数

---

## 2. 新しい概念の解説

### インターフェースって何？（I-prefix）

**「全メソッドが純粋仮想関数な抽象クラス。データを持たない」** ものです。

```cpp
// I プレフィックス = Interface
class ICharacter {
public:
    // virtual デストラクタ (必須)
    virtual ~ICharacter(void) {}
    // 全メソッドが = 0
    virtual std::string const& getName(
    ) const = 0;
    virtual void equip(AMateria* m) = 0;
    virtual void unequip(int idx) = 0;
    virtual void use(
        int idx, ICharacter& target
    ) = 0;
    // データメンバなし!
};
```

**インターフェースの特徴**:

- **名前の先頭に `I`** を付ける慣習（`ICharacter`, `IMateriaSource`）
- **全メソッドが `= 0`**
- **データメンバ（変数）なし**
- **virtual デストラクタは必須**（実装は `{}` でOK）
- **OCF は不要**（状態を持たないため）

### インターフェースと抽象クラスの違い

| | 抽象クラス (A-prefix) | インターフェース (I-prefix) |
|---|---|---|
| 例 | `AMateria` | `ICharacter` |
| 純粋仮想関数 | 1つ以上 | 全メソッド |
| 普通の実装メソッド | 持てる | なし |
| データメンバ | 持てる | なし |
| コンストラクタ | 必要 | 不要 |
| OCF | 必要 | virtual デストラクタだけ |

### プロトタイプパターンって何？

**「オブジェクトが自分のコピーを作る」デザインパターン** です。
`clone()` メソッドで実現します。

```cpp
class AMateria {
public:
    // これがプロトタイプパターン
    virtual AMateria* clone(void) const = 0;
};

class Ice : public AMateria {
public:
    // 自分の型を知っている
    virtual AMateria* clone(void) const {
        // 自分自身を new でコピー
        return new Ice(*this);
    }
};
```

### なぜプロトタイプパターンが便利？

**抽象クラスのポインタから、正しい型のコピーが作れる** からです。

```cpp
// AMateria* で持っている (中身は Ice か Cure か分からない)
AMateria* m = src->createMateria("ice");

// 普通ならコピーできない:
// new AMateria(*m);  ← NG! 抽象クラスは new できない
// new Ice(*m);       ← NG! m が Ice か分からない

// ↓ clone() なら OK
AMateria* copy = m->clone();
// 中身が Ice なら → new Ice(*this) が呼ばれる
// 中身が Cure なら → new Cure(*this) が呼ばれる
```

!!! info "具体例"
    `AMateria* inventory[4]` に Ice と Cure が混在して入っているとします。
    `inventory[i]->clone()` だけで、
    Ice なら Ice のコピー、Cure なら Cure のコピーが作れる。
    **呼ぶ側は実際の型を知らなくていい**。これがプロトタイプパターンの真価です。

### 所有権 (ownership) って何？

**「誰が new して、誰が delete する責任を負うか」のルール** です。

```
所有権のパターン:
  A が B に対して new した X を渡す
    |
    +-- A がまだ責任を持つ  (B は借りてるだけ)
    |
    +-- 所有権が B に移る   (B が delete 責任を負う)
```

C++98 には所有権を明示する仕組みがないので、
**コメントや命名で管理する** 必要があります。

### この exercise の所有権ルール

subject で指定された所有権ルール:

| 操作 | 所有権 |
|---|---|
| `learnMateria(AMateria*)` | 所有権が **source に移る**。source が delete 責任 |
| `createMateria()` の戻り値 | 所有権が **呼び出し側に移る**。呼び出し側が delete 責任 |
| `equip(AMateria*)` | 所有権が **character に移る** |
| `unequip(idx)` | **delete してはいけない**。外すだけ |

### unequip で delete しない理由

subject の仕様:

> "Your unequip() member function must not delete Materia!"

「外したマテリアを後で再利用できる可能性がある」からです。
でも **リークは許されない**。
この矛盾を解決するために **`_floor[1024]` 配列に退避** します。

```
unequip(idx) の挙動:
  _inventory[idx] を _floor に移動
  _inventory[idx] = NULL にする

Character のデストラクタ:
  _inventory と _floor の全要素を delete
  → 誰も落ちてない、全員回収
```

### なぜ `_floor[1024]` が必要？

```
問題: unequip したマテリアを誰が delete するか?

選択肢1: unequip で delete する
  → subject 禁止! "must not delete"

選択肢2: 何もしない
  → リーク (誰も delete しない)

選択肢3: _floor に退避して
        デストラクタで delete
  → OK! これが正解
```

!!! info "1024 は余裕を見た数"
    subject の spec に明確な数値はなく、十分大きな数（1024）にしておけば
    実用的にはリーク対策として十分です。

### 前方宣言って何？

**「クラスがあることだけ伝えて、中身は後で教える」** 宣言です。

```cpp
// AMateria.hpp の先頭
class ICharacter;   // 前方宣言

class AMateria {
    // ICharacter& を使うだけなら
    // 完全な定義は不要
    virtual void use(ICharacter& target);
};
```

### なぜ前方宣言が必要？

**ヘッダの循環 include** を避けるためです。

```
AMateria.hpp が ICharacter.hpp を include
ICharacter.hpp が AMateria.hpp を include
  → お互いを include し合う = 循環参照!
  → コンパイルエラー

解決:
  AMateria.hpp で ICharacter を前方宣言
  ICharacter.hpp で AMateria を前方宣言
  → include し合わない
  → ポインタ/参照としてだけ使う
```

---

## 3. 課題仕様

### クラス一覧（7クラス）

| クラス名 | 種別 | 役割 |
|---|---|---|
| **AMateria** | 抽象クラス | マテリアの基底 |
| **Ice** | 具象 | 氷マテリア |
| **Cure** | 具象 | 回復マテリア |
| **ICharacter** | インターフェース | キャラの契約 |
| **Character** | 具象 | キャラの実装 |
| **IMateriaSource** | インターフェース | 工房の契約 |
| **MateriaSource** | 具象 | 工房の実装 |

### クラス関係図

```
         ┌────────────────┐
         │  ICharacter    │ interface
         │  ────────────  │
         │  getName()  =0 │
         │  equip()    =0 │
         │  unequip()  =0 │
         │  use()      =0 │
         └────────┬───────┘
                  │ implements
         ┌────────▼───────┐
         │   Character    │
         │  ────────────  │
         │  _name         │
         │  _inventory[4] │ ── owns ──┐
         │  _floor[1024]  │ ── owns ──┤
         │  _floorCount   │           │
         └────────────────┘           │
                                      ▼
┌────────────────┐          ┌──────────────────┐
│ IMateriaSource │ intf     │    AMateria      │ abstract
│ ────────────── │          │  ──────────────  │
│ learnMateria() │=0        │  _type           │
│ createMateria()│=0        │  clone()      =0 │
└───────┬────────┘          │  use(ICharacter&)│
        │ implements        └────────┬─────────┘
┌───────▼────────┐               ┌───┴───┐
│ MateriaSource  │               │       │
│ ────────────── │          ┌────▼─┐  ┌──▼───┐
│ _learned[4]────┼──owns───→│ Ice  │  │ Cure │
└────────────────┘          └──────┘  └──────┘
```

### 各クラスの仕様

**AMateria**:

- `protected: std::string _type`
- OCF + `AMateria(std::string const& type)` (型指定コンストラクタ)
- `std::string const& getType() const`
- `virtual AMateria* clone() const = 0` — 純粋仮想
- `virtual void use(ICharacter& target)` — デフォルト実装あり

**Ice / Cure**:

- `AMateria` を継承
- OCF 完備
- `clone()` で `new Ice(*this)` / `new Cure(*this)` を返す
- `use()` で特定のメッセージを出力

**ICharacter (interface)**:

- `virtual ~ICharacter() {}`
- `virtual std::string const& getName() const = 0`
- `virtual void equip(AMateria* m) = 0`
- `virtual void unequip(int idx) = 0` **← delete 禁止**
- `virtual void use(int idx, ICharacter& target) = 0`

**Character**:

- `std::string _name`
- `AMateria* _inventory[4]` — 装備スロット
- `AMateria* _floor[1024]` — unequip で外したマテリアの退避
- OCF + `Character(std::string const& name)`
- ICharacter の全メソッド実装
- ディープコピー: `clone()` でインベントリ複製

**IMateriaSource (interface)**:

- `virtual ~IMateriaSource() {}`
- `virtual void learnMateria(AMateria*) = 0`
- `virtual AMateria* createMateria(std::string const& type) = 0`

**MateriaSource**:

- `AMateria* _learned[4]` — 学習済みの原本
- OCF 完備
- `learnMateria()`: 原本を保存（所有権を受け取る）
- `createMateria()`: 原本の `clone()` を返す（所有権は呼び出し側へ）

---

## 4. 実行例（subject 指定 main）

```console
$ make && ./materia
* shoots an ice bolt at bob *
* heals bob's wounds *
```

subject で指定されている main:

```cpp
int main(void) {
    IMateriaSource* src = new MateriaSource();
    src->learnMateria(new Ice());
    src->learnMateria(new Cure());

    ICharacter* me = new Character("me");

    AMateria* tmp;
    tmp = src->createMateria("ice");
    me->equip(tmp);
    tmp = src->createMateria("cure");
    me->equip(tmp);

    ICharacter* bob = new Character("bob");

    me->use(0, *bob);
    me->use(1, *bob);

    delete bob;
    delete me;
    delete src;

    return 0;
}
```

---

## 5. C と C++ の比較

C で同じことをやろうとすると **関数ポインタの構造体を手動で管理** する必要があります。

!!! info "インターフェース = 「全関数が関数ポインタの構造体」の言語版"
    C でインターフェースを作るには:

    1. 関数ポインタだけの構造体 (vtable) を定義
    2. 実装ごとに vtable を作って差し替える
    3. 呼び出しは `obj->vt->func(obj, ...)` のように冗長

    C++ の `class I { virtual = 0; };` はこの仕組みを
    **言語機能として組み込んだもの**。さらに `clone()` (プロトタイプパターン) は
    **C の「コピー関数ポインタを構造体に入れる」の自動化** です。

=== "C の書き方（手動 vtable）"

    ```c
    /* printf 用 */
    #include <stdio.h>
    /* malloc / free 用 */
    #include <stdlib.h>
    /* strcpy 用 */
    #include <string.h>

    /* ── 「インターフェース」 ── */
    /* 関数ポインタの集まり (手動 vtable) */
    /* C++ の class ICharacter に相当 */
    typedef struct s_icharacter_vt {
        /* self = 自分自身 (this 相当) */
        /* Cでは明示的に渡す必要あり */
        const char* (*getName)(void* self);
        void (*equip)(
            void* self, void* materia
        );
        void (*unequip)(
            void* self, int idx
        );
        void (*use)(
            void* self, int idx,
            void* target
        );
    } t_icharacter_vt;

    /* ── Character 実装 ── */
    typedef struct s_character {
        /* 手動 vtable のポインタ */
        /* C++ の virtual は自動で持つ */
        t_icharacter_vt* vt;
        /* 名前 (固定サイズ配列) */
        char name[64];
        /* インベントリ (void* で型汎用化) */
        void* inventory[4];
    } t_character;

    /* ── AMateria の C 版 ── */
    /* プロトタイプパターンも関数ポインタで */
    typedef struct s_amateria {
        /* 型名 */
        char type[32];
        /* コピー関数 = clone のC版 */
        /* 「型を知らなくても複製できる」を */
        /* 関数ポインタで表現 */
        void* (*clone)(void* self);
        /* use 関数ポインタ */
        void (*use)(
            void* self, void* target
        );
    } t_amateria;

    /* ── 所有権管理 ── */
    /* C には所有権の仕組みがないので */
    /* コメントで「takes ownership」と */
    /* 明記するしかない */
    void learnMateria(
        void* src, t_amateria* m
    ) {
        /* src が m の delete 責任を持つ */
        /* ← ルールを守るのは人間次第 */
    }
    ```

=== "C++ の書き方（virtual で自動）"

    ```cpp
    // 前方宣言:
    // ICharacter.hpp と AMateria.hpp が
    // 互いに include すると循環参照になる
    // → 「名前があること」だけ伝えて回避
    class AMateria;

    // ── 純粋インターフェース ──
    // 全メソッドが = 0、データなし
    // C の関数ポインタ構造体の進化形
    class ICharacter {
    public:
        // virtual デストラクタ必須
        // (基底* delete で派生も呼ぶため)
        virtual ~ICharacter() {}
        // const& = 読み取り専用の参照
        // (コピーを避けて速い)
        virtual std::string const& getName(
        ) const = 0;
        // AMateria* を受け取る
        // (所有権を受け取る契約)
        virtual void equip(
            AMateria* m
        ) = 0;
        virtual void unequip(
            int idx
        ) = 0;
        virtual void use(
            int idx,
            ICharacter& target
        ) = 0;
    };

    // ── 抽象基底 ──
    // データと実装を一部持てる
    // (これが抽象クラスと
    //  インターフェースの違い)
    class AMateria {
    protected:
        // 派生から書き換え可能
        std::string _type;
    public:
        // ── プロトタイプパターン ──
        // Cの「コピー関数ポインタ」を
        // virtual で言語機能化
        // 派生で必ず実装
        virtual AMateria* clone(
        ) const = 0;
        // virtual だが実装あり
        // 派生で上書き可能
        virtual void use(
            ICharacter& target
        );
    };

    // ── 具象クラス Ice ──
    class Ice : public AMateria {
    public:
        // clone 実装 (プロトタイプの核心)
        // 自分の型 (Ice) を知ってるので
        // new Ice(*this) と書ける
        // 呼び出し側は AMateria* で受け取る
        // → 型を知らずにコピーできる!
        virtual AMateria* clone(
        ) const {
            return new Ice(*this);
        }
    };
    ```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|---|---|
| 関数ポインタの構造体 | `class I { virtual = 0; };` | インターフェースを言語で表現 |
| `void*` 汎用ポインタ | 抽象クラスのポインタ | 型安全な多態性 |
| `memcpy` で手動コピー | `clone()` メソッド | プロトタイプパターン |
| 所有権はコメント | 所有権は設計で明確 | デストラクタで自動解放 |
| 循環 include の工夫 | 前方宣言 | 言語機能でスッキリ |

---

## 6. コード解説

### プログラム全体の流れ

```
1. MateriaSource を new
     → 空の _learned[4] = NULL x 4
2. learnMateria(new Ice())
     → _learned[0] = Ice (所有権 src へ)
3. learnMateria(new Cure())
     → _learned[1] = Cure
4. Character "me" を new
     → 空の _inventory, _floor
5. tmp = src->createMateria("ice")
     → _learned[0]->clone() → 新 Ice
     → tmp が所有
6. me->equip(tmp)
     → _inventory[0] = tmp
     → 所有権 me へ
7. 同様に cure も装備
8. Character "bob" を new
9. me->use(0, *bob)
     → _inventory[0]->use(*bob)
     → virtual で Ice::use が呼ばれる
     → "* shoots an ice bolt at bob *"
10. delete bob
      → 空なので何もしない
11. delete me
      → _inventory の Ice/Cure を delete
12. delete src
      → _learned の Ice/Cure を delete
```

### 所有権の流れ（図解）

```
  new Ice()
     |
     |  learnMateria(Ice)
     v
  MateriaSource._learned[0]   ← 所有: src
     |
     |  createMateria("ice")
     |  → _learned[0]->clone()
     |  → new Ice を生成
     v
  tmp (呼び出し側)             ← 所有: caller
     |
     |  equip(tmp)
     v
  Character._inventory[0]     ← 所有: character
     |
     |  unequip(0)
     v
  Character._floor[N]         ← 所有: character
     |
     |  ~Character()
     v
  delete されて終了
```

### AMateria.hpp（抽象基底）

```cpp title="AMateria.hpp" linenums="1"
#ifndef AMATERIA_HPP
#define AMATERIA_HPP

#include <string>
#include <iostream>

// 前方宣言:
// 「ICharacter というクラスがある」
// とだけ伝える
// 完全な定義はこのヘッダでは不要
// 循環 include を避けるため
class ICharacter;

// A プレフィックス = Abstract
// インスタンス化不可、継承が前提
class AMateria {
protected:
    std::string _type;

public:
    AMateria(void);
    // 型名を指定して作るコンストラクタ
    // std::string const& = 書き換えない参照
    AMateria(std::string const& type);
    AMateria(const AMateria& other);
    AMateria& operator=(
        const AMateria& other
    );
    // virtual デストラクタ (必須)
    virtual ~AMateria(void);

    std::string const& getType(void) const;
    // = 0 純粋仮想 (プロトタイプパターン)
    // 派生で必ず実装
    virtual AMateria* clone(
        void
    ) const = 0;
    // virtual だが実装あり (派生で上書き可)
    virtual void use(ICharacter& target);
};

#endif
```

### AMateria.cpp

```cpp title="AMateria.cpp" linenums="1"
#include "AMateria.hpp"
// use() 内で target.getName() を
// 呼ぶため完全定義が必要
#include "ICharacter.hpp"

AMateria::AMateria(void)
    : _type("default") {
}

// 型名を受け取るコンストラクタ
AMateria::AMateria(
    std::string const& type
) : _type(type) {
}

AMateria::AMateria(const AMateria& other)
    : _type(other._type) {
}

AMateria& AMateria::operator=(
    const AMateria& other
) {
    if (this != &other)
        _type = other._type;
    return *this;
}

AMateria::~AMateria(void) {
}

std::string const& AMateria::getType(
    void
) const {
    return _type;
}

// 基底のデフォルト use
// 派生で上書きされる
void AMateria::use(ICharacter& target) {
    // 未使用引数の警告抑制
    (void)target;
}
```

### Ice.hpp / Ice.cpp

```cpp title="Ice.hpp" linenums="1"
#ifndef ICE_HPP
#define ICE_HPP

#include "AMateria.hpp"

// AMateria を継承した具象クラス
class Ice : public AMateria {
public:
    Ice(void);
    Ice(const Ice& other);
    Ice& operator=(const Ice& other);
    virtual ~Ice(void);

    // 純粋仮想の実装
    virtual AMateria* clone(void) const;
    // 基底の use を上書き
    virtual void use(ICharacter& target);
};

#endif
```

```cpp title="Ice.cpp" linenums="1"
#include "Ice.hpp"
#include "ICharacter.hpp"

// 基底のコンストラクタに "ice" を渡す
// 初期化子リストで _type が設定される
Ice::Ice(void) : AMateria("ice") {
}

// 基底のコピーコンストラクタを明示呼び出し
Ice::Ice(const Ice& other)
    : AMateria(other) {
}

Ice& Ice::operator=(const Ice& other) {
    if (this != &other)
        // 基底の operator= を :: で呼ぶ
        AMateria::operator=(other);
    return *this;
}

Ice::~Ice(void) {
}

// プロトタイプパターンの核
// 自分自身のコピーを new で作って返す
// 呼び出し側は AMateria* で受け取れる
AMateria* Ice::clone(void) const {
    return new Ice(*this);
}

// 動的ディスパッチで呼ばれる
// target はインターフェース参照
// → getName() だけ呼べる
void Ice::use(ICharacter& target) {
    std::cout
        << "* shoots an ice bolt at "
        << target.getName()
        << " *" << std::endl;
}
```

Cure.cpp は Ice.cpp とほぼ同じ。違いは `_type = "cure"` と
`use` のメッセージが `"* heals <name>'s wounds *"` なだけ。

### ICharacter.hpp（インターフェース）

```cpp title="ICharacter.hpp" linenums="1"
#ifndef ICHARACTER_HPP
#define ICHARACTER_HPP

#include <string>

// 前方宣言
// AMateria* だけ使うので完全定義不要
class AMateria;

// I プレフィックス = Interface
// 全メソッドが = 0
// データメンバなし
class ICharacter {
public:
    // 空のインライン virtual デストラクタ
    // インターフェースの約束事:
    // 他のメソッドは = 0 だが
    // デストラクタだけは {} を書く
    virtual ~ICharacter(void) {}
    // std::string const& = コピー避けるため
    virtual std::string const& getName(
        void
    ) const = 0;
    virtual void equip(AMateria* m) = 0;
    // subject: unequip は delete 禁止!
    virtual void unequip(int idx) = 0;
    virtual void use(
        int idx,
        ICharacter& target
    ) = 0;
};

#endif
```

!!! info "インターフェースに OCF を書かない理由"
    インターフェースは **データを持たない** ので、
    コピーコンストラクタや代入演算子で「コピーすべきデータ」が存在しません。
    コンパイラのデフォルトで十分 → 明示的に書かない。
    **OCF が必要なのは `Character` のような実装クラス** の方。

### Character.hpp

```cpp title="Character.hpp" linenums="1"
#ifndef CHARACTER_HPP
#define CHARACTER_HPP

#include "ICharacter.hpp"
#include "AMateria.hpp"

// ICharacter インターフェースの実装
// subject 要件:
//   - 装備スロット 4
//   - unequip は delete しない
//   - でもリークも許されない
//   → _floor に退避して
//     デストラクタでまとめて delete
class Character : public ICharacter {
private:
    std::string _name;
    // 装備中のマテリア (4 スロット)
    AMateria*   _inventory[4];
    // unequip で外したマテリアの退避場所
    // 所有権はここにある
    AMateria*   _floor[1024];
    int         _floorCount;

public:
    Character(void);
    Character(std::string const& name);
    Character(const Character& other);
    Character& operator=(
        const Character& other
    );
    virtual ~Character(void);

    // インターフェースの純粋仮想関数を
    // 全て実装する必要がある
    virtual std::string const& getName(
        void
    ) const;
    virtual void equip(AMateria* m);
    virtual void unequip(int idx);
    virtual void use(
        int idx,
        ICharacter& target
    );
};

#endif
```

### Character.cpp（最大のコード）

```cpp title="Character.cpp (default constructor)" linenums="1"
// デフォルトコンストラクタ
// 初期化子リストで _name と _floorCount
// 本体で配列を NULL 埋め
Character::Character(void)
    : _name("unnamed"), _floorCount(0) {
    // NULL = ポインタが何も指してない値
    // (C++98 では NULL / 0 を使う)
    for (int i = 0; i < 4; i++)
        _inventory[i] = NULL;
    for (int i = 0; i < 1024; i++)
        _floor[i] = NULL;
}

// 名前指定コンストラクタ
Character::Character(
    std::string const& name
) : _name(name), _floorCount(0) {
    for (int i = 0; i < 4; i++)
        _inventory[i] = NULL;
    for (int i = 0; i < 1024; i++)
        _floor[i] = NULL;
}
```

```cpp title="Character.cpp (copy constructor)" linenums="1"
// ディープコピー版コピーコンストラクタ
// AMateria は抽象クラスなので
// new AMateria(*p) と書けない
// → clone() でディープコピー
//    (プロトタイプパターンの真価)
Character::Character(
    const Character& other
) : _name(other._name), _floorCount(0) {
    for (int i = 0; i < 4; i++) {
        if (other._inventory[i])
            // clone() で型を気にせずコピー
            _inventory[i]
                = other._inventory[i]->clone();
        else
            _inventory[i] = NULL;
    }
    // _floor は空で初期化
    // (元の _floor はコピーしない)
    for (int i = 0; i < 1024; i++)
        _floor[i] = NULL;
}
```

```cpp title="Character.cpp (operator=)" linenums="1"
// 代入演算子もディープコピー
Character& Character::operator=(
    const Character& other
) {
    if (this != &other) {   // 自己代入防止
        _name = other._name;
        for (int i = 0; i < 4; i++) {
            // 旧 _inventory を解放
            // (NULL への delete は安全)
            delete _inventory[i];
            if (other._inventory[i])
                // clone でディープコピー
                _inventory[i]
                    = other._inventory[i]
                          ->clone();
            else
                _inventory[i] = NULL;
        }
        // 旧 _floor も解放してリセット
        for (int i = 0; i < _floorCount; i++)
            delete _floor[i];
        _floorCount = 0;
    }
    return *this;
}
```

```cpp title="Character.cpp (destructor)" linenums="1"
// デストラクタ:
// 所有している全マテリアを解放
Character::~Character(void) {
    // 装備中のマテリアを全解放
    for (int i = 0; i < 4; i++)
        delete _inventory[i];
    // 退避したマテリアも全解放
    for (int i = 0; i < _floorCount; i++)
        delete _floor[i];
}

// 名前を返す
std::string const& Character::getName(
    void
) const {
    return _name;
}
```

```cpp title="Character.cpp (equip)" linenums="1"
// equip: マテリアを装備する
// 渡された m の所有権が Character に移る
// → 以後、Character が delete 責任を負う
void Character::equip(AMateria* m) {
    if (!m)          // NULL チェック
        return;
    // 空きスロットを探す (先頭から)
    for (int i = 0; i < 4; i++) {
        if (!_inventory[i]) {
            _inventory[i] = m;
            return;
        }
    }
    // 空きがない場合
    // (subject には仕様が明示されてないが
    //  リーク防止のため _floor に退避)
    if (_floorCount < 1024)
        _floor[_floorCount++] = m;
    else
        delete m;  // 最終手段
}
```

```cpp title="Character.cpp (unequip)" linenums="1"
// unequip: マテリアを外す
// subject 要件: delete してはいけない!
// → _floor に退避して後でまとめて delete
void Character::unequip(int idx) {
    if (idx < 0 || idx >= 4
        || !_inventory[idx])
        return;
    // _floor に退避
    if (_floorCount < 1024) {
        _floor[_floorCount]
            = _inventory[idx];
        _floorCount++;
    } else {
        // _floor が溢れたら仕方なく delete
        delete _inventory[idx];
    }
    // スロットは空にする
    _inventory[idx] = NULL;
}
```

```cpp title="Character.cpp (use)" linenums="1"
// use: スロットのマテリアを使う
void Character::use(
    int idx, ICharacter& target
) {
    if (idx < 0 || idx >= 4
        || !_inventory[idx])
        return;
    // virtual なので動的ディスパッチ
    // → Ice なら Ice::use
    // → Cure なら Cure::use
    _inventory[idx]->use(target);
}
```

### IMateriaSource.hpp

```cpp title="IMateriaSource.hpp" linenums="1"
#ifndef IMATERIASOURCE_HPP
#define IMATERIASOURCE_HPP

#include "AMateria.hpp"

// I プレフィックス = Interface
// 全メソッドが = 0
class IMateriaSource {
public:
    // virtual デストラクタ (必須)
    virtual ~IMateriaSource(void) {}
    // learnMateria:
    //   渡されたマテリアの所有権を受け取る
    //   → src が delete 責任を持つ
    virtual void learnMateria(
        AMateria*
    ) = 0;
    // createMateria:
    //   clone() で新マテリアを生成して返す
    //   → 呼び出し側が delete 責任を持つ
    virtual AMateria* createMateria(
        std::string const& type
    ) = 0;
};

#endif
```

### MateriaSource.hpp / MateriaSource.cpp

```cpp title="MateriaSource.hpp" linenums="1"
#ifndef MATERIASOURCE_HPP
#define MATERIASOURCE_HPP

#include "IMateriaSource.hpp"

class MateriaSource : public IMateriaSource {
private:
    // 学習中のマテリア原本 (4 つまで)
    AMateria* _learned[4];

public:
    MateriaSource(void);
    MateriaSource(const MateriaSource& other);
    MateriaSource& operator=(
        const MateriaSource& other
    );
    virtual ~MateriaSource(void);

    virtual void learnMateria(AMateria* m);
    virtual AMateria* createMateria(
        std::string const& type
    );
};

#endif
```

```cpp title="MateriaSource.cpp" linenums="1"
#include "MateriaSource.hpp"

MateriaSource::MateriaSource(void) {
    // 空で初期化
    for (int i = 0; i < 4; i++)
        _learned[i] = NULL;
}

// ディープコピー (clone() を使う)
MateriaSource::MateriaSource(
    const MateriaSource& other
) {
    for (int i = 0; i < 4; i++) {
        if (other._learned[i])
            // 抽象クラスなので clone()
            _learned[i]
                = other._learned[i]->clone();
        else
            _learned[i] = NULL;
    }
}

MateriaSource& MateriaSource::operator=(
    const MateriaSource& other
) {
    if (this != &other) {
        for (int i = 0; i < 4; i++) {
            delete _learned[i];
            if (other._learned[i])
                _learned[i]
                    = other._learned[i]
                          ->clone();
            else
                _learned[i] = NULL;
        }
    }
    return *this;
}

// 所有している全原本を解放
MateriaSource::~MateriaSource(void) {
    for (int i = 0; i < 4; i++)
        delete _learned[i];
}

// learnMateria:
// 渡されたポインタの所有権を受け取る
void MateriaSource::learnMateria(
    AMateria* m
) {
    if (!m)
        return;
    for (int i = 0; i < 4; i++) {
        if (!_learned[i]) {
            _learned[i] = m;  // 原本を保存
            return;
        }
    }
    // 空きがない場合
    // リーク防止のため delete
    delete m;
}

// createMateria:
// 型名にマッチする原本の clone を返す
// 原本はそのまま保持 (コピーを渡すだけ)
// 呼び出し側が delete 責任を持つ
AMateria* MateriaSource::createMateria(
    std::string const& type
) {
    for (int i = 0; i < 4; i++) {
        if (_learned[i]
            && _learned[i]->getType()
               == type)
            // プロトタイプパターン
            return _learned[i]->clone();
    }
    return 0;   // 見つからなければ NULL
}
```

### main.cpp（subject 指定）

```cpp title="main.cpp" linenums="1"
#include "AMateria.hpp"
#include "Ice.hpp"
#include "Cure.hpp"
#include "ICharacter.hpp"
#include "Character.hpp"
#include "IMateriaSource.hpp"
#include "MateriaSource.hpp"

int main(void) {
    // IMateriaSource* で持つ
    // → 実装クラス MateriaSource を知らなくても
    //    インターフェース経由で操作できる
    IMateriaSource* src
        = new MateriaSource();

    // learn すると所有権が src に移る
    // → main 側は以後このポインタを触らない
    src->learnMateria(new Ice());
    src->learnMateria(new Cure());

    ICharacter* me = new Character("me");

    AMateria* tmp;
    // createMateria は新しい Ice を返す
    // 所有権は呼び出し側 (tmp)
    tmp = src->createMateria("ice");
    // equip で所有権が me に移る
    me->equip(tmp);

    tmp = src->createMateria("cure");
    me->equip(tmp);

    ICharacter* bob = new Character("bob");

    // virtual use() のおかげで
    // Ice::use / Cure::use が呼ばれる
    // *bob は ICharacter& として渡される
    me->use(0, *bob);  // ice bolt!
    me->use(1, *bob);  // heals!

    // インターフェース* 経由 delete
    // virtual デストラクタで
    // 実装クラスのデストラクタが呼ばれ
    // 中のマテリアも全部解放される
    delete bob;
    delete me;
    delete src;

    return 0;
}
```

### 所有権テーブル（まとめ）

| 操作 | 生成者 | 所有者（終的） | 解放者 |
|---|---|---|---|
| `new Ice()` 直後 | main | main（一時的） | — |
| `learnMateria(ice)` | — | MateriaSource | `~MateriaSource()` |
| `createMateria("ice")` | MateriaSource（clone） | main（一時的） | → equip で Character へ |
| `equip(tmp)` | — | Character | `~Character()` |
| `unequip(idx)` | — | Character（`_floor`） | `~Character()` |

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Implement the following concrete classes in the
    > canonical form: Ice, Cure. ... Write the orthodox
    > canonical form for Character and MateriaSource."
    >
    > "If any class is not in OCF (except the interfaces),
    > the exercise is graded 0."

    **OCF の徹底** と **メモリ管理** が評価の核心。

- [ ] `make` がエラー/警告なく通る
- [ ] subject の main が期待通りに動作する
  - [ ] `* shoots an ice bolt at bob *`
  - [ ] `* heals bob's wounds *`
- [ ] OCF が以下のクラスに揃っている
  - [ ] AMateria
  - [ ] Ice
  - [ ] Cure
  - [ ] Character
  - [ ] MateriaSource
- [ ] インターフェース (ICharacter, IMateriaSource) は OCF なし（正しい）
- [ ] インターフェースに virtual デストラクタがある
- [ ] メモリリークなし (valgrind / leaks で確認)
- [ ] `AMateria::clone()` が `= 0`
- [ ] `unequip()` が delete してない

---

## 8. テストチェックリスト

### インターフェース

- [ ] `ICharacter`: 全メソッド `= 0`、virtual デストラクタあり、データなし
- [ ] `IMateriaSource`: 同上
- [ ] インターフェースに OCF を書いてない（正しい）

### AMateria

- [ ] `clone()` が `= 0`
- [ ] `_type` が `protected`
- [ ] OCF 4 関数
- [ ] `AMateria(std::string const& type)` コンストラクタあり

### Ice / Cure

- [ ] `clone()` が `new Ice(*this)` / `new Cure(*this)`
- [ ] `use()` が正しいメッセージ
- [ ] OCF 4 関数

### Character

- [ ] `_inventory[4]` で管理
- [ ] `_floor[1024]` で unequip 退避
- [ ] `equip()`: 空きスロットに装備、満杯なら `_floor` へ
- [ ] `unequip()`: **delete せず** スロットを NULL に
- [ ] `use()`: 正しいスロットのマテリアを使う
- [ ] ディープコピー: `clone()` でインベントリ複製
- [ ] OCF 4 関数

### MateriaSource

- [ ] `learnMateria()`: 原本を保存 (4 つまで)
- [ ] `createMateria()`: 原本の `clone()` を返す
- [ ] 型名が合わなければ NULL を返す
- [ ] OCF 4 関数

### メモリ管理

- [ ] subject の main でメモリリークなし
- [ ] equip/unequip を繰り返してもリークなし
- [ ] Character のコピー後に delete してもリークなし
- [ ] MateriaSource のコピー後に delete してもリークなし
- [ ] 無効インデックス `use(-1, ...)` でクラッシュしない

### 規約

- [ ] `malloc` / `free` / `printf` 不使用
- [ ] `using namespace std;` なし
- [ ] インターフェース以外の全クラスが OCF 完備

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|---|---|
| インターフェースとは？ | 全メソッドが `= 0` でデータを持たない抽象クラス。I-prefix が慣習 |
| 抽象クラスとの違いは？ | 抽象クラスはデータや実装メソッドを持てるが、インターフェースは持たない |
| プロトタイプパターンとは？ | `clone()` メソッドでオブジェクト自身のコピーを作るパターン |
| なぜ clone() が必要？ | AMateria は抽象なので `new AMateria(*p)` できない。派生が自分の型を知ってるので clone で解決 |
| 所有権とは？ | 「誰が new して、誰が delete する責任を負うか」のルール |
| `learnMateria` の所有権は？ | 引数で受け取った AMateria* の所有権が source に移る |
| `createMateria` の所有権は？ | 戻り値の所有権は呼び出し側に移る |
| `unequip` で delete しない理由は？ | subject の明確な要件。外したマテリアを後で使える可能性を残す |
| _floor[1024] の目的は？ | unequip で外したマテリアの退避場所。リーク防止のためデストラクタでまとめて delete |
| 前方宣言とは？ | `class ICharacter;` のように宣言だけ。ヘッダの循環 include を避けるため |
| なぜ ICharacter に OCF がない？ | データを持たないので、コピーすべき状態がない。コンパイラのデフォルトで十分 |
| なぜ ICharacter に virtual デストラクタがある？ | `ICharacter*` 経由 delete で派生のデストラクタを呼ぶため |
| vtable とは？ | virtual 関数の関数ポインタの表。コンパイラが自動生成 |
| equip で空きがないとどうする？ | subject 明記なしだが、この実装では `_floor` に退避してリーク防止 |
| 二重 equip（同じポインタを2度）を防げるか？ | コード上は防げない。呼び出し側の責任（subject 前提）|

---

## 10. よくあるミス

!!! warning "メモリリーク: unequip で放置"
    ```cpp
    me->equip(tmp);
    me->unequip(0);
    // tmp はどこへ?
    // _floor に退避されていれば OK
    // 誰も管理してないならリーク
    ```

    解決策: `_floor[1024]` に退避し、デストラクタで delete する。

!!! warning "所有権の混乱: 同じマテリアを2回 equip"
    ```cpp
    AMateria* tmp
        = src->createMateria("ice");
    me->equip(tmp);
    me->equip(tmp);
    // 同じポインタが2つのスロットに入る
    // → 二重 delete でクラッシュ
    ```

!!! warning "createMateria の戻り値を放置"
    ```cpp
    src->createMateria("ice");
    // 戻り値を誰も受け取らない
    // → リーク
    ```

!!! warning "コピーコンストラクタで clone() を使い忘れ"
    ```cpp
    Character::Character(
        const Character& other
    ) {
        for (int i = 0; i < 4; i++)
            // NG: ポインタだけコピー
            _inventory[i]
                = other._inventory[i];
    }
    // → 二重 delete
    //
    // OK: clone() を使う
    _inventory[i]
        = other._inventory[i]->clone();
    ```

!!! warning "インターフェースに OCF を書く"
    ```cpp
    // NG: インターフェースに OCF は不要
    class ICharacter {
        ICharacter(void);
        ICharacter(const ICharacter&);
        ICharacter& operator=(
            const ICharacter&
        );
        virtual ~ICharacter();
    };
    // 状態がないので意味がなく設計として不適切
    ```

!!! warning "AMateria / Character / MateriaSource の OCF を省略"
    ```cpp
    // NG: OCF 省略 → 自動で 0 点!
    class MateriaSource : public IMateriaSource {
        // コンストラクタと learnMateria だけ
        // コピーコンストラクタ/代入/デストラクタなし
    };
    ```

!!! warning "前方宣言を忘れて循環 include"
    ```cpp
    // AMateria.hpp
    #include "ICharacter.hpp"  // NG
    // ICharacter.hpp
    #include "AMateria.hpp"    // 循環!
    //
    // 解決: 前方宣言
    // AMateria.hpp
    class ICharacter;  // 前方宣言
    ```

!!! warning "virtual デストラクタがない"
    ```cpp
    class ICharacter {
        ~ICharacter();  // virtual なし!
    };
    ICharacter* c = new Character("x");
    delete c;
    // → Character::~Character が呼ばれない
    // → 内部のマテリアが全部リーク!
    ```

!!! warning "unequip で delete してしまう"
    ```cpp
    void Character::unequip(int idx) {
        delete _inventory[idx];  // NG!
        _inventory[idx] = NULL;
    }
    // subject: "must not delete"
    ```

!!! warning "equip で NULL チェックしない"
    ```cpp
    void Character::equip(AMateria* m) {
        // NG: NULL チェックなし
        _inventory[0] = m;
        // m が NULL でも入ってしまう
    }
    ```

!!! warning "clone() を自分の型で書かない"
    ```cpp
    // NG: AMateria* で new しようとしても
    //     抽象クラスだから new できない
    AMateria* Ice::clone(void) const {
        return new AMateria(*this);  // NG
    }

    // OK: 自分の型を使う
    AMateria* Ice::clone(void) const {
        return new Ice(*this);
    }
    ```

---

## 11. 次のステップ

cpp04 モジュールはこれで完了!

cpp00〜04 で学んだことを総復習してから評価に臨みましょう:

- cpp00: 基本、クラス、I/O
- cpp01: new/delete、参照、ポインタ
- cpp02: 演算子オーバーロード、Fixed-point
- cpp03: 継承
- cpp04: ポリモーフィズム、抽象、インターフェース

詳しい Q&A と模範回答は [評価対策](eval.md) ページへ。
