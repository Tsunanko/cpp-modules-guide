# cpp03 評価対策

## このモジュールの評価テーマ

> **クラスの継承、構築/破棄順序、ダイヤモンド問題と virtual 継承を理解しているか**

---

## 評価前チェック（全 exercise 共通）

以下を **1 つでも満たしていなければ、そもそも採点対象外** になる。

- [ ] `c++ -Wall -Wextra -Werror -std=c++98` でコンパイルが通る
- [ ] ヘッダに関数の実装がない（テンプレートを除く）
- [ ] Makefile が `c++` コンパイラと必須フラグを使っている

!!! danger "即 Forbidden Function フラグ（= 0点）"
    以下のいずれかに該当すると、**その exercise は 0 点**。

    - C 関数の使用: `*alloc`, `*printf`, `free`
    - `using namespace std;` の使用
    - `friend` キーワードの使用
    - 外部ライブラリの使用
    - C++98 以外の機能（`auto`, `nullptr`, ラムダ, range-for 等）

---

## Exercise 別 Q&A

### Ex00: ClapTrap

!!! note "評価形式: 基本動作確認 + 口頭質問"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| ClapTrap の初期値は？ | HP=10, EP=10, AD=0 | 仕様通りの初期化を確認 |
| EP=0 のときどうなる？ | attack も beRepaired も実行できない。メッセージのみ表示 | エネルギーシステムの理解 |
| HP=0 のときどうなる？ | 全アクション不可。takeDamage も "already dead" と表示 | 死亡状態の処理 |
| unsigned int で注意すべき点は？ | アンダーフロー。`3 - 5` が約 42 億になる。先に比較してクランプする | C/C++ 共通の型の罠 |
| OCF とは？ | Orthodox Canonical Form。デフォルトコンストラクタ、コピーコンストラクタ、代入演算子、デストラクタの 4 つ | C++ の基本作法 |

!!! info "よく聞かれる口頭質問"
    **Q: なぜ _attackDamage の初期値は 0 なのか？**

    A: ClapTrap は最も基本的なユニット。攻撃力 0 は「弱い」ことを表現する設計。
    派生クラス (ScavTrap, FragTrap) でより高い値に設定する。

    **Q: attack と beRepaired で EP を消費するのに takeDamage で消費しないのはなぜ？**

    A: takeDamage は**受動的なアクション** (攻撃を受ける) なので、エネルギーを消費しない。
    attack と beRepaired は**能動的なアクション**なのでコストがかかる。

---

### Ex01: ScavTrap

!!! note "評価形式: 構築/破棄順序の確認 + 口頭質問 (最も質問が多い)"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| なぜ private を protected に変えた？ | 派生クラス (ScavTrap) から `_hitPoints` 等に直接アクセスするため。private のままでは派生クラスからもアクセスできない | **protected の理解** |
| 構築順序は？ | **ClapTrap → ScavTrap** (親→子)。親が先に完成しないと、子が親のメンバを使えない | **構築順序の理解** |
| 破棄順序は？ | **ScavTrap → ClapTrap** (子→親)。構築の逆順 (LIFO) | **破棄順序の理解** |
| なぜ attack をオーバーライドした？ | ClapTrap::attack は "ClapTrap" と表示するが、ScavTrap のインスタンスには "ScavTrap" と表示したいため | メソッドオーバーライドの目的 |
| guardGate の出力は？ | ScavTrap が Gate keeper モードに入ったことを示すメッセージ | 固有メソッドの確認 |

!!! info "よく聞かれる口頭質問"
    **Q: コンストラクタの初期化リストで親クラスのコンストラクタをどう呼ぶ？**

    A: `ScavTrap(name) : ClapTrap(name) { ... }` のように初期化リストに書く。
    省略するとデフォルトコンストラクタが呼ばれるが、明示的に書くのが推奨。

    **Q: public 継承と private 継承の違いは？**

    A: public 継承は「is-a 関係」。ScavTrap は ClapTrap の一種。親の public メンバは外部からも呼べる。
    private 継承は「実装の流用」目的。親の public メンバも外部からは呼べなくなる。
    ここでは is-a 関係が適切なので public 継承を使う。

---

### Ex02: FragTrap

!!! note "評価形式: Ex01 と同じパターンの確認"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| FragTrap のステータスは？ | HP=100, EP=100, AD=30 | ScavTrap との違いを確認 |
| 構築/破棄順序は？ | ClapTrap → FragTrap / FragTrap → ClapTrap | ex01 と同じパターン |
| attack はオーバーライドしている？ | していない。ClapTrap::attack がそのまま使われる | ScavTrap との違いの理解 |
| highFivesGuys の出力は？ | ポジティブなハイタッチをリクエストするメッセージ | 固有メソッドの確認 |

---

### Ex03: DiamondTrap -- **最も重要**

