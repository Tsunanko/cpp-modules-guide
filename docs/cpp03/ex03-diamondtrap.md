# ex03 — DiamondTrap

---

## このプログラムは何？

**ScavTrap と FragTrap の両方** を親に持つ DiamondTrap を作るプログラム。

DiamondTrap は ScavTrap の能力（警備モード）と
FragTrap の能力（ハイタッチ）を **両方** 使えるスーパーロボット。

```
             ClapTrap（共通の祖先）
            /        \
        ScavTrap   FragTrap    ← 両方が ClapTrap を継承
            \        /
           DiamondTrap          ← 両方を同時に継承
```

形がひし形（ダイヤモンド）なので **ダイヤモンド継承** と呼ばれます。
ただし、単純に両方を継承すると **ダイヤモンド問題** という落とし穴にハマります。
これを **virtual 継承** で解決するのが、この exercise の核心。

これは cpp モジュール全体でもトップクラスの難所です。
焦らずゆっくり読んでください。

---

## 1. この exercise で学ぶこと

- **多重継承** — 2つ以上の親クラスを同時に継承する
- **ダイヤモンド問題** — 共通の祖先が 2 つできてしまう問題
- **virtual 継承** — ダイヤモンド問題を解決する C++ の仕組み
- **using 宣言** — 曖昧なメソッドを明示的に選ぶ
- **スコープ解決演算子** — `_name` vs `ClapTrap::_name`

---

## 2. 新しい概念の解説

### 多重継承って何？

**2つ以上の親クラスを同時に継承すること**。

```cpp
// 単一継承（これまでの ex01/ex02）
class ScavTrap : public ClapTrap { };

// 多重継承（今回の ex03）
class DiamondTrap :
    public FragTrap,    // ← 親1つ目
    public ScavTrap     // ← 親2つ目
{
};
```

カンマ `,` で区切って複数の親を指定します。
これで DiamondTrap は FragTrap と ScavTrap の **両方** の機能を持ちます。

```
DiamondTrap d("Diamond");
d.attack("Enemy");        // ScavTrap の機能
d.guardGate();            // ScavTrap の機能
d.highFivesGuys();        // FragTrap の機能
d.takeDamage(10);         // ClapTrap の機能（両親から継承）
d.beRepaired(5);          // ClapTrap の機能（両親から継承）
```

### ダイヤモンド問題って何？

**2つの親が同じ親を持つと、孫の中に共通の祖先が 2 つできてしまう問題**。

DiamondTrap の継承関係を描くと:

```
          ClapTrap         ← 共通の祖先
         /        \
     ScavTrap   FragTrap    ← どちらも ClapTrap を継承
         \        /
        DiamondTrap         ← 両方を継承
```

virtual なしで継承すると、DiamondTrap の中はこんな状態:

```
DiamondTrap のメモリ構造（virtual なしの場合）:

  ┌─────────────────────┐
  │ ClapTrap (1つ目)    │ ← ScavTrap 経由で入ってきた
  │  _name              │
  │  _hitPoints         │
  │  _energyPoints      │
  │  _attackDamage      │
  ├─────────────────────┤
  │ ClapTrap (2つ目)    │ ← FragTrap 経由で入ってきた
  │  _name              │    同じ変数が2個！
  │  _hitPoints         │
  │  _energyPoints      │
  │  _attackDamage      │
  └─────────────────────┘

  → _name を読もうとすると
    「どっちの _name？」で曖昧
  → コンパイルエラー
```

**同じ `_name` が2つある** ので、プログラムは「どっちを使うの？」となって
エラーになります。これがダイヤモンド問題。

### `virtual` 継承って何？

**ダイヤモンド問題を解決する C++ の仕組み**。
親クラスの継承に `virtual` を付けると、共通の祖先を **1つに統合** できます。

```cpp
// ScavTrap.hpp
class ScavTrap : virtual public ClapTrap { };
//               ^^^^^^^
//               これを追加！

// FragTrap.hpp
class FragTrap : virtual public ClapTrap { };
//               ^^^^^^^
//               これも追加！
```

こうすると DiamondTrap の中はこうなります:

