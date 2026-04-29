# ex00 — My First Class in Orthodox Canonical Form

---

## このプログラムは何？

**`Fixed` というクラスを作って、
OCF（正統派カノニカルフォーム）を学ぶ** プログラムです。

オブジェクトが「作られた」「コピーされた」「消された」時に
メッセージを表示して、**何がいつ起こっているか** を
目で見て確認するのがゴールです。

```
$ ./fixed
Default constructor called     ← 作られた！
Copy constructor called        ← コピーされた！
Copy assignment operator called ← 上書きされた！
...
Destructor called              ← 片付けられた！
```

---

## 1. このexerciseで学ぶこと

C 言語の構造体を卒業して、
**C++ のクラスの作り方の基本**を覚える exercise です。

- **クラス (class)** — C の `struct` の進化版
- **OCF の 4 関数** — クラスに必ず書くセット
- **コンストラクタ / デストラクタ** — 自動で呼ばれる関数
- **`static const` メンバ** — クラス全体で共有する定数
- **初期化子リスト** — `: _value(0)` の書き方
- **自己代入チェック** — `if (this != &rhs)` の意味

---

## 2. 新しい概念の解説

### クラス (class) って何？

**データと関数をまとめた「箱」** です。
C の `struct` の進化版だと思ってください。

```cpp
class Fixed {
private:
    int _value;              // データ
public:
    int getRawBits() const;  // 関数
};
```

`private:` は「外から触れない」、
`public:` は「外から使える」という意味です。

### コンストラクタ って何？

**オブジェクトが作られる時に、
自動で呼ばれる初期化関数** です。

C で言うと「`memset` で初期化」の代わりです。

```cpp
Fixed a;  // この瞬間に Fixed() が自動で呼ばれる
          // ← プログラマは呼ばなくていい！
```

名前は**クラス名と同じ**にするルールです。
戻り値の型は**書かない**（暗黙で自分のクラス型を返す）。

### デストラクタ って何？

**オブジェクトが消える時に、
自動で呼ばれる後片付け関数** です。

```cpp
void func() {
    Fixed a;     // コンストラクタが呼ばれる
    // ... a を使う処理
}                // ← この } で自動的にデストラクタが呼ばれる
```

名前は `~` + クラス名。関数の終わりやスコープを抜けた時に
自動実行されます。C で言うと `free` を書かなくていい感じです。

### コピーコンストラクタ って何？

**「既存のオブジェクトをコピーして新しいオブジェクトを作る」**
時に呼ばれる関数です。

```cpp
Fixed a;       // a を作る（デフォルトコンストラクタ）
Fixed b(a);    // a をコピーして b を作る
               // ↑ ここでコピーコンストラクタが呼ばれる
```

引数は必ず **`const Fixed &src`** (const 参照)。
`&` を付けないと「引数に渡す時にまたコピーする」ことになり、
**無限ループ**になってしまいます。

### 代入演算子 (`operator=`) って何？

**「既存のオブジェクトに別の値を上書き」** する時に呼ばれる関数です。

```cpp
Fixed a;       // a を作る
Fixed c;       // c を作る（既に存在している）
c = a;         // a の値を c に上書き
               // ↑ ここで代入演算子が呼ばれる
```

**コピーコンストラクタとの違いは「作る or 上書きする」** です。

### 自己代入チェック (`if (this != &rhs)`) って何？

**`a = a;` のように自分自身に代入された時**
の安全対策です。

```cpp
Fixed a;
a = a;  // 自己代入！
```

今回の `int _value` だけなら問題ないですが、
ポインタメンバがあるクラスで先に `delete` してから
コピーしようとすると**自分のデータが消えてから
自分からコピーしようとして壊れる**事故が起きます。

だから `if (this != &rhs)` で
**「自分と同じアドレスじゃない時だけ処理」**
と書くのが定石です。

### なぜ `operator=` は `Fixed&` を返す？

