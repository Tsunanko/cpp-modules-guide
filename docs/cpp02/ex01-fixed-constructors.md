# ex01 — Towards a more useful fixed-point number class

---

## このプログラムは何？

ex00 で作った `Fixed` クラスに、
**`int` や `float` との変換機能を追加する** プログラムです。

ex00 では箱だけでしたが、今回で
**数字を入れたり、数字に戻したり** できるようになります。

```
int 10      → Fixed に変換 → Fixed から float に戻す
float 42.42 → Fixed に変換 → Fixed から int に戻す

$ ./fixed
a is 1234.43
b is 10
c is 42.4219     ← 42.42 が少しずれる（精度の限界）
d is 10
```

---

## 1. このexerciseで学ぶこと

- **変換コンストラクタ** — `int` や `float` から `Fixed` を作る
- **ビットシフト** — `<< 8` で 256 倍、`>> 8` で 256 で割る
- **`roundf`** — 四捨五入で精度を保つ
- **`toFloat` / `toInt`** — Fixed から元の数値に戻す
- **`operator<<` の非メンバオーバーロード** — `cout << fixed` を可能に
- **`static_cast`** — 安全な型変換

---

## 2. 新しい概念の解説

### 変換コンストラクタって何？

**「別の型から自分のクラスを作る」コンストラクタ**です。

普通のコンストラクタは引数なしですが、
変換コンストラクタは**値を受け取って変換**します。

```cpp
// デフォルトコンストラクタ（引数なし）
Fixed a;

// 変換コンストラクタ（int から作る）
Fixed b(10);

// 変換コンストラクタ（float から作る）
Fixed c(42.42f);
```

C で書いていたこういう変換関数が...

```c
int to_fixed(int n) { return n * 256; }
int fixed = to_fixed(10);
```

C++ ではコンストラクタに**組み込める**のです。

```cpp
Fixed b(10);  // ← 自動で 10 * 256 してくれる
```

### ビットシフト `<< 8` って何？

**「2 進数で左に 8 桁ずらす」** 操作です。
結果的に **`× 256`** と同じ意味になります。

```
  10        = 0000 1010  (2進数)
  10 << 8   = 1010 0000 0000  (左に 8 桁ずらす)
            = 2560     (10進数)
            = 10 × 256

  逆に >> 8 は右に 8 桁ずらす = ÷ 256
  2560 >> 8 = 10
```

ビットシフトは**超高速**なので、
掛け算や割り算よりも好まれます。

### なぜ「8 ビット」= 256？

小数部のビット数を **8 bit** と決めているからです。

```
  32 ビットの int を 2 つに分ける

  ┌──────────────────────┬──────────┐
  │  整数部 (24 ビット)    │ 小数部    │
  │                      │ (8 ビット) │
  └──────────────────────┴──────────┘

  8 ビットで表せる段階数 = 2^8 = 256
  最小精度 = 1/256 = 0.00390625
```

### `roundf` って何？ なぜ必要？

**float を四捨五入する関数** (`<cmath>` に入っている)。
**精度を保つため**に必要です。

```
  42.42 * 256 = 10859.52

  切り捨て (何もしないと):
    10859 → 10859 / 256 = 42.41796875
                           ↑ ずれが大きい

  四捨五入 (roundf):
    10860 → 10860 / 256 = 42.421875
                           ↑ ずれが小さい
```

float → int に変換する時、**デフォルトは切り捨て**
なので、`roundf` で四捨五入しないと精度が落ちます。

### `static_cast` って何？

**C++ 流の安全な型変換**です。

```cpp
// C 流のキャスト（何でも無理やり変換）
(float)x

// C++ 流の static_cast（意図が明確で安全）
static_cast<float>(x)
```

C++ には 4 種類のキャストがあります:

| キャスト | 用途 |
|---------|------|
| `static_cast<T>` | 普通の型変換（**これを使う**） |
| `const_cast<T>` | const を外す（怖い） |
| `dynamic_cast<T>` | 継承のダウンキャスト |
| `reinterpret_cast<T>` | ビットの再解釈（超危険） |