```
DiamondTrap のメモリ構造（virtual ありの場合）:

  ┌─────────────────────┐
  │ ClapTrap（1つだけ） │ ← ScavTrap と FragTrap が共有
  │  _name              │
  │  _hitPoints         │
  │  _energyPoints      │
  │  _attackDamage      │
  └─────────────────────┘
  ┌─────────────────────┐
  │ ScavTrap 部分       │
  ├─────────────────────┤
  │ FragTrap 部分       │
  └─────────────────────┘
```

ClapTrap が **1つだけ** になったので、`_name` を読んでも曖昧にならない！

!!! warning "両方の親に virtual が必要"
    片方だけ virtual にしても ClapTrap は 1 つに統合されません。
    ScavTrap と FragTrap の **両方** に virtual を付ける必要があります。

### なぜ DiamondTrap から ClapTrap を直接呼ぶ必要がある？

**virtual 基底クラスは「最も派生したクラス」から初期化する** のが C++ のルール。

普通の継承なら、子のコンストラクタが親を呼べば済みます。

```cpp
// 普通の継承（virtual なし）
class ScavTrap : public ClapTrap {
public:
    ScavTrap(const std::string& name)
        : ClapTrap(name) {}   // ← ScavTrap が親を呼ぶ
};
```

でも virtual 継承の場合、共通の祖先は **最派生クラス（DiamondTrap）** が直接初期化します。

```cpp
// virtual 継承の場合
// ScavTrap の初期化リストの ClapTrap() は無視される！
// DiamondTrap が直接 ClapTrap を呼ぶ

DiamondTrap::DiamondTrap(const std::string& name)
    : ClapTrap(name + "_clap_name"),   // ← 最派生が直接！
      FragTrap(name),
      ScavTrap(name),
      _name(name)
{
    // ...
}
```

**なぜ？** → 共通の祖先（ClapTrap）は1つだけ。
ScavTrap と FragTrap のどっちが ClapTrap を初期化すべきか曖昧になるので、
一番下の DiamondTrap が責任を持って初期化する、というルール。

### `using ScavTrap::attack;` って何？

**曖昧な候補の中から1つを選ぶ宣言**。

DiamondTrap は ScavTrap と FragTrap 経由で attack を継承しています。
ScavTrap::attack と、FragTrap 経由の ClapTrap::attack がある状態。

```
DiamondTrap が持つ attack の候補:
  - ScavTrap::attack（ScavTrap がオーバーライドした版）
  - ClapTrap::attack（FragTrap はオーバーライドしないので親のまま）
```

どれを使うか指定しないとコンパイラは「どっちの attack？」となって曖昧エラー。

```cpp
class DiamondTrap :
    public FragTrap, public ScavTrap
{
public:
    // ScavTrap 版の attack を使うと宣言
    using ScavTrap::attack;   // ← これ！
};
```

これで `diamond.attack("Enemy")` は **ScavTrap の attack** が呼ばれるようになります。

### スコープ解決演算子 `ClapTrap::_name` って何？

**「どのクラスの `_name` か」を明示的に指定する書き方**。

DiamondTrap には同じ名前のメンバが2つあります:

```cpp
class DiamondTrap : public FragTrap, public ScavTrap {
private:
    std::string _name;   // DiamondTrap 独自の _name
    // ClapTrap からも _name を継承している
};
```

```cpp
_name            → DiamondTrap::_name（近いスコープが優先）
ClapTrap::_name  → 親 ClapTrap の _name（スコープ解決で明示）
```

同じ名前だけどスコープが違うので、**共存できます**。
`whoAmI()` ではこの2つを両方表示します。

```cpp
void DiamondTrap::whoAmI(void) {
    std::cout << "I am DiamondTrap " << _name
        // ↑ DiamondTrap::_name（自分の名前）
        << " and my ClapTrap name is "
        << ClapTrap::_name << std::endl;
        // ↑ ClapTrap::_name（親の名前）
}
```

---

## 3. 課題仕様

### DiamondTrap クラス

- **`FragTrap` と `ScavTrap` を public 継承**（多重継承）
- ScavTrap と FragTrap を `virtual public ClapTrap` に変更