**連鎖代入 (`a = b = c`)** を可能にするためです。

```cpp
Fixed a, b, c;
a = b = c;   // c → b → a の順で代入
```

これは `a = (b = c)` と解釈されます。
`b = c` の結果が「`b` 自身」を返すから、
続けて `a = b` ができるのです。
もし `void` を返すと連鎖代入ができません。

### `static const` って何？

**クラス全体で 1 つだけ持つ、変更できない値** です。

```cpp
static const int _fractionalBits = 8;
```

- `static`: インスタンスごとじゃなく **クラス全体で共有**
- `const`: **変更できない**

`_fractionalBits` は「小数部のビット数」で、
全 `Fixed` オブジェクトで同じ値なので `static const` が最適です。

### 固定小数点 って何？

**小数を整数で表す方法**です。

```
  普通の整数:    10 はそのまま 10
  固定小数点:    10 を 256 倍して 2560 として保存
                 使う時は 256 で割って 10.0 に戻す

  ┌─────────────────────┬──────────┐
  │  整数部 (24 ビット)   │ 小数部   │
  │                     │ (8 ビット)│
  └─────────────────────┴──────────┘
```

今回の ex00 では **箱だけ** 作ります。
実際の変換や計算は ex01 以降で追加します。

### `getRawBits` / `setRawBits` って何？

**内部の生の整数値 (`_value`) を
読み書きするための関数**です。

```cpp
int   getRawBits() const;   // 値を読む
void  setRawBits(int raw);  // 値を書く
```

`getRawBits` の末尾の `const` は
**「この関数は自分の中を変更しない」** という約束です。

---

## 3. 課題仕様

| 項目 | 内容 |
|------|------|
| クラス名 | `Fixed` |
| private メンバ | `int _value`（値を保存） |
| private メンバ | `static const int _fractionalBits = 8` |
| public 関数 | OCF の 4 関数 + `getRawBits` + `setRawBits` |
| デバッグ出力 | 各関数でメッセージを表示 |

---

## 4. 実行例

```console
$ make
$ ./fixed
Default constructor called
Copy constructor called
Copy assignment operator called
getRawBits member function called
Default constructor called
Copy assignment operator called
getRawBits member function called
getRawBits member function called
0
getRawBits member function called
0
getRawBits member function called
0
Destructor called
Destructor called
Destructor called
```

**メッセージの順番に注目！**

- `Fixed b(a)` → コピーコンストラクタが呼ばれる
- `c = b` → 代入演算子が呼ばれる
- デストラクタは**作った逆順**（c → b → a）で呼ばれる

---

## 5. C と C++ の比較

### OCF って結局 C で言うと何？

**C で構造体を「ちゃんと扱う」ために必要な 4 つの処理を
自動化する仕組み** です。

C でクラスっぽいものを扱うと、毎回こういう自作関数を書きます:

```c
/* ステップ1: 初期化関数を自作 */
/* 例: void init_fixed(t_fixed *f) { f->value=0; } */

/* ステップ2: コピー関数を自作 (deep copy) */
/* 例: void copy_fixed(t_fixed *dst, t_fixed *src) {
 *        dst->value = src->value;
 *    }
 */

/* ステップ3: 代入処理も自分で書く */
/* C の = は memcpy と同じで「中身を単純コピー」するだけ */
/* ポインタメンバがあると危険 */

/* ステップ4: 後片付け関数を自作 */
/* 例: void free_fixed(t_fixed *f) { free(f->buf); } */
```

C++ の OCF はこの 4 つを **クラスに組み込む** 仕組みです。

| C で自作する関数 | C++ の OCF |
|-----------------|-----------|
| `init_xxx()` | デフォルトコンストラクタ |
| `copy_xxx()` | コピーコンストラクタ |
| `=` の処理 (手動) | 代入演算子 (カスタマイズ可) |
| `free_xxx()` | デストラクタ |

