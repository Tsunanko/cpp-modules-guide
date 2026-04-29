# ex01 — ScavTrap

---

## このプログラムは何？

**ClapTrap を「親」として、ScavTrap という「子」クラスを作るプログラム** です。

ScavTrap は ClapTrap の機能を全部受け継ぎますが、
**HP・EP・AD の数値がパワーアップ** していて、
**警備モード（guardGate）** という独自機能も持っています。

これが cpp03 で初めての **継承（けいしょう）** の exercise。
「親クラス → 子クラス」の関係を実際に体験します。

```
   ClapTrap（親）
      HP: 10, EP: 10, AD: 0
         |
         ↓ 継承（機能を受け継ぐ）
         |
   ScavTrap（子）
      HP: 100, EP: 50, AD: 20   ← 数値がパワーアップ
      + guardGate()              ← 独自の機能
      + attack() を書き直し      ← 親の関数を上書き
```

---

## 1. この exercise で学ぶこと

- **public 継承** — `class ScavTrap : public ClapTrap`
- **`protected`** — private と public の中間
- **構築順序（親→子）** と **破棄順序（子→親）**
- **メソッドオーバーライド** — 親の関数を子で書き直す
- **親の代入演算子を呼ぶ** — `ClapTrap::operator=(other)`

---

## 2. 新しい概念の解説

### `class Child : public Parent` って何？

**「Child は Parent を受け継ぐ」** と宣言する書き方です。
これが **継承** の構文。

```cpp
class ScavTrap : public ClapTrap
//         ↑     ↑       ↑
//         |     |       親クラスの名前
//         |     public継承（後で説明）
//         子クラスの名前
{
    // ScavTrap 独自のメンバをここに書く
};
```

これを書くだけで、ScavTrap は ClapTrap の **全部のメンバ** を持ちます。

```
ClapTrap が持っていたもの:
  _name, _hitPoints, _energyPoints, _attackDamage
  attack(), takeDamage(), beRepaired()

↓ ScavTrap : public ClapTrap で継承

ScavTrap が持つもの:
  親から受け継いだ全て + ScavTrap 独自のもの
```

### public 継承って何？

継承には3種類あります: `public`、`protected`、`private`。
42Tokyo の課題では **public 継承だけ** を使います。

| 継承の種類 | 意味 |
|------|------|
| `public` 継承 | 「ScavTrap は ClapTrap の一種」という is-a 関係 |
| `protected` 継承 | 実装の流用（使わない） |
| `private` 継承 | 実装の流用（使わない） |

**public 継承** の意味: 親の public メンバは子でも public、
親の protected メンバは子でも protected（そのまま引き継がれる）。

### `protected` って何？

**private と public の中間** のアクセス修飾子です。

| 修飾子 | クラス内 | 派生クラス | クラス外 |
|:---:|:---:|:---:|:---:|
| `public` | 〇 | 〇 | 〇 |
| `protected` | 〇 | 〇 | × |
| `private` | 〇 | × | × |

- `private`: クラスの **中だけ** 触れる。派生クラスからも触れない。
- `protected`: クラスの中 + **派生クラスからも** 触れる。でも外からは触れない。
- `public`: どこからでも触れる。

### なぜ `private` を `protected` に変えるの？

**子クラスから親のメンバ変数を書き換えるため** です。

ex00 では ClapTrap のメンバは `private` でした。
でも ex01 で ScavTrap を作ると、ScavTrap のコンストラクタで
`_hitPoints = 100;` と書きたくなります。

```cpp
// ScavTrap のコンストラクタで…
ScavTrap::ScavTrap(const std::string& name)
    : ClapTrap(name)
{
    _hitPoints = 100;   // ← _hitPoints を書き換えたい
    _energyPoints = 50;
    _attackDamage = 20;
}
```

でも `private` だと、派生クラスからも触れません。
だから `protected` に **格上げ** する必要があります。

```cpp
// ex00: private（派生からも触れない）
private:
    unsigned int _hitPoints;

// ex01: protected（派生から触れる）
protected:
    unsigned int _hitPoints;
```

### 構築順序（親→子）って何？

**子クラスのオブジェクトを作るとき、まず親のコンストラクタが呼ばれて、その後に子のコンストラクタが呼ばれる**。

