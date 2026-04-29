# ex06 — Harl filter

---

## このプログラムは何？

**指定したレベル以上の文句を全部表示するプログラム** です。

ex05 と同じ Harl キャラクターですが、
指定したレベル「以上」のメッセージを全部出すのがポイント。

```
./harlFilter DEBUG   → DEBUG + INFO + WARNING + ERROR を全部表示
./harlFilter WARNING → WARNING + ERROR だけ表示
./harlFilter ERROR   → ERROR のみ表示
```

実装には `switch` 文の **fall-through**
（break を書かないと次の case に流れる）
という機能を意図的に使います。

---

## 1. このexerciseで学ぶこと

- **`switch` 文**: C と同じ、整数を値で分岐する制御構文
- **fall-through**: break を書かないと次の case に落ちる
- **意図的な fall-through**: 「指定以上を全部実行」のテク
- **`default`**: どの case にも当てはまらない時の受け皿
- **制約**: `switch` は整数型（int, char, enum）のみ
- **文字列 → 整数変換**: `std::string` を switch で使う回避策

---

## 2. 新しい概念の解説

### `switch` って何？

**整数値で分岐する制御構文** です。
C と全く同じ構文で C++ でも使えます。

```cpp
int n = 2;

switch (n)
{
    case 1:
        std::cout << "一";
        break;
    case 2:
        std::cout << "二";
        break;
    case 3:
        std::cout << "三";
        break;
    default:
        std::cout << "その他";
}
// → "二"
```

構造:

- `switch (式)`: 評価する式（**整数型のみ**）
- `case 値:`: その値にジャンプする場所
- `break;`: switch を抜ける
- `default:`: どれにも一致しない時の受け皿

### fall-through って何？

**`break` を書かないと次の case に流れ込む性質** です。
英語で「落ちる」という意味。

!!! info "C と C++ で同じ挙動"
    fall-through は C から受け継がれた機能で、
    C++ でも **全く同じ動き** をします。
    ex06 では「この機能を意図的に使う」のがポイント。

```cpp
int n = 1;

switch (n)
{
    case 1:
        // n が 1 なのでここに飛ぶ
        std::cout << "1 ";
        // break がない！ → 落ちる
    case 2:
        // 落ちてきて実行される
        std::cout << "2 ";
        // break がない！ → 落ちる
    case 3:
        // 落ちてきて実行される
        std::cout << "3 ";
        // break で switch を抜ける
        break;
    case 4:
        // n=1 からは到達しない
        std::cout << "4 ";
        break;
}
// → "1 2 3 "  （1 から 3 まで全部出力される）
```

**動きの図解:**

```
case 1 に飛ぶ
  ↓
"1 " を出す
  ↓ break がないので落ちる
case 2 の処理へ
  ↓
"2 " を出す
  ↓ break がないので落ちる
case 3 の処理へ
  ↓
"3 " を出す
  ↓ break で脱出
switch 終了
```

C でも同じ動きです。**意図的に使うと強力** ですが、
うっかり break を忘れるとバグになりやすい機能。

### 意図的な fall-through を使うと何ができる？

**「指定レベル以上を全部実行する」のような処理が自然に書ける** のです。

```cpp
switch (lvl)
{
    case 0:  // DEBUG
        debug();
        /* falls through */
    case 1:  // INFO
        info();
        /* falls through */
    case 2:  // WARNING
        warning();
        /* falls through */
    case 3:  // ERROR
        error();
        break;
}
```

例: `lvl == 2` (WARNING) の場合:

```
case 2 に飛ぶ
  ↓
warning() 実行
  ↓ break がないので落ちる
case 3 (error) も実行
  ↓ break で終了
```

結果 → `warning()` と `error()` が実行される。
これが subject の要件「以上のレベルを全部出す」です。

!!! tip "`/* falls through */` コメント"
    モダンなコンパイラは「break忘れかも？」と警告を出します。
    意図的だと示すために **`/* falls through */`** と
    コメントを書くのが慣習です。
    C++17 以降は `[[fallthrough]]` 属性もありますが、
    C++98 では使えません。

### なぜ `switch` は `std::string` を使えない？

**`switch` は整数型（int, char, enum）のみ** だからです。

```cpp
std::string s = "DEBUG";

// ❌ コンパイルエラー
switch (s) {
    case "DEBUG":
    ...
}
```

理由: `switch` は**ジャンプテーブル**という
高速化のための仕組みで、整数値での分岐しかできません。
文字列は長さが可変で比較コストが大きいため対象外。