つまり OCF は **「C でバラバラに書いてた関数を、クラスの
中にまとめて、自動呼び出しにする」** 仕組みです。

### 並べて比較

=== "C の書き方"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>
    /* memset, memcpy 用のヘッダ */
    #include <string.h>

    /* 構造体で Fixed を定義 */
    /* C には class がない */
    struct Fixed {
        /* 固定小数点の値 */
        int value;
    };

    /* ── ステップ1: 初期化 (自作) ── */
    /* 変数を宣言 */
    struct Fixed a;
    /* memset: 指定バイト数を 0 で埋める C の関数 */
    /* &a: a のアドレス, 0: 埋める値, sizeof: バイト数 */
    memset(&a, 0, sizeof(a));

    /* ── ステップ2: コピー (自作) ── */
    /* コピー先の変数を宣言 */
    struct Fixed b;
    /* memcpy: メモリをバイト単位でコピーする C の関数 */
    /* &b: コピー先, &a: コピー元, sizeof: バイト数 */
    /* ポインタがあると浅いコピーになって危険 */
    memcpy(&b, &a, sizeof(a));

    /* ── ステップ3: 代入 (手動) ── */
    /* C の = は単純コピー。カスタマイズできない */
    b = a;

    /* ── ステップ4: 後片付け (自作) ── */
    /* 今回は free 不要だが、ポインタがあれば必須 */
    /* 例: if (a.buf) free(a.buf); */
    ```

=== "C++ の書き方"

    ```cpp
    // cout 用のヘッダ
    #include <iostream>

    // ── クラスで Fixed を定義 ──
    // class はデータ + 関数をまとめる箱
    // OCF の 4 関数が自動で呼ばれる

    // ステップ1: 初期化 (自動)
    // Fixed() コンストラクタが自動で呼ばれる
    // → _value = 0 にセットされる
    Fixed a;

    // ステップ2: コピー (自動)
    // Fixed(const Fixed&) が自動で呼ばれる
    // → a の中身が b にコピーされる
    Fixed b(a);

    // ステップ3: 代入 (自動でカスタマイズ処理)
    // operator= が自動で呼ばれる
    // 自己代入チェックなど好きな処理を書ける
    Fixed c;
    c = b;

    // ステップ4: 後片付け (自動)
    // スコープを抜けると ~Fixed() が自動で呼ばれる
    // → 後片付けの書き忘れが起きない
    ```

**ステップ数で比較すると:**

```
C の場合: 4 ステップ（全部自分で書く）
  ①memset で初期化
  ②memcpy でコピー
  ③= は単純コピーだけ（カスタマイズ不可）
  ④free は自分で呼ぶ

C++ の場合: 0 ステップ（全部自動）
  ①コンストラクタが自動実行
  ②コピーコンストラクタが自動実行
  ③operator= が自動実行（中身は自由に書ける）
  ④デストラクタが自動実行
```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|------------|
| `memset` で初期化 | コンストラクタが自動 | 初期化忘れがなくなる |
| `memcpy` でコピー | コピーコンストラクタ | 深いコピーも自動化 |
| `=` は単純コピー | `operator=` でカスタマイズ | 自己代入対策など可能 |
| `free` を手動で呼ぶ | デストラクタが自動 | 片付け忘れがなくなる |

---

## 6. コード解説

### プログラムの流れ (ASCII図)

```
main() start
   |
   v
Fixed a;          → Default constructor called
   |
   v
Fixed b(a);       → Copy constructor called
   |                 (中で *this = src で
   |                  Copy assignment operator called)
   v
Fixed c;          → Default constructor called
   |
   v
c = b;            → Copy assignment operator called
   |
   v
a,b,c の値を表示  → getRawBits 3回
   |
   v
main() end        → Destructor x3 (c → b → a の順)
```

### Fixed.hpp（ヘッダファイル）

```cpp title="Fixed.hpp" linenums="1"
#ifndef FIXED_HPP
# define FIXED_HPP

