# ex03 — BSP (Binary Space Partitioning)

---

## このプログラムは何？

**ある点 (Point) が三角形の内側にあるか外側にあるかを判定する**
プログラムです。

ex02 までで育てた `Fixed` クラスを使って、
**2D 幾何学の基本アルゴリズム**を実装します。

```
        C (0,10)
        |\
        | \
        |  \    P (2,2)  ← 内側 → true
        |   \
        |    \
   A ---+-----B
  (0,0)      (10,0)

  (10,10) は外側 → false
  (5, 0)  は辺上 → false (課題仕様)
```

---

## 1. このexerciseで学ぶこと

- **`const` メンバ変数** — 生成後に変更不可のメンバ
- **初期化子リストの必須性** — const メンバはここでしか初期化できない
- **自由関数 (non-member function)** — クラスに属さない関数
- **外積 (cross product)** — 点が線のどちら側にあるか判定
- **三角形の内外判定アルゴリズム** — 3 辺の外積で判定

---

## 2. 新しい概念の解説

### const メンバ変数 って何？

**「作った後は変更できない」メンバ変数**です。

```cpp
class Point {
private:
    Fixed const _x;   // ← const 付き
    Fixed const _y;
};
```

座標 `(1.0, 2.0)` の `Point` を作ったら、
**その後ずっと `(1.0, 2.0)` のまま**で、
勝手に `(3.0, 4.0)` に変わったりしません。

!!! info "Point は「位置」なので変わらない設計がふさわしい"
    `Point` は「2D 空間上の点の位置」を表します。
    位置は勝手に動かない方が自然なので、
    **`const` で不変性を保証**しておきます。

### なぜ const メンバは初期化子リストでしか初期化できない？

**`const` は「代入できない」**という意味だからです。

```cpp
// NG: 本体で代入しようとする
Point::Point(float x, float y)
{
    this->_x = x;   // ❌ コンパイルエラー
                    //   const メンバは代入不可
}

// OK: 初期化子リストで初期化
Point::Point(float x, float y)
    : _x(x), _y(y)   // ← ここで「初期化」
{
    // 本体は空
}
```

**初期化と代入は違います**:

```
初期化: 変数を作る瞬間に値を与える
代入:   既にある変数に値を上書きする
                 ↑ const は代入できない
```

だから **const メンバは必ず初期化子リストを使う** 必要があります。

### なぜ const メンバがあると `*this = src` が使えない？

**`operator=` が何もできない**からです。

```cpp
// ex00 の Fixed ではこれが動いた:
Point::Point(const Point &src)
{
    *this = src;  // ❌ Point では動かない！
}

// 理由: operator= は const メンバを代入できない
//      → 中身がコピーされない
```

`Point` の `operator=` は**実質何もしない**関数
（後述）なので、`*this = src` をしても
`_x` や `_y` がコピーされません。

**コピーコンストラクタも初期化子リスト**を使うのが正解:

```cpp
Point::Point(const Point &src)
    : _x(src._x), _y(src._y)   // 初期化
{
}
```

### const メンバを持つクラスの `operator=` はどうする？

**何もしない (no-op) 実装にする**しかありません。

```cpp
Point &Point::operator=(const Point &rhs)
{
    (void)rhs;       // 未使用警告の抑制
    return *this;    // 自分自身を返すだけ
}
```

- **なぜ書くの？** → OCF のルールで必須
- **何をするの？** → 実質何もしない
- **(void)rhs?** → 引数を使わないと警告が出るので抑制

### 自由関数 (non-member function) って何？

**クラスに属さない、普通の関数**です。

```cpp
// Point クラスの外に書く
bool bsp(Point const a, Point const b,
         Point const c, Point const point);
```

`bsp` は 4 つの `Point` を受け取る関数で、
特定の `Point` に紐づく操作ではないので
**クラスのメンバにしない方が自然**です。

### ベクトルの外積 (cross product) って何？

**2 つのベクトルから方向を計算する**演算です。
今回は**「点が線のどちら側にあるか」**を知るために使います。

```
  外積 (P1→P2) × (P1→P3) の符号:

  正 (+) → P3 は P1→P2 の「左側」
  負 (-) → P3 は P1→P2 の「右側」
  ゼロ   → P3 は P1→P2 の「線上」

        P2
       /
      /
     /        → 正 (左側)
  P1────────────
              → 負 (右側)
```

**計算式** (2D):

```
cross = (P2.x - P1.x) * (P3.y - P1.y)
      - (P2.y - P1.y) * (P3.x - P1.x)
```

