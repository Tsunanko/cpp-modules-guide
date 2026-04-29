# ex02 — Now we're talking

---

## このプログラムは何？

`Fixed` クラスに**ほとんど全ての演算子を追加**して、
**`int` みたいに自然に使える数値型にする** プログラムです。

```cpp
Fixed a(10);
Fixed b(5);

// こんなことができるようになる！
Fixed c = a + b;      // 足し算
bool  x = a > b;      // 比較
++a;                  // インクリメント
Fixed m = Fixed::min(a, b);  // 小さい方を取得
```

C 言語だと `fixed_add(a, b)` と書いていたのが、
**`a + b` と自然に** 書けるようになります。

---

## 1. このexerciseで学ぶこと

- **演算子オーバーロード** — `+` `-` `*` `/` `<` `>` `==` `!=` `<=` `>=` `++` `--`
- **前置と後置** のインクリメント/デクリメントの違い
- **ダミー int 引数** — 前置と後置を区別する方法
- **`static` メンバ関数** — インスタンスなしで呼べる関数
- **const/非const オーバーロード** — 同じ名前で引数違い
- **参照戻り値 vs 値戻り値** の使い分け

---

## 2. 新しい概念の解説

### 演算子オーバーロード って何？

**「自作クラスに `+` や `==` を使えるようにする仕組み」**です。

C++ では `+` や `==` の動作を**自分で定義**できます。
これを**演算子オーバーロード**と呼びます。

```cpp
// int は最初から + が使える
int a = 10 + 5;      // → 15

// Fixed も + を定義すれば使える！
Fixed a = Fixed(10) + Fixed(5);  // → 15
```

関数名を **`operator+`** のように書くと、
コンパイラが「`+` が来た時にこれを使う」と
解釈してくれます。

今回追加する演算子一覧:

```
  追加する演算子と関数
  ====================
  比較 (6個):  >   <   >=   <=   ==   !=
  算術 (4個):  +   -   *   /
  前置 (2個):  ++a  --a
  後置 (2個):  a++  a--
  static (2個): min()   max()
```

### 比較演算子 (`>` `<` `==` など) って何？

**2 つを比較して `bool` (true/false) を返す**演算子です。

```cpp
Fixed a(10), b(5);
if (a > b)          // a の方が大きい？
    std::cout << "a is bigger";
```

Fixed 同士の比較は、**内部の `_value` をそのまま比較**
するだけで OK です。両方とも 256 倍されていて
**同じスケール**なので、大小関係は変わりません。

### 算術演算子 (`+` `-` `*` `/`) って何？

**計算をして新しい値を返す**演算子です。

```cpp
Fixed c = a + b;   // a と b を足して新しい Fixed を返す
```

内部で「float に戻して計算してから Fixed に作り直す」
のが簡潔で安全です。

### なぜ float 経由で計算する？

**直接 `_value` を掛け算するとスケールが壊れる**からです。

```
直接掛け算:
  _value * _value
  2560 * 512 = 1310720
  これは 「256^2 × (実数の積)」のスケール
  → 実数に戻すには 256 で 2 回割らないとダメ

float 経由:
  10.0 * 2.0 = 20.0
  Fixed(20.0) → 正解！
```

### 前置/後置インクリメントの違いは？

`++a` と `a++` の**タイミング**の違いです。

```
  前置 (++a):  先に足してから、足した後の値を返す
  後置 (a++):  足す前の値を返してから、足す

  例: a = 5 のとき
  ┌──────────┬──────────────┬──────────────┐
  │ 書き方    │ 返る値       │ 実行後の a   │
  ├──────────┼──────────────┼──────────────┤
  │ ++a      │ 6（足した後）│ 6            │
  │ a++      │ 5（足す前）  │ 6            │
  └──────────┴──────────────┴──────────────┘
```

### ダミー int 引数って何？

**前置と後置を区別するため**だけの仕様上のルールです。

```cpp
// 前置: 引数なし
Fixed &operator++(void);

// 後置: ダミー int 引数 あり
Fixed operator++(int);
//              ↑ ここ。使わないけど書かないと
//                コンパイラが前置と区別できない
```

C++ の文法上の**慣習**で、`int` の値は**使いません**。
「こう書くと後置ですよ」と区別するためだけのものです。

### なぜ前置は参照、後置は値を返す？

**返すべき値が違う**からです。

```
前置 (++a):
  自分自身を「変更後」に返す → 参照で OK
  (自分はこの関数の後も生きている)

後置 (a++):
  「変更前のコピー」を返す → 値で返すしかない
  (ローカル変数 tmp を参照で返すと
   関数終了で消えてダングリング参照になる)
```

