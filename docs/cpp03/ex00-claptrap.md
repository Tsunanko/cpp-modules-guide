# ex00 — ClapTrap

---

## このプログラムは何？

**ロボット同士が戦うプログラム** です。

`ClapTrap` というロボットのクラスを作ります。
このロボットは **HP（体力）** と **EP（エネルギー）** を持っていて、
**攻撃（attack）** と **ダメージを受ける（takeDamage）** と
**回復（beRepaired）** ができます。

このロボットは **これから作る全クラスの親（ベース）** になります。
だから、しっかり設計することが大事です。

```
ClapTrap "Alice"
  HP: 10       ← 体力
  EP: 10       ← エネルギー
  AD: 0        ← 攻撃力

Alice.attack("Bob")    → EP を 1 使って Bob に攻撃
Alice.takeDamage(5)    → HP を 5 減らす
Alice.beRepaired(3)    → EP を 1 使って HP を 3 回復
```

---

## 1. この exercise で学ぶこと

- **基底クラス（親クラス）** の作り方
- **`unsigned int`** という「負になれない整数型」の使い方
- **エネルギー管理** の仕組み（行動にコストがかかる設計）
- **コンストラクタ/デストラクタのメッセージ出力**
- **OCF（Orthodox Canonical Form）** の4つの関数

---

## 2. 新しい概念の解説

### 基底クラスって何？

**継承のベースになる（親になる）クラス** のことです。

cpp03 では、まず ClapTrap という基底クラスを作ります。
次の ex01 で ScavTrap がこの ClapTrap を「継承」します。
ex02 では FragTrap が、ex03 では DiamondTrap が同じく継承します。

```
     ClapTrap ← これが基底クラス（親）
    /   |   \
ScavTrap FragTrap DiamondTrap
   ↑ 全部 ClapTrap の子クラス
```

だから、ClapTrap の設計は **すごく大事**。
ここでバグがあると、全ての子クラスに影響します。

### `unsigned int` って何？

**符号なし整数型** — つまり **負の数になれない整数** です。

```cpp
int          x = -5;   // OK（負の数にできる）
unsigned int y = -5;   // ダメ！ 変な巨大な数になる
```

| | `int` | `unsigned int` |
|---|---|---|
| 範囲 | 約 -21億 〜 +21億 | 0 〜 約 42億 |
| 負の数 | できる | できない |
| 使う場面 | 普通の数 | 「絶対マイナスにならない」値 |

### なぜ HP/EP に unsigned int を使うの？

**HP や EP は負の数になることが絶対にない** からです。

- HP が -5 のロボット → 存在しない（0 で死亡）
- EP が -3 のロボット → 存在しない（0 で行動不能）

型レベルで「負の値はあり得ない」と宣言することで、バグを防ぎます。

!!! warning "unsigned int のワナ: アンダーフロー"
    `unsigned int` は 0 より小さくなれないので、引き算で **おかしな値になる** ことがあります。

    ```cpp
    unsigned int hp = 3;
    unsigned int damage = 5;
    hp -= damage;   // 期待: -2
                    // 実際: 約 42 億！（アンダーフロー）
    ```

    対策: 引き算の前にチェックする。

    ```cpp
    if (damage >= hp)
        hp = 0;       // 0 にクランプ（固定）
    else
        hp -= damage; // 安全に引き算
    ```

### なぜエネルギーチェックが必要？

**無限に行動できないようにする** ためです。

ClapTrap は attack と beRepaired のたびに EP を 1 消費します。
EP が 0 になると、もうこれらの行動はできません。

```
初期 EP: 10
attack → EP: 9
attack → EP: 8
...
attack → EP: 0
attack → 「can't attack, no energy points!」と表示して何もしない
```

これは **リソース管理** の練習です。ゲームでもよく使うパターン。

### コンストラクタ/デストラクタのメッセージ

クラスのオブジェクトを作ると **コンストラクタ** が自動で呼ばれ、
オブジェクトが消えると **デストラクタ** が自動で呼ばれます。