```
ScavTrap a("Alpha"); と書くと…

  1. ClapTrap のコンストラクタが呼ばれる（親が先！）
     → _name="Alpha", _hitPoints=10, _energyPoints=10,
        _attackDamage=0 で初期化
     → "ClapTrap constructor called for Alpha"

  2. ScavTrap のコンストラクタが呼ばれる（子が後！）
     → _hitPoints=100, _energyPoints=50,
        _attackDamage=20 に上書き
     → "ScavTrap constructor called for Alpha"
```

**なぜ親が先？** → 子は親の上に成り立っているから。
家を建てるときに土台を先に作るのと同じ。

### 破棄順序（子→親）って何？

**オブジェクトが消えるとき、まず子のデストラクタが呼ばれて、その後に親のデストラクタが呼ばれる**。

```
main 終了で a が消えるとき…

  1. ScavTrap のデストラクタが呼ばれる（子が先！）
     → "ScavTrap destructor called for Alpha"

  2. ClapTrap のデストラクタが呼ばれる（親が後！）
     → "ClapTrap destructor called for Alpha"
```

**なぜ子が先？** → これを **LIFO（Last In, First Out）** と言います。
「最後に作られたものが最初に壊される」のがルール。

```
 構築: [ClapTrap] → [ScavTrap]   （土台から積み上げ）
 破棄: [ScavTrap] → [ClapTrap]   （上から順に取り壊し）
```

子が先に壊れないといけない理由:
子は親のメンバに依存しているので、親を先に壊すと子が参照先を失ってしまう。
だから **子を先に綺麗に片付けてから、親を壊す**。

### メソッドオーバーライドって何？

**親の関数と同じ名前の関数を子で書き直す** ことです。

ClapTrap の `attack()` は `"ClapTrap Alice attacks..."` と表示します。
でも ScavTrap は `"ScavTrap Alpha attacks..."` と表示したい。

そこで、ScavTrap の中で **同じ名前の `attack()` を定義** します。
これが **オーバーライド**。

```cpp
class ClapTrap {
public:
    void attack(const std::string& target) {
        std::cout << "ClapTrap ... attacks";
    }
};

class ScavTrap : public ClapTrap {
public:
    // 親と同じ名前で書き直し！
    void attack(const std::string& target) {
        std::cout << "ScavTrap ... attacks";
    }
};

ScavTrap a("Alpha");
a.attack("Bob");   // ScavTrap 版が呼ばれる
```

!!! info "C++98 では厳密には「名前隠蔽」"
    `virtual` キーワードなしのオーバーライドは、
    C++ の用語では **名前隠蔽（name hiding）** と呼ばれます。
    cpp04 で学ぶ `virtual` 関数とは動作が少し違います。
    今は「同じ名前で書き直すと子版が使われる」と理解すればOK。

### `ClapTrap::operator=(other)` って何？

**子の代入演算子の中から、親の代入演算子を呼び出す** 書き方です。

ScavTrap の代入演算子を書くとき、親（ClapTrap）部分のコピーは
親に任せる方が確実です。

```cpp
ScavTrap& ScavTrap::operator=(
    const ScavTrap& other)
{
    if (this != &other)
        // ↓ 親クラスの代入演算子を呼ぶ
        ClapTrap::operator=(other);
    return *this;
}
```

- `ClapTrap::` → スコープ解決演算子。「ClapTrap の」という指定
- `operator=(other)` → 代入演算子を直接呼び出し

こうすることで、親の `_name` や `_hitPoints` のコピーを親に任せて、
子は **自分独自の処理だけ** に集中できます。

---

## 3. 課題仕様

### ScavTrap クラス

- **`ClapTrap` を public 継承**
- ClapTrap のメンバ変数を `private` → `protected` に変更（親クラスの変更）

### ステータス

| 属性 | 初期値（ClapTrap からの変更） |
|------|------------------------------|
| `_hitPoints` | **100** （10 → 100） |
| `_energyPoints` | **50** （10 → 50） |
| `_attackDamage` | **20** （0 → 20） |

### メンバ関数

- コンストラクタ/デストラクタで **ScavTrap 固有のメッセージ** を出力
- `attack()` を **オーバーライド**（ScavTrap 用のメッセージに変更）
- 固有メソッド: **`guardGate()`** — Gate keeper モードを表示
- OCF を全て実装