```cpp
// 前置: 参照を返す（コピーなし、速い）
Fixed &operator++(void)
{
    this->_value++;
    return *this;   // 自分自身
}

// 後置: 値を返す（コピーあり、少し遅い）
Fixed operator++(int)
{
    Fixed tmp(*this);  // 足す前のコピー
    this->_value++;
    return tmp;        // コピーを返す
}
```

!!! tip "前置の方が速い"
    前置はコピーを作らないので速いです。
    ループでは `++i` を使うのが C++ 流の推奨です。

### `static` メンバ関数って何？

**オブジェクトなしで、クラス名から直接呼べる関数**です。

```cpp
// 普通のメンバ関数（オブジェクトが必要）
a.getRawBits();

// static メンバ関数（クラス名で呼べる）
Fixed::min(a, b);
Fixed::max(a, b);
```

`static` 関数は `this` を持ちません。
メンバ変数にアクセスしない用途で使います。

### なぜ const/非 const 版の 2 つ？

**const オブジェクトも非 const オブジェクトも**
どちらからも使えるようにするためです。

```cpp
Fixed a(1), b(2);
Fixed::min(a, b);       // 非 const 版が呼ばれる

const Fixed x(1), y(2);
Fixed::min(x, y);       // const 版が呼ばれる
                        // ← これがないとエラー
```

const オブジェクトを非 const 版に渡すと、
**const が外れてしまう危険**があるので、
コンパイラが**エラーにします**。
両方用意しておけばどちらでも使えます。

---

## 3. 課題仕様

ex01 の `Fixed` に以下を追加する。

| 種類 | 演算子 | 補足 |
|------|--------|------|
| 比較 | `> < >= <= == !=` | `bool` を返す、末尾 `const` |
| 算術 | `+ - * /` | 新しい `Fixed` を返す、末尾 `const` |
| 前置 | `++x`, `--x` | 参照を返す |
| 後置 | `x++`, `x--` | 値を返す（ダミー `int`） |
| static | `min`, `max` | const 版と非 const 版の 2 つずつ |

!!! note "デバッグメッセージの除去"
    ex02 ではコンストラクタ/デストラクタの
    メッセージを**消してよい**（出力が長くなるため）。

---

## 4. 実行例

```console
$ make
$ ./fixed
0
0.00390625
0.00390625
0.00390625
0.0078125
10.1016
10.1016
```

**出力の解説:**

```
0             ← a の初期値（0）
0.00390625    ← ++a（前置: 足した後の値）
0.00390625    ← a の現在値
0.00390625    ← a++（後置: 足す前の値を返す）
0.0078125     ← a の現在値（後置で足された後）
10.1016       ← 5.05f * 2 の結果
10.1016       ← max(a, b) → b が大きい
```

!!! info "0.00390625 って何？"
    固定小数点の最小単位 = `1/256 = 0.00390625`。
    `_value++` は内部値を 1 増やすので、
    実数としては 0.00390625 だけ増えます。

---

## 5. C と C++ の比較

### 演算子オーバーロードって C で言うと何？

**C で「+ 関数」「比較関数」を自作していたものを、
演算子の形で書く** 仕組みです。

C では、構造体同士の計算は毎回関数を自作します:

```c
/* 自作関数の例 */
/* t_fixed fixed_add(t_fixed a, t_fixed b) {
 *     t_fixed r;
 *     r.value = a.value + b.value;
 *     return r;
 * }
 */

/* int fixed_greater(t_fixed a, t_fixed b) {
 *     return a.value > b.value;
 * }
 */

/* t_fixed fixed_increment(t_fixed a) {
 *     a.value++;
 *     return a;
 * }
 */

/* 使う時は毎回関数を呼ぶ */
t_fixed c = fixed_add(a, b);
if (fixed_greater(a, b)) { /* ... */ }
a = fixed_increment(a);
```

C++ では、関数名を `operator+` のように書くだけで
**`+` や `>` の記号で自然に書ける** ようになります:

| C の自作関数 | C++ の書き方 |
|-------------|-------------|
| `fixed_add(a, b)` | `a + b` |
| `fixed_sub(a, b)` | `a - b` |
| `fixed_greater(a, b)` | `a > b` |
| `fixed_equal(a, b)` | `a == b` |
| `fixed_increment(a)` | `++a` |

つまり演算子オーバーロードは、
**「C の関数を記号に置き換える」** 仕組みです。

### 並べて比較