### ステータス

| 属性 | 値の出所 |
|------|----------|
| `_hitPoints` | **FragTrap** (100) |
| `_energyPoints` | **ScavTrap** (50) |
| `_attackDamage` | **FragTrap** (30) |

### 特殊仕様

- **DiamondTrap 独自の `_name`** を持つ（ClapTrap::_name とは別の変数）
- ClapTrap::_name は `<name>_clap_name` に設定（例: "Diamond_clap_name"）
- **`using ScavTrap::attack;`** で ScavTrap の attack を使用
- **`whoAmI()`** — DiamondTrap::_name と ClapTrap::_name の両方を表示
- コンストラクタ/デストラクタで DiamondTrap 固有のメッセージを出力
- OCF を全て実装

---

## 4. 実行例

```console
$ ./DiamondTrap
=== Creating DiamondTrap ===
ClapTrap constructor called for Diamond_clap_name    ← (1) virtual 基底が最初
FragTrap constructor called for Diamond              ← (2) 宣言順: FragTrap が先
ScavTrap constructor called for Diamond              ← (3) 宣言順: ScavTrap が次
DiamondTrap constructor called for Diamond           ← (4) 自分自身が最後

=== DiamondTrap actions ===
ScavTrap Diamond attacks Enemy, causing 30 points of damage!
ClapTrap Diamond_clap_name takes 25 points of damage! HP: 75
ClapTrap Diamond_clap_name repairs itself for 10 hit points! HP: 85

=== Special abilities ===
ScavTrap Diamond is now in Gate keeper mode.
FragTrap Diamond requests a positive high five!
I am DiamondTrap Diamond and my ClapTrap name is Diamond_clap_name

=== Destruction ===
DiamondTrap destructor called for Diamond            ← (4) 自分自身が最初
ScavTrap destructor called for Diamond               ← (3) 構築の逆順
FragTrap destructor called for Diamond               ← (2)
ClapTrap destructor called for Diamond_clap_name     ← (1) virtual 基底が最後
```

**注目ポイント**:

- **構築順**: ClapTrap → FragTrap → ScavTrap → DiamondTrap
  （virtual 基底 → 宣言順 → 自身）
- **破棄順**: DiamondTrap → ScavTrap → FragTrap → ClapTrap
  （構築の完全な逆順）
- attack で "ScavTrap" と表示される（using 宣言の効果）
- whoAmI で 2 つの異なる名前が表示される
- takeDamage/beRepaired は "ClapTrap Diamond_clap_name" と表示（ClapTrap::_name を使う）

---

## 5. C と C++ の比較

C言語には **多重継承に相当する機能がない**。
C で無理に再現しようとすると:

### 多重継承とダイヤモンド問題って C で言うと何？

**C で 2 つの親構造体を両方埋め込むと、共通の祖先が
2 つになってしまう問題** です。

C では解決できないので、**「1 つの場所にだけ親を埋め込む」
独自ルール** を手動で守るか、別の設計にするしかありません。

C++ の `virtual` 継承は **コンパイラが自動で統合してくれる**
仕組みです。

=== "C で擬似多重継承 (問題発生)"

    ```c
    /* 共通の祖先 */
    typedef struct s_clap {
        char name[50];
        unsigned int hp;
    } t_clap;

    /* 子1: ScavTrap は ClapTrap を埋め込む */
    typedef struct s_scav {
        t_clap parent;  /* ← ClapTrap その1 */
    } t_scav;

    /* 子2: FragTrap も ClapTrap を埋め込む */
    typedef struct s_frag {
        t_clap parent;  /* ← ClapTrap その2 */
    } t_frag;

    /* ── 孫: DiamondTrap (2 つの親を埋め込む) ── */
    typedef struct s_diamond {
        /* ScavTrap 部分 (中に ClapTrap を含む) */
        t_scav scav;
        /* FragTrap 部分 (中にも ClapTrap を含む) */
        t_frag frag;
    } t_diamond;

    /* 問題: ClapTrap が 2 回登場する！ */
    /* d.scav.parent.name と d.frag.parent.name で */
    /* 名前が 2 つ、HP も 2 つ存在してしまう */
    /* → どちらを使うかを毎回決めないといけない */
    /* → 同期漏れで不整合バグが起きる */
    ```