subject は「コンストラクタ/デストラクタで **メッセージを出力する** こと」を要求しています。
なぜ？ → 評価のとき「いつ作られていつ消えたか」を目で確認するため。

```cpp
ClapTrap a("Alice");  → "ClapTrap constructor called for Alice"
// ... 処理 ...
                       → "ClapTrap destructor called for Alice"
// （スコープを抜けるとき自動）
```

---

## 3. 課題仕様

### ClapTrap クラスのメンバ変数

| 属性 | 型 | 初期値 |
|------|----|--------|
| `_name` | `std::string` | コンストラクタ引数 |
| `_hitPoints` | `unsigned int` | 10 |
| `_energyPoints` | `unsigned int` | 10 |
| `_attackDamage` | `unsigned int` | 0 |

### メンバ関数

| 関数 | 動作 |
|------|------|
| `attack(target)` | EP を 1 消費、ダメージ量を表示 |
| `takeDamage(amount)` | HP を減らす |
| `beRepaired(amount)` | EP を 1 消費、HP を回復 |

### ルール

- HP = 0 または EP = 0 のとき attack / beRepaired は実行不可
- コンストラクタ/デストラクタでメッセージを出力
- **OCF**: デフォルトコンストラクタ、コピーコンストラクタ、代入演算子、デストラクタ
- Makefile: `all`, `clean`, `fclean`, `re`

---

## 4. 実行例

```console
$ ./ClapTrap
=== Creating ClapTraps ===
ClapTrap constructor called for Alice
ClapTrap constructor called for Bob

=== Combat ===
ClapTrap Alice attacks Bob, causing 0 points of damage!
ClapTrap Bob takes 0 points of damage! HP: 10
ClapTrap Bob attacks Alice, causing 0 points of damage!
ClapTrap Alice takes 0 points of damage! HP: 10

=== Repair ===
ClapTrap Alice repairs itself for 5 hit points! HP: 15

=== Heavy damage ===
ClapTrap Alice takes 100 points of damage! HP: 0
ClapTrap Alice can't attack, no hit points!
ClapTrap Alice can't repair, no hit points!

=== Destruction ===
ClapTrap destructor called for Bob
ClapTrap destructor called for Alice
```

**注目ポイント**:

- `_attackDamage` の初期値は 0 → 攻撃しても 0 ダメージ
- HP が 0 になると一切の行動ができない
- 破棄は **構築の逆順**（最後に作った方から消える）

---

## 5. C と C++ の比較

### クラスって C で言うと何？

**C の「構造体 + それを操作する関数群」を、
1 つの箱にまとめた仕組み** です。

C でオブジェクトっぽいものを書くと、毎回こうなります:

```c
/* ステップ1: データを構造体で定義 */
/* typedef struct { ... } t_claptrap; */

/* ステップ2: 初期化関数を自作 */
/* void ct_init(t_claptrap *ct, ...); */

/* ステップ3: 操作関数を毎回 ポインタ渡しで自作 */
/* void ct_attack(t_claptrap *ct, ...); */
/* void ct_take_damage(t_claptrap *ct, ...); */

/* ステップ4: 使う時は構造体と関数を両方渡す */
/* ct_attack(&a, "Bob"); */
```

C++ のクラスはこれを **1 つの class {} にまとめる** 仕組み:

| C で書いていたもの | C++ のクラス |
|-------------------|------------|
| `struct { ... }` | `class { ... }` |
| `ct_init()` (自作) | コンストラクタ (自動) |
| `ct_attack(&a, ...)` | `a.attack(...)` |
| 関数に毎回ポインタ渡し | this が自動で渡される |

### 並べて比較