結果の符号だけ見れば、「どちら側か」が分かります。

### 三角形の内外判定アルゴリズムは？

**3 辺全ての外積の符号が同じなら内側**、という原理です。

```
        C
        |\
        | \
        |  \    P ← 全辺の「左側」にある → 内側
        |   \
        |    \
   A ---+-----B

  辺 AB に対する P の外積: 正 (左側)
  辺 BC に対する P の外積: 正 (左側)
  辺 CA に対する P の外積: 正 (左側)
  → 全て同じ符号 → 内側！

  もし符号が混在したら (正と負が両方) → 外側
  どれかが 0 だったら → 辺上 → 課題仕様では false
```

**手順:**

1. 3 辺 (AB, BC, CA) それぞれに対して、点 P との外積を計算
2. **外積が 0** → 辺上 → `false`
3. **全て同じ符号** (全て正 or 全て負) → 内部 → `true`
4. **符号が混在** → 外部 → `false`

### なぜ引数が値渡しで参照じゃないの？

**課題仕様に `Point const a` と書かれている**から。

```cpp
// 課題の仕様: 値渡し
bool bsp(Point const a, ...);

// 参照渡しの方が速いが、仕様を優先する
bool bsp(Point const &a, ...);  // ← これは違反
```

パフォーマンスよりも**仕様どおりに書く**のが大事です。

---

## 3. 課題仕様

**Point クラス**:

| 項目 | 内容 |
|------|------|
| private メンバ | `Fixed const _x` / `Fixed const _y` |
| OCF 4 関数 | 必須 |
| コンストラクタ | `Point()`, `Point(float x, float y)` |
| getter | `Fixed getX() const`, `Fixed getY() const` |

**bsp 関数**:

| 項目 | 内容 |
|------|------|
| シグネチャ | `bool bsp(Point const a, Point const b, Point const c, Point const point)` |
| 内側 | `true` |
| 外側 | `false` |
| 辺上・頂点上 | `false` (仕様) |

---

## 4. 実行例

```console
$ make
$ ./bsp
Inside (2,2): 1
Outside (10,10): 0
On edge (5,0): 0
On vertex (0,0): 0
On hypotenuse (5,5): 0
Inside (1,1): 1
Outside (-1,-1): 0
```

**注目ポイント:**

- 三角形 `(0,0)-(10,0)-(0,10)` の内部にある `(2,2)` → `true (1)`
- 辺上の `(5,0)` や頂点上の `(0,0)` → `false (0)` (課題仕様)
- 斜辺上の `(5,5)` → `false`

---

## 5. C と C++ の比較

### const メンバって C で言うと何？

**C には「生成後に変更不可のメンバ」を強制する仕組みが
ない** ので、ドキュメントとコーディング規約で守るだけです。

```c
/* C では「変更するな」を型で保証できない */
typedef struct s_point {
    float x;  /* 変更可能 (規約で守るだけ) */
    float y;
} t_point;

/* 外部から書き換え放題 */
t_point p;
p.x = 10;  /* コンパイル通る (規約違反でも気付かない) */
```

C++ では **const キーワードで型レベルで保証** できます:

```cpp
class Point {
private:
    Fixed const _x;  // 生成後は書き換え不可
};

// 書き換えようとするとコンパイルエラー
p._x = 10;  // ❌ error: assignment of read-only member
```

### 並べて比較