=== "C++ の virtual 継承 (解決)"

    ```cpp
    // ── 子1: virtual 付きで継承 ──
    // virtual = ClapTrap を「共有する意思」を宣言
    class ScavTrap :
        virtual public ClapTrap { };

    // ── 子2: 同じく virtual 付き ──
    // 両方に virtual が必要 (片方だけではダメ)
    class FragTrap :
        virtual public ClapTrap { };

    // ── 孫: 多重継承 ──
    // カンマ区切りで 2 つの親を指定
    class DiamondTrap :
        public FragTrap,
        public ScavTrap
    {
        // ClapTrap は 1 つだけになる
        // 曖昧さはコンパイラレベルで解決
        // _name は 1 つしか存在しない
    };
    ```

C++ の virtual 継承はこの「2 つの ClapTrap 問題」をコンパイラレベルで解決します。

### virtual 継承って C で言うと何？

**C で「親構造体の重複を防ぐ」ために設計を工夫する必要が
あったものを、コンパイラが自動で統合する** 仕組みです。

```
C の場合:
  方法1: DiamondTrap に親を 1 つだけ埋め込む
         → 兄弟に渡す時に設計が複雑化
  方法2: ClapTrap へのポインタを共有
         → メモリ確保/解放を手動管理
  方法3: 「重複を受け入れて、使う時だけ気をつける」
         → バグの温床

C++ の場合:
  ScavTrap : virtual public ClapTrap
  FragTrap : virtual public ClapTrap
  → コンパイラが自動で「ClapTrap は 1 つだけ」にする
```

### using 宣言って C で言うと何？

**「同じ名前の関数が複数あるとき、どれを使うかを宣言」**
する仕組みです。

C では同名関数を複数作れないので、この問題自体が発生しません
(関数名を `scav_attack`, `frag_attack` のように区別する)。

C++ の多重継承では、親両方が `attack()` を持っていると
「どっちの attack?」で曖昧になるので、`using` で指定します:

```cpp
// ScavTrap も ClapTrap (FragTrap 経由) も attack を持つ
// → どっちを使うか曖昧 → コンパイルエラー

class DiamondTrap : public FragTrap, public ScavTrap {
public:
    // ── ScavTrap 版の attack を採用すると宣言 ──
    // これで diamond.attack() は ScavTrap::attack が呼ばれる
    using ScavTrap::attack;
};
```

| C で言うと | C++ の using 宣言 |
|-----------|------------------|
| `scav_attack()` を呼ぶ | `using ScavTrap::attack;` |
| 関数名で区別する文化 | 同名でも選べる仕組み |

---

## 6. コード解説

### ダイヤモンド問題のビジュアル

```
  ★ virtual 継承なしの場合（問題発生）★

     ClapTrap(1)    ClapTrap(2)  ← ClapTrap が 2 つ！
         |               |
     ScavTrap        FragTrap
         \              /
          DiamondTrap

  → _name が 2 つ、_hitPoints が 2 つ...
  → どちらの _name を使う？
  → コンパイルエラー（曖昧）
```

```
  ★ virtual 継承ありの場合（解決）★

        ClapTrap              ← 1 つだけ（virtual 基底）
       /        \
   ScavTrap   FragTrap        ← どちらも virtual public ClapTrap
       \        /
      DiamondTrap

  → _name は 1 つだけ
  → 曖昧さがない
```

### ScavTrap.hpp（virtual 追加）

```cpp title="ScavTrap.hpp (ex03 版)" linenums="1"
#ifndef SCAVTRAP_HPP
# define SCAVTRAP_HPP

# include "ClapTrap.hpp"

// ── ここが変更点 ──
// ex01: public ClapTrap
// ex03: virtual public ClapTrap
class ScavTrap :
    virtual public ClapTrap          // (1)
{
public:
    ScavTrap(void);
    ScavTrap(const std::string& name);
    ScavTrap(const ScavTrap& other);
    ScavTrap& operator=(
        const ScavTrap& other);
    ~ScavTrap(void);

    void attack(
        const std::string& target);
    void guardGate(void);
};

#endif
```