42 では**意図が明確**な `static_cast` が推奨されます。

### `operator<<` のオーバーロードって何？

**`std::cout << fixed` と書けるようにする仕組み**です。

普段 `std::cout << 42` と書けるのは、
`int` 用の `<<` が最初から定義されているからです。
自作クラスの `Fixed` を `cout` で表示するには、
**自分で `<<` を定義する必要があります**。

```cpp
Fixed c(42.42f);
std::cout << c << std::endl;  // これが動くようにする
// 出力: 42.4219
```

### なぜ `operator<<` は非メンバ関数？

**左側が `Fixed` じゃないから**です。

`<<` の書き方は **`左辺 << 右辺`** です。
`cout << fixed` の左辺は `std::ostream` (`cout` の型)
なので、`Fixed` のメンバ関数にはできません。

```cpp
// もしメンバ関数にすると...
fixed << std::cout;  // ← 逆で変！

// 非メンバ関数なら...
std::cout << fixed;  // ← 自然！
```

だからクラスの**外**に自由関数として定義します。

### `toFloat` / `toInt` って何？

**Fixed から元の数値に戻す関数**です。

```
Fixed → float に変換  =  toFloat()
Fixed → int に変換    =  toInt()

例:
  Fixed b(10);
  b.toFloat()  → 10.0f   (float に変換)
  b.toInt()    → 10      (int に変換)
```

### 変換の早見表

| 変換 | 計算 | 例 |
|------|------|-----|
| int → Fixed | `n << 8`（= n * 256） | 10 → 2560 |
| float → Fixed | `roundf(f * 256)` | 42.42 → 10860 |
| Fixed → float | `value / 256.0f` | 10860 → 42.421875 |
| Fixed → int | `value >> 8`（= value / 256） | 10860 → 42 |

---

## 3. 課題仕様

ex00 の `Fixed` に以下を追加する。

| 追加するもの | 説明 |
|------------|------|
| `Fixed(const int n)` | int から Fixed を作る |
| `Fixed(const float n)` | float から Fixed を作る |
| `float toFloat() const` | Fixed を float に変換 |
| `int toInt() const` | Fixed を int に変換 |
| `operator<<` (非メンバ) | `cout << fixed` を可能にする |

---

## 4. 実行例

```console
$ make
$ ./fixed
Default constructor called
Int constructor called
Float constructor called
Copy constructor called
Copy assignment operator called
Float constructor called
Copy assignment operator called
Destructor called
a is 1234.43
b is 10
c is 42.4219
d is 10
a is 1234 as integer
b is 10 as integer
c is 42 as integer
d is 10 as integer
Destructor called
Destructor called
Destructor called
Destructor called
```

**注目ポイント:**

- `42.42f` が `42.4219` になる（固定小数点の精度の限界）
- `toInt()` は小数部を切り捨てる（42.4219 → 42）

---

## 5. C と C++ の比較

### 固定小数点って C で言うと何？

**C では「整数で小数を表現する仕組み」を
毎回自作する必要がある** ものです。

C で固定小数点を扱う時は、以下を全部自分で書きます:

```c
/* ステップ1: 値を保存する型を定義 */
/* typedef int t_fixed;  (ただの int として扱う) */

/* ステップ2: int → Fixed の変換関数を自作 */
/* int to_fixed_int(int n) { return n << 8; } */

/* ステップ3: float → Fixed の変換関数を自作 */
/* int to_fixed_float(float f) {
 *     return (int)roundf(f * 256);
 * }
 */

/* ステップ4: Fixed → int の変換関数を自作 */
/* int fixed_to_int(int v) { return v >> 8; } */

/* ステップ5: Fixed → float の変換関数を自作 */
/* float fixed_to_float(int v) {
 *     return (float)v / 256.0f;
 * }
 */

/* ステップ6: 表示するときも毎回関数呼び出し */
/* printf("%f\n", fixed_to_float(v)); */
```

C++ では、これらを **クラスに組み込めます**:

| C で自作する関数 | C++ の書き方 |
|-----------------|-------------|
| `to_fixed_int(10)` | `Fixed b(10);` |
| `to_fixed_float(42.42f)` | `Fixed c(42.42f);` |
| `fixed_to_int(v)` | `b.toInt()` |
| `fixed_to_float(v)` | `b.toFloat()` |
| `printf("%f", ...)` | `std::cout << b` |

つまり **変換コンストラクタは「C で書いていた変換関数を、
コンストラクタの形で書く」** 仕組みです。

### 並べて比較

=== "C の書き方"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>
    /* roundf 用のヘッダ */
    #include <math.h>

    /* ── int → 固定小数点 (自作) ── */
    /* n を 256 倍する関数を自作 */
    /* << 8 = 左に 8 桁ずらす = × 256 */
    int to_fixed_int(int n)
    {
        /* 256 倍してそのまま返す */
        return n << 8;
    }

    /* ── float → 固定小数点 (自作) ── */
    /* roundf: float を四捨五入する C の関数 */
    /* (int) は C 流のキャスト (型変換) */
    int to_fixed_float(float f)
    {
        /* 256 倍して四捨五入して int に変換 */
        return (int)roundf(f * 256);
    }

    /* ── 固定小数点 → float (自作) ── */
    /* (float) は C 流のキャスト */
    float to_float(int v)
    {
        /* 256.0 で割って float に戻す */
        return (float)v / 256.0f;
    }

    /* ── 使う時 ── */
    /* 変換関数を手動で呼ぶ */
    int fixed = to_fixed_float(42.42f);
    /* printf: C の出力関数 */
    /* %f で float を表示、変換も手動 */
    printf("%f\n", to_float(fixed));
    ```

=== "C++ の書き方"

    ```cpp
    // cout 用のヘッダ
    #include <iostream>
    // roundf 用のヘッダ (C++ 版)
    #include <cmath>

    // ── int から Fixed を作る ──
    // 変換コンストラクタが自動で呼ばれる
    // 内部で n << 8 を実行
    Fixed b(10);

    // ── float から Fixed を作る ──
    // 変換コンストラクタが自動で呼ばれる
    // 内部で roundf(n * 256) を実行
    Fixed c(42.42f);

    // ── cout で直接表示 ──
    // operator<< が自動で toFloat() を呼ぶ
    // 型に合った表示を自動選択してくれる
    std::cout << c << std::endl;
    ```

**ステップ数で比較すると:**

```
C の場合: 毎回 2 ステップ
  ①変換関数を呼んで値を得る
  ②printf で表示

C++ の場合: 1 ステップ
  ①cout << c で全部完了
    (コンストラクタと operator<< が裏で動く)
```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|------------|
| 関数 `to_fixed_int(10)` | `Fixed(10)` | コンストラクタに組み込み |
| `(int)roundf(...)` | `static_cast<int>(...)` | 型変換が安全に |
| `printf("%f", ...)` | `std::cout << ...` | 型に応じて自動選択 |

---

## 6. コード解説

### プログラムの流れ (ASCII図)

```
main() start
   |
   v
Fixed a;              → Default constructor
   |
   v
Fixed b(10);          → Int constructor
   |                    _value = 10 << 8 = 2560
   v
Fixed c(42.42f);      → Float constructor
   |                    _value = roundf(42.42 * 256)
   |                           = 10860
   v
Fixed d(b);           → Copy constructor
   |
   v
std::cout << a << ... → operator<< x 4 回
   |                    (toFloat を経由して表示)
   v
a.toInt() 等を表示     → toInt x 4 回
   |
   v
main() end            → Destructor x 4
```

### Fixed.hpp

```cpp title="Fixed.hpp" linenums="1"
#ifndef FIXED_HPP
# define FIXED_HPP

// std::ostream の宣言が必要なため
# include <iostream>

class Fixed
{
public:
    // --- OCF の 4 関数（ex00 と同じ）---
    Fixed(void);
    Fixed(const Fixed &src);
    Fixed &operator=(const Fixed &rhs);
    ~Fixed(void);

