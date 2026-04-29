# cpp03 モジュール概要

---

## このモジュールは何をするの？

**「クラスの継承（けいしょう）」を学ぶモジュール** です。

「継承」とは、**親クラスの機能を子クラスがそのまま受け継ぐ** こと。
C言語の「構造体の埋め込み」を、もっと便利で安全にしたものです。

このモジュールでは、ClapTrap というロボットのクラスを作り、
それを親として ScavTrap（警備ロボ）や FragTrap（パーティロボ）を作ります。
最後は両方を同時に継承する DiamondTrap を作って、
**ダイヤモンド問題** という継承の落とし穴を学びます。

---

## 1. 継承って何？

**親クラスの機能を、子クラスがそのまま受け継ぐ仕組み** です。

例えるなら「親の車を子が受け継ぐ」感じ。
親の車（ClapTrap）にはエンジンとタイヤがあって、
子（ScavTrap）はそこに「警備モード」機能を追加したカスタム車。

```
    ClapTrap（親クラス）
      attack()       ← 攻撃
      takeDamage()   ← ダメージを受ける
      beRepaired()   ← 回復する
          |
          ↓ 受け継ぐ（継承する）
          |
    ScavTrap（子クラス）
      attack()       ← 親から受け継いで、さらに書き直す
      takeDamage()   ← 親のまま使う
      beRepaired()   ← 親のまま使う
      guardGate()    ← 子だけの新しい機能
```

### Cの構造体埋め込みとの違い

Cにも似たような書き方はあります。
でもC++の継承は、それを **もっと賢く** したものです。

```c
// C: 構造体の中に親構造体を埋め込む
typedef struct s_scavtrap {
    t_claptrap parent;   // 親を埋め込む
    // ScavTrap 独自のメンバ
} t_scavtrap;

// → 初期化を自分で呼ばないといけない
// → 関数の上書きもできない
// → アクセス制御もできない
```

```cpp
// C++: 継承の書き方
class ScavTrap : public ClapTrap {
    // 親の初期化は自動！
    // 関数の書き直しもOK！
    // アクセス制御もある！
};
```

---

## 2. なぜ継承を使うの？

**同じ処理を何度も書かなくて済む** からです。

もし継承がなかったら、ScavTrap と FragTrap の両方に
「HPを減らす処理」「EPを減らす処理」を別々にコピペしないといけません。
バグがあったら両方直す必要があります。

継承を使えば、**共通部分は親クラスに1回だけ書く** だけで済みます。
子クラスは独自の機能だけを書けばOK。

```
継承なし（コピペ）:
  ScavTrap → attack/takeDamage/beRepaired を全部書く
  FragTrap → attack/takeDamage/beRepaired を全部書く
  → バグを直すときに両方直す必要がある

継承あり:
  ClapTrap → attack/takeDamage/beRepaired を1回だけ書く
  ScavTrap → 独自の機能だけ書く（残りは親から受け継ぐ）
  FragTrap → 独自の機能だけ書く（残りは親から受け継ぐ）
  → バグを直すのは親の1箇所だけでOK
```

---

## 3. コンストラクタ・デストラクタの呼ばれる順番って何？

子クラスのオブジェクトを作ると、**親クラスのコンストラクタも自動で呼ばれます**。
このとき、呼ばれる順番はちゃんと決まっています。

### 構築の順番: 親 → 子

```
ScavTrap a("Alpha"); と書くと…

  1. ClapTrap のコンストラクタが呼ばれる（親が先）
     → _hitPoints=10, _energyPoints=10, _attackDamage=0

  2. ScavTrap のコンストラクタが呼ばれる（子が後）
     → _hitPoints=100, _energyPoints=50, _attackDamage=20
```

**なぜ親が先？** → 子は親の土台の上に作られるから。
家を建てるとき、土台を先に作ってから壁を立てるのと同じです。

### 破棄の順番: 子 → 親（構築の逆順）

```
関数が終わって a が消えるとき…

  1. ScavTrap のデストラクタが呼ばれる（子が先）

  2. ClapTrap のデストラクタが呼ばれる（親が後）
```

**なぜ子が先？** → 最後に作ったものを最初に壊す（**LIFO: Last In First Out**）。
子は親に依存しているので、親を先に壊すと子がおかしくなります。
だから親は最後まで残ります。

