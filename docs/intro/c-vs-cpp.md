# C → C++ 入門 (15分)

---

## このページは何？

**C 言語の経験者が C++ を最短で理解するための道しるべ** です。

C++ は C の「進化版」です。
C のコードはほとんどそのまま C++ としても動きます。
でも、42 の cpp00〜04 を解くには**新しい概念を 7 個**覚える必要があります。

このページではその 7 個を、C と並べて比較しながら解説します。

---

## C++ は C の何を変えたか？

```
    C 言語
     |
     | (進化)
     v
   C++ 言語
     |
     +- ① 名前空間 (namespace)  ← 関数を「箱」で整理
     +- ② std::cout で出力       ← printf の代わり
     +- ③ std::string            ← 文字列の自動管理
     +- ④ クラス (class)         ← 構造体 + 関数の融合
     +- ⑤ コンストラクタ/デストラクタ ← 自動初期化/後片付け
     +- ⑥ 参照 (&)              ← ポインタの安全版
     +- ⑦ static メンバ          ← クラス全体で共有する値
```

---

## 1. 名前空間 (namespace) って何？

**関数や変数を「箱」に入れて整理する仕組み** です。

C では全ての関数が世界中で唯一無二でなければなりません。
C++ では**同じ名前の関数を「別の箱」に入れて共存させられます**。

=== "C の書き方"

    ```c
    #include <stdio.h>

    int main(void) {
        /* printf は世界にひとつだけ */
        printf("hello\n");
        return 0;
    }
    ```

=== "C++ の書き方"

    ```cpp
    #include <iostream>

    int main(void) {
        // std:: は「std という箱の中」という意味
        std::cout << "hello" << std::endl;
        return 0;
    }
    ```

### `std::` って何？

- `std` = **Standard（標準）ライブラリの箱**
- `::` = **スコープ解決演算子**（箱の中身を指す）
- `std::cout` = 「std 箱の中の cout」

```cpp
std::cout    // 標準の cout
std::endl    // 標準の endl
std::string  // 標準の string
```

### なぜ「箱」が必要？

大きなプロジェクトでは、
自分の `print` と誰かのライブラリの `print` が
**衝突する**可能性があります。

名前空間があれば：

```cpp
mylib::print();   // 自分の print
otherlib::print(); // 他人の print
// 衝突しない！
```

!!! danger "`using namespace std;` は 42 では禁止"
    これを書くと `std::` を省略できますが、
    衝突のリスクが出るため **42 では禁止**。

    ```cpp
    // ❌ 禁止
    using namespace std;
    cout << "hello" << endl;

    // ✅ 毎回 std:: を書く
    std::cout << "hello" << std::endl;
    ```

---

## 2. `std::cout` って何？

**画面に文字を出す道具** です。C の `printf` の代わり。

=== "C の書き方"

    ```c
    int x = 42;
    printf("x = %d\n", x);
    // フォーマット指定子 %d が必要
    ```

=== "C++ の書き方"

    ```cpp
    int x = 42;
    std::cout << "x = " << x << std::endl;
    // 型は自動判別、%d 不要
    ```

### `<<` 演算子の特徴

1. **型は自動判別** — `int` でも `double` でも `string` でも OK
2. **連結できる** — `<< a << b << c` と続けられる
3. **型エラーはコンパイル時に検出** — `printf("%d", "hello")` のようなバグが出ない

### `std::endl` と `"\n"` の違い

| | `"\n"` | `std::endl` |
|---|--------|-------------|
| 改行する | はい | はい |
| バッファをフラッシュ（強制出力） | **しない** | **する** |
| 速度 | 速い | 少し遅い |

普段は同じ結果ですが、
**大量出力のときは `"\n"` の方が速い**。

ディフェンスでよく聞かれるので覚えておきましょう。

---

## 3. `std::string` って何？

**自動でメモリを管理してくれる文字列型** です。
C の `char*` の進化版。

