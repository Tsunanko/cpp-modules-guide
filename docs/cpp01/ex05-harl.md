# ex05 — Harl 2.0

---

## このプログラムは何？

**クレーマーが文句を言うシミュレーションプログラム** です。

コマンドラインで「どのレベルで文句を言うか」を指定すると、
その強さの文句を Harl（キャラクター名）が画面に出します。

```
入力:  ./harl DEBUG
出力:  [ DEBUG ] I love having extra bacon...

入力:  ./harl WARNING
出力:  [ WARNING ] I think I deserve extra bacon...
```

普通なら `if` や `switch` を書くところを、
**関数のアドレスを配列に入れて呼び出す** という
C++ 特有のテクニックで実装します。

---

## 1. このexerciseで学ぶこと

- **関数ポインタ**: 関数のアドレスを変数に入れる
- **メンバ関数ポインタ**: クラスのメンバ関数用の特殊なポインタ
- **`&Harl::debug`**: メンバ関数のアドレスの取り方
- **`(this->*func)()`**: メンバ関数ポインタの呼び方
- **並列配列パターン**: 文字列と関数ポインタを同じ添字で対応させる
- **禁止**: if/else 連鎖は使わない（設計思想の学び）

---

## 2. 新しい概念の解説

### 関数ポインタって何？

**関数のアドレスを入れられる変数** です。
C にもある概念ですが、慣れていないと難しい。

```c
/* 普通の関数 */
void greet(void) {
    /* printf で標準出力に書く */
    printf("Hello\n");
}

int main(void) {
    /* 関数ポインタ変数の宣言 */
    /* 「関数 greet のアドレス」を格納 */
    /* & は省略可能 (配列と同じ) */
    void (*func)(void) = &greet;
    /* 呼び出し: 普通の関数と同じ構文 */
    func();  /* → "Hello" */
    return 0;
}
```

読み方:

- `void` = 戻り値の型
- `(*func)` = `func` という**ポインタ変数**
- `(void)` = 引数の型

「戻り値の型 (*変数名)(引数の型)」
がテンプレートです。

### メンバ関数ポインタって何？

**クラスのメンバ関数専用の特殊なポインタ** です。
普通の関数ポインタとは**型が違います**。

C の関数ポインタは「関数のアドレスだけ」を持ちます。
C++ のメンバ関数ポインタは「関数のアドレス + this 引数の型」を持ちます。

#### C で関数ポインタを使う場合（1 種類）

```c
/* 型: 戻り値 (*名前)(引数) */
void (*func)(void) = &greet;
/* 呼び出し: func() */
```

#### C++ でメンバ関数ポインタを使う場合（クラス名付き）

```cpp
class Harl {
public:
    void debug(void);
};

// ── この1つが C の関数ポインタに足された ──
// Harl:: でどのクラスのメンバかを明示
// (隠れた this 引数の型が決まる)
// ❌ 普通の関数ポインタではダメ:
// void (*func)(void) = &Harl::debug;
//   → コンパイルエラー
// ✅ Harl::* をつける:
void (Harl::*func)(void) = &Harl::debug;
//    ^^^^^^^ クラス名:: が必要
```

**なぜ型が違うのか？**

メンバ関数は **隠れた `this` 引数** を持っています。
C の関数ポインタは「そのまま呼べる」アドレス。
C++ のメンバ関数ポインタは「this をつけないと呼べない」アドレス。

```cpp
// 内部的にはこういう感じ
harl.debug();
// ≒ Harl_debug(&harl);
// ↑ 実は this ( = &harl ) が渡されている
```

だから普通の関数ポインタとは互換がありません。

### `&Harl::debug` って何？

**「Harl クラスの debug メンバ関数のアドレス」** です。

```cpp
// メンバ関数のアドレスは
// &クラス名::関数名 で取る
void (Harl::*f)(void) = &Harl::debug;

// ❌ & を忘れるとエラー（C++98）
void (Harl::*f)(void) = Harl::debug;
```

通常の関数は `&` 省略できますが、
メンバ関数では **`&` を必ず書く** のがルール。

### `(this->*func)()` って何？

**メンバ関数ポインタを呼び出す構文** です。
C の関数ポインタ呼び出しに「どのオブジェクト経由か」が追加された形。

#### C の関数ポインタ呼び出し

```c
/* 型: void (*f)(void) */
void (*func)(void) = &greet;
/* 呼び出し: 普通の関数と同じ */
/* 呼び出し対象のオブジェクトは存在しない */
func();
```

#### C++ のメンバ関数ポインタ呼び出し