    // --- 変換コンストラクタ（今回追加）---
    // int から Fixed を作る
    Fixed(const int n);
    // float から Fixed を作る
    Fixed(const float n);

    // --- 変換関数（今回追加）---
    // Fixed → float
    float toFloat(void) const;
    // Fixed → int
    int   toInt(void) const;

    // --- ex00 から引き続き ---
    int   getRawBits(void) const;
    void  setRawBits(int const raw);

private:
    int              _value;
    static const int _fractionalBits = 8;
};

// --- operator<< 非メンバ関数 ---
// std::cout << fixed を可能にする
// 左辺が ostream なのでクラス外に置く
std::ostream &operator<<(
    std::ostream &out,
    const Fixed &fixed
);

#endif
```

### Fixed.cpp（変換部分）

```cpp title="Fixed.cpp" linenums="1"
// cmath: roundf を使うため
#include <cmath>
#include "Fixed.hpp"

// --- int → Fixed ---
// n を 256 倍して保存する（n << 8 = n * 256）
// 例: 10 → _value = 2560
Fixed::Fixed(const int n)
    : _value(n << _fractionalBits)
{
    std::cout
        << "Int constructor called"
        << std::endl;
}

// --- float → Fixed ---
// n を 256 倍して四捨五入して保存する
// 例: 42.42 → _value = 10860
Fixed::Fixed(const float n)
    : _value(
        roundf(n * (1 << _fractionalBits))
    )
{
    std::cout
        << "Float constructor called"
        << std::endl;
}

// --- Fixed → float ---
// 保存している値を 256.0 で割って戻す
// static_cast で int→float 変換
float Fixed::toFloat(void) const
{
    return static_cast<float>(this->_value)
         / (1 << _fractionalBits);
}

// --- Fixed → int ---
// 保存している値を 256 で割る（>> 8 で切り捨て）
// 例: 10860 >> 8 = 42
int Fixed::toInt(void) const
{
    return this->_value >> _fractionalBits;
}

// --- operator<< （非メンバ関数）---
// cout << fixed と書けるようにする
// out を参照で返すことで <<の連鎖が可能
std::ostream &operator<<(
    std::ostream &out,
    const Fixed &fixed)
{
    // toFloat() で float に戻してから出力
    out << fixed.toFloat();
    // 連鎖用に out を返す
    return out;
}
```

### 大事なポイントをひとつずつ

#### なぜ `roundf` が必要？

`float` から `int` に変換する時、
**何もしないと切り捨て** になります。

```
42.42 * 256 = 10859.52

切り捨て:  10859 → 10859/256 = 42.41796875
                                 ↑ ずれが大きい

四捨五入:  10860 → 10860/256 = 42.421875
                                 ↑ ずれが小さい
```

`roundf` で四捨五入することで精度が上がります。

#### なぜ `operator<<` は非メンバ？

**`<<` の左側が `std::ostream`** なので、
`Fixed` のメンバ関数にはできないからです。

```cpp
// メンバ関数にすると書き方が変
fixed << std::cout;  // ← 逆！変！

// 非メンバ関数なら自然
std::cout << fixed;  // ← こう書きたい
```

`friend` を使わずに書けるのは、
`toFloat()` が `public` なおかげです。

#### なぜ `toFloat` で `static_cast` が必要？

**`int / int` は整数除算**になって、
小数部が消えてしまうからです。

```cpp
// NG: 整数除算 → 小数部が消える
return this->_value / 256;   // 10860 / 256 = 42
                             //          ↑ 小数が消えた