=== "C の書き方"

    ```c
    char name[100];
    strcpy(name, "Alice");
    strcat(name, " Liddell");
    /* バッファサイズを常に気にする必要 */
    ```

=== "C++ の書き方"

    ```cpp
    std::string name = "Alice";
    name += " Liddell";
    // サイズは自動で調整される
    ```

### string の便利機能

```cpp
std::string a = "Hello";
std::string b = "World";

a == b              // 比較（C の strcmp 相当）
a + " " + b         // 連結
a.length()          // 長さ
a.substr(0, 3)      // 部分文字列 → "Hel"
a.find("ell")       // 検索 → 1（見つかった位置）
a.empty()           // 空？
```

**バッファあふれの心配なし**、
**`strcpy`/`strcat` 不要** がポイントです。

---

## 4. クラス (class) って何？

**データと関数を「箱」にまとめたもの** です。
C の `struct` が進化した形。

=== "C の書き方"

    ```c
    /* データだけのstruct */
    typedef struct s_point {
        int x;
        int y;
    } t_point;

    /* 関数は別に定義 */
    t_point create_point(int x, int y) {
        t_point p;
        p.x = x;
        p.y = y;
        return p;
    }

    void print_point(t_point *p) {
        printf("(%d, %d)\n", p->x, p->y);
    }
    ```

=== "C++ の書き方"

    ```cpp
    // データと関数を1つの箱に
    class Point {
    private:
        int _x;  // 外から見えない
        int _y;

    public:
        // コンストラクタ: 作る時に呼ばれる
        Point(int x, int y)
            : _x(x), _y(y) {}

        // メンバ関数: クラスの機能
        void print(void) const {
            std::cout << "(" << _x
                      << ", " << _y << ")"
                      << std::endl;
        }
    };
    ```

### `private` と `public` の違い

| | private | public |
|---|---------|--------|
| 外部からアクセス | ❌ できない | ✅ できる |
| クラス内部から | ✅ できる | ✅ できる |
| 用途 | 内部データ | 公開するインターフェース |

```
┌─── Point クラス ───┐
│                    │
│  private:          │  ← 外から見えない
│    _x ───┐         │
│    _y ───┤         │
│          │         │
│  public: │         │
│    Point() ← コンストラクタ
│    print()         │  ← 外から呼べる
│                    │
└────────────────────┘
```

### なぜ private にするの？

**データ保護**のためです。
外部から勝手に `_x = -999` とされたら困るので、
setter 関数でバリデーションを入れられるようにしておきます。

---

## 5. コンストラクタ/デストラクタって何？

### コンストラクタ

**オブジェクトを作る時に自動で呼ばれる関数** です。
初期化を安全に行えます。

```cpp
class Point {
public:
    // コンストラクタ
    Point(int x, int y) : _x(x), _y(y) {
        std::cout << "Point created\n";
    }
};

int main(void) {
    Point p(3, 4);  // ここでコンストラクタが呼ばれる
    return 0;
}
```

### デストラクタ

**オブジェクトが消える時に自動で呼ばれる関数** です。
後片付け（メモリ解放など）に使います。

```cpp
class Point {
public:
    ~Point(void) {  // ~ + クラス名
        std::cout << "Point destroyed\n";
    }
};

int main(void) {
    Point p(3, 4);
    return 0;
    // main終了時に自動で ~Point() が呼ばれる
}
```

### C との比較

| | C | C++ |
|---|---|-----|
| 初期化 | 関数を手動で呼ぶ | コンストラクタが自動 |
| 後片付け | 関数を手動で呼ぶ | デストラクタが自動 |
| メモリ確保 | `malloc` + 初期化 | `new` が両方やる |
| メモリ解放 | `free` | `delete` がデストラクタ呼んでから解放 |

**自動化が C++ の大きな強み** です。

!!! note "cpp00 段階の読者へ"
    上の表のうち `new` / `delete` の行 (下 2 行) は **cpp01 から登場** します。cpp00 を読んでいる段階では、上 2 行 (コンストラクタ・デストラクタの自動呼び出し) だけ理解できれば十分です。`new`/`delete` の話は cpp01 で戻ってきましょう。