---

## 4. 実行例

```console
$ ./ScavTrap
=== Creating ScavTrap ===
ClapTrap constructor called for Alpha       ← 親が先
ScavTrap constructor called for Alpha       ← 子が後

=== ScavTrap actions ===
ScavTrap Alpha attacks Enemy, causing 20 points of damage!
ClapTrap Alpha takes 30 points of damage! HP: 70
ClapTrap Alpha repairs itself for 10 hit points! HP: 80
ScavTrap Alpha is now in Gate keeper mode.

=== Destruction ===
ScavTrap destructor called for Alpha        ← 子が先
ClapTrap destructor called for Alpha        ← 親が後
```

**注目ポイント**:

- **構築順**: ClapTrap → ScavTrap（親から子へ）
- **破棄順**: ScavTrap → ClapTrap（子から親へ、逆順）
- `attack()` は ScavTrap がオーバーライド → "ScavTrap" と表示
- `takeDamage()` / `beRepaired()` はオーバーライドしていない → "ClapTrap" と表示

---

## 5. C と C++ の比較

### 継承って C で言うと何？

**C で「構造体の先頭に親構造体を埋め込む」テクニックを、
言語機能として正式にサポートしたもの** です。

C でクラスっぽい派生を作ると、毎回こう書きます:

```c
/* ステップ1: 親構造体を先頭に埋め込む */
/* typedef struct {
 *     t_clap parent;  ← ここが親
 *     ... 子独自のメンバ ...
 * } t_scav;
 */

/* ステップ2: 子の初期化で親の初期化を「手動」で呼ぶ */
/* void scav_init(t_scav *s, ...) {
 *     clap_init(&s->parent, ...);  ← 呼び忘れ注意
 *     ... 子独自の初期化 ...
 * }
 */

/* ステップ3: 親のメンバに触るには parent 経由 */
/* s->parent.hp = 100; */

/* ステップ4: アクセス制限がないので外から触り放題 */
```

C++ の継承はこれを **言語機能で自動化** する仕組み:

| C の擬似継承 | C++ の継承 |
|-------------|-----------|
| `t_clap parent;` を先頭に埋め込み | `class Child : public Parent` |
| `clap_init()` を手動呼び出し | 親コンストラクタが自動呼び出し |
| `clap_destroy()` を手動呼び出し | 親デストラクタが自動呼び出し |
| `s->parent.hp` でアクセス | `_hitPoints` で直接 |
| 関数ポインタで上書き | メソッドオーバーライド |
| アクセス制限なし | `public/protected/private` |

つまり継承は **「C で書いていた構造体埋め込みパターンを、
自動化 + 安全化 する」** 仕組みです。

### 並べて比較

=== "C の書き方（擬似継承）"

    ```c
    /* C には継承がない！ */
    /* 構造体埋め込みで擬似的に書く */

    /* ── 親の構造体 ── */
    typedef struct s_clap {
        /* 名前 */
        char         name[50];
        /* HP/EP/AD */
        unsigned int hp;
        unsigned int ep;
        unsigned int ad;
    } t_clap;

    /* ── 子の構造体 (親を埋め込む) ── */
    typedef struct s_scav {
        /* 先頭に親を埋め込むのがミソ */
        /* アドレスが同じになり、擬似的に
           「scav は clap」として扱える */
        t_clap parent;
        /* ScavTrap 独自のメンバ */
    } t_scav;

    /* ── 子の初期化関数 (自作) ── */
    /* 親の初期化を手動で呼ぶ必要あり */
    void scav_init(t_scav *s,
                   const char *name) {
        /* 忘れると親が初期化されないバグ！ */
        clap_init(&s->parent, name);
        /* 親のメンバを上書き (parent 経由) */
        s->parent.hp = 100;
        s->parent.ep = 50;
        s->parent.ad = 20;
    }

    /* ── 子の攻撃関数 (自作) ── */
    /* 親のメンバは parent. 経由でアクセス */
    void scav_attack(t_scav *s,
                     const char *t) {
        /* EP を 1 消費 (parent 経由) */
        s->parent.ep--;
        /* printf で出力 */
        printf("ScavTrap attacks %s\n", t);
    }
    ```