**回避策**: 文字列を先に **整数インデックスに変換** する。

```cpp
int lvl = -1;
std::string levels[4] = {
    "DEBUG", "INFO", "WARNING", "ERROR"
};

for (int i = 0; i < 4; i++) {
    if (levels[i] == s) {
        lvl = i;
        break;  // ← これは for を抜ける break
    }
}

switch (lvl) {
    case 0: debug(); /* falls through */
    case 1: info();  /* falls through */
    ...
}
```

### `default` って何？

**どの `case` にも一致しなかった時に実行される節** です。
C の `default` と同じ。

```cpp
switch (n)
{
    case 1: ...; break;
    case 2: ...; break;
    default:
        // n が 1 でも 2 でもない時
        std::cout << "不明な値" << std::endl;
}
```

書かなくてもコンパイルは通りますが、
**不正な入力への対応** に便利。

### for ループの break と switch の break

同じ `break` でも、どこから抜けるか意識しましょう。

```cpp
for (int i = 0; i < 4; i++)
{
    if (levels[i] == level)
    {
        lvl = i;
        break;  // ← この break は for を抜ける
    }
}

switch (lvl)
{
    case 3:
        error();
        break;  // ← この break は switch を抜ける
}
```

**`break` は最も内側のループ/switch を抜ける** のがルール。
もし for の中に switch があって、内側の switch で
`break` を書くと for は抜けないので注意。

---

## 3. 課題仕様

| 項目 | 内容 |
|------|------|
| プログラム名 | `harlFilter` |
| 引数 | レベル文字列（DEBUG/INFO/WARNING/ERROR） |
| 動作 | 指定レベル **以上** を全部表示 |
| 不正な引数 | `[ Probably complaining about insignificant problems ]` |
| 必須 | `switch` 文を使う |

**レベルと出力の対応表:**

| 引数 | 表示するメッセージ |
|------|-------------------|
| DEBUG | DEBUG + INFO + WARNING + ERROR |
| INFO | INFO + WARNING + ERROR |
| WARNING | WARNING + ERROR |
| ERROR | ERROR のみ |
| その他 | insignificant problems メッセージ |

---

## 4. 実行例

```console
$ make
$ ./harlFilter DEBUG
[ DEBUG ]
I love having extra bacon...

[ INFO ]
I cannot believe adding extra bacon...

[ WARNING ]
I think I deserve some extra bacon...

[ ERROR ]
This is unacceptable!

$ ./harlFilter WARNING
[ WARNING ]
I think I deserve some extra bacon...

[ ERROR ]
This is unacceptable!

$ ./harlFilter ERROR
[ ERROR ]
This is unacceptable!

$ ./harlFilter INVALID
[ Probably complaining about insignificant problems ]
```

---

## 5. C と C++ の比較

=== "C の書き方"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>
    /* strcmp 用のヘッダ */
    #include <string.h>

    /* 各レベルの関数 */
    void debug(void)   { printf("[DEBUG]\n"); }
    void info(void)    { printf("[INFO]\n"); }
    void warning(void) { printf("[WARN]\n"); }
    void error(void)   { printf("[ERR]\n"); }

    int main(int argc, char **argv) {
        /* レベル文字列配列 */
        const char *levels[] = {
            "DEBUG","INFO","WARNING","ERROR"
        };
        /* 文字列 → 整数に変換する変数 */
        /* -1 = 「該当なし」の印 */
        int lvl = -1;
        /* ループで一致するレベルを探す */
        for (int i = 0; i < 4; i++)
            /* strcmp: 一致で 0 を返す */
            if (strcmp(argv[1],
                       levels[i]) == 0)
                lvl = i;  /* 整数に変換 */
        /* switch: 整数値で分岐 */
        switch (lvl) {
            case 0: debug();
                    /* break なし → 落ちる */
                    /* falls through */
            case 1: info();
                    /* 落ちる */
                    /* falls through */
            case 2: warning();
                    /* 落ちる */
                    /* falls through */
            case 3: error();
                    break;  /* ここで脱出 */
            default:
                /* lvl = -1 の時ここ */
                printf("[ ... ]\n");
        }
        return 0;
    }
    ```

=== "C++ の書き方"

    ```cpp
    // Harl クラスの定義を読む
    #include "Harl.hpp"

    void Harl::complain(std::string level) {
        // レベル文字列配列 (std::string)
        std::string levels[4] = {
            "DEBUG", "INFO",
            "WARNING", "ERROR"
        };
        // 文字列 → 整数に変換
        // switch は string を受け取れないので
        // 事前に int に変換する必要がある
        int lvl = -1;
        // ループで一致を探す
        for (int i = 0; i < 4; i++) {
            // std::string は == で比較可能
            // (C の strcmp の代わり)
            if (levels[i] == level) {
                lvl = i;
                // for を抜ける break
                break;
            }
        }
        // ── switch + fall-through ──
        // C と全く同じ挙動、意図的に使う
        switch (lvl) {
            case 0:
                // this-> でメンバ関数を呼ぶ
                this->debug();
                // break なし → 落ちる
                /* falls through */
            case 1:
                this->info();
                /* falls through */
            case 2:
                this->warning();
                /* falls through */
            case 3:
                this->error();
                // switch を抜ける break
                break;
            default:
                // lvl = -1 の時ここ
                // (不正な入力)
                std::cout
                    << "[ ... ]"
                    << std::endl;
        }
    }
    ```

**C と C++ の違い（このexerciseでの核心）:**

| C | C++ | 一言で言うと |
|---|-----|-------------|
| `strcmp(a, b) == 0` | `a == b` | string は == で比較可 |
| `printf` | `std::cout <<` | ストリーム出力 |
| グローバル関数 | メンバ関数 `this->debug()` | クラスに内包 |

**switch の動きは C と同じ** です。

---

## 6. コード解説

### プログラムの流れ

```
main (argc チェック)
  |
  v