1. **`virtual public ClapTrap`** —
   通常の `public ClapTrap` に `virtual` を追加。
   これにより ClapTrap が **仮想基底クラス** になる。

### FragTrap.hpp（virtual 追加）

```cpp title="FragTrap.hpp (ex03 版)" linenums="1"
#ifndef FRAGTRAP_HPP
# define FRAGTRAP_HPP

# include "ClapTrap.hpp"

// ── ここが変更点 ──
// ex02: public ClapTrap
// ex03: virtual public ClapTrap
class FragTrap :
    virtual public ClapTrap          // (1)
{
public:
    FragTrap(void);
    FragTrap(const std::string& name);
    FragTrap(const FragTrap& other);
    FragTrap& operator=(
        const FragTrap& other);
    ~FragTrap(void);

    void highFivesGuys(void);
};

#endif
```

1. **`virtual public ClapTrap`** — ScavTrap と同じく virtual 継承。
   **両方** virtual でないとダイヤモンド問題は解決しない。

### DiamondTrap.hpp

```cpp title="DiamondTrap.hpp" linenums="1"
#ifndef DIAMONDTRAP_HPP
# define DIAMONDTRAP_HPP

# include "ScavTrap.hpp"
# include "FragTrap.hpp"

// ── 多重継承 ──
// カンマ区切りで複数の親を指定
class DiamondTrap :
    public FragTrap,                 // (1)
    public ScavTrap
{
public:
    DiamondTrap(void);
    DiamondTrap(
        const std::string& name);
    DiamondTrap(
        const DiamondTrap& other);
    DiamondTrap& operator=(
        const DiamondTrap& other);
    ~DiamondTrap(void);

    // ── using 宣言 ──
    // 両親で曖昧になる attack を
    // ScavTrap 版に統一
    using ScavTrap::attack;           // (2)

    void whoAmI(void);

private:
    // DiamondTrap 独自の _name
    // ClapTrap::_name とは別の変数
    std::string _name;                // (3)
};

#endif
```

1. **`public FragTrap, public ScavTrap`** — カンマ区切りで複数の親を指定。
   宣言順は **FragTrap → ScavTrap**（この順で構築される）。
2. **`using ScavTrap::attack;`** — 両親に attack が存在して曖昧になるため、
   ScavTrap 版を使うと明示。
3. **DiamondTrap 独自の `_name`** — ClapTrap::_name とは **別の変数**。
   同名だがスコープが異なるので共存できる。

### DiamondTrap.cpp（コンストラクタ）

```cpp title="DiamondTrap.cpp (コンストラクタ)" linenums="1"
#include "DiamondTrap.hpp"
#include <iostream>

// ── デフォルトコンストラクタ ──
// virtual 基底の ClapTrap を
// 最派生クラス（DiamondTrap）が直接呼ぶ！
DiamondTrap::DiamondTrap(void)
    : ClapTrap("default_clap_name"), // (1)
      FragTrap(),
      ScavTrap(),
      _name("default")
{
    // 課題仕様のステータスに上書き:
    // HP = FragTrap (100)
    // EP = ScavTrap (50)
    // AD = FragTrap (30)
    _hitPoints = 100;                 // (2)
    _energyPoints = 50;
    _attackDamage = 30;
    std::cout
        << "DiamondTrap default "
        << "constructor called"
        << std::endl;
}

// ── 名前付きコンストラクタ ──
DiamondTrap::DiamondTrap(
    const std::string& name)
    : ClapTrap(name + "_clap_name"),  // (3)
      FragTrap(name),
      ScavTrap(name),
      _name(name)                     // (4)
{
    _hitPoints = 100;
    _energyPoints = 50;
    _attackDamage = 30;
    std::cout
        << "DiamondTrap constructor "
        << "called for " << _name
        << std::endl;
}

// ── コピーコンストラクタ ──
DiamondTrap::DiamondTrap(
    const DiamondTrap& other)
    : ClapTrap(other),
      FragTrap(other),
      ScavTrap(other),
      _name(other._name)
{
    std::cout
        << "DiamondTrap copy constructor"
        << " called for " << _name
        << std::endl;
}

// ── 代入演算子 ──
DiamondTrap& DiamondTrap::operator=(
    const DiamondTrap& other)
{
    std::cout
        << "DiamondTrap copy assignment"
        << " operator called"
        << std::endl;
    if (this != &other)
    {
        // virtual 基底なので1回呼べば十分
        ClapTrap::operator=(other);   // (5)
        _name = other._name;
    }
    return *this;
}

// ── デストラクタ ──
DiamondTrap::~DiamondTrap(void)
{
    std::cout
        << "DiamondTrap destructor "
        << "called for " << _name
        << std::endl;
}
```