=== "C の書き方"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>

    /* ── 構造体で Point を定義 ── */
    /* C には class がないので struct を使う */
    /* const メンバが作れないので、x, y は
       外から自由に書き換えられてしまう */
    typedef struct s_point {
        /* x 座標 (変更可) */
        float x;
        /* y 座標 (変更可) */
        float y;
    } t_point;

    /* ── 外積 (自作関数) ── */
    /* 2D ベクトルの外積を計算 */
    /* 符号で点の左右を判定できる */
    float cross(t_point p1, t_point p2,
                t_point p3)
    {
        /* 構造体メンバに直接アクセス */
        /* (p2 - p1) × (p3 - p1) を計算 */
        return (p2.x - p1.x)
             * (p3.y - p1.y)
             - (p2.y - p1.y)
             * (p3.x - p1.x);
    }

    /* ── 三角形判定 (自作関数) ── */
    /* 点が三角形の内側か判定 */
    int bsp(t_point a, t_point b,
            t_point c, t_point p)
    {
        /* 3 辺それぞれの外積を計算 */
        float d1 = cross(a, b, p);
        float d2 = cross(b, c, p);
        float d3 = cross(c, a, p);

        /* 外積が 0 = 辺上 → 外とみなす */
        if (d1 == 0 || d2 == 0 || d3 == 0)
            return 0;

        /* 負の符号があるか (int で代用) */
        int neg = (d1<0) || (d2<0) || (d3<0);
        /* 正の符号があるか */
        int pos = (d1>0) || (d2>0) || (d3>0);
        /* 混在=外側 / 全部同じ=内側 */
        return !(neg && pos);
    }
    ```

=== "C++ の書き方"

    ```cpp
    // Point クラスの宣言を読み込む
    #include "Point.hpp"

    // ── 外積 (自由関数) ──
    // static: このファイル内だけで使える
    // 演算子オーバーロードで自然に書ける
    static Fixed cross(Point p1, Point p2,
                       Point p3)
    {
        // getX/getY で値を取得 (カプセル化)
        // Fixed の operator- と operator* が
        // ex02 で定義済みなので自然に書ける
        return (p2.getX() - p1.getX())
             * (p3.getY() - p1.getY())
             - (p2.getY() - p1.getY())
             * (p3.getX() - p1.getX());
    }

    // ── 三角形判定 (自由関数) ──
    // 戻り値は bool (int で代用しない)
    // Point const: 値渡しだが変更不可と保証
    bool bsp(Point const a, Point const b,
             Point const c, Point const p)
    {
        // 3 辺それぞれの外積を計算
        Fixed d1 = cross(a, b, p);
        Fixed d2 = cross(b, c, p);
        Fixed d3 = cross(c, a, p);

        // Fixed(0) は int→Fixed 変換
        // 一時オブジェクトで 0 を作って比較
        if (d1 == Fixed(0)
            || d2 == Fixed(0)
            || d3 == Fixed(0))
            return false;

        // 負の符号があるか (bool を直接使える)
        bool neg = (d1 < Fixed(0))
            || (d2 < Fixed(0))
            || (d3 < Fixed(0));
        // 正の符号があるか
        bool pos = (d1 > Fixed(0))
            || (d2 > Fixed(0))
            || (d3 > Fixed(0));
        // 混在=外側 / 全部同じ=内側
        return !(neg && pos);
    }
    ```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|------------|
| `typedef struct` | `class Point` | クラスで書く |
| `p.x` で直接アクセス | `p.getX()` で getter | カプセル化 |
| `float` で計算 | `Fixed` で計算 | 固定小数点で正確 |
| メンバは自由に変更可 | `const` で不変 | 安全 |
| `int` で true/false | `bool` で true/false | 型が明確 |

---

## 6. コード解説

### プログラムの流れ (ASCII図)

```
main() start
   |
   v
Point a(0, 0);          → Fixed(float) 2 回
Point b(10, 0);
Point c(0, 10);
   |
   v
bsp(a, b, c, Point(2,2))
   |
   +─→ crossProduct(a, b, P)
   |    → 外積 d1 を計算
   |
   +─→ crossProduct(b, c, P)
   |    → 外積 d2 を計算
   |
   +─→ crossProduct(c, a, P)
   |    → 外積 d3 を計算
   |
   +─→ d1, d2, d3 のどれか 0 → false
   |
   +─→ 符号が全部同じ → true (内部)
   |    符号が混在   → false (外部)
   v
結果を表示
   |
   v
main() end
```

### Point.hpp

```cpp title="Point.hpp" linenums="1"
#ifndef POINT_HPP
# define POINT_HPP

# include "Fixed.hpp"

// Point: 2D 座標点を表すクラス
// 座標は Fixed (固定小数点) で保持
class Point
{
public:
    // --- OCF の 4 関数 ---
    // デフォルトコンストラクタ: 原点(0,0)
    Point(void);
    // 座標指定コンストラクタ
    Point(const float x, const float y);
    // コピーコンストラクタ
    Point(const Point &src);
    // 代入演算子 (const メンバのため no-op)
    Point &operator=(const Point &rhs);
    // デストラクタ
    ~Point(void);

    // --- getter (const メンバ関数) ---
    // _x, _y は private なので
    // getter で読み取る
    Fixed getX(void) const;
    Fixed getY(void) const;

private:
    // const メンバ: 生成後に変更不可
    // 「Fixed const」は「const Fixed」と同じ
    // 初期化子リストでしか初期化できない
    Fixed const _x;
    Fixed const _y;
};

// --- 自由関数 ---
// クラスに属さない。Point.hpp に宣言を置く
bool bsp(
    Point const a,
    Point const b,
    Point const c,
    Point const point
);