Harl オブジェクトを作る
  |
  v
harl.complain(argv[1]) を呼ぶ
  |
  v
[complain の中]
  |
  v
levels[4] = {"DEBUG","INFO","WARNING","ERROR"}
lvl = -1 で初期化
  |
  v
for で levels を走査
  |
  +-- 一致 --> lvl = i, break (for を抜ける)
  |
  v
switch (lvl) に飛ぶ
  |
  +-- 0 → debug() → 落ちる ↓
  |                        info() → 落ちる ↓
  |                                 warning() → 落ちる ↓
  |                                          error() → break
  +-- 1 → info() から実行（以下同様）
  +-- 2 → warning() から実行
  +-- 3 → error() のみ実行
  +-- -1 (default) → insignificant メッセージ
  |
  v
終了
```

### ファイル別の解説

#### Harl.hpp — ex05 とほぼ同じ

```cpp title="Harl.hpp" linenums="1"
#ifndef HARL_HPP
#define HARL_HPP

#include <string>
#include <iostream>

class Harl
{
private:
    // 各レベルのメッセージ出力関数
    void debug(void);
    void info(void);
    void warning(void);
    void error(void);

public:
    // ex06 では「以上のレベルを全部」表示
    void complain(std::string level);
};

#endif
```

#### Harl.cpp — switch で fall-through

```cpp title="Harl.cpp" linenums="1"
#include "Harl.hpp"

// 各レベルのメッセージ（ex05 と同じ）
void Harl::debug(void)
{
    std::cout << "[ DEBUG ]" << std::endl;
    std::cout
        << "I love having extra bacon..."
        << std::endl;
}

void Harl::info(void)
{
    std::cout << "[ INFO ]" << std::endl;
    std::cout
        << "I cannot believe..."
        << std::endl;
}

void Harl::warning(void)
{
    std::cout << "[ WARNING ]" << std::endl;
    std::cout
        << "I deserve extra bacon..."
        << std::endl;
}

void Harl::error(void)
{
    std::cout << "[ ERROR ]" << std::endl;
    std::cout
        << "This is unacceptable!"
        << std::endl;
}

// ─── complain: switch + fall-through ───
void Harl::complain(std::string level)
{
    // ── レベル文字列の配列 ──
    std::string levels[4] = {
        "DEBUG",    // 0
        "INFO",     // 1
        "WARNING",  // 2
        "ERROR"     // 3
    };

    // ── 文字列をインデックスに変換 ──
    // switch は std::string を使えないので
    // 事前に整数に変換する
    int lvl = -1;
    for (int i = 0; i < 4; i++)
    {
        if (levels[i] == level)
        {
            lvl = i;
            break;  // for を抜ける
        }
    }

    // ── switch + fall-through ──
    switch (lvl)
    {
        case 0:  // DEBUG
            this->debug();
            // break を書かない！
            /* falls through */
        case 1:  // INFO
            this->info();
            /* falls through */
        case 2:  // WARNING
            this->warning();
            /* falls through */
        case 3:  // ERROR
            this->error();
            break;  // ここだけ break
        default:  // -1（不正な入力）
            std::cout
                << "[ Probably complaining "
                   "about insignificant "
                   "problems ]"
                << std::endl;
    }
}
```

#### main.cpp — エントリーポイント

```cpp title="main.cpp" linenums="1"
#include "Harl.hpp"