```
 構築: [ClapTrap] → [ScavTrap]   （土台から積み上げ）
 破棄: [ScavTrap] → [ClapTrap]   （上から順に取り壊し）
```

---

## 4. ダイヤモンド問題って何？

**2つの親クラスが、さらに同じ親クラスを持っているとき** に起きる問題です。

cpp03 の ex03 で登場する DiamondTrap がその例です。

```
          ClapTrap
         /        \
     ScavTrap   FragTrap      ← どちらも ClapTrap を継承
         \        /
       DiamondTrap             ← 両方を継承

      ↑ 形がひし形（ダイヤモンド）なので「ダイヤモンド継承」
```

何も対策をしないと、DiamondTrap の中に **ClapTrap が2つ** できてしまいます。

```
DiamondTrap の中身（virtualなし）:
  ┌──────────────────────┐
  │ ClapTrap (1つ目)     │ ← ScavTrap 経由
  │  _name = ?           │
  │  _hitPoints = ?      │
  ├──────────────────────┤
  │ ClapTrap (2つ目)     │ ← FragTrap 経由
  │  _name = ?           │   同じ変数が2個！
  │  _hitPoints = ?      │
  └──────────────────────┘

  → _name を読むとき「どっちの _name？」と曖昧になる
  → コンパイルエラー
```

解決するには **`virtual` 継承** を使います。詳しくは ex03 で解説します。

---

## 5. 学ぶ概念マップ

```
                        ClapTrap（ベースクラス）
                       /    |    \
                      /     |     \
                HP:10  EP:10  AD:0
                           |
              ┌────────────┴────────────┐
              │                         │
           ScavTrap                  FragTrap
        (public継承)              (public継承)
        HP:100 EP:50 AD:20       HP:100 EP:100 AD:30
        + guardGate()             + highFivesGuys()
        + attack() を書き直し
              │                         │
              └────────────┬────────────┘
                           │
                      DiamondTrap
                   （多重継承: ダイヤモンド）
            HP:FragTrap  EP:ScavTrap  AD:FragTrap
                + whoAmI()
                + using ScavTrap::attack
```

---

## 6. 4つのexerciseと所要時間

| # | 名前 | 難度 | 所要時間 | 主要トピック |
|---|---|:---:|:---:|---|
| [ex00](ex00-claptrap.md) | ClapTrap | ★★ | 1時間 | ベースクラス、HP/EP管理 |
| [ex01](ex01-scavtrap.md) | ScavTrap | ★★★ | 1〜2時間 | public継承、protected、構築/破棄順序、オーバーライド |
| [ex02](ex02-fragtrap.md) | FragTrap | ★★ | 30分〜1時間 | 兄弟クラスの練習 |
| [ex03](ex03-diamondtrap.md) | DiamondTrap | ★★★★★ | 3〜5時間 | 多重継承、ダイヤモンド問題、virtual継承、using宣言 |

!!! warning "ex03 がこのモジュール最大の山場"
    ex03 のダイヤモンド継承は cpp モジュール全体でもトップクラスの難所。
    ex00〜ex02 を確実に理解してから取り組むこと。

---

## 7. 共通ルール

- コンパイル: `c++ -Wall -Wextra -Werror -std=c++98`
- `using namespace std;` は **禁止**
- `printf` / `malloc` / `free` は **使用禁止**
- ヘッダに関数実装を書かない（テンプレートを除く）
- `friend` キーワードは禁止

---

## 8. アクセス修飾子の範囲（重要）

継承で一番大事なのが「**どこからアクセスできるか**」の制御です。

| 修飾子 | クラス内 | 派生クラス | クラス外 |
|:---:|:---:|:---:|:---:|
| `public`    | 〇 | 〇 | 〇 |
| `protected` | 〇 | 〇 | × |
| `private`   | 〇 | × | × |

!!! info "ex00 と ex01 の違い"
    - ex00 では ClapTrap のメンバ変数は `private`
    - ex01 からは `protected` に変更

    なぜ？ → 派生クラス（ScavTrap）から `_hitPoints` を書き換えるには
    `private` だと触れないので、`protected` に格上げする必要があるため。

---

次は [ex00 ClapTrap](ex00-claptrap.md) から始めましょう。