!!! note "評価形式: ダイヤモンド問題の理解が核心"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| ダイヤモンド問題とは？ | 多重継承で共通の基底クラスが 2 つ存在してしまう問題。_name や _hitPoints が 2 セット作られ、どちらを使うか曖昧になる | **最重要概念** |
| virtual 継承はどう解決する？ | ScavTrap と FragTrap が `virtual public ClapTrap` で継承すると、DiamondTrap 内の ClapTrap が **1 つだけ**になる | virtual の仕組み |
| 構築順序は？ | ClapTrap → FragTrap → ScavTrap → DiamondTrap。virtual 基底が最初、次に宣言順、最後に自身 | **virtual 基底の構築順序** |
| 破棄順序は？ | DiamondTrap → ScavTrap → FragTrap → ClapTrap。構築の完全な逆順 | LIFO の確認 |
| whoAmI は何を表示する？ | DiamondTrap::_name ("Diamond") と ClapTrap::_name ("Diamond_clap_name") の両方 | スコープ解決の理解 |
| using ScavTrap::attack の意味は？ | 多重継承で attack が曖昧になるため、ScavTrap 版を使うと明示する | using 宣言の理解 |

!!! info "よく聞かれる口頭質問"
    **Q: なぜ ClapTrap のコンストラクタを DiamondTrap から直接呼ぶ？**

    A: virtual 基底クラスは DiamondTrap 内に 1 つしかない。FragTrap と ScavTrap のどちらが
    初期化するか曖昧になるため、**最も派生したクラスが責任を持つ** (C++ の言語仕様)。
    FragTrap/ScavTrap の初期化リスト内の ClapTrap() は無視される。

    **Q: virtual 継承と virtual 関数は同じもの？**

    A: **全く別の概念**。virtual 継承は「基底クラスの共有」、virtual 関数は「ポリモーフィズム」(cpp04 で学ぶ)。
    キーワードが同じだが、目的も仕組みも異なる。

    **Q: DiamondTrap::_name と ClapTrap::_name はなぜ別の値？**

    A: DiamondTrap は独自の `_name` メンバを持っている。C++ のスコープ規則により、
    クラス内で `_name` と書くと最も近いスコープ (DiamondTrap 自身) の _name が参照される。
    `ClapTrap::_name` とスコープ解決演算子で明示して初めて親の _name にアクセスできる。

---

## 概念の深掘り

### ダイヤモンド継承の構造

```
                    ClapTrap
                  (virtual 基底)
                   _name: "X_clap_name"
                   _hitPoints: 10
                   _energyPoints: 10
                   _attackDamage: 0
                  /              \
                 /                \
          ScavTrap              FragTrap
        (virtual public)      (virtual public)
         HP→100               HP→100
         EP→50                EP→100
         AD→20                AD→30
         + guardGate()        + highFivesGuys()
         + attack() override
                 \                /
                  \              /
                   DiamondTrap
                    _name: "X"  (独自メンバ)
                    HP=100, EP=50, AD=30
                    using ScavTrap::attack
                    + whoAmI()
```

### virtual 基底クラスの初期化メカニズム

```
通常の継承:
  DiamondTrap → FragTrap → ClapTrap()  ← FragTrap が ClapTrap を初期化
  DiamondTrap → ScavTrap → ClapTrap()  ← ScavTrap も ClapTrap を初期化
  → ClapTrap が 2 回初期化される (2 つのインスタンス)

virtual 継承:
  DiamondTrap → ClapTrap()             ← 最派生クラスが直接初期化 (1 回だけ)
  DiamondTrap → FragTrap()             ← FragTrap 内の ClapTrap() は無視
  DiamondTrap → ScavTrap()             ← ScavTrap 内の ClapTrap() も無視
  → ClapTrap は 1 つだけ
```

### 構築/破棄順序まとめ

| クラス | 構築順 | 破棄順 |
|--------|--------|--------|
| ClapTrap | 1 | 1 |
| ScavTrap | ClapTrap(1) → ScavTrap(2) | ScavTrap(1) → ClapTrap(2) |
| FragTrap | ClapTrap(1) → FragTrap(2) | FragTrap(1) → ClapTrap(2) |
| DiamondTrap | ClapTrap(1) → FragTrap(2) → ScavTrap(3) → DiamondTrap(4) | DiamondTrap(1) → ScavTrap(2) → FragTrap(3) → ClapTrap(4) |

---

## 即不合格フラグ一覧

| フラグ | 条件 |
|--------|------|
| **Empty work** | 提出物が空 |
| **Incomplete work** | exercise が未完成 |
| **Invalid compilation** | `c++ -Wall -Wextra -Werror` でコンパイルできない |
| **Cheat** | 不正行為の疑い |
| **Crash** | segfault やクラッシュ |
| **Leaks** | メモリリーク |
| **Forbidden function** | C 関数, `using namespace`, `friend`, 外部ライブラリ等 |
| **Can't support / explain code** | **自分のコードを説明できない** |

!!! danger "「Can't support / explain code」に注意"
    コードが動いていても、**なぜそう書いたかを説明できなければ不合格**になりうる。
    特に以下は確実に答えられるようにしておくこと:

    - **protected にした理由** (ex01)
    - **構築/破棄順序とその理由** (ex01〜ex03)
    - **ダイヤモンド問題の説明と virtual の仕組み** (ex03)
    - **using 宣言の必要性** (ex03)
    - **whoAmI で 2 つの名前が表示される理由** (ex03)
