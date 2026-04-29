# cpp04 評価対策

## このモジュールの評価テーマ

> **ポリモーフィズム、抽象クラス、インターフェース、ディープコピーを理解しているか**

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

!!! danger "OCF 特別チェック — cpp04 固有ルール"
    **インターフェースクラス以外の全てのクラスが Orthodox Canonical Form を満たしていなければ、
    そのexerciseは採点しない。**

    OCF = デフォルトコンストラクタ + コピーコンストラクタ + 代入演算子 + デストラクタ の 4 点セット。

    | クラス | OCF 必須？ | 理由 |
    |--------|:---:|------|
    | Animal | YES | 抽象クラスでも OCF は必要 |
    | Dog | YES | |
    | Cat | YES | |
    | Brain | YES | |
    | WrongAnimal | YES | |
    | WrongCat | YES | |
    | AMateria | YES | 抽象クラスでも OCF は必要 |
    | Ice | YES | |
    | Cure | YES | |
    | Character | YES | |
    | MateriaSource | YES | |
    | ICharacter | **NO** | インターフェース (状態なし) |
    | IMateriaSource | **NO** | インターフェース (状態なし) |

    **全ファイルのヘッダを開いて 4 関数の宣言があるかチェックされる。**

---

## Exercise 別 Q&A

### Ex00: Polymorphism

!!! note "評価形式: virtual キーワードの確認 + 動作テスト"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| `virtual` とは何か？ | 仮想関数。基底ポインタ越しでも実行時に実際の型の関数が呼ばれる（動的ディスパッチ）。コンパイラが vtable を自動生成する | **この exercise の核心** |
| WrongAnimal と Animal の違いは？ | Animal の `makeSound` は `virtual`、WrongAnimal の `makeSound` は非 virtual。`WrongAnimal*` 経由で WrongCat を呼ぶと WrongAnimal の実装が呼ばれる（静的バインディング） | virtual の効果を対比で理解しているか |
| virtual デストラクタがないとどうなるか？ | `Animal*` 経由で `delete` した時に派生のデストラクタが呼ばれない。派生が持つリソース（ex01 の Brain）がリークする。未定義動作 | **最頻出質問** |
| `Animal*` 経由で Dog の makeSound が呼ばれるのはなぜ？ | vtable（仮想関数テーブル）のおかげ。各オブジェクトは vtable へのポインタ (vptr) を持ち、virtual 関数呼び出し時にこのテーブルを参照して実際の関数アドレスを取得する | vtable の仕組みを知っているか |

!!! info "よく聞かれる口頭質問"
    **Q: vtable はいつ作られるか？**

    A: **コンパイル時**にクラスごとに 1 つ作られる。各 virtual 関数のアドレスが格納されたテーブル。
    オブジェクト生成時にコンストラクタが vptr (vtable へのポインタ) をオブジェクトに書き込む。

    **Q: virtual を付けるとオーバーヘッドはあるか？**

    A: vptr 分のメモリ（通常 8 バイト）と、関数呼び出し時の間接参照 1 回分。
    通常は無視できるレベルだが、パフォーマンスクリティカルなコードでは考慮する場合がある。

    **Q: コンストラクタの呼び出し順序は？**

    A: 基底 → 派生の順。`new Dog()` なら `Animal::Animal()` → `Dog::Dog()`。
    デストラクタは逆順: `Dog::~Dog()` → `Animal::~Animal()`。

---

### Ex01: I don't want to set the world on fire

!!! note "評価形式: ディープコピーの動作確認 + メモリリークチェック"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| ディープコピーのテストをして | Dog を作成 → Brain に値を設定 → コピーコンストラクタでコピー → コピーの Brain を変更 → オリジナルが変わっていないことを確認 | **ディープコピーの実証** |
| シャローコピーとは何か？ | ポインタの値だけをコピーすること。2 つのオブジェクトが同じメモリを指す。片方を delete すると、もう片方がダングリングポインタになり二重 free が起きる | ディープコピーの理由を理解しているか |
| 代入演算子でもディープコピーか？ | はい。自己代入チェック → 旧 Brain を delete → 新しい Brain を new で確保してコピー | コピーコンストラクタとの違いも確認 |
| メモリリークはないか？ | Animal* 配列を作成して全て delete。`leaks` コマンドで確認 | メモリ管理の基本 |