#endif
```

### Point.cpp

```cpp title="Point.cpp" linenums="1"
#include "Point.hpp"

// --- デフォルトコンストラクタ ---
// 原点 (0, 0) に初期化
// : _x(0), _y(0) は初期化子リスト
// const メンバだからここでしか初期化できない
Point::Point(void) : _x(0), _y(0)
{
}

// --- 座標指定コンストラクタ ---
// float x, y から Fixed を作って保存
// _x(x) は Fixed(float) 変換コンストラクタ呼び出し
Point::Point(const float x, const float y)
    : _x(x), _y(y)
{
}

// --- コピーコンストラクタ ---
// *this = src は使えない！
// (operator= が no-op だからコピーされない)
// → 初期化子リストで src の値をコピー
Point::Point(const Point &src)
    : _x(src._x), _y(src._y)
{
}

// --- 代入演算子 ---
// const メンバは代入できない
// → 何もしない (no-op)
// (void)rhs は「使わない引数」の警告抑制
Point &Point::operator=(const Point &rhs)
{
    (void)rhs;
    return *this;
}

// --- デストラクタ ---
Point::~Point(void)
{
}

// --- getX ---
// _x を返す (const なので変更されない)
Fixed Point::getX(void) const
{
    return this->_x;
}

// --- getY ---
Fixed Point::getY(void) const
{
    return this->_y;
}
```

!!! warning "コピーコンストラクタで `*this = src` を使ってはいけない"
    ```cpp
    // ex00 の Fixed ではこれで動いた:
    Point::Point(const Point &src)
    {
        *this = src;  // ❌ Point では動かない！
    }
    ```
    `*this = src` パターンは **const メンバがない場合のみ** 有効。
    const メンバがある場合は**初期化子リスト**を使う。

### bsp.cpp (外積による判定)

```cpp title="bsp.cpp" linenums="1"
#include "Point.hpp"

// --- 外積のヘルパー関数 ---
// static: このファイル内だけで使える
//         (C の static 関数と同じ)
// 外積 (P1→P2) × (P1→P3) の符号で
// 点が線のどちら側にあるかが分かる
//   正 = 左側, 負 = 右側, 0 = 線上
static Fixed crossProduct(
    Point const p1,
    Point const p2,
    Point const p3)
{
    // ex02 の operator-, operator*, operator-
    // がここで活用される
    return (p2.getX() - p1.getX())
         * (p3.getY() - p1.getY())
         - (p2.getY() - p1.getY())
         * (p3.getX() - p1.getX());
}

// --- bsp 本体 ---
// 三角形 abc の内部に point があれば true
bool bsp(
    Point const a,
    Point const b,
    Point const c,
    Point const point)
{
    // 3 辺それぞれに対して外積を計算
    Fixed d1 = crossProduct(a, b, point);
    Fixed d2 = crossProduct(b, c, point);
    Fixed d3 = crossProduct(c, a, point);

    // 外積が 0 = 辺上 → 仕様で false
    // Fixed(0) は int→Fixed 変換で
    // 一時オブジェクトを作って比較
    if (d1 == Fixed(0)
        || d2 == Fixed(0)
        || d3 == Fixed(0))
        return false;

    // 負の符号があるか？
    bool hasNeg = (d1 < Fixed(0))
               || (d2 < Fixed(0))
               || (d3 < Fixed(0));
    // 正の符号があるか？
    bool hasPos = (d1 > Fixed(0))
               || (d2 > Fixed(0))
               || (d3 > Fixed(0));

    // 負と正が混在 → 外部 → false
    // 全部同じ符号  → 内部 → true
    return !(hasNeg && hasPos);
}
```

### 外積の数学的イメージ

```
  外積 (P1→P2) × (P1→P3):

        P2
       /
      /
     /        → 正 (左側)
  P1────────────
              → 負 (右側)
      \
       \
        P3
