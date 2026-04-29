# ex02 — FragTrap

---

## このプログラムは何？

**ClapTrap を親として、FragTrap という子クラスを作るプログラム** です。

これは ex01 の ScavTrap と **同じパターン** の練習。
でも FragTrap は ScavTrap と **兄弟関係**（どちらも ClapTrap の子）で、
お互いに独立しています。

```
     ClapTrap（共通の親）
      HP: 10, EP: 10, AD: 0
        /        \
       /          \
   ScavTrap    FragTrap      ← 兄弟クラス（お互いに独立）
  HP:100       HP:100
  EP:50        EP:100
  AD:20        AD:30
  + guardGate   + highFivesGuys
```

ScavTrap と似ていますが、数値と固有メソッドが違います。
**ScavTrap は attack をオーバーライドするけど、FragTrap はしない** のも大事な違い。

---

## 1. この exercise で学ぶこと

- **兄弟クラス（並列の継承）** のパターン
- ScavTrap と同じ構造で FragTrap を書く **反復練習**
- **attack をオーバーライドしない** 場合の動作
- 違うステータス値と固有メソッドの実装

---

## 2. 新しい概念の解説

### 兄弟クラスって何？

**同じ親を持つ、横並びのクラス** のこと。

ScavTrap と FragTrap はどちらも ClapTrap の子です。
でもお互いには **関係がない**（ScavTrap は FragTrap を知らない）。

```
       ClapTrap
        /    \
   ScavTrap  FragTrap    ← 横並び、お互いに独立
```

この構造は「is-a 関係」が複数ある場合によく使います。
例えるなら:
- Animal（親）
  - Dog（子）
  - Cat（子）
  - Bird（子）

Dog と Cat は兄弟で、お互いに直接関係はありません。

### 反復練習で何を身につける？

ex02 は **ex01 と同じパターンを繰り返す** ことで、
継承の書き方を体に染み込ませる練習です。

```
ex01 でやったこと:
  - class ScavTrap : public ClapTrap
  - コンストラクタで親を呼ぶ : ClapTrap(name)
  - 親の operator= に委譲
  - 構築/破棄順序（親→子/子→親）

ex02 でやること:
  - class FragTrap : public ClapTrap  ← 全く同じパターン！
  - コンストラクタで親を呼ぶ : ClapTrap(name)
  - 親の operator= に委譲
  - 構築/破棄順序（親→子/子→親）

違うのは: 数値（HP/EP/AD）と固有メソッド（highFivesGuys）と
          attack をオーバーライドしない点
```

### attack をオーバーライドしないとどうなる？

subject は FragTrap に attack のオーバーライドを **要求していません**。
なので、FragTrap は親（ClapTrap）の attack をそのまま使います。

```cpp
FragTrap a("Frag");
a.attack("Enemy");
// 呼ばれるのは ClapTrap::attack
// → "ClapTrap Frag attacks Enemy, ..." と表示
```

これが ScavTrap との **大きな違い**:

| | ScavTrap | FragTrap |
|---|---------|----------|
| attack オーバーライド | **あり** | **なし** |
| 表示 | "ScavTrap ... attacks" | "ClapTrap ... attacks" |

---

## 3. 課題仕様

### FragTrap クラス

- **`ClapTrap` を public 継承**

### ステータス

| 属性 | 初期値（ClapTrap からの変更） |
|------|------------------------------|
| `_hitPoints` | **100** （10 → 100） |
| `_energyPoints` | **100** （10 → 100） |
| `_attackDamage` | **30** （0 → 30） |

### メンバ関数

- コンストラクタ/デストラクタで **FragTrap 固有のメッセージ** を出力
- 固有メソッド: **`highFivesGuys()`** — ポジティブなハイタッチをリクエスト
- `attack()` のオーバーライドは **不要**（ClapTrap の attack をそのまま使う）
- OCF を全て実装

!!! info "ScavTrap との違いを整理"
    | | ScavTrap | FragTrap |
    |---|---------|----------|
    | EP | 50 | **100** |
    | AD | 20 | **30** |
    | 固有メソッド | `guardGate()` | `highFivesGuys()` |
    | attack オーバーライド | あり | **なし** |

---

## 4. 実行例