```cpp
void Harl::complain() {
    // 型: void (Harl::*f)(void)
    void (Harl::*func)(void)
        = &Harl::debug;

    // ❌ C と同じ書き方ではダメ
    //    (this が渡されないので)
    // func();

    // ✅ ->* 演算子で this を指定
    // → 「この this オブジェクトの」
    //   「func が指すメンバ関数」を呼ぶ
    (this->*func)();
    //  ^^^ this を明示的に指定する必要あり
}
```

読み方:

- `this->*func` = 「このオブジェクトの、func が指すメンバ関数」
- 外側の `()` = 演算子の優先順位のため **必須**
- 内側の `()` = 関数を呼び出す

オブジェクト経由で呼ぶ場合:

```cpp
Harl h;
void (Harl::*f)(void) = &Harl::debug;

(h.*f)();     // ← . の場合は .*
// または
Harl *p = &h;
(p->*f)();    // ← -> の場合は ->*
```

### 並列配列パターンって何？

**2 つの配列を「同じ添字」で対応させるデザイン** です。

```cpp
std::string levels[4] = {
    "DEBUG", "INFO", "WARNING", "ERROR"
};

void (Harl::*funcs[4])(void) = {
    &Harl::debug,
    &Harl::info,
    &Harl::warning,
    &Harl::error
};

// levels[i] と funcs[i] が対応する
// levels[0] = "DEBUG"     funcs[0] = debug
// levels[1] = "INFO"      funcs[1] = info
// levels[2] = "WARNING"   funcs[2] = warning
// levels[3] = "ERROR"     funcs[3] = error
```

使い方は:

1. `levels` を走査して引数と一致するインデックスを探す
2. 同じインデックスの `funcs[i]` を呼ぶ

これで if/else 連鎖を書かずに済みます。

### なぜ if/else を使わないのか？

subject の要件ですが、実用的にも利点があります。

```cpp
// ❌ if/else チェーン
// レベル追加のたびに if 分岐を書き足す
if (level == "DEBUG")    debug();
else if (level == "INFO")    info();
else if (level == "WARNING") warning();
else if (level == "ERROR")   error();

// ✅ 並列配列
// レベル追加はテーブルに 1 行追加するだけ
// ロジックは変えなくてよい
```

**コードの構造を変えずに拡張できる** のが利点。

---

## 3. 課題仕様

| 項目 | 内容 |
|------|------|
| クラス名 | `Harl` |
| private | `debug()`, `info()`, `warning()`, `error()` |
| public | `complain(std::string level)` |
| 禁止 | `if/else` 連鎖（メンバ関数ポインタで実装） |
| 実行例 | `./harl DEBUG` → DEBUG メッセージ |

**レベル文字列とメッセージの対応:**

| レベル | メッセージ（抜粋） |
|--------|-------------------|
| DEBUG | I love having extra bacon... |
| INFO | I cannot believe adding extra bacon costs more... |
| WARNING | I think I deserve some extra bacon for free... |
| ERROR | This is unacceptable! I want to speak to the manager... |

---

## 4. 実行例

```console
$ make
$ ./harl DEBUG
[ DEBUG ]
I love having extra bacon for my 7XL-double-cheese...

$ ./harl INFO
[ INFO ]
I cannot believe adding extra bacon costs more money...

$ ./harl WARNING
[ WARNING ]
I think I deserve to have some extra bacon for free.

$ ./harl ERROR
[ ERROR ]
This is unacceptable! I want to speak to the manager now.

$ ./harl INVALID
（何も出ない、またはサイレントに終了）
```

---

## 5. C と C++ の比較