// OK: 片方を float に → 小数点演算
return static_cast<float>(this->_value) / 256;
// → 42.421875
```

`int / int` を避けるために
**片方を float にキャスト**する必要があります。

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Check for the fixed-point conversion constructors
    > taking `int` and `float`.
    > Check the `toFloat()` and `toInt()` member functions.
    > Check the overload of `operator<<`."

- [ ] int から Fixed を作る変換コンストラクタがある
- [ ] float から Fixed を作る変換コンストラクタがある
- [ ] `toFloat()` で float に戻せる
- [ ] `toInt()` で int に戻せる
- [ ] `operator<<` で `cout << fixed` ができる

---

## 8. テストチェックリスト

### 基本動作

- [ ] `make` がエラーも警告もなく通る
- [ ] 出力が実行例と一致する
- [ ] `Fixed(10)` → `cout` で `10` と表示
- [ ] `Fixed(42.42f)` → `cout` で `42.4219`

### 変換の精度

- [ ] `Fixed(0)` → `toFloat()` が `0`
- [ ] `Fixed(1)` → `toFloat()` が `1`
- [ ] `Fixed(-10)` → `toFloat()` が `-10`
- [ ] `Fixed(0.5f)` → `toFloat()` が `0.5`

### エッジケース

- [ ] 大きい数 `Fixed(10000)` でも動く
- [ ] 負の数 `Fixed(-42.42f)` でも動く
- [ ] `Fixed(0.0f)` → `toInt()` が `0`

### Makefile / ルール

- [ ] `make re` でクリーンビルド成功
- [ ] ヘッダに実装がない
- [ ] `friend` 不使用
- [ ] `<cmath>` をインクルードしている

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| 固定小数点数とは？ | 整数のビットを「整数部+小数部」に分けて小数を表現する方式。精度が固定で決定論的 |
| `n << 8` は何をしている？ | n を 256 倍して固定小数点の内部表現に変換。小数部 8 ビット分の空間を確保する |
| なぜ `roundf` が必要？ | float → int の変換はデフォルトで切り捨て。四捨五入しないと精度が落ちる |
| 42.42 が 42.4219 になる理由は？ | 8 ビットの精度は 1/256 = 0.00390625。42.42 に最も近い表現可能な値が 42.421875 |
| `operator<<` はなぜ非メンバ？ | 左側が `ostream` なので `Fixed` のメンバにできない。`friend` なしで `toFloat()` 経由で実装 |
| `static_cast` と C キャストの違いは？ | 機能は近いが、`static_cast` は意図が明確でコンパイラのチェックが厳しい |
| なぜ `operator<<` が `ostream&` を返す？ | `cout << a << b` のような連鎖を可能にするため |
| ビットシフトと掛け算の違いは？ | 結果は同じだがビットシフトの方が高速。2 の冪乗の場合のみ使える |

---

## 10. よくあるミス

!!! warning "`roundf` を忘れる"
    ```cpp
    // NG: 切り捨てで精度が落ちる
    _value = n * (1 << _fractionalBits);

    // OK: 四捨五入で精度を保つ
    _value = roundf(
        n * (1 << _fractionalBits)
    );
    ```

!!! warning "整数除算のワナ"
    ```cpp
    // NG: int / int = 小数部が消える
    return this->_value
         / (1 << _fractionalBits);

    // OK: float にキャストしてから割る
    return static_cast<float>(this->_value)
         / (1 << _fractionalBits);
    ```
    `int / int` は整数除算になるので、
    片方を `float` にする必要があります。

!!! warning "`<cmath>` のインクルード忘れ"
    `roundf` は `<cmath>` に入っています。
    インクルードしないと**コンパイルエラー**。

!!! warning "`operator<<` を `friend` で書く"
    ```cpp
    // NG: 42 では friend は禁止
    friend std::ostream &operator<<(...);

    // OK: toFloat() が public なので不要
    ```

!!! warning "`operator<<` をメンバ関数にする"
    ```cpp
    // NG: 書き方が逆になる
    class Fixed {
        std::ostream &operator<<(
            std::ostream &out);
    };
    // → fixed << cout; ← 変！
    ```
    必ず**クラスの外**に自由関数として書く。

---

## 11. 次の exercise へ

次の [ex02 Now we're talking](ex02-fixed-operators.md) では、
`+`, `-`, `*`, `/` や `>`, `<` など
**全ての演算子を追加**します。

`Fixed` クラスが `int` みたいに使えるようになります。
