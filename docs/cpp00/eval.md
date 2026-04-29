# cpp00 評価対策

## このモジュールの評価テーマ

> **C++ の基本的な I/O、クラス設計、static メンバを理解しているか**

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

### Ex00: Megaphone

!!! note "評価形式: Yes / No（動くか動かないか）"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| 引数を大文字に変換して出力するか？ | subject の 3 例を実行して見せる。引数間にスペースは入れない | 基本動作確認 |
| 引数なしで `* LOUD AND UNBEARABLE FEEDBACK NOISE *` と出力するか？ | `./megaphone` を実行して見せる | デフォルト動作の確認 |
| C++ らしいアプローチで解いているか？ | `std::toupper` と `std::cout` を使っている。`printf` や `toupper`（C版）は使っていない | **C ではなく C++ で書けているか**が最大のチェックポイント |

!!! info "よく聞かれる口頭質問"
    **Q: `std::endl` と `"\n"` の違いは？**

    A: `std::endl` は改行 **+ バッファのフラッシュ**。`"\n"` は改行のみ。
    大量出力時は `"\n"` の方が高速だが、今回の規模では違いはない。

    **Q: `(char)` キャストはなぜ必要？**

    A: `std::toupper()` は `int` を返す。そのまま `<<` に流すと **整数として出力されてしまう**ため、
    `(char)` で文字型に戻す必要がある。

---

### Ex01: My Awesome PhoneBook

!!! note "評価形式: Yes/No + 0-5点の混合"
    完璧に動作しなくても、**採点できる部分は採点される**。

#### Error management [0-5点]

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| エラー処理はあるか？ | 空入力で再プロンプト、EOF (`Ctrl+D`) で安全に終了。segfault は起きない | subject に具体的な指定はないが、**segfault だけは絶対 NG** |

#### EXIT command [Yes/No]

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| EXIT が正しく動作するか？ | `EXIT` と入力するとプログラムが終了する。連絡先データは保存されない（subject 仕様通り） | 基本動作確認 |

#### Visibility [0-5点] — **最も質問されやすい**

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| Contact クラスの属性は private か？ | はい。`_firstName` 等は全て `private` で、対応する `getFirstName()` 等の getter を `public` に用意している | 初心者は何でも public にしがち。**カプセル化を理解しているか**のチェック |
| なぜ private にした？ | **カプセル化**のため。外部から直接データを書き換えられると、バリデーション（空文字チェック等）をバイパスされる。setter を経由させることで入力制御ができる | OOP の基本概念を説明できるか |
| 必要最低限のものだけ public になっているか？ | 内部ヘルパー（`_truncate` 等）は `private`。外部から使う `addContact()`, `searchContacts()` だけ `public` | アクセス修飾子の使い分け |

!!! tip "評価シート原文"
    > "Beginners tend to make everything public. That's what you need to check here."

    → **「なぜ private にしたのか」は確実に聞かれる**と思って準備すること。

#### Contact and PhoneBook Classes [Yes/No]

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| Contact クラスに必要な属性があるか？ | `firstName`, `lastName`, `nickname`, `phoneNumber`, `darkestSecret` の 5 フィールドがある | 仕様どおりのクラス設計か |
| PhoneBook に Contact の配列があるか？ | `Contact _contacts[8]` として固定長配列で保持。動的確保 (`new`) は使っていない | 動的確保禁止の要件を満たしているか |

#### Read/Eval Loop [Yes/No]

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| 入力ループは C++ らしい方法か？ | `std::getline(std::cin, command)` で入力を読み、`while(true)` ループで処理。`scanf` や `gets` は不使用 | C ではなく C++ の入出力を使えているか |

#### ADD command [0-5点]

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| ADD は subject 通りに動作するか？ | 5 フィールドを順番に入力、空フィールドは再入力を要求。9 件目以降は最古の連絡先を上書き（リングバッファ） | 基本動作 + 上書きロジック |

#### SEARCH command [0-5点]

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| SEARCH は表形式で表示されるか？ | 各列は幅 10、右寄せ、`\|` 区切り。10 文字超は 9 文字 + `.` に切り詰め | **iomanip の使い方**がポイント |
| iomanip の何を使った？ | `std::setw(10)` で幅指定、`std::right` で右寄せ。`<iomanip>` をインクルード | C++ の整形出力を学んだか |