```

- **正**: P3 は P1→P2 の**左側**
- **負**: P3 は P1→P2 の**右側**
- **ゼロ**: P3 は P1→P2 の**線上**

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Point class with `Fixed const` members.
    > Check the `bsp` function.
    > Verify that points inside the triangle return true,
    > points outside return false, and points on edges
    > or vertices return false."

- [ ] Point クラスに `Fixed const` メンバがある
- [ ] Point の OCF 4 関数が揃っている
- [ ] `bsp()` が自由関数として実装されている
- [ ] 内部の点 → `true`
- [ ] 外部の点 → `false`
- [ ] 辺上・頂点上の点 → `false`

---

## 8. テストチェックリスト

### 基本動作

- [ ] `make` が警告なく通る
- [ ] 三角形内部の点 → `true`
- [ ] 三角形外部の点 → `false`

### エッジケース

- [ ] 辺上の点 → `false`
- [ ] 頂点上の点 → `false`
- [ ] 三角形のすぐ外側の点 → `false`
- [ ] 別の三角形（逆回り）でも正しく動作する

### Point クラスの確認

- [ ] `Fixed const _x, _y` が const メンバになっている
- [ ] コピーコンストラクタが初期化子リストを使っている
- [ ] `operator=` が no-op（const メンバを変更しない）
- [ ] `getX()` / `getY()` が const メンバ関数

### OCF / 規約

- [ ] Point が OCF の 4 要素を全て持つ
- [ ] Fixed が ex02 の全演算子を持つ
- [ ] `friend` / `printf` / `using namespace` 不使用

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| なぜ `_x`, `_y` を `const` にする？ | 点の座標は作った後に変わらない方が自然。不変性を型で保証できる |
| const メンバはなぜ初期化子リストが必須？ | const は代入不可。本体 `{}` では代入になるので使えない。初期化子リストは「初期化」なので OK |
| コピーコンストラクタで `*this = src` を使うとどうなる？ | `operator=` が const メンバを代入できず no-op なので、`_x`, `_y` が初期化されず未定義動作になる |
| `operator=` の `(void)rhs` は何？ | 未使用引数の警告抑制。const メンバのため実際には rhs を使わない |
| 外積は何を判定する？ | 点が線 (P1→P2) のどちら側にあるか。正=左側、負=右側、0=線上 |
| 三角形内外判定の原理は？ | 3 辺それぞれに対する外積の符号が全て同じなら内側。混在したら外側 |
| 辺上を false にする方法は？ | 外積が 0 になる辺があれば即 false を返す |
| bsp が自由関数なのはなぜ？ | 4 つの Point を操作する関数で、特定の Point に紐づく操作ではないため |
| `Fixed(0)` は何？ | int 0 から Fixed を作る一時オブジェクト。変換コンストラクタが呼ばれる |

---

## 10. よくあるミス

!!! warning "const メンバを本体で代入しようとする"
    ```cpp
    Point &Point::operator=(const Point &rhs)
    {
        this->_x = rhs._x;  // ❌ コンパイルエラー
        this->_y = rhs._y;
        return *this;
    }
    ```
    const メンバを持つクラスの `operator=` は
    **実質 no-op** にするしかない。
    `(void)rhs;` で未使用警告を抑制する。

!!! warning "辺上の判定を忘れる"
    ```cpp
    // 外積 == 0 のチェックを省略すると
    // 辺上の点が true になる
    bool neg = (d1 < Fixed(0)) || ...;
    bool pos = (d1 > Fixed(0)) || ...;
    return !(neg && pos);
    // d1==0, d2>0, d3>0 の場合:
    //   neg=false → !(false && true) = true ❌
    ```
    課題仕様では辺上は `false`。
    **外積 == 0 のチェックを最初に**行う。

!!! warning "コピーコンストラクタで `*this = src` を使う"
    ```cpp
    Point::Point(const Point &src)
    {
        *this = src;  // ❌ _x, _y が初期化されない
    }
    ```
    ex00 では動いたパターンだが、
    **const メンバがある ex03 では壊れる**。
    初期化子リスト `: _x(src._x), _y(src._y)` を使う。

!!! warning "bsp の引数を参照にしてしまう"
    ```cpp
    bool bsp(Point const &a, ...);
    // ❌ subject の仕様は値渡し
    ```
    subject では `Point const a` (値渡し) と明記されている。
    参照渡しの方が効率的だが、**仕様に従うこと**。

!!! warning "外積の計算式を間違える"
    ```cpp
    // NG: 符号が逆になる
    return (p3.x - p1.x) * (p2.y - p1.y)
         - (p3.y - p1.y) * (p2.x - p1.x);

    // OK: 仕様通り
    return (p2.x - p1.x) * (p3.y - p1.y)
         - (p2.y - p1.y) * (p3.x - p1.x);
    ```

---

## 11. 次のモジュールへ

これで cpp02 は完走です！

次の [cpp03](../cpp03/index.md) では、
クラスの最重要機能**継承 (inheritance)** を学びます。
ClapTrap という戦闘ロボットをベースに、
**ScavTrap / FragTrap / DiamondTrap** と
どんどん派生クラスを作っていきます。

cpp02 で学んだ OCF の 4 関数と初期化子リストは、
**継承でも必ず使う**ので、しっかり押さえておきましょう。