=== "C の書き方"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>
    /* strcmp 用のヘッダ */
    #include <string.h>

    /* 各レベルの関数 (普通のグローバル関数) */
    void debug(void)   { printf("[DEBUG]\n"); }
    void info(void)    { printf("[INFO]\n"); }
    void warning(void) { printf("[WARN]\n"); }
    void error(void)   { printf("[ERR]\n"); }

    int main(int argc, char **argv) {
        /* ── 普通の関数ポインタ配列 ── */
        /* void (*funcs[4])(void) の読み方: */
        /*   void = 戻り値型 */
        /*   (*funcs[4]) = 4 要素のポインタ配列 */
        /*   (void) = 引数型 */
        void (*funcs[4])(void) = {
            debug, info, warning, error
        };
        /* 対応する文字列配列 */
        const char *levels[4] = {
            "DEBUG","INFO","WARNING","ERROR"
        };
        /* 4 レベルをループで比較 */
        for (int i = 0; i < 4; i++) {
            /* strcmp: 文字列比較 */
            /* 一致すると 0 を返す */
            if (strcmp(argv[1],
                       levels[i]) == 0) {
                /* 普通に呼ぶ (this 不要) */
                funcs[i]();
                return 0;
            }
        }
        /* どれにも一致しない場合 */
        return 1;
    }
    ```

=== "C++ の書き方"

    ```cpp
    #include "Harl.hpp"

    void Harl::complain(std::string level) {
        // ── メンバ関数ポインタ配列 ──
        // void (Harl::*funcs[4])(void) の意味:
        //   void = 戻り値型
        //   (Harl::* ...) = Harl の
        //                   メンバ関数ポインタ
        //   funcs[4] = 4 要素の配列
        //   (void) = 引数型
        // C の関数ポインタに Harl:: が足された
        void (Harl::*funcs[4])(void) = {
            // & と クラス名:: が必須
            &Harl::debug,
            &Harl::info,
            &Harl::warning,
            &Harl::error
        };
        // 対応する文字列配列
        std::string levels[4] = {
            "DEBUG", "INFO",
            "WARNING", "ERROR"
        };
        // ループで一致を探す
        for (int i = 0; i < 4; i++) {
            // std::string は == で比較可能
            // (strcmp 不要、C++ の強み)
            if (levels[i] == level) {
                // ── メンバ関数ポインタ呼び出し ──
                // (this->*funcs[i])() の意味:
                //   this->*funcs[i]
                //     = このオブジェクトの
                //       funcs[i] が指す関数
                //   外側の () で呼び出し
                //   (優先順位のため必須)
                (this->*funcs[i])();
                return;
            }
        }
    }
    ```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|-------------|
| `void (*f)(void)` | `void (Harl::*f)(void)` | クラス名:: が追加 |
| `f = debug` | `f = &Harl::debug` | `&` と `クラス名::` が必須 |
| `f();` | `(this->*f)();` | `->*` 演算子で呼ぶ |
| `strcmp(a, b)==0` | `a == b` | string は `==` で比較可 |

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
文字列配列 levels[4] を用意
メンバ関数ポインタ配列 funcs[4] を用意
  |
  v
for ループで levels[i] と level を比較
  |
  +-- 一致 --> (this->*funcs[i])() で呼ぶ → return
  |
  v
どれにも一致しない → そのまま終了
```

### ファイル別の解説

#### Harl.hpp — クラスの宣言

```cpp title="Harl.hpp" linenums="1"
#ifndef HARL_HPP
#define HARL_HPP

#include <string>
#include <iostream>

// ─── Harl クラス ───
// レベル別に文句を言うキャラクター
class Harl
{
private:
    // ── 各レベルの private メンバ関数 ──
    // 外部から直接呼べない。
    // complain() 経由でのみ呼ぶ設計。
    void debug(void);
    void info(void);
    void warning(void);
    void error(void);

public:
    // ── 公開インターフェース ──
    // レベル文字列を受け取り、
    // 対応する private 関数を呼ぶ
    void complain(std::string level);
};

#endif
```

#### Harl.cpp — メンバ関数ポインタの核心

```cpp title="Harl.cpp" linenums="1"
#include "Harl.hpp"

// ─── 各レベルのメッセージ ───
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
        << "I cannot believe adding bacon..."
        << std::endl;
}

void Harl::warning(void)
{
    std::cout << "[ WARNING ]" << std::endl;
    std::cout
        << "I deserve extra bacon for free."
        << std::endl;
}

void Harl::error(void)
{
    std::cout << "[ ERROR ]" << std::endl;
    std::cout
        << "This is unacceptable!"
        << std::endl;
}

// ─── complain: ディスパッチ関数 ───
void Harl::complain(std::string level)
{
    // ── メンバ関数ポインタ配列 ──
    // void        : 戻り値型
    // (Harl::*    : Harl のメンバ関数ポインタ
    // funcs[4])   : 配列名と要素数
    // (void)      : 引数リスト
    void (Harl::*funcs[4])(void) = {
        &Harl::debug,    // 0
        &Harl::info,     // 1
        &Harl::warning,  // 2
        &Harl::error     // 3
    };

    // ── 対応する文字列配列 ──
    // funcs[i] と同じ順番にする
    std::string levels[4] = {
        "DEBUG",
        "INFO",
        "WARNING",
        "ERROR"
    };

    // ── 一致するレベルを探す ──
    for (int i = 0; i < 4; i++)
    {
        // std::string は == で比較可能
        // （strcmp 不要）
        if (levels[i] == level)
        {
            // メンバ関数ポインタを呼び出し:
            //   this->* で「このオブジェクトの
            //   関数ポインタ」
            //   外側の () は優先順位のため必須
            (this->*funcs[i])();
            return;
        }
    }
    // どれにも一致しなければ何もしない
}
```