=== "C の書き方"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>
    /* strcpy 用のヘッダ */
    #include <string.h>

    /* ── 構造体でデータをまとめる ── */
    /* C には class がないので struct を使う */
    typedef struct s_claptrap {
        /* 名前 (50 文字の配列) */
        char         name[50];
        /* HP (体力) */
        unsigned int hp;
        /* EP (エネルギー) */
        unsigned int ep;
        /* AD (攻撃力) */
        unsigned int ad;
    } t_claptrap;

    /* ── 初期化関数 (自作) ── */
    /* 毎回ポインタ渡しで書く */
    void ct_init(t_claptrap *ct,
                 const char *name) {
        /* strcpy: 文字列をコピーする C 関数 */
        strcpy(ct->name, name);
        /* メンバを1つずつ埋める */
        ct->hp = 10;
        ct->ep = 10;
        ct->ad = 0;
    }

    /* ── 攻撃関数 (自作) ── */
    /* 毎回 ct ポインタを渡す必要あり */
    void ct_attack(t_claptrap *ct,
                   const char *target) {
        /* HP/EP チェック (自分で書く) */
        if (ct->hp == 0 || ct->ep == 0)
            return;
        /* EP を 1 消費 */
        ct->ep--;
        /* printf: C の出力関数 */
        printf("%s attacks %s\n",
               ct->name, target);
    }

    int main(void) {
        /* 変数宣言 (初期化は手動) */
        t_claptrap a;
        /* 初期化関数を手動で呼ぶ (忘れがち) */
        ct_init(&a, "Alice");
        /* 攻撃関数にポインタを渡す */
        ct_attack(&a, "Bob");
        /* 後片付け関数も手動で呼ぶ必要あり */
        return 0;
    }
    ```

=== "C++ の書き方"

    ```cpp
    // cout 用のヘッダ
    #include <iostream>
    // std::string 用のヘッダ
    #include <string>

    // ── クラスでデータと関数をまとめる ──
    class ClapTrap {
    // private: 外から直接触れない
    private:
        // std::string は自動でサイズ管理
        std::string  _name;
        // unsigned int: 負にならない整数
        unsigned int _hitPoints;
        unsigned int _energyPoints;
        unsigned int _attackDamage;

    // public: クラス外から使える
    public:
        // ── コンストラクタ ──
        // ClapTrap a("Alice") で自動で呼ばれる
        // : _name(name) は初期化子リスト
        ClapTrap(const std::string& name)
            : _name(name),
              _hitPoints(10),
              _energyPoints(10),
              _attackDamage(0) {}

        // ── メンバ関数 ──
        // ct ポインタを渡す必要なし
        // this が自動で渡される
        void attack(
            const std::string& target) {
            // HP/EP チェック
            if (_hitPoints == 0
                || _energyPoints == 0)
                return;
            // EP を 1 消費
            _energyPoints--;
            // std::cout で出力
            std::cout << _name
                << " attacks " << target
                << std::endl;
        }
    };

    int main(void) {
        // ClapTrap a("Alice") で
        // コンストラクタが自動で呼ばれる
        ClapTrap a("Alice");
        // a.attack(...) で自然に呼べる
        a.attack("Bob");
        // スコープ終了でデストラクタが自動実行
        return 0;
    }
    ```

**ステップ数で比較すると:**

```
C の場合: 4 ステップ
  ①構造体を定義
  ②初期化関数を自作して呼ぶ
  ③操作関数に毎回ポインタを渡す
  ④後片付けも手動

C++ の場合: 1 ステップ
  ①クラスを定義するだけ
    コンストラクタ、メンバ関数、
    デストラクタが自動で連動する
```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|------------|
| `struct` + 関数 | `class`（1つにまとめる） | データと処理が一体 |
| `ct_init(&a, "Alice")` | `ClapTrap a("Alice")` | 初期化が自動 |
| `ct_attack(&a, "Bob")` | `a.attack("Bob")` | ポインタを渡さない |
| `printf` | `std::cout` | 出力方法が変わった |

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
ClapTrap a("Alice") を作る
  |  → コンストラクタが呼ばれる
  |  → "ClapTrap constructor called for Alice"
  v
a.attack("Bob")
  |  → HP/EP チェック
  |  → EP を 1 減らす
  |  → ダメージ量を表示
  v
b.takeDamage(5)
  |  → HP から 5 引く（0 未満にならないよう保護）
  v
a.beRepaired(3)
  |  → EP を 1 減らす
  |  → HP に 3 足す
  v
main 終了
  |  → 逆順でデストラクタが呼ばれる
  |  → "ClapTrap destructor called for Bob"
  |  → "ClapTrap destructor called for Alice"
  v
終了
```