```console
$ ./FragTrap
=== Creating FragTrap ===
ClapTrap constructor called for Frag        ← 親が先
FragTrap constructor called for Frag        ← 子が後

=== FragTrap actions ===
ClapTrap Frag attacks Enemy, causing 30 points of damage!
ClapTrap Frag takes 40 points of damage! HP: 60
ClapTrap Frag repairs itself for 20 hit points! HP: 80
FragTrap Frag requests a positive high five!

=== Destruction ===
FragTrap destructor called for Frag         ← 子が先
ClapTrap destructor called for Frag         ← 親が後
```

**注目ポイント**:

- attack のメッセージが "**ClapTrap**" になっている
  （FragTrap は attack をオーバーライドしない）
- ScavTrap と同じ構築/破棄順序（親→子 / 子→親）
- AD = 30 なので、攻撃すると 30 ダメージ

---

## 5. C と C++ の比較

ex02 は ex01 と同じパターンの繰り返し。
C では構造体埋め込みで 2 種類の「派生構造体」を作ると、
共通部分の初期化コードが重複します。
C++ の継承なら ClapTrap のロジックは 1 箇所に集約されます。

### 兄弟クラスって C で言うと何？

**C では、同じ親から派生する構造体を複数作ると、
親の init を「全ての子で手動で呼ぶ」必要がある** 状態です。

C++ の継承なら、親のロジックは ClapTrap に 1 箇所だけ書いて、
子 (ScavTrap, FragTrap) は自動で使い回せます。

=== "C の場合（重複）"

    ```c
    /* ── 親の構造体 ── */
    typedef struct s_clap {
        char name[50];
        unsigned int hp;
    } t_clap;

    /* ── 子1: ScavTrap ── */
    /* 親を埋め込む */
    typedef struct s_scav {
        t_clap parent;
    } t_scav;

    /* ── 子2: FragTrap (同じパターン) ── */
    /* 親を埋め込む (同じ記述の繰り返し) */
    typedef struct s_frag {
        t_clap parent;
    } t_frag;

    /* ── ScavTrap の初期化関数 ── */
    void scav_init(t_scav *s,
                   const char *name) {
        /* 親の init を手動で呼ぶ */
        clap_init(&s->parent, name);
        /* 独自の初期化を続ける */
    }

    /* ── FragTrap の初期化関数 ── */
    /* 同じパターンを書き直す (重複) */
    void frag_init(t_frag *f,
                   const char *name) {
        /* 親の init を手動で呼ぶ */
        clap_init(&f->parent, name);
        /* 独自の初期化を続ける */
    }

    /* ↑ どちらも clap_init の呼び忘れリスクあり */
    /* ↑ 兄弟が増えるほど重複コードが増える */
    ```

=== "C++ の場合（自動）"

    ```cpp
    // ── 子1: ScavTrap ──
    // : public ClapTrap で継承を宣言
    class ScavTrap : public ClapTrap {
    public:
        // 初期化子リストで親を呼ぶ
        ScavTrap(const std::string& name)
            // ClapTrap(name) は親コンストラクタ
            // 省略しても自動で呼ばれる
            : ClapTrap(name) { }
    };

    // ── 子2: FragTrap (同じパターン) ──
    // 宣言の書き方も同じ = 学習コスト低い
    class FragTrap : public ClapTrap {
    public:
        // こちらも親コンストラクタを明示
        FragTrap(const std::string& name)
            // 呼び忘れは言語仕様上不可能
            // (デフォルトが自動実行される)
            : ClapTrap(name) { }
    };
    ```

**ステップ数で比較すると:**

```
C の場合: 子クラスの数 × 毎回同じステップ
  ScavTrap: 親埋め込み + clap_init 呼び出し
  FragTrap: 親埋め込み + clap_init 呼び出し
  → 兄弟が増えるほど重複コードが増える

C++ の場合: 子クラスの数 × 1 行宣言だけ
  class ScavTrap : public ClapTrap { ... };
  class FragTrap : public ClapTrap { ... };
  → 兄弟が増えても親のロジックは 1 箇所
```

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
FragTrap a("Frag") を作る
  |
  ├→ [1] ClapTrap コンストラクタ（親が先）
  |      "ClapTrap constructor called for Frag"
  |
  └→ [2] FragTrap コンストラクタ（子が後）
         HP=100, EP=100, AD=30 に上書き
         "FragTrap constructor called for Frag"
  v
a.attack("Enemy")
  → ClapTrap::attack が呼ばれる（オーバーライドなし）
  → "ClapTrap Frag attacks ..."
  v
a.highFivesGuys()
  → FragTrap 固有メソッド
  → "FragTrap Frag requests a positive high five!"
  v