=== "C++ の書き方（継承）"

    ```cpp
    // ── 継承の構文 ──
    // : public ClapTrap で「ClapTrap の子」を宣言
    // これだけで親の全メンバが自動で組み込まれる
    class ScavTrap : public ClapTrap {
    public:
        // ── コンストラクタ ──
        // : ClapTrap(name) で親を「明示的に」呼ぶ
        // (書かなくても親のデフォルトが自動で呼ばれる)
        ScavTrap(const std::string& name)
            : ClapTrap(name)
        {
            // 親の protected メンバを直接書き換え
            // parent. のようなプレフィックス不要
            _hitPoints = 100;
            _energyPoints = 50;
            _attackDamage = 20;
        }

        // ── attack のオーバーライド ──
        // 親と同名のメソッドを子で再定義
        // 呼び出し時に子版が優先される
        void attack(
            const std::string& target)
        {
            // 親メンバにそのままアクセス
            if (_hitPoints == 0
                || _energyPoints == 0)
                return;
            _energyPoints--;
            // std::cout で出力
            std::cout << "ScavTrap "
                << _name << " attacks "
                << target << std::endl;
        }

        // ── 子独自のメソッド ──
        // 親にはない機能を追加
        void guardGate(void) {
            std::cout << _name
                << " is now in Gate "
                << "keeper mode."
                << std::endl;
        }
    };
    ```

**ステップ数で比較すると:**

```
C の場合: 4 ステップ (毎回書く)
  ①親構造体を先頭に埋め込む
  ②子の初期化で親の init を「手動」で呼ぶ
     (忘れるとバグ)
  ③親メンバアクセスは parent. 経由
  ④後片付けも親の destroy を手動で呼ぶ

C++ の場合: 1 ステップ (宣言だけ)
  ①: public ClapTrap と書くだけ
     親コンストラクタの呼び出し、
     親メンバへの直接アクセス、
     親デストラクタの呼び出し、
     全部自動でやってくれる
```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|------------|
| `struct` 埋め込み | `class : public` 継承 | 本物の継承 |
| `init_clap()` を手動呼び出し | 親コンストラクタが自動呼び出し | 呼び忘れ不可能 |
| `destroy_clap()` を手動呼び出し | 親デストラクタが自動呼び出し | 片付け忘れ不可能 |
| `s->parent.hp` でアクセス | `_hitPoints` で直接アクセス | protected 経由で自然な書き方 |
| 関数ポインタで上書き | メソッドオーバーライド | 言語レベルでサポート |

### アクセス権限 (public/protected/private) って何？

**メンバに「どこから触れるか」のレベルを設定** する仕組み。

C には全くない C++ 独自の機能です。

```
┌─────────────┬────────┬──────────┬────────┐
│ 修飾子      │ 自分   │ 子クラス │ 外部   │
├─────────────┼────────┼──────────┼────────┤
│ public      │  〇    │   〇     │  〇    │
│ protected   │  〇    │   〇     │  ×     │
│ private     │  〇    │   ×      │  ×     │
└─────────────┴────────┴──────────┴────────┘
```

- **public**: 玄関から誰でも入れる (外部 API)
- **protected**: 家族 (子クラス) までなら入れる
- **private**: 自分だけ (外部からも子からも隔離)

### 構築順序 (親→子) って C で言うと何？

**C の「親構造体を先に初期化してから子を初期化」を、
言語が自動でやってくれる** 仕組みです。

```
C の場合 (手動):
  scav_init(&s, "Alice") の中で…
    ①clap_init(&s->parent, "Alice")  ← 手動で呼ぶ
    ②s->parent.hp = 100;              ← 上書き
    (呼び忘れたらバグ)

C++ の場合 (自動):
  ScavTrap a("Alice") の瞬間に…
    ①ClapTrap コンストラクタが自動実行 (親が先)
    ②ScavTrap コンストラクタが実行 (子が後)
    (書き忘れようがない)
```

**なぜ親が先？** → 子は親のメンバに依存するから。
家を建てる時、土台 (親) を先に作ってから
2 階 (子) を積むのと同じ。

### 破棄順序 (子→親) って C で言うと何？

**「構築の逆順で壊す」のを C では手動、C++ では自動**。