1. **`ClapTrap("default_clap_name")`** — virtual 基底のコンストラクタは
   **最も派生したクラスから直接呼ぶ**。
   FragTrap/ScavTrap の初期化リスト内の ClapTrap() は無視される。
2. **ステータス上書き** — HP=FragTrap(100), EP=ScavTrap(50), AD=FragTrap(30) に設定。
3. **`name + "_clap_name"`** — ClapTrap::_name に渡す値。
   DiamondTrap::_name とは異なる値にする（subject 仕様）。
4. **`_name(name)`** — DiamondTrap 独自の _name を初期化。
   ClapTrap::_name とは別の変数。
5. **`ClapTrap::operator=(other)`** — virtual 基底なので1回呼べば十分。
   重複呼び出しの心配なし。

### DiamondTrap.cpp（whoAmI）

```cpp title="DiamondTrap.cpp (whoAmI)" linenums="1"
// ── whoAmI メソッド ──
// 2つの _name を両方表示する
void DiamondTrap::whoAmI(void)
{
    std::cout
        << "I am DiamondTrap "
        << _name                      // (1)
        << " and my ClapTrap name is "
        << ClapTrap::_name            // (2)
        << std::endl;
}
```

1. **`_name`** — スコープが近い DiamondTrap::_name が自動で参照される。
2. **`ClapTrap::_name`** — スコープ解決演算子で明示的に親の _name を指定。

### コンストラクタ呼び出し順序の詳細

```
DiamondTrap("Diamond") の構築順:

 (1) ClapTrap("Diamond_clap_name")
     ← virtual 基底が最初！
     _name = "Diamond_clap_name"
     _hitPoints=10, _energyPoints=10,
     _attackDamage=0

 (2) FragTrap("Diamond")
     ← 宣言順で FragTrap が先
     _hitPoints=100, _energyPoints=100,
     _attackDamage=30
     （ClapTrap() は呼ばれない、既に (1) で完了）

 (3) ScavTrap("Diamond")
     ← 宣言順で ScavTrap が次
     _hitPoints=100, _energyPoints=50,
     _attackDamage=20
     （ClapTrap() は呼ばれない、既に (1) で完了）

 (4) DiamondTrap("Diamond")
     ← 自身が最後
     _hitPoints=100, _energyPoints=50,
     _attackDamage=30
     DiamondTrap::_name = "Diamond"

破棄順: (4) → (3) → (2) → (1)
  （構築の完全な逆順）
```

!!! warning "ステータスの上書き連鎖に注意"
    FragTrap が HP=100, EP=100, AD=30 にした後、
    ScavTrap が HP=100, EP=50, AD=20 に上書きする。
    最終的に DiamondTrap のコンストラクタで HP=100, EP=50, AD=30 に設定し直す。
    **上書き順序を意識すること**。

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex03/"
    > "Files to turn in: Makefile, ClapTrap.{h, hpp}, ClapTrap.cpp,
    >  ScavTrap.{h, hpp}, ScavTrap.cpp, FragTrap.{h, hpp}, FragTrap.cpp,
    >  DiamondTrap.{h, hpp}, DiamondTrap.cpp, main.cpp"

    virtual 継承、多重継承、using 宣言、スコープ解決、
    構築/破棄順序の全てが評価のポイント。