!!! info "よく聞かれる口頭質問"
    **Q: なぜ代入演算子で自己代入チェックが必要なのか？**

    A: `dog = dog;` の場合、チェックなしだと:
    1. `delete _brain` で自分の Brain を消す
    2. `new Brain(*other._brain)` で消した Brain を参照 → 未定義動作

    **Q: Rule of Three とは？**

    A: コピーコンストラクタ・代入演算子・デストラクタのいずれかをカスタマイズしたら、
    3 つ全てをカスタマイズすべきという原則。ヒープメンバがある場合に特に重要。

---

### Ex02: Abstract class

!!! note "評価形式: `= 0` の確認 + コンパイルテスト"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| `= 0` とは何か？ | 純粋仮想関数。実装を持たず、派生クラスで必ずオーバーライドが必要。1 つでも純粋仮想関数があるクラスは抽象クラスになる | **この exercise の核心** |
| なぜ Animal をインスタンス化できないのか？ | `makeSound` が `= 0` なので Animal は抽象クラス。抽象クラスは直接インスタンス化できない (コンパイルエラー) | 抽象クラスの制約を理解しているか |
| 抽象クラスにコンストラクタはあるか？ | ある。`new Dog()` した時に `Animal::Animal()` が先に呼ばれる。抽象クラスは「直接生成できない」だけで、派生の一部としては構築される | よくある誤解の確認 |
| `Animal*` ポインタは使えるか？ | 使える。`Animal* ptr = new Dog();` は合法。禁止されているのは `Animal` の実体生成だけ | 抽象クラスの正確な制約 |

!!! info "よく聞かれる口頭質問"
    **Q: abstract class と interface の違いは？**

    A: **abstract class** は一部のメソッドが純粋仮想で、データメンバや実装を持てる (AMateria)。
    **interface** は全メソッドが純粋仮想で、データメンバを持たない (ICharacter)。
    C++ では言語レベルで区別されないが、42 では命名規則で区別する (A-prefix vs I-prefix)。

    **Q: 純粋仮想関数に実装を持たせることはできるか？**

    A: 可能。`.cpp` に `Animal::makeSound()` の定義を書ける。
    ただし、派生から `Animal::makeSound()` と明示的に呼ばない限り使われない。
    42 の課題では通常使わない。

---

### Ex03: Interface & recap

!!! note "評価形式: 全クラスの動作確認 + メモリリーク + 口頭質問"

    **最も質問が多い exercise。特にメモリ管理と所有権について深く聞かれる。**

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| インターフェースとは何か？ | 全メソッドが純粋仮想 (`= 0`) で、データメンバを持たないクラス。「何ができるか」の契約を定義する。C++ では言語機能ではなく設計パターン | インターフェースの概念 |
| プロトタイプパターンとは？ | `clone()` で具体的な型を知らなくてもオブジェクトのコピーを生成するパターン。AMateria* から Ice/Cure のコピーを作りたいが、`new AMateria()` はできない (抽象)。`clone()` が自分の型を知っていて適切に `new` する | 最も難しい概念 |
| 誰がどのメモリを所有するか？ | `learnMateria` → MateriaSource が所有。`createMateria` → 返り値は呼び出し側。`equip` → Character が所有。`unequip` → Character の _floor が所有 | **メモリリークの有無を決める** |
| unequip で delete しないのはなぜ？ | subject の仕様。外したマテリアは後で再利用できる可能性がある。ただしリークも許されないので、_floor に退避してデストラクタでまとめて delete する | 所有権管理の設計判断 |
| Character のディープコピーはどう実装したか？ | `clone()` を使う。`new AMateria(*p)` はできない (抽象クラス)。`p->clone()` で実際の型のコピーが得られる | プロトタイプパターンの実践 |