```
C の場合 (手動):
  scav_destroy(&s) の中で…
    ①s->own_buffer を free(子の片付け)
    ②clap_destroy(&s->parent) を呼ぶ(親を壊す)
    (順序を間違えたら未定義動作)

C++ の場合 (自動):
  スコープを抜ける瞬間に…
    ①ScavTrap デストラクタ自動実行 (子が先)
    ②ClapTrap デストラクタ自動実行 (親が後)
```

**なぜ子が先？** → LIFO (Last In First Out)。
子は親のメンバを使うので、親より先に片付けないと
「参照先がない」状態になってしまう。

!!! info "C には継承が存在しない"
    Cの構造体埋め込みは見た目は似ていますが、
    **コンストラクタ自動呼び出し**、**アクセス制御（protected）**、
    **ポリモーフィズム** のどれも持ちません。
    C++ の継承は言語レベルでこれらを保証します。

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
ScavTrap a("Alpha") を作る
  |
  ├→ [1] ClapTrap コンストラクタ（親が先）
  |      "ClapTrap constructor called for Alpha"
  |
  └→ [2] ScavTrap コンストラクタ（子が後）
         HP/EP/AD を上書き（100/50/20）
         "ScavTrap constructor called for Alpha"
  v
a.attack("Enemy")  → ScavTrap::attack（オーバーライド）
  v
a.takeDamage(30)   → ClapTrap::takeDamage（継承）
  v
a.beRepaired(10)   → ClapTrap::beRepaired（継承）
  v
a.guardGate()      → ScavTrap 固有メソッド
  v
main 終了
  |
  ├→ [1] ScavTrap デストラクタ（子が先）
  |      "ScavTrap destructor called for Alpha"
  |
  └→ [2] ClapTrap デストラクタ（親が後）
         "ClapTrap destructor called for Alpha"
```

### ClapTrap.hpp（ex00 からの変更）

```cpp title="ClapTrap.hpp (変更箇所)" linenums="1"
class ClapTrap
{
public:
    // OCF と戦闘メソッドは同じ

// ── ここが変更点 ──
// ex00: private だった
// ex01: protected に変更！
protected:               // (1)
    std::string  _name;
    unsigned int _hitPoints;
    unsigned int _energyPoints;
    unsigned int _attackDamage;
};
```

1. **`protected`** — クラス外からは触れない（privateと同じ）が、
   **派生クラスからは触れる**。ScavTrap のコンストラクタで
   `_hitPoints = 100;` と書けるようになる。

### ScavTrap.hpp

```cpp title="ScavTrap.hpp" linenums="1"
#ifndef SCAVTRAP_HPP
# define SCAVTRAP_HPP

// 親クラスの定義が必要
# include "ClapTrap.hpp"

// ── 継承の書き方 ──
// class Child : public Parent
class ScavTrap : public ClapTrap   // (1)
{
public:
    // OCF は子クラスでも自分で書く
    ScavTrap(void);
    ScavTrap(const std::string& name);
    ScavTrap(const ScavTrap& other);
    ScavTrap& operator=(
        const ScavTrap& other);
    ~ScavTrap(void);

    // 親の attack() を書き直す
    void attack(
        const std::string& target);  // (2)

    // ScavTrap 独自のメソッド
    void guardGate(void);            // (3)
};

#endif
```

1. **`class ScavTrap : public ClapTrap`** — ScavTrap は ClapTrap を public 継承。
   親の public/protected メンバを引き継ぐ。
2. **`attack()` のオーバーライド** — 親と同名のメソッドを再定義。
   呼び出し時に ScavTrap 版が優先される。
3. **`guardGate()`** — ScavTrap 固有の機能。ClapTrap にはない。

### ScavTrap.cpp（コンストラクタ）

```cpp title="ScavTrap.cpp (コンストラクタ)" linenums="1"
#include "ScavTrap.hpp"
#include <iostream>

// ── デフォルトコンストラクタ ──
// : ClapTrap() で親コンストラクタを呼ぶ
ScavTrap::ScavTrap(void)
    : ClapTrap()                       // (1)
{
    // 親の初期化後、ここで上書き
    _hitPoints = 100;                   // (2)
    _energyPoints = 50;
    _attackDamage = 20;
    std::cout
        << "ScavTrap default "
        << "constructor called"
        << std::endl;
}