main 終了
  |
  ├→ [1] FragTrap デストラクタ（子が先）
  |
  └→ [2] ClapTrap デストラクタ（親が後）
```

### FragTrap.hpp

```cpp title="FragTrap.hpp" linenums="1"
#ifndef FRAGTRAP_HPP
# define FRAGTRAP_HPP

// 親クラスの定義が必要
# include "ClapTrap.hpp"

// ── ScavTrap と同じパターン ──
// class Child : public Parent
class FragTrap : public ClapTrap    // (1)
{
public:
    // OCF は子クラスでも自分で書く
    FragTrap(void);
    FragTrap(const std::string& name);
    FragTrap(const FragTrap& other);
    FragTrap& operator=(
        const FragTrap& other);
    ~FragTrap(void);

    // FragTrap 固有メソッド
    void highFivesGuys(void);        // (2)

    // attack のオーバーライドは「書かない」
    // → ClapTrap::attack がそのまま使われる
};

#endif
```

1. **`class FragTrap : public ClapTrap`** —
   ScavTrap と同じく ClapTrap を public 継承。
2. **`highFivesGuys()`** — FragTrap 固有のメソッド。
   attack のオーバーライド宣言はない。

### FragTrap.cpp（コンストラクタ）

```cpp title="FragTrap.cpp (コンストラクタ)" linenums="1"
#include "FragTrap.hpp"
#include <iostream>

// ── デフォルトコンストラクタ ──
// 親の ClapTrap() を明示的に呼ぶ
FragTrap::FragTrap(void)
    : ClapTrap()
{
    _hitPoints = 100;
    _energyPoints = 100;              // (1)
    _attackDamage = 30;
    std::cout
        << "FragTrap default "
        << "constructor called"
        << std::endl;
}

// ── 名前付きコンストラクタ ──
FragTrap::FragTrap(
    const std::string& name)
    : ClapTrap(name)
{
    _hitPoints = 100;
    _energyPoints = 100;
    _attackDamage = 30;
    std::cout
        << "FragTrap constructor "
        << "called for " << _name
        << std::endl;
}

// ── コピーコンストラクタ ──
// 親の ClapTrap(other) に委譲
FragTrap::FragTrap(
    const FragTrap& other)
    : ClapTrap(other)
{
    std::cout
        << "FragTrap copy constructor "
        << "called for " << _name
        << std::endl;
}

// ── 代入演算子 ──
FragTrap& FragTrap::operator=(
    const FragTrap& other)
{
    std::cout
        << "FragTrap copy assignment "
        << "operator called"
        << std::endl;
    if (this != &other)
        // 親の operator= に委譲
        ClapTrap::operator=(other);   // (2)
    return *this;
}