// Fixed クラス: OCF の基本を学ぶための箱
class Fixed
{
public:
    // --- OCF の 4 つの関数 ---
    // (1) デフォルトコンストラクタ
    //     Fixed a; の時に呼ばれる
    Fixed(void);

    // (2) コピーコンストラクタ
    //     Fixed b(a); の時に呼ばれる
    //     const 参照 (Fixed &) で受ける
    Fixed(const Fixed &src);

    // (3) コピー代入演算子
    //     c = a; の時に呼ばれる
    //     戻り値 Fixed& で連鎖代入を可能に
    Fixed &operator=(const Fixed &rhs);

    // (4) デストラクタ
    //     ~ + クラス名
    //     スコープ抜けで自動で呼ばれる
    ~Fixed(void);

    // --- メンバ関数 ---
    // const: 自分を変更しないと約束する印
    int   getRawBits(void) const;
    void  setRawBits(int const raw);

private:
    // 固定小数点の値を保存する変数
    int              _value;
    // クラス全体で共有する定数 (8 ビット)
    static const int _fractionalBits = 8;
};

#endif
```

### Fixed.cpp（実装ファイル）

```cpp title="Fixed.cpp" linenums="1"
// iostream: std::cout のため
#include <iostream>
#include "Fixed.hpp"

// --- (1) デフォルトコンストラクタ ---
// : _value(0) は「初期化子リスト」
// 本体 {} に入る前にメンバを初期化
Fixed::Fixed(void) : _value(0)
{
    std::cout
        << "Default constructor called"
        << std::endl;
}

// --- (2) コピーコンストラクタ ---
// src の値をコピーして自分を作る
Fixed::Fixed(const Fixed &src)
{
    std::cout
        << "Copy constructor called"
        << std::endl;
    // 代入演算子に処理を任せる
    // (*this は自分自身を指す)
    *this = src;
}

// --- (3) コピー代入演算子 ---
// rhs (右辺) の値を自分にコピーする
Fixed &Fixed::operator=(const Fixed &rhs)
{
    std::cout
        << "Copy assignment operator called"
        << std::endl;
    // 自己代入 (a = a) のチェック
    // this は自分のアドレス
    // &rhs は右辺のアドレス
    if (this != &rhs)
        this->_value = rhs.getRawBits();
    // 自分自身を返す（連鎖代入 a=b=c のため）
    return *this;
}

// --- (4) デストラクタ ---
// オブジェクト消滅時に自動で呼ばれる
Fixed::~Fixed(void)
{
    std::cout
        << "Destructor called"
        << std::endl;
}

// --- getRawBits ---
// 内部の値をそのまま返す
// 末尾 const: 自分を変更しない約束
int Fixed::getRawBits(void) const
{
    std::cout
        << "getRawBits member function called"
        << std::endl;
    return this->_value;
}

// --- setRawBits ---
// 内部の値を設定する
void Fixed::setRawBits(int const raw)
{
    this->_value = raw;
}
```

### 大事なポイントをひとつずつ

#### 初期化子リスト `: _value(0)` って何？

コンストラクタの `{}` の **前** に
メンバを初期化する書き方です。

```cpp
// ✅ 初期化子リスト（推奨）
Fixed::Fixed(void) : _value(0)
{
}