!!! tip "評価シート原文"
    > "A slight deviation from the expected format is not important. This part is about using iomanips in C++."

    → フォーマットの多少のずれは減点されにくい。**iomanip を使っていることが重要**。

---

### Ex02: The Job Of Your Dreams (任意)

!!! note "評価形式: Yes / No（ログが一致するか否か）"

| 質問 | 模範回答 | なぜ聞かれるか |
|------|---------|---------------|
| 出力がログファイルと一致するか？ | タイムスタンプを除去して `diff` で比較。差分ゼロ | 一致しなければ即不正解 |

```bash
# 一致確認コマンド
./account 2>&1 | cut -c20- > got.log
cut -c20- 19920104_091532.log > expect.log
diff got.log expect.log  # 差分ゼロなら OK
```

!!! info "よく聞かれる口頭質問"
    **Q: static メンバ変数とは何か？**

    A: クラスの**全インスタンスで共有**される変数。`Account::_nbAccounts` のように 1 つだけ存在する。
    口座の総数や合計残高のように、個々のオブジェクトではなくクラス全体の情報を管理するのに使う。
    `.cpp` ファイルで `int Account::_nbAccounts = 0;` と**実体定義が必要**。

    **Q: static メンバ関数とは何か？**

    A: インスタンスなしで呼べる関数。`Account::getNbAccounts()` のように使う。
    `this` ポインタを持たないので、非 static メンバには直接アクセスできない。

    **Q: なぜデストラクタの順番が逆になることがある？**

    A: C++ のスタック上のオブジェクトは **構築の逆順 (LIFO)** で破棄される。
    `tests.cpp` の `vector` 内のオブジェクトも、配列末尾から順にデストラクタが呼ばれる場合がある
    （ただし `vector` の実装依存。評価では順序の違いは許容される）。

    **Q: コンストラクタの初期化リストとは？**

    A: `Account(int d) : _amount(d), _nbDeposits(0) { ... }` のように、
    コンストラクタ本体の前でメンバを初期化する方法。代入より効率的で、
    **const メンバや参照メンバには必須**。

    **Q: `_displayTimestamp` はどう実装した？**

    A: `<ctime>` の `std::time()` で現在時刻を取得し、
    `std::strftime(buf, sizeof(buf), "[%Y%m%d_%H%M%S] ", std::localtime(&now))`
    でフォーマット。末尾の半角スペースを忘れると期待ログとずれる。

---

## 概念の深掘り

### カプセル化（Ex01 で最も重要な概念）

```
┌───────────────────────────────┐
│         Contact クラス          │
│                               │
│  private:                     │
│    _firstName ─────────────┐  │
│    _lastName               │  │
│    _nickname               │  │
│    _phoneNumber            │  │
│    _darkestSecret          │  │
│                            │  │
│  public:                   │  │
│    getFirstName() ←────────┘  │  ← getter 経由でのみ読める
│    setFirstName(val) ─────→┘  │  ← setter 経由でのみ書ける
│    isEmpty()                  │
└───────────────────────────────┘
```

**なぜカプセル化するのか？**

1. **データ保護**: 外部から直接 `_phoneNumber = ""` と書き換えられない
2. **バリデーション**: setter 内で「空文字は拒否」等のチェックを入れられる
3. **実装の自由**: 内部の保持方法を変えても、getter/setter の API が同じなら外部コードは影響を受けない

### static メンバ（Ex02 の核心）

```
┌──────────── Account クラス ────────────┐
│                                        │
│  static int _nbAccounts = 0  ← 共有   │
│  static int _totalAmount = 0 ← 共有   │
│                                        │
│  ┌─ account[0] ─┐  ┌─ account[1] ─┐  │
│  │ _index: 0     │  │ _index: 1     │  │
│  │ _amount: 42   │  │ _amount: 54   │  │
│  └───────────────┘  └───────────────┘  │
│                                        │
│  → _nbAccounts == 2                    │
│  → _totalAmount == 96                  │
└────────────────────────────────────────┘
```

- **インスタンス変数** (`_amount`): オブジェクトごとに別の値
- **static 変数** (`_nbAccounts`): クラス全体で 1 つだけ

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
    特に Ex02 の static メンバや Ex01 の private/public の理由は必ず答えられるようにしておくこと。