// ── デストラクタ ──
FragTrap::~FragTrap(void)
{
    std::cout
        << "FragTrap destructor "
        << "called for " << _name
        << std::endl;
}
```

1. **EP = 100** — ScavTrap（EP=50）と違って FragTrap はエネルギーが豊富。
   AD=30 で攻撃力も高い。
2. **親 operator= への委譲** — ScavTrap と同じパターン。
   共通メンバのコピーを ClapTrap に任せる。

### FragTrap.cpp（固有メソッド）

```cpp title="FragTrap.cpp (固有メソッド)" linenums="1"
// ── FragTrap 独自メソッド ──
// ClapTrap にはない機能
void FragTrap::highFivesGuys(void)
{
    std::cout << "FragTrap " << _name
        << " requests a positive "
        << "high five!" << std::endl;
}
```

!!! info "なぜ FragTrap は attack をオーバーライドしないのか"
    subject に「FragTrap は attack をオーバーライドせよ」という指示がないため。
    結果として `a.attack("Enemy")` は ClapTrap::attack が呼ばれ、
    メッセージは `"ClapTrap Frag attacks..."` になります。
    ScavTrap との違いを理解するための重要ポイント。

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex02/"
    > "Files to turn in: Makefile, ClapTrap.{h, hpp}, ClapTrap.cpp,
    >  ScavTrap.{h, hpp}, ScavTrap.cpp, FragTrap.{h, hpp}, FragTrap.cpp, main.cpp"

    ex01 と同じパターンが FragTrap でも書けるか、
    attack のオーバーライドが「ない」ことの理解が評価のポイント。

- [ ] `make` が警告なく通る
- [ ] FragTrap が `public ClapTrap` で継承している
- [ ] 構築順序が親→子になっている
- [ ] 破棄順序が子→親になっている
- [ ] attack のメッセージが "ClapTrap ..." になっている（オーバーライドなし）
- [ ] highFivesGuys() が動作する
- [ ] ScavTrap と FragTrap を同じ main で使える

---

## 8. テストチェックリスト

### 構築/破棄順序

- [ ] FragTrap 生成時に ClapTrap コンストラクタが **先に** 呼ばれる
- [ ] FragTrap 破棄時に FragTrap デストラクタが **先に** 呼ばれる
- [ ] ClapTrap デストラクタが **後に** 呼ばれる

### ステータス

- [ ] FragTrap の HP = 100, EP = 100, AD = 30
- [ ] attack で 30 ダメージが表示される

### メソッド

- [ ] `attack()` が "ClapTrap" と表示する（オーバーライドなし）
- [ ] `highFivesGuys()` がハイタッチメッセージを表示する

### ScavTrap との共存

- [ ] ScavTrap と FragTrap を同じ main で使える
- [ ] それぞれ独立したオブジェクトとして動作する
- [ ] 破棄順序が正しい

### OCF / 規約

- [ ] OCF が全て実装されている
- [ ] `using namespace std;` なし
- [ ] `printf` / `malloc` / `free` 不使用

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| FragTrap と ScavTrap の関係は？ | 兄弟クラス。どちらも ClapTrap を親に持つ、お互いに独立 |
| なぜ FragTrap は attack をオーバーライドしない？ | subject がオーバーライドを要求していないから |
| オーバーライドしないとどうなる？ | 親 ClapTrap の attack がそのまま使われる |
| attack のメッセージに "FragTrap" は出る？ | 出ない。"ClapTrap ..." になる（親の attack が呼ばれる） |
| ScavTrap と違う点は？ | EP/AD の値、固有メソッド、attack のオーバーライドの有無 |
| FragTrap のステータスは？ | HP=100, EP=100, AD=30 |
| ScavTrap のステータスは？ | HP=100, EP=50, AD=20 |
| 継承の書き方は？ | `class FragTrap : public ClapTrap` |
| 親コンストラクタの呼び方は？ | 初期化子リストで `: ClapTrap(name)` |
| 構築/破棄順序は？ | 構築は親→子、破棄は子→親（LIFO） |

---

## 10. よくあるミス

!!! warning "ScavTrap のコードをコピーして attack のオーバーライドを残す"
    ```cpp
    // ❌ FragTrap は attack をオーバーライドしない
    // ScavTrap からコピペして残してしまうミス
    void FragTrap::attack(
        const std::string& target) { ... }

    // ✅ attack の宣言/定義を書かない
    // → ClapTrap::attack が使われる
    ```

!!! warning "ステータス値の間違い"
    ScavTrap と FragTrap でステータスが異なる。
    コピペ時に値を変え忘れないこと。

    | | HP | EP | AD |
    |---|---|---|---|
    | ClapTrap | 10 | 10 | 0 |
    | ScavTrap | 100 | **50** | **20** |
    | FragTrap | 100 | **100** | **30** |

!!! warning "ClapTrap.hpp の include を忘れる"
    ```cpp
    // ❌ include を忘れる
    // → ClapTrap の定義が見つからず
    //   コンパイルエラー
    #ifndef FRAGTRAP_HPP
    # define FRAGTRAP_HPP
    class FragTrap : public ClapTrap {
        // コンパイルエラー
    };

    // ✅ include を書く
    #ifndef FRAGTRAP_HPP
    # define FRAGTRAP_HPP
    # include "ClapTrap.hpp"    // ← 必須
    class FragTrap : public ClapTrap {
        // OK
    };
    ```

!!! warning "固有メソッド名のタイプミス"
    `highFivesGuys`（複数形・末尾にs）が正しい。
    `highFiveGuy` や `HighFivesGuys`（大文字始まり）はダメ。
    subject の表記に厳密に従うこと。

---

## 11. 次の exercise へ

次の [ex03 DiamondTrap](ex03-diamondtrap.md) は **cpp03 最大の山場**。

ScavTrap と FragTrap の **両方** を継承する DiamondTrap を作り、
**ダイヤモンド問題** と **virtual 継承** を学びます。
ここまでの理解がしっかりしていないと迷子になるので、
ex00〜ex02 を確実に理解してから進みましょう。