### ClapTrap.hpp（ヘッダファイル）

```cpp title="ClapTrap.hpp" linenums="1"
#ifndef CLAPTRAP_HPP
# define CLAPTRAP_HPP

// std::string を使うために必要
# include <string>

// ── ClapTrap クラスの定義 ──
class ClapTrap
{
// public: クラス外から自由にアクセス可能
public:
    // OCF（4つの特別な関数）
    ClapTrap(void);                        // (1)
    ClapTrap(const std::string& name);     // (2)
    ClapTrap(const ClapTrap& other);       // (3)
    ClapTrap& operator=(
        const ClapTrap& other);            // (4)
    ~ClapTrap(void);                       // (5)

    // 戦闘メソッド
    void attack(const std::string& target);
    void takeDamage(unsigned int amount);
    void beRepaired(unsigned int amount);

// private: クラス外からアクセス不可
// ex01 では protected に変える！
private:
    std::string  _name;
    unsigned int _hitPoints;
    unsigned int _energyPoints;
    unsigned int _attackDamage;
};

#endif
```

1. **デフォルトコンストラクタ** — 引数なしで作るとき
2. **名前付きコンストラクタ** — `ClapTrap a("Alice")`
3. **コピーコンストラクタ** — `ClapTrap b(a)`（既存から複製）
4. **代入演算子** — `b = a`（既存への代入）
5. **デストラクタ** — オブジェクトが消えるとき

!!! info "なぜ今は private？ ex01 で protected に変える理由"
    ex00 ではまだ派生クラスがないので `private` で十分です。
    でも ex01 で ScavTrap が ClapTrap を継承すると、
    ScavTrap のコンストラクタで `_hitPoints = 100;` と書きたい。
    `private` のままだと派生クラスからも触れないので、
    **ex01 で `protected` に格上げ** します。

### ClapTrap.cpp（コンストラクタ4種 + デストラクタ）

```cpp title="ClapTrap.cpp" linenums="1"
#include "ClapTrap.hpp"
#include <iostream>

// ── デフォルトコンストラクタ ──
// 引数なし版。"default" という名前で初期化。
// : _name(...), _hitPoints(...), ...
// ↑ 初期化子リスト
//   本体 {} に入る前にメンバを初期化
ClapTrap::ClapTrap(void)
    : _name("default"),
      _hitPoints(10),
      _energyPoints(10),
      _attackDamage(0)
{
    std::cout
        << "ClapTrap default "
        << "constructor called"
        << std::endl;
}

// ── 名前付きコンストラクタ ──
// "Alice" などを渡せる版
ClapTrap::ClapTrap(
    const std::string& name)
    : _name(name),
      _hitPoints(10),
      _energyPoints(10),
      _attackDamage(0)
{
    std::cout
        << "ClapTrap constructor "
        << "called for " << _name
        << std::endl;
}

// ── コピーコンストラクタ ──
// ClapTrap b(a); のように
// 別のオブジェクトから複製するとき
ClapTrap::ClapTrap(const ClapTrap& other)
    : _name(other._name),
      _hitPoints(other._hitPoints),
      _energyPoints(other._energyPoints),
      _attackDamage(other._attackDamage)
{
    std::cout
        << "ClapTrap copy constructor "
        << "called for " << _name
        << std::endl;
}

// ── 代入演算子 ──
// b = a; のときに呼ばれる
ClapTrap& ClapTrap::operator=(
    const ClapTrap& other)
{
    std::cout
        << "ClapTrap copy assignment "
        << "operator called"
        << std::endl;
    // 自己代入チェック（a = a; で壊れないように）
    if (this != &other)
    {
        _name = other._name;
        _hitPoints = other._hitPoints;
        _energyPoints = other._energyPoints;
        _attackDamage = other._attackDamage;
    }
    // *this で自分自身を返す（連鎖代入可能に）
    return *this;
}

// ── デストラクタ ──
// オブジェクトが消えるとき自動で呼ばれる
ClapTrap::~ClapTrap(void)
{
    std::cout
        << "ClapTrap destructor "
        << "called for " << _name
        << std::endl;
}
```