#### main.cpp — エントリーポイント

```cpp title="main.cpp" linenums="1"
#include "Harl.hpp"

int main(int argc, char **argv)
{
    // 引数チェック: プログラム名 + レベル = 2
    if (argc != 2)
    {
        std::cerr
            << "Usage: ./harl <LEVEL>"
            << std::endl;
        return (1);
    }

    // Harl オブジェクトを作る（スタック）
    Harl harl;
    // complain を呼ぶ → 該当レベルを出力
    harl.complain(argv[1]);
    return (0);
}
```

!!! info "`.*` と `->*` の違い"
    ```cpp
    Harl h;
    Harl *p = &h;
    void (Harl::*f)(void) = &Harl::debug;

    (h.*f)();   // オブジェクト経由（.*）
    (p->*f)();  // ポインタ経由（->*）
    ```
    `complain` の中では `this` がポインタなので
    `->*` を使います。

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex05/"
    > "Files to turn in: Makefile, main.cpp,
    > Harl.{h, hpp}, Harl.cpp"

    禁止事項: if/else/else if の連鎖での分岐。

- [ ] `Harl` クラスを作成している
- [ ] `debug/info/warning/error` は private
- [ ] `complain` は public
- [ ] メンバ関数ポインタを使って分岐している
- [ ] if/else の連鎖になっていない

---

## 8. テストチェックリスト

- [ ] `make` が警告なく通る
- [ ] `./harl DEBUG` → DEBUG メッセージ
- [ ] `./harl INFO` → INFO メッセージ
- [ ] `./harl WARNING` → WARNING メッセージ
- [ ] `./harl ERROR` → ERROR メッセージ
- [ ] `./harl INVALID` → 何も出ない（クラッシュしない）
- [ ] `./harl`（引数なし）→ エラー処理
- [ ] **メンバ関数ポインタ配列を使っている**（コード確認）
- [ ] **if/else の連鎖になっていない**（コード確認）

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| 普通の関数ポインタとメンバ関数ポインタの違いは？ | メンバ関数は隠れた `this` 引数を持つので型が違う。クラス名:: が必要 |
| `&Harl::debug` の `&` は省略できる？ | C++98 ではメンバ関数のアドレスは `&` 必須 |
| `(this->*func)()` の外側の `()` は何のため？ | 演算子の優先順位を揃えるため。ないとコンパイルエラー |
| なぜ if/else を使わない？ | subject の要件。レベル追加時にコード構造を変えずに済む拡張性もある |
| `strcmp` じゃなくて `==` が使える理由は？ | `std::string` は `==` 演算子がオーバーロードされている |
| 並列配列パターンって何？ | 2 つの配列を同じ添字で対応させる設計。一方で文字列、一方で関数ポインタ |
| `this->*f` と `(*this).*f` は同じ？ | 同じ意味。`this->*` は `(*this).*` の糖衣構文 |

---

## 10. よくあるミス

!!! warning "メンバ関数ポインタの型を間違える"
    ```cpp
    // ❌ 普通の関数ポインタでは受け取れない
    void (*f)(void) = &Harl::debug;

    // ✅ Harl:: を型に入れる
    void (Harl::*f)(void) = &Harl::debug;
    ```

!!! warning "呼び出し時の括弧忘れ"
    ```cpp
    // ❌ 演算子優先順位エラー
    this->*funcs[i]();

    // ✅ 外側を () で囲む
    (this->*funcs[i])();
    ```

!!! warning "`&` を忘れる"
    ```cpp
    // ❌ C++98 ではエラー
    void (Harl::*f)(void) = Harl::debug;

    // ✅ メンバ関数のアドレスは & 必須
    void (Harl::*f)(void) = &Harl::debug;
    ```

!!! warning "`strcmp` を使う"
    ```cpp
    // ❌ C 風（動くが C++ らしくない）
    if (strcmp(level.c_str(), "DEBUG") == 0)

    // ✅ string は == で比較可能
    if (level == "DEBUG")
    ```

!!! warning "配列の並び順がずれる"
    `levels[i]` と `funcs[i]` の順序がズレると
    `DEBUG` を指定したのに `info()` が呼ばれる、
    などのバグになります。

---

## 11. 次の exercise へ

次の [ex06 Harl filter](ex06-harl-filter.md) では、
同じ Harl を使いつつ **`switch` 文の fall-through**
を学びます。

「DEBUG を指定したら DEBUG + INFO + WARNING + ERROR
を全部表示」のような「以上のレベルを全部出す」処理を
`switch` のフォールスルー機能で実装します。