int main(int argc, char **argv)
{
    // 引数チェック
    if (argc != 2)
    {
        std::cerr
            << "Usage: ./harlFilter <LEVEL>"
            << std::endl;
        return (1);
    }

    Harl harl;
    // 指定レベル以上のメッセージを全部出す
    harl.complain(argv[1]);
    return (0);
}
```

!!! info "fall-through の動き（WARNING 指定時）"
    ```
    lvl = 2 (WARNING)
      ↓
    switch(2):
      case 0: スキップ（飛ぶ先ではない）
      case 1: スキップ
      case 2: warning() 実行
              ↓ break なし → 落ちる
      case 3: error() 実行
              ↓ break で終了
    ```
    結果: WARNING と ERROR の両方が表示される

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex06/"
    > "Files to turn in: Makefile, main.cpp,
    > Harl.{h, hpp}, Harl.cpp"

    必須: **`switch` 文を使う**。

- [ ] `switch` 文を使って実装している
- [ ] fall-through で「以上のレベル」を全部出す
- [ ] 不正なレベルで insignificant メッセージが出る
- [ ] DEBUG で 4 つ全部、WARNING で 2 つ、などの動き
- [ ] `default` が書かれている

---

## 8. テストチェックリスト

- [ ] `make` が警告なく通る
- [ ] `./harlFilter DEBUG` → 4 つ全て表示
- [ ] `./harlFilter INFO` → INFO + WARNING + ERROR
- [ ] `./harlFilter WARNING` → WARNING + ERROR
- [ ] `./harlFilter ERROR` → ERROR のみ
- [ ] `./harlFilter INVALID` → insignificant メッセージ
- [ ] `./harlFilter`（引数なし）→ エラー処理
- [ ] **switch 文を使っている**（コード確認）
- [ ] **fall-through を意図的に使っている**（コード確認）

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| fall-through って何？ | break を書かないと次の case に流れ込む機能 |
| なぜ switch に std::string を渡せない？ | switch は整数型（int/char/enum）のみ。ジャンプテーブル最適化のため |
| どうやって文字列で分岐した？ | 事前に for ループで文字列 → インデックスに変換した |
| fall-through と break 忘れの違いは？ | 意図的かどうか。`/* falls through */` コメントを書くと意図的だと示せる |
| `default` は必要？ | 不正な入力の受け皿に便利。subject でも要求されている |
| for の break と switch の break は同じ？ | どちらも最も内側を抜ける。ネストしていると注意が必要 |
| ex05 との違いは？ | ex05 は 1 つだけ呼ぶ。ex06 は「以上」を全部呼ぶ |
| なぜ switch の最後に break を書く？ | なくても動くが、後でcaseを追加した時に事故らないためのお作法 |

---

## 10. よくあるミス

!!! warning "うっかり break を書いてしまう"
    ```cpp
    case 0:
        this->debug();
        break;  // ❌ これがあると DEBUG のみ表示
    case 1:
        this->info();
    ```
    `/* falls through */` コメントを書く習慣で
    防げます。

!!! warning "switch に std::string を渡す"
    ```cpp
    // ❌ コンパイルエラー
    switch (level) {
        case "DEBUG": ...
    }
    ```
    先に文字列を整数に変換すること。

!!! warning "for の break を switch の break と混同"
    for と switch がネストしていると、
    内側の `break` は内側の方を抜けます。
    **どっちの break か意識する**。

!!! warning "`default` を書き忘れる"
    `default` がないと不正な入力で何も起こらず、
    デバッグしにくくなります。

!!! warning "配列と case 番号のズレ"
    `levels[2] = "WARNING"` なのに
    `case 2: info()` を書いてしまう、など。
    配列の順序と case 番号を一致させる。

---

## 11. 次の exercise へ

cpp01 はここで終了！
次のモジュール [cpp02](../cpp02/index.md) では、
**演算子オーバーロードと正規形（Orthodox Canonical Form）**
を学びます。

- `Fixed` クラスで固定小数点数を実装
- `+`, `-`, `*`, `/` などの演算子を自作
- コンストラクタ・デストラクタ・コピーコンストラクタ・
  代入演算子の **4 点セット（正規形）**

C++ の真骨頂である「型を自分で設計する」世界が
本格的に始まります。