!!! info "初期化子リストって何？"
    `: _name(name), _hitPoints(10), ...` の部分。

    コンストラクタの本体 `{}` に入る **前に** メンバを初期化する書き方です。
    本体で `_name = name;` と代入するより **効率が良い** ので、
    常にこの書き方を使うのがC++の作法。

### ClapTrap.cpp（戦闘メソッド）

```cpp title="ClapTrap.cpp (戦闘メソッド)" linenums="1"
// ── 攻撃 ──
void ClapTrap::attack(
    const std::string& target)
{
    // (1) HP チェック（死亡中は行動不可）
    if (_hitPoints == 0)
    {
        std::cout << "ClapTrap " << _name
            << " can't attack, "
            << "no hit points!"
            << std::endl;
        return;
    }
    // (2) EP チェック（エネルギー切れ）
    if (_energyPoints == 0)
    {
        std::cout << "ClapTrap " << _name
            << " can't attack, "
            << "no energy points!"
            << std::endl;
        return;
    }
    // (3) EP を 1 消費
    _energyPoints--;
    // (4) 攻撃メッセージ
    std::cout << "ClapTrap " << _name
        << " attacks " << target
        << ", causing " << _attackDamage
        << " points of damage!"
        << std::endl;
}

// ── ダメージを受ける ──
void ClapTrap::takeDamage(
    unsigned int amount)
{
    if (_hitPoints == 0)
    {
        std::cout << "ClapTrap " << _name
            << " is already dead!"
            << std::endl;
        return;
    }
    // (5) アンダーフロー防止！
    //     amount >= _hitPoints なら 0 にする
    if (amount >= _hitPoints)
        _hitPoints = 0;
    else
        _hitPoints -= amount;
    std::cout << "ClapTrap " << _name
        << " takes " << amount
        << " points of damage! HP: "
        << _hitPoints << std::endl;
}

// ── 回復 ──
void ClapTrap::beRepaired(
    unsigned int amount)
{
    if (_hitPoints == 0)
    {
        std::cout << "ClapTrap " << _name
            << " can't repair, "
            << "no hit points!"
            << std::endl;
        return;
    }
    if (_energyPoints == 0)
    {
        std::cout << "ClapTrap " << _name
            << " can't repair, "
            << "no energy points!"
            << std::endl;
        return;
    }
    // EP を 1 消費して HP を回復
    _energyPoints--;
    _hitPoints += amount;
    std::cout << "ClapTrap " << _name
        << " repairs itself for "
        << amount << " hit points! HP: "
        << _hitPoints << std::endl;
}
```

1. **HP チェック** — 死亡中は行動不可
2. **EP チェック** — エネルギー切れでも行動不可
3. **EP 消費** — attack と beRepaired は EP を 1 消費、takeDamage は消費しない
4. **メッセージ出力** — ダメージ量とターゲット名を表示
5. **アンダーフロー防止** — `unsigned int` なので `3 - 5` は -2 ではなく約 42 億になる

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex00/"
    > "Files to turn in: Makefile, ClapTrap.{h, hpp}, ClapTrap.cpp, main.cpp"

    ClapTrap クラスの実装、OCF、エネルギーシステムの動作が評価のポイント。

- [ ] `make` が警告なく通る
- [ ] コンストラクタ/デストラクタのメッセージが出力される
- [ ] `attack()` がダメージ量を表示する
- [ ] `takeDamage()` が HP を減らす
- [ ] `beRepaired()` が HP を回復する
- [ ] HP = 0 または EP = 0 で行動不可
- [ ] OCF（4つの特別な関数）が全て実装されている

---

## 8. テストチェックリスト

### 基本動作