!!! info "よく聞かれる口頭質問"
    **Q: 前方宣言 (forward declaration) は何のために使うのか？**

    A: ヘッダの循環 include を避けるため。AMateria.hpp は ICharacter の参照を使い、
    ICharacter.hpp は AMateria のポインタを使う。両方で include すると循環する。
    ポインタや参照だけなら前方宣言で十分 (完全な定義は .cpp で include)。

    **Q: Character を new してインターフェースポインタで管理するメリットは？**

    A: `ICharacter* me = new Character("me");` とすることで、
    呼び出し側は Character の実装詳細を知らなくても API を使える。
    後で Character の実装を変更しても、ICharacter を使うコードは変更不要。
    **「実装ではなくインターフェースに対してプログラミングする」** という OOP の原則。

    **Q: `_floor` 配列のサイズが足りなくなったらどうなるか？**

    A: 1024 個を超えた場合は仕方なく delete する。実際のプロジェクトでは
    `std::vector` を使うべきだが、42 では C++98 + STL 制限があるため固定配列で対処。

---

## 概念の深掘り

### vtable (仮想関数テーブル) の仕組み

```
Animal クラスの vtable:             Dog クラスの vtable:
┌──────────────────────┐           ┌──────────────────────┐
│ [0] ~Animal          │           │ [0] ~Dog             │ ← オーバーライド
│ [1] makeSound → ... │           │ [1] makeSound → Dog::│ ← オーバーライド
│ [2] getType → Animal::│          │ [2] getType → Animal::│ ← 継承
└──────────────────────┘           └──────────────────────┘

Animal* a = new Dog();
┌─────────────┐
│ Dog object  │
│ ┌─────────┐ │
│ │ vptr ───┼─┼──→ Dog の vtable
│ ├─────────┤ │
│ │ _type   │ │
│ ├─────────┤ │
│ │ _brain  │ │
│ └─────────┘ │
└─────────────┘

a->makeSound();
→ a の vptr を辿る → Dog の vtable を見る → Dog::makeSound を呼ぶ
```

### ディープコピー vs シャローコピー (図解)

```
シャローコピー (デフォルト):

  Dog A          Dog B
  ┌─────┐      ┌─────┐
  │brain─┼──┐──┼─brain│   ← 同じアドレスを指す
  └─────┘  │  └─────┘
           ▼
     ┌──────────┐
     │  Brain   │   ← 共有 → 二重 free!
     └──────────┘

ディープコピー (自作):

  Dog A          Dog B
  ┌─────┐      ┌─────┐
  │brain─┼─┐  ┌┼─brain│
  └─────┘ │  │ └─────┘
          ▼  ▼
    ┌────────┐ ┌────────┐
    │ Brain  │ │ Brain  │  ← 独立したコピー
    └────────┘ └────────┘
```

### 所有権の流れ (ex03)

```
new Ice() ─── learnMateria() ──→ MateriaSource._learned[]
                                        │
                                   clone()
                                        │
                                        ▼
              tmp (一時変数)  ←─── createMateria()
               │
          equip(tmp)
               │
               ▼
          Character._inventory[]
               │
          unequip(idx)          use(idx, target)
               │                     │
               ▼                     ▼
          Character._floor[]    AMateria::use() → Ice/Cure::use()
               │
          ~Character()
               │
               ▼
            delete (全マテリア解放)
```

---

## 即不合格フラグ一覧

| フラグ | 条件 |
|--------|------|
| **OCF 不足** | **インターフェース以外のクラスに 4 関数が揃っていない → そのexerciseは採点しない** |
| **Empty work** | 提出物が空 |
| **Incomplete work** | exercise が未完成 |
| **Invalid compilation** | `c++ -Wall -Wextra -Werror` でコンパイルできない |
| **Cheat** | 不正行為の疑い |
| **Crash** | segfault やクラッシュ |
| **Leaks** | メモリリーク (特に ex03 で頻発) |
| **Forbidden function** | C 関数, `using namespace`, `friend`, 外部ライブラリ等 |
| **Can't support / explain code** | **自分のコードを説明できない** |

!!! danger "「OCF 不足」は cpp04 で最も多い不合格理由"
    コードが完璧に動いていても、**1 つのクラスに OCF が欠けているだけで 0 点**になりうる。
    提出前に全ヘッダファイルを開いて 4 関数の宣言をチェックすること。

!!! danger "「Can't support / explain code」に注意"
    特に ex03 は複雑なので、以下を必ず答えられるようにしておくこと:

    - virtual 関数と vtable の仕組み
    - ディープコピーとシャローコピーの違い
    - 純粋仮想関数と抽象クラスの関係
    - インターフェースの定義と目的
    - プロトタイプパターンの仕組みとなぜ必要か
    - 全マテリアの所有権がどこにあるか