// ── 名前付きコンストラクタ ──
ScavTrap::ScavTrap(
    const std::string& name)
    : ClapTrap(name)                   // (3)
{
    _hitPoints = 100;
    _energyPoints = 50;
    _attackDamage = 20;
    std::cout
        << "ScavTrap constructor "
        << "called for " << _name
        << std::endl;
}

// ── コピーコンストラクタ ──
// 親のコピコンに委譲: ClapTrap(other)
ScavTrap::ScavTrap(
    const ScavTrap& other)
    : ClapTrap(other)                  // (4)
{
    std::cout
        << "ScavTrap copy constructor "
        << "called for " << _name
        << std::endl;
}

// ── 代入演算子 ──
ScavTrap& ScavTrap::operator=(
    const ScavTrap& other)
{
    std::cout
        << "ScavTrap copy assignment "
        << "operator called"
        << std::endl;
    if (this != &other)
        // 親の代入演算子に委譲
        ClapTrap::operator=(other);    // (5)
    return *this;
}

// ── デストラクタ ──
ScavTrap::~ScavTrap(void)
{
    std::cout
        << "ScavTrap destructor "
        << "called for " << _name
        << std::endl;
}
```

1. **`: ClapTrap()`** — 初期化子リストで親コンストラクタを明示的に呼ぶ。
   省略するとデフォルトコンストラクタが自動で呼ばれる。明示的に書くのが推奨。
2. **ステータス上書き** — 親の初期化リストで `_hitPoints=10` 等がセットされた後、
   ここで 100/50/20 に上書き。
3. **`: ClapTrap(name)`** — 親の名前付きコンストラクタを呼ぶ。
   `_name` が `name` で初期化される。
4. **`: ClapTrap(other)`** — ScavTrap は ClapTrap の子なので、
   `other` を ClapTrap& として扱える（アップキャスト）。
5. **`ClapTrap::operator=(other)`** — 親の代入演算子に委譲。
   共通メンバのコピーを親に任せる。

### ScavTrap.cpp（メソッド）

```cpp title="ScavTrap.cpp (メソッド)" linenums="1"
// ── attack() のオーバーライド ──
// 親の attack() と同じロジックだが
// メッセージが "ScavTrap" に変わる
void ScavTrap::attack(
    const std::string& target)
{
    if (_hitPoints == 0)
    {
        std::cout << "ScavTrap " << _name
            << " can't attack, "
            << "no hit points!"
            << std::endl;
        return;
    }
    if (_energyPoints == 0)
    {
        std::cout << "ScavTrap " << _name
            << " can't attack, "
            << "no energy points!"
            << std::endl;
        return;
    }
    _energyPoints--;
    std::cout << "ScavTrap " << _name
        << " attacks " << target
        << ", causing " << _attackDamage
        << " points of damage!"
        << std::endl;
}