=== "C の書き方"

    ```c
    /* ── 足し算 (関数呼び出し) ── */
    /* fixed_add は自作関数 */
    /* 毎回関数を呼んで結果を受け取る */
    int result = fixed_add(a, b);

    /* ── 比較 (関数呼び出し) ── */
    /* fixed_greater は自作関数 */
    /* true/false を int で受け取る */
    if (fixed_greater(a, b))
        /* printf: C の出力関数 */
        printf("a is bigger\n");

    /* ── 最小値 (関数呼び出し) ── */
    /* fixed_min は自作関数 */
    int m = fixed_min(a, b);

    /* ── インクリメント (関数呼び出し) ── */
    /* C の ++ は int/ポインタにしか使えない */
    /* 構造体には自作関数が必要 */
    a = fixed_increment(a);
    ```

=== "C++ の書き方"

    ```cpp
    // ── 足し算 (演算子で自然に) ──
    // operator+ が自動で呼ばれる
    // 中身は C の fixed_add と同じロジック
    Fixed result = a + b;

    // ── 比較 (演算子で自然に) ──
    // operator> が自動で呼ばれる
    // bool を直接返す
    if (a > b)
        // std::cout: C++ の出力
        std::cout << "a is bigger";

    // ── 最小値 (static 関数) ──
    // Fixed::min でクラスに紐づいた関数を呼ぶ
    // グローバル名前空間を汚さない
    Fixed m = Fixed::min(a, b);

    // ── インクリメント (演算子) ──
    // 組み込み型 (int) と同じ書き方
    // operator++ が自動で呼ばれる
    ++a;
    ```

**ステップ数で比較すると:**

```
C の場合: 毎回「関数名を思い出して呼ぶ」1 ステップ
  → fixed_add? fixed_greater? fixed_equal?
    関数名がバラバラで覚えにくい

C++ の場合: 演算子を書くだけ
  → a + b, a > b, a == b
    int と同じ書き方で使える
```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|------------|
| `fixed_add(a, b)` | `a + b` | 演算子で書ける |
| `fixed_greater(a, b)` | `a > b` | 自然な比較 |
| `fixed_min(a, b)` | `Fixed::min(a, b)` | クラスに紐づく |
| `fixed_increment(a)` | `++a` | 組み込み型のように |

---

## 6. コード解説

### プログラムの流れ (ASCII図)

```
main() start
   |
   v
Fixed a;                → _value = 0
   |
   v
std::cout << a;         → operator<< → 0
   |
   v
std::cout << ++a;       → 前置 operator++
   |                      _value = 1
   |                      参照で返す → 0.00390625
   v
std::cout << a++;       → 後置 operator++
   |                      tmp = 1 のコピー
   |                      _value = 2
   |                      tmp(=1) を返す → 0.00390625
   v
Fixed b(Fixed(5.05f)    → 算術演算子
        * Fixed(2));    → Fixed(10.1) を作成
   |
   v
std::cout << Fixed::max → static 関数
           (a, b);         b の方が大きい → 10.1016
   |
   v
main() end
```

### 比較演算子（6 個）

**内部値 `_value` をそのまま比較する**だけ。
両方とも 256 倍されているから、大小関係はそのまま。

```cpp title="Fixed.cpp (比較演算子)" linenums="1"
// --- a > b ---
// 内部値を直接比較
bool Fixed::operator>(
    const Fixed &rhs) const
{
    return this->_value > rhs._value;
}

// --- a < b ---
bool Fixed::operator<(
    const Fixed &rhs) const
{
    return this->_value < rhs._value;
}

// --- a >= b ---
bool Fixed::operator>=(
    const Fixed &rhs) const
{
    return this->_value >= rhs._value;
}

// --- a <= b ---
bool Fixed::operator<=(
    const Fixed &rhs) const
{
    return this->_value <= rhs._value;
}

// --- a == b ---
bool Fixed::operator==(
    const Fixed &rhs) const
{
    return this->_value == rhs._value;
}

// --- a != b ---
bool Fixed::operator!=(
    const Fixed &rhs) const
{
    return this->_value != rhs._value;
}
```

### 算術演算子（4 個）

**float に変換してから計算して、Fixed に戻す**。

```cpp title="Fixed.cpp (算術演算子)" linenums="1"
// --- a + b ---
// toFloat() で実数に戻して計算
// 結果を新しい Fixed として返す
Fixed Fixed::operator+(
    const Fixed &rhs) const
{
    return Fixed(
        this->toFloat() + rhs.toFloat()
    );
}

// --- a - b ---
Fixed Fixed::operator-(
    const Fixed &rhs) const
{
    return Fixed(
        this->toFloat() - rhs.toFloat()
    );
}

// --- a * b ---
// 掛け算は特に float 経由が必須
// (_value * _value だとスケールが壊れる)
Fixed Fixed::operator*(
    const Fixed &rhs) const
{
    return Fixed(
        this->toFloat() * rhs.toFloat()
    );
}

// --- a / b ---
Fixed Fixed::operator/(
    const Fixed &rhs) const
{
    return Fixed(
        this->toFloat() / rhs.toFloat()
    );
}
```