- [ ] `make` が警告なく通る
- [ ] ScavTrap と FragTrap が `virtual public ClapTrap` で継承している
- [ ] DiamondTrap が FragTrap と ScavTrap を多重継承している
- [ ] 構築順: ClapTrap → FragTrap → ScavTrap → DiamondTrap
- [ ] 破棄順: DiamondTrap → ScavTrap → FragTrap → ClapTrap
- [ ] ClapTrap のコンストラクタは **1 回だけ** 呼ばれる（virtual 基底）
- [ ] attack で "ScavTrap" と表示される（using 宣言）
- [ ] whoAmI で 2 つの異なる _name が表示される

---

## 8. テストチェックリスト

### 構築/破棄順序

- [ ] 構築順: ClapTrap → FragTrap → ScavTrap → DiamondTrap
- [ ] 破棄順: DiamondTrap → ScavTrap → FragTrap → ClapTrap
- [ ] ClapTrap のコンストラクタは **1 回だけ** 呼ばれる（virtual 基底）

### ステータス

- [ ] HP = 100（FragTrap 由来）
- [ ] EP = 50（ScavTrap 由来）
- [ ] AD = 30（FragTrap 由来）

### メソッド

- [ ] `attack()` が "ScavTrap" と表示する（using 宣言）
- [ ] `guardGate()` が動作する（ScavTrap から継承）
- [ ] `highFivesGuys()` が動作する（FragTrap から継承）
- [ ] `whoAmI()` が DiamondTrap::_name と ClapTrap::_name の両方を表示する

### whoAmI の出力

- [ ] DiamondTrap の名前が表示される（"Diamond"）
- [ ] ClapTrap の名前が `<name>_clap_name` 形式で表示される（"Diamond_clap_name"）
- [ ] 2 つの名前が異なる

### OCF / 規約

- [ ] OCF が全て実装されている
- [ ] ScavTrap と FragTrap が `virtual public ClapTrap` で継承している
- [ ] `using namespace std;` なし
- [ ] `printf` / `malloc` / `free` 不使用

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| 多重継承とは？ | 2つ以上の親クラスを同時に継承すること |
| ダイヤモンド問題とは？ | 2つの親が同じ親を持つと、孫の中に共通の祖先が2つできてしまう問題 |
| ダイヤモンド問題を具体的に | virtual なしだと `_name` や `_hitPoints` が DiamondTrap 内に2つでき、曖昧になる |
| どう解決する？ | ScavTrap と FragTrap の両方を `virtual public ClapTrap` にする |
| virtual 継承は何をする？ | 共通の祖先を1つに統合し、ダイヤモンド問題を解決する |
| 片方だけ virtual じゃだめ？ | だめ。両方 virtual でないと ClapTrap は1つに統合されない |
| virtual 基底の初期化はどこでする？ | **最も派生したクラス**（DiamondTrap）から直接呼ぶ。C++ の仕様 |
| なぜ最派生から呼ぶ？ | ScavTrap と FragTrap のどちらが ClapTrap を初期化すべきか曖昧だから |
| using 宣言の意味は？ | 曖昧な候補（ScavTrap::attack と ClapTrap::attack）から1つを明示的に選ぶ |
| using なしだとどうなる？ | `diamond.attack()` が曖昧呼び出しでコンパイルエラー |
| 構築順は？ | ClapTrap → FragTrap → ScavTrap → DiamondTrap |
| 破棄順は？ | DiamondTrap → ScavTrap → FragTrap → ClapTrap（構築の逆順） |
| ClapTrap コンストラクタは何回呼ばれる？ | 1回だけ（virtual 基底なので） |
| なぜ DiamondTrap に独自 `_name` が必要？ | subject の仕様。`_name` と `ClapTrap::_name` を別々に管理するため |
| `ClapTrap::_name` の `::` は？ | スコープ解決演算子。「ClapTrap クラスの _name」と明示する |
| `_name` と `ClapTrap::_name` の違いは？ | `_name` は DiamondTrap 独自のもの、`ClapTrap::_name` は継承した親のもの |
| whoAmI は何を表示する？ | DiamondTrap::_name と ClapTrap::_name の両方 |
| HP はどっちから？ EP はどっちから？ | HP は FragTrap(100)、EP は ScavTrap(50)、AD は FragTrap(30) |