// △ 本体で代入（動くけど非推奨）
Fixed::Fixed(void)
{
    _value = 0;   // 1度デフォルト初期化してから代入
}
```

初期化子リストの方が**効率的**で、
**const メンバには初期化子リストしか使えません**
（ex03 で必要になります）。

#### `this` って何？

**自分自身のアドレスを指すポインタ**です。
C でいうと `self` ポインタに相当します。

```cpp
// this の型は Fixed*（Fixedへのポインタ）
this->_value    // 自分の _value
&rhs            // rhs のアドレス
*this           // 自分自身の本体（参照）
```

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Check the Orthodox Canonical Form.
    > Check the implementation of the `getRawBits()`
    > and `setRawBits()` member functions.
    > Ensure the expected debug output is printed."

- [ ] デフォルトコンストラクタがある
- [ ] コピーコンストラクタがある
- [ ] コピー代入演算子がある
- [ ] デストラクタがある
- [ ] 各関数でデバッグメッセージが表示される
- [ ] `getRawBits` / `setRawBits` が正しく動く

---

## 8. テストチェックリスト

### 基本動作

- [ ] `make` がエラーも警告もなく通る
- [ ] 出力が実行例と完全に一致する
- [ ] デストラクタが 3 回出力される

### OCF の各要素

- [ ] デフォルトコンストラクタで `_value = 0`
- [ ] コピーコンストラクタで `_value` がコピーされる
- [ ] 代入演算子で `_value` が上書きされる
- [ ] デストラクタが `}` で自動的に呼ばれる

### エッジケース

- [ ] `setRawBits(42)` → `getRawBits()` が `42`
- [ ] `const Fixed` で `getRawBits()` が呼べる
- [ ] `a = a`（自己代入）でクラッシュしない

### Makefile / ルール

- [ ] `make re` → クリーンビルドが通る
- [ ] ヘッダに実装がない
- [ ] `printf` / `using namespace` / `friend` 不使用

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| OCF とは何？ | クラスに必ず書く 4 つの関数。デフォルトコンストラクタ、コピーコンストラクタ、代入演算子、デストラクタ |
| コピーコンストラクタと代入演算子の違いは？ | コピーコンストラクタは **新しく作る** とき。代入演算子は **既にあるものに上書き** するとき |
| 自己代入チェックはなぜ必要？ | ポインタメンバがある場合、先に `delete` して元データが消えるのを防ぐため |
| デストラクタが逆順で呼ばれるのはなぜ？ | C++ のスタックは LIFO（後入れ先出し）。a,b,c の順に作ったら c,b,a の順に壊す |
| `static const int` は何？ | クラス全体で共有する定数。インスタンスごとに持たない |
| 初期化子リストと本体代入の違いは？ | 初期化子リストは **初期化**。本体は **代入**。const メンバには初期化子リストが必須 |
| なぜ `operator=` は参照を返す？ | 連鎖代入 `a = b = c` を可能にするため |
| なぜコピーコンストラクタは const 参照を取る？ | 参照でないと引数渡しでまたコピーが起きて無限ループ。const で変更しない約束 |

---

## 10. よくあるミス

!!! warning "コピーコンストラクタを書き忘れる"
    コンパイラが自動で作ってくれるので
    **コンパイルは通る** けど、
    デバッグメッセージが出ないから
    **出力が一致しない**。
    OCF は 4 つ全部書くこと。

!!! warning "自己代入チェックを忘れる"
    今回は `int` だけなので動くけど、
    **評価で「自己代入チェックは？」と聞かれる**。
    `if (this != &rhs)` を常に書く。

!!! warning "`_fractionalBits` を普通の変数にする"
    ```cpp
    int _fractionalBits;              // NG
    static const int _fractionalBits = 8; // OK
    ```
    全オブジェクトで同じ値だから `static const` が正解。

!!! warning "コピーコンストラクタを参照なしで書く"
    ```cpp
    Fixed(const Fixed src);   // NG: & がない
    Fixed(const Fixed &src);  // OK
    ```
    `&` がないと引数渡しでまたコピーが起きて**無限ループ**。

!!! warning "`operator=` の戻り値を void にする"
    ```cpp
    void operator=(const Fixed &rhs);   // NG
    Fixed &operator=(const Fixed &rhs); // OK
    ```
    void だと `a = b = c` ができない。

---

## 11. 次の exercise へ

次の [ex01 Towards a more useful class](ex01-fixed-constructors.md) では、
`int` や `float` から `Fixed` を作れるようにします。

`Fixed` クラスがだんだん
**「使えるクラス」** に育っていきます。