!!! info "なぜ float 経由で計算する？"
    直接 `_value` 同士で掛け算するとスケールが壊れるから。

    ```
    直接掛け算 (NG):
      2560 * 512 = 1310720
      これを 256 で割っても 5120 ← 正解は 20！

    float 経由 (OK):
      10.0 * 2.0 = 20.0
      Fixed(20.0) → 正解！
    ```

### 前置/後置インクリメント

ここが**一番混乱しやすい**ところです。

```cpp title="Fixed.cpp (インクリメント)" linenums="1"
// ── 前置 (++a) ──
// 先に足して、足した後の自分を返す
// 参照で返す = コピーなし、速い
Fixed &Fixed::operator++(void)
{
    this->_value++;     // 内部値を +1
    return *this;       // 自分自身を返す
}

// ── 後置 (a++) ──
// 足す前の値を返して、自分は足す
// ダミー int 引数で前置と区別
Fixed Fixed::operator++(int)
{
    // 足す前の自分をコピー
    Fixed tmp(*this);
    // 内部値を +1
    this->_value++;
    // コピー(足す前)を値で返す
    return tmp;
}

// --- デクリメント (--) も同じパターン ---
Fixed &Fixed::operator--(void)
{
    this->_value--;
    return *this;
}

Fixed Fixed::operator--(int)
{
    Fixed tmp(*this);
    this->_value--;
    return tmp;
}
```

前置と後置の違いを図にすると:

```
  前置 (++a): 足す → 自分を返す
  ─────────────────────────
  _value: 0 → 1
  返す値:      1（足した後）
  返し方:   参照（コピーなし、速い）

  後置 (a++): コピーを保存 → 足す → コピーを返す
  ─────────────────────────
  _value: 0 → 1
  返す値: 0    （足す前のコピー）
  返し方: 値（コピーあり、少し遅い）
```

### static min/max

const 版と非 const 版の **2 つずつ** 作ります。

```cpp title="Fixed.cpp (static min/max)" linenums="1"
// ── 非 const 版 min ──
// 普通の Fixed を受け取る
// 呼び出し例: Fixed::min(a, b)
Fixed &Fixed::min(Fixed &a, Fixed &b)
{
    if (a < b)
        return a;
    return b;
}

// ── const 版 min ──
// const Fixed を受け取る
// const オブジェクトからも使える
const Fixed &Fixed::min(
    const Fixed &a, const Fixed &b)
{
    if (a < b)
        return a;
    return b;
}

// --- max も同じパターン ---
Fixed &Fixed::max(Fixed &a, Fixed &b)
{
    if (a > b)
        return a;
    return b;
}

const Fixed &Fixed::max(
    const Fixed &a, const Fixed &b)
{
    if (a > b)
        return a;
    return b;
}
```

!!! info "なぜ 2 バージョン必要？"
    ```cpp
    Fixed a(1), b(2);
    Fixed::min(a, b);  // ← 非 const 版

    const Fixed x(1), y(2);
    Fixed::min(x, y);  // ← const 版
    ```
    const オブジェクトを渡した時に
    const 版がないと**コンパイルエラー**になります。

!!! note "static 関数の定義には static を書かない"
    ```cpp
    // ヘッダ (宣言): static を書く
    static Fixed &min(Fixed &a, Fixed &b);

    // cpp (定義): static を書かない
    Fixed &Fixed::min(Fixed &a, Fixed &b)
    {
        // ...
    }
    ```

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Check the 6 comparison operators.
    > Check the 4 arithmetic operators.
    > Check the 4 increment/decrement operators.
    > Check the 4 overloads of `min` and `max`."

- [ ] 6 つの比較演算子が全て動く
- [ ] 4 つの算術演算子が全て動く
- [ ] 前置/後置インクリメントが正しい
- [ ] 前置/後置デクリメントが正しい
- [ ] `min` / `max` が const/非 const 両方動く

---

## 8. テストチェックリスト

### 基本動作

- [ ] `make` がエラーも警告もなく通る
- [ ] 出力が実行例と一致する

### 比較演算子