- [ ] `make` が警告なく通る
- [ ] `attack()` が動作してダメージ量が表示される
- [ ] `takeDamage()` が HP を減らす
- [ ] `beRepaired()` が HP を回復する

### エネルギーシステム

- [ ] attack / beRepaired で EP が 1 減る
- [ ] EP = 0 のとき attack / beRepaired が実行されない
- [ ] EP を 10 回使い切ったら行動不能になる

### HP システム

- [ ] HP = 0 のとき全アクションが実行されない
- [ ] HP 以上のダメージを受けたとき HP が 0 になる（負にならない）
- [ ] beRepaired で HP が回復する

### エッジケース

- [ ] `takeDamage(0)` が正常動作する
- [ ] `beRepaired(0)` が正常動作する

### 規約

- [ ] `using namespace std;` なし
- [ ] `printf` / `malloc` / `free` 不使用
- [ ] OCF が全て実装されている

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| なぜ `unsigned int` を使う？ | HP/EP/AD は負の値にならないから。型レベルで「負は無効」と保証できる |
| `unsigned int` のワナは？ | アンダーフロー。`3 - 5` が -2 ではなく約 42 億になる |
| 対策は？ | 引き算の前に `amount >= _hitPoints` でチェックし、超えたら 0 にする |
| OCF の4つは？ | デフォルトコンストラクタ、コピーコンストラクタ、代入演算子、デストラクタ |
| 初期化子リストとは？ | コンストラクタ本体の前に `: _name(name)` でメンバを初期化する書き方 |
| 初期化子リストの利点は？ | 本体で代入するより効率が良い。const / 参照メンバには必須 |
| なぜ EP チェックが必要？ | 無限に行動できないようにするため（リソース管理） |
| なぜコンストラクタ/デストラクタでメッセージを出す？ | 評価のときにオブジェクトの生成/破棄タイミングを目で確認するため |
| 自己代入チェックとは？ | `a = a;` で壊れないようにする `if (this != &other)` の定番パターン |
| private と public の違い は？ | private はクラス外から触れない、public は自由にアクセス可能 |

---

## 10. よくあるミス

!!! warning "unsigned int のアンダーフロー"
    ```cpp
    // ❌ 危険: _hitPoints=3, amount=5 のとき約42億になる
    _hitPoints -= amount;

    // ✅ 安全: 先にチェックして 0 にクランプ
    if (amount >= _hitPoints)
        _hitPoints = 0;
    else
        _hitPoints -= amount;
    ```

!!! warning "EP チェックの欠落"
    ```cpp
    // ❌ EP チェックなし → 無限に攻撃できてしまう
    void ClapTrap::attack(
        const std::string& target) {
        std::cout << "attacks "
                  << target << std::endl;
    }

    // ✅ HP と EP の両方をチェック
    void ClapTrap::attack(
        const std::string& target) {
        if (_hitPoints == 0
            || _energyPoints == 0)
            return;
        _energyPoints--;
        // ...
    }
    ```

!!! warning "デストラクタメッセージの欠落"
    subject はコンストラクタ/デストラクタでメッセージを出力することを要求している。
    評価時に「どのオブジェクトがいつ破棄されたか」を確認するため、
    デストラクタのメッセージは必須。

!!! warning "自己代入チェックの省略"
    ```cpp
    // ❌ a = a; したとき自分を壊してしまう可能性
    ClapTrap& operator=(const ClapTrap& other) {
        _name = other._name;
        // ...
        return *this;
    }

    // ✅ 自己代入チェックあり
    ClapTrap& operator=(const ClapTrap& other) {
        if (this != &other) {
            _name = other._name;
            // ...
        }
        return *this;
    }
    ```

---

## 11. 次の exercise へ

次の [ex01 ScavTrap](ex01-scavtrap.md) では、**継承** の第一歩を学びます。

ClapTrap を親クラスとして、その機能を受け継ぐ ScavTrap を作ります。
今 `private` になっているメンバを `protected` に変える理由、
**構築順序（親→子）** と **破棄順序（子→親）** も体験します。