### 初期化子リストって何？

```cpp
// 方法1: 本体で代入（非推奨）
Point::Point(int x, int y) {
    _x = x;  // 代入
    _y = y;
}

// 方法2: 初期化子リスト（推奨）
Point::Point(int x, int y)
    : _x(x), _y(y) {  // ← 初期化
    // 本体に入る前に初期化される
}
```

**`const` メンバや参照メンバには初期化子リストが必須** です。

---

## 6. 参照 (&) って何？

**変数の「別名（あだ名）」** です。
ポインタに似ていますが、もっとシンプル。

=== "C の書き方（ポインタのみ）"

    ```c
    void increment(int *n) {
        (*n)++;  // * が必要
    }

    int main(void) {
        int x = 10;
        increment(&x);  // アドレスを渡す
        /* x は 11 になる */
        return 0;
    }
    ```

=== "C++ の書き方（参照）"

    ```cpp
    void increment(int &n) {
        n++;  // * 不要、普通に使える
    }

    int main(void) {
        int x = 10;
        increment(x);  // x そのまま渡す
        // x は 11 になる
        return 0;
    }
    ```

### 参照 vs ポインタ

| | ポインタ | 参照 |
|---|---------|------|
| 宣言 | `int *p = &x;` | `int &r = x;` |
| NULL にできる？ | はい | **いいえ** |
| 後から別の対象に変更？ | はい | **いいえ** |
| 宣言時に初期化 | 任意 | **必須** |
| アクセス | `*p` | `r`（そのまま） |

### どっちを使うべき？

```
対象が NULL（不在）になる可能性がある？
  ├── YES → ポインタを使う
  └── NO → 参照を使う（初期化子リストで束縛）
```

---

## 7. static メンバって何？

**クラス全体で共有される変数/関数** です。

=== "普通のメンバ"

    ```cpp
    class Counter {
    public:
        int count;  // オブジェクトごと
    };

    Counter a, b;
    a.count = 10;
    b.count = 20;
    // a.count と b.count は別物
    ```

=== "static メンバ"

    ```cpp
    class Counter {
    public:
        static int count;  // クラス全体で1個
    };
    int Counter::count = 0;  // 実体定義

    Counter a, b;
    Counter::count = 10;
    // 1つしか存在しない
    ```

### 使いどころ

- **口座の総数**（各口座ではなくクラス全体の情報）
- **生成されたオブジェクト数**のカウンタ
- **クラス共通の設定値**

```
┌─── Account クラス ───┐
│                      │
│  static _nbAccounts  │ ← 全員で共有する1個
│                      │
│  a[0]: _amount=100   │ ← 各オブジェクト固有
│  a[1]: _amount=200   │
│  a[2]: _amount=300   │
└──────────────────────┘
```

---

## まとめ：C と C++ の対応表

| やりたいこと | C | C++ |
|------|---|-----|
| 画面に文字を出す | `printf` | `std::cout <<` |
| 文字列を扱う | `char*` + `strcpy` | `std::string` |
| メモリ確保 | `malloc` | `new` |
| メモリ解放 | `free` | `delete` |
| データ+関数のまとめ | `struct` + 関数 | `class` |
| 変数の別名 | ポインタのみ | 参照 (`&`) |
| 衝突防止 | なし | 名前空間 (`namespace`) |

---

## 次に読むページ

- **[開発環境](setup.md)** — コンパイラとフラグの設定
- **[対応表チートシート](cheatsheet.md)** — C と C++ の対応を一覧で確認
- **[cpp00 概要](../cpp00/index.md)** — 最初のモジュール

!!! tip "ここまで読んだら"
    「C と C++ の違い」の大枠はつかめました。
    細かいことは各 exercise ページで学べるので、
    まずは [cpp00](../cpp00/index.md) から手を動かしてみましょう。