- [ ] `Fixed(10) > Fixed(5)` → `true`
- [ ] `Fixed(5) < Fixed(10)` → `true`
- [ ] `Fixed(5) == Fixed(5)` → `true`
- [ ] `Fixed(5) != Fixed(10)` → `true`
- [ ] `Fixed(5) >= Fixed(5)` → `true`
- [ ] `Fixed(5) <= Fixed(5)` → `true`

### 算術演算子

- [ ] `Fixed(10) + Fixed(5)` → `15`
- [ ] `Fixed(10) - Fixed(5)` → `5`
- [ ] `Fixed(5.05f) * Fixed(2)` → 約 `10.1016`
- [ ] `Fixed(10) / Fixed(2)` → `5`

### インクリメント/デクリメント

- [ ] `++a` が足した**後**の値を返す
- [ ] `a++` が足す**前**の値を返す
- [ ] `--a` / `a--` も同様に動作する
- [ ] `++a = b` のような書き方ができる（前置は参照）

### min/max

- [ ] `Fixed::min(a, b)` が動く
- [ ] `const Fixed` でも `min` が動く
- [ ] `Fixed::max(a, b)` が動く

### Makefile / ルール

- [ ] ヘッダに実装がない
- [ ] `friend` / `printf` / `using namespace` 不使用

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| 演算子オーバーロードとは？ | 自作クラスに `+` や `==` などの演算子の動作を定義する仕組み。`operator+` のような名前で関数を書く |
| 前置と後置の違いは？ | 前置は変更後の値を参照で返す。後置は変更前の値をコピーで返す。ダミー `int` で区別する |
| なぜ前置は参照で後置はコピー？ | 前置は「変更後の自分」を返すから参照で OK。後置は「変更前の値」を返すからコピーが必要（ローカル変数の参照は返せない） |
| 算術を float 経由にする理由は？ | 掛け算で `_value * _value` だとスケールが 256^2 になって壊れる。float 経由なら安全 |
| min/max が 2 バージョンある理由は？ | const オブジェクトを渡した時に const 版が必要。ないとコンパイルエラー |
| 比較で toFloat を使わない理由は？ | 両方同じスケールなので `_value` 同士で比較した方が正確で効率的 |
| ダミー int 引数は何に使う？ | 前置と後置を区別するための仕様上の慣習。実際の値は使わない |
| static メンバ関数とは？ | オブジェクトなしでクラス名から呼べる関数。`this` を持たない |
| なぜ前置の方が推奨されるの？ | コピーを作らないぶん速い。C++ のループでは `++i` が推奨 |

---

## 10. よくあるミス

!!! warning "前置と後置を取り違える"
    ```cpp
    // 前置: 参照を返す、ダミー引数なし
    Fixed &operator++(void);

    // 後置: 値を返す、ダミー int あり
    Fixed operator++(int);
    ```
    ダミー `int` を忘れると
    前置が 2 つになって**コンパイルエラー**。

!!! warning "掛け算で _value を直接掛ける"
    ```cpp
    // NG: スケールが壊れる
    result._value
        = this->_value * rhs._value;

    // OK: float 経由
    return Fixed(
        this->toFloat() * rhs.toFloat()
    );
    ```

!!! warning "後置でローカル変数の参照を返す"
    ```cpp
    // NG: tmp は関数が終わると消える
    Fixed &operator++(int)
    {
        Fixed tmp(*this);
        this->_value++;
        return tmp;  // ← ダングリング参照！
    }

    // OK: 値で返す
    Fixed operator++(int)
    {
        Fixed tmp(*this);
        this->_value++;
        return tmp;  // ← コピーして返す
    }
    ```

!!! warning "min/max の const 版を忘れる"
    ```cpp
    const Fixed a(1), b(2);
    Fixed::min(a, b);
    // ← const 版がないとコンパイルエラー
    ```

!!! warning "算術演算子の戻り値を参照にする"
    ```cpp
    // NG: ローカル Fixed の参照を返すとダングリング
    Fixed &operator+(const Fixed &rhs) const
    {
        return Fixed(...);  // 一時オブジェクト
    }

    // OK: 値で返す
    Fixed operator+(const Fixed &rhs) const;
    ```

!!! warning "末尾 const を忘れる"
    ```cpp
    // NG: const Fixed で比較できない
    bool operator>(const Fixed &rhs);

    // OK: 末尾 const で自分を変更しない約束
    bool operator>(const Fixed &rhs) const;
    ```

---

## 11. 次の exercise へ

次の [ex03 BSP](ex03-bsp.md) では、
`Fixed` クラスを使って**三角形の内外判定**を実装します。

ex00-ex02 で育てた `Fixed` の全機能を活用する応用問題です。
**const メンバ**と**初期化子リストの必須性**
を学ぶ重要な exercise です。