// ── ScavTrap 固有メソッド ──
void ScavTrap::guardGate(void)
{
    std::cout << "ScavTrap " << _name
        << " is now in Gate keeper "
        << "mode." << std::endl;
}
```

!!! info "なぜ attack() をオーバーライドするの？"
    ClapTrap の attack() は `"ClapTrap Alice attacks..."` と出力します。
    でも ScavTrap のインスタンスなのに "ClapTrap" と表示されたら不自然。
    **派生クラスにふさわしい表示** にするためにオーバーライドします。

!!! info "takeDamage と beRepaired はオーバーライドしない"
    subject はこれらのオーバーライドを要求していないので、
    親の ClapTrap 版がそのまま使われます。
    結果として、メッセージは `"ClapTrap ... takes ..."` になります。

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex01/"
    > "Files to turn in: Makefile, ClapTrap.{h, hpp}, ClapTrap.cpp,
    >  ScavTrap.{h, hpp}, ScavTrap.cpp, main.cpp"

    継承の構文、構築/破棄順序、attack のオーバーライド、
    protected への変更が評価のポイント。

- [ ] `make` が警告なく通る
- [ ] ClapTrap のメンバ変数が `protected` になっている
- [ ] ScavTrap が `public ClapTrap` で継承している
- [ ] 構築順序が親→子になっている
- [ ] 破棄順序が子→親になっている
- [ ] attack() がオーバーライドされている
- [ ] guardGate() が動作する

---

## 8. テストチェックリスト

### 構築/破棄順序

- [ ] ScavTrap 生成時に ClapTrap コンストラクタが **先に** 呼ばれる
- [ ] ScavTrap 破棄時に ScavTrap デストラクタが **先に** 呼ばれる
- [ ] ClapTrap デストラクタが **後に** 呼ばれる

### ステータス

- [ ] ScavTrap の HP = 100, EP = 50, AD = 20
- [ ] attack で 20 ダメージが表示される

### メソッド

- [ ] `attack()` が "ScavTrap" と表示する（オーバーライド）
- [ ] `takeDamage()` が "ClapTrap" と表示する（継承）
- [ ] `beRepaired()` が "ClapTrap" と表示する（継承）
- [ ] `guardGate()` が Gate keeper モードのメッセージを表示する

### OCF

- [ ] コピーコンストラクタが動作する
- [ ] 代入演算子が動作する
- [ ] コピー/代入でも構築/破棄順序が正しい

### 規約

- [ ] ClapTrap のメンバ変数が `protected` になっている
- [ ] `using namespace std;` なし
- [ ] `printf` / `malloc` / `free` 不使用

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| public 継承とは？ | 親の public/protected がそのまま子に引き継がれる継承。is-a 関係を表す |
| なぜ `private` を `protected` に変えた？ | 派生クラス（ScavTrap）から `_hitPoints` などを書き換えるため |
| private と protected の違いは？ | protected は派生クラスから触れる、private は触れない |
| 構築順序は？ | 親 → 子（親の土台が先にできる） |
| 破棄順序は？ | 子 → 親（構築の逆順、LIFO） |
| なぜ親が先に構築される？ | 子は親のメンバに依存するから。親が完成してから子を積む |
| なぜ子が先に破棄される？ | 子は親に依存するので、子を先に片付けてから親を壊す |
| `: ClapTrap(name)` は何？ | 初期化子リストで親のコンストラクタを明示的に呼ぶ書き方 |
| オーバーライドとは？ | 親と同じ名前の関数を子で書き直すこと |
| `ClapTrap::operator=(other)` の意味は？ | 親の代入演算子を呼び出して、親部分のコピーを任せる |
| なぜ attack だけオーバーライド？ | subject が「attack のメッセージを ScavTrap にせよ」と要求しているから |

---

## 10. よくあるミス

!!! warning "親コンストラクタの呼び忘れ"
    ```cpp
    // ❌ 親コンストラクタを初期化リストで呼んでいない
    ScavTrap::ScavTrap(
        const std::string& name) {
        _name = name;
        // → ClapTrap のデフォルトコンストラクタが
        //   勝手に呼ばれた後に代入される
    }

    // ✅ 初期化リストで親コンストラクタを明示的に呼ぶ
    ScavTrap::ScavTrap(
        const std::string& name)
        : ClapTrap(name)
    {
        _hitPoints = 100;
    }
    ```

!!! warning "private のまま継承しようとする"
    ```cpp
    // ❌ ClapTrap.hpp で private のまま
    private:
        unsigned int _hitPoints;
        // → ScavTrap からアクセスできない！
        //   コンパイルエラー

    // ✅ protected に変更する
    protected:
        unsigned int _hitPoints;
    ```

!!! warning "コンストラクタメッセージの順序が逆"
    親のコンストラクタが先に呼ばれることを理解していないと、
    メッセージの順序が仕様と合わない。
    **初期化リストの呼び出し順は常に親→子**。

!!! warning "operator= で親を呼び忘れる"
    ```cpp
    // ❌ 親の operator= を呼ばない
    ScavTrap& ScavTrap::operator=(
        const ScavTrap& other) {
        // _name などのコピーが漏れる！
        return *this;
    }

    // ✅ 親の operator= に委譲
    ScavTrap& ScavTrap::operator=(
        const ScavTrap& other) {
        if (this != &other)
            ClapTrap::operator=(other);
        return *this;
    }
    ```

---

## 11. 次の exercise へ

次の [ex02 FragTrap](ex02-fragtrap.md) では、
**ScavTrap と同じパターン** で FragTrap を作ります。

今度は ScavTrap と **兄弟関係** の派生クラスを作る練習です。
パターンを定着させる復習回なので、ex01 の理解を確認する良い機会。