---

## 10. よくあるミス

!!! warning "virtual の付け忘れ"
    ```cpp
    // ❌ virtual なし → ClapTrap が2つできてしまう
    class ScavTrap : public ClapTrap { };
    class FragTrap : public ClapTrap { };

    // ✅ virtual 付き → ClapTrap が1つに統合
    class ScavTrap : virtual public ClapTrap { };
    class FragTrap : virtual public ClapTrap { };
    ```

!!! warning "片方だけ virtual にする"
    ```cpp
    // ❌ 片方だけだと統合されない
    class ScavTrap : virtual public ClapTrap { };
    class FragTrap : public ClapTrap { };  // ← virtual なし

    // ✅ 両方に virtual
    class ScavTrap : virtual public ClapTrap { };
    class FragTrap : virtual public ClapTrap { };
    ```

!!! warning "ClapTrap コンストラクタを DiamondTrap から直接呼ばない"
    ```cpp
    // ❌ ClapTrap を初期化リストで呼んでいない
    //   → ClapTrap のデフォルトコンストラクタが呼ばれる
    //   → _name が "default" になってしまう
    DiamondTrap::DiamondTrap(
        const std::string& name)
        : FragTrap(name),
          ScavTrap(name),
          _name(name) { }

    // ✅ ClapTrap を明示的に直接初期化
    DiamondTrap::DiamondTrap(
        const std::string& name)
        : ClapTrap(name + "_clap_name"),
          FragTrap(name),
          ScavTrap(name),
          _name(name) { }
    ```

!!! warning "using 宣言の欠落で曖昧エラー"
    ```cpp
    // ❌ using 宣言なし
    class DiamondTrap :
        public FragTrap, public ScavTrap
    {
        // diamond.attack("enemy");
        // → コンパイルエラー: 曖昧な呼び出し
    };

    // ✅ using 宣言で ScavTrap::attack を選択
    class DiamondTrap :
        public FragTrap, public ScavTrap
    {
        using ScavTrap::attack;
    };
    ```

!!! warning "whoAmI で ClapTrap::_name を忘れる"
    ```cpp
    // ❌ _name だけ表示
    //    → DiamondTrap::_name しか出ない
    void DiamondTrap::whoAmI(void) {
        std::cout << "I am " << _name
                  << std::endl;
    }

    // ✅ 両方の _name を表示
    void DiamondTrap::whoAmI(void) {
        std::cout << "I am DiamondTrap "
                  << _name
                  << " and my ClapTrap "
                  << "name is "
                  << ClapTrap::_name
                  << std::endl;
    }
    ```

!!! warning "ステータスの上書き忘れ"
    ```cpp
    // ❌ ステータスを上書きしない
    //    → ScavTrap の値(HP=100,EP=50,AD=20) のまま
    //    → subject 仕様と合わない
    DiamondTrap::DiamondTrap(
        const std::string& name)
        : ClapTrap(name + "_clap_name"),
          FragTrap(name),
          ScavTrap(name),
          _name(name) { }

    // ✅ 仕様通りに上書き
    //    HP=FragTrap(100), EP=ScavTrap(50),
    //    AD=FragTrap(30)
    DiamondTrap::DiamondTrap(
        const std::string& name)
        : ClapTrap(name + "_clap_name"),
          FragTrap(name),
          ScavTrap(name),
          _name(name)
    {
        _hitPoints = 100;
        _energyPoints = 50;
        _attackDamage = 30;
    }
    ```

---

## 11. 次のモジュールへ

cpp03 お疲れ様でした！ 一番の山場を越えました。

次の [cpp04](../cpp04/index.md) では **ポリモーフィズム** と **抽象クラス** を学びます:

- **virtual 関数** — cpp03 の virtual 継承とは **別の概念** に注意！
- **ポリモーフィズム** — 基底クラスのポインタ/参照で派生クラスのメソッドを呼ぶ
- **抽象クラス** — 純粋仮想関数を持つ、インスタンス化できないクラス
- **interface** — すべて純粋仮想関数のクラス

cpp03 で学んだ継承を土台に、さらに強力な設計テクニックを学びます。
