# ex02 — HI THIS IS BRAIN

---

## このプログラムは何？

**ポインタと参照が「同じものを指している」ことを証明するプログラム** です。

1つの文字列に対して、3通りの方法でアドレスと値を表示します。
全部同じ結果になることで、**参照はただの「別名」** だとわかります。

```
str        ─→ "HI THIS IS BRAIN"  (元の変数)
stringPTR  ─→ "HI THIS IS BRAIN"  (ポインタ)
stringREF  ─→ "HI THIS IS BRAIN"  (参照 = 別名)

3つとも同じアドレス、同じ値！
```

---

## 1. このexerciseで学ぶこと

- **参照 (`&`)** は元の変数の「別名」であること
- ポインタと参照の **アドレスの出力結果** を比較する
- `&` の意味がコンテキストで変わることを理解する

---

## 2. 新しい概念の解説

### 参照って何？

**変数の「別名（ニックネーム）」** です。

本名が「田中太郎」、ニックネームが「タロー」みたいなもの。
どちらの名前で呼んでも、同じ人です。

C でポインタを使う時は3つのステップが必要でした。
C++ の参照はそれを1行にまとめたものです。

#### C でポインタを使う場合（3 ステップ必要）

```c
// ── ステップ1: ポインタ変数を作る ──
// & で str のアドレスを取得
char *p = &str;

// ── ステップ2: NULL チェック ──
// ポインタは NULL の可能性があるので
// 使う前に毎回チェックが必要
if (p != NULL) {
    // ── ステップ3: * でデリファレンス ──
    // ポインタから元の値を取り出すため
    // 毎回 * をつけて使う
    printf("%s\n", *p);
}
```

#### C++ で参照を使う場合（1 行で完了）

```cpp
// ── この1行で以下が全部保証される ──
//   ①str の「別名」になる
//     (アドレスを持つがデリファレンス不要)
//   ②NULL にならない (参照は必ず何かを指す)
//   ③以後は普通の変数と同じ書き方で使える
std::string &ref = str;

// 参照は使う時も普通の変数と同じ
// * も & もいらない！
std::cout << ref << std::endl;  // "HI ..."

// ref を変えると str も変わる
// 同じものだから当然
ref = "HELLO";
```

**図で比較すると:**

```
C のポインタ:              C++ の参照:

char *p = &str;             std::string &ref = str;
  ↓                              ↓
毎回 NULL チェック            NULL にならない
  ↓                              ↓
毎回 * でデリファレンス       普通の変数のように使う
  ↓                              ↓
printf("%s", *p);           cout << ref;
（3手順）                     （1手順）
```

### ポインタとの違いは？

| | ポインタ | 参照 |
|---|---------|------|
| 宣言 | `int *p = &x;` | `int &r = x;` |
| 値を読む | `*p`（`*` が必要） | `r`（そのまま） |
| NULL にできる？ | はい | **いいえ** |
| 後から別の変数に変更？ | はい | **いいえ** |
| 初期化は？ | 後からでもOK | **宣言時に必須** |

```
ポインタは「住所が書かれたメモ用紙」
  → メモ用紙は独立した物体
  → 別の住所に書き換えられる
  → メモ用紙がない（NULL）こともある

参照は「あだ名」
  → あだ名は独立した物体ではない
  → あだ名を別の人に付け替えられない
  → あだ名は必ず誰かにつく
```

### `&` の3つの意味

C++ では `&` がどこに書いてあるかで意味が変わります。

| 場所 | 意味 | 例 |
|------|------|-----|
| **型の後ろ**（宣言時） | 参照型 | `int &ref = x;` |
| **変数の前**（式の中） | アドレス取得 | `int *p = &x;` |
| **二項演算**（式の中） | ビット AND | `a & b` |

```cpp
int x = 42;
int &ref = x;    // ← 参照の宣言
int *ptr = &x;   // ← アドレスの取得
int y = x & 0xFF; // ← ビット AND
```

---

## 3. 課題仕様

| 項目 | 内容 |
|------|------|
| 変数 | `std::string str = "HI THIS IS BRAIN"` |
| ポインタ | `std::string *stringPTR` — str を指す |
| 参照 | `std::string &stringREF` — str の別名 |
| 出力 | 3つのアドレスと3つの値 |

---

## 4. 実行例

```console
$ make
$ ./brain
Address of str:       0x7ffeefbff5a8
Address of stringPTR: 0x7ffeefbff5a8
Address of stringREF: 0x7ffeefbff5a8
Value of str:         HI THIS IS BRAIN
Value of *stringPTR:  HI THIS IS BRAIN
Value of stringREF:   HI THIS IS BRAIN
```

**注目**: 3つのアドレスが **全て同じ** です。
参照は独立した変数ではなく、元の変数 **そのもの** だからです。

---

## 5. C と C++ の比較

=== "C の書き方（ポインタだけ）"

    ```c
    /* printf 用のヘッダ */
    #include <stdio.h>

    int main(void) {
        /* C には参照がないので */
        /* ポインタしか使えない */
        /* char * = 文字列のアドレスを持つ変数 */
        char *str = "HI THIS IS BRAIN";
        /* str のアドレスを持つポインタ */
        /* (ポインタのポインタになる) */
        char **ptr = &str;

        /* %p: アドレスを表示する書式指定 */
        /* (void *) でキャストするのがお作法 */
        printf("Address: %p\n", (void *)str);
        /* *ptr で ptr が指す先 = str の値 */
        printf("Address: %p\n", (void *)*ptr);
        /* %s: 文字列を表示する書式指定 */
        printf("Value:   %s\n", str);
        /* *ptr をデリファレンスして値を取得 */
        printf("Value:   %s\n", *ptr);
        return 0;
    }
    ```

=== "C++ の書き方（参照が使える）"

    ```cpp
    // cout 用のヘッダ
    #include <iostream>
    // std::string 用のヘッダ
    #include <string>

    int main(void) {
        // 元になる文字列を用意
        std::string str = "HI THIS IS BRAIN";
        // ── ポインタの作成 ──
        // &str で str のアドレスを取得
        // string * は string のアドレス型
        std::string *stringPTR = &str;
        // ── 参照の作成 ──
        // string & は「string の別名」型
        // 宣言と同時に初期化が必須
        std::string &stringREF = str;

        // ── アドレスは3つとも同じになる ──
        // &str: 元の変数のアドレス
        std::cout << "Address: "
                  << &str << std::endl;
        // stringPTR: ポインタの中身=str のアドレス
        std::cout << "Address: "
                  << stringPTR << std::endl;
        // &stringREF: 参照のアドレス
        // → 参照は別名なので str と同じ
        std::cout << "Address: "
                  << &stringREF << std::endl;

        // ── 値も3つとも同じになる ──
        // str はそのまま値を表示
        std::cout << "Value: "
                  << str << std::endl;
        // *stringPTR: ポインタをデリファレンス
        // (C と同じく * で中身を取り出す)
        std::cout << "Value: "
                  << *stringPTR << std::endl;
        // stringREF: 参照はそのまま使える
        // (* も & もいらない！)
        std::cout << "Value: "
                  << stringREF << std::endl;
        return 0;
    }
    ```

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
str に "HI THIS IS BRAIN" を入れる
  |
  v
stringPTR に str のアドレスを入れる
  |  → ポインタ: 住所メモを持っている
  v
stringREF に str の別名をつける
  |  → 参照: 同一人物にあだ名をつけた
  v
3つのアドレスを出力
  |  → 全部同じ！
  v
3つの値を出力
  |  → 全部同じ！
  v
終了（参照はエイリアスだと証明された）
```

### main.cpp（ソースコード）

```cpp title="main.cpp" linenums="1"
// 画面出力に必要
#include <iostream>
// std::string に必要
#include <string>

int main(void) {
    // ── 変数の準備 ──
    // 元になる文字列
    std::string str = "HI THIS IS BRAIN";

    // ポインタ: str のアドレスを格納
    // &str で str のアドレスを取得
    std::string *stringPTR = &str;

    // 参照: str の別名
    // 宣言と同時に初期化が必須
    std::string &stringREF = str;

    // ── アドレスの表示 ──
    // &str: 元の変数のアドレス
    std::cout << "Address of str:       "
              << &str << std::endl;
    // stringPTR: ポインタの中身=アドレス
    std::cout << "Address of stringPTR: "
              << stringPTR << std::endl;
    // &stringREF: 参照のアドレス
    // → str と同じアドレス！
    std::cout << "Address of stringREF: "
              << &stringREF << std::endl;

    // ── 値の表示 ──
    // str: そのまま値を表示
    std::cout << "Value of str:         "
              << str << std::endl;
    // *stringPTR: ポインタをデリファレンス
    // * をつけて「中身を見る」
    std::cout << "Value of *stringPTR:  "
              << *stringPTR << std::endl;
    // stringREF: 参照はそのまま使える
    // * も & もいらない！
    std::cout << "Value of stringREF:   "
              << stringREF << std::endl;

    return 0;
}
```

!!! info "参照の一番の利点"
    ポインタは `*` でデリファレンスが必要ですが、
    参照は **普通の変数と同じ書き方** で使えます。

    ```cpp
    // ポインタ: * が必要
    std::cout << *stringPTR;

    // 参照: そのまま使える！
    std::cout << stringREF;
    ```

    コードが読みやすくなるのが参照の大きなメリットです。

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex02/"
    > "Files to turn in: Makefile, main.cpp"

    ポインタと参照の基本的な違いを理解しているかが評価のポイント。

- [ ] 3つのアドレスが全て同じ
- [ ] 3つの値が全て同じ
- [ ] ポインタと参照の違いを説明できる

---

## 8. テストチェックリスト

- [ ] `make` が警告なく通る
- [ ] 3つのアドレスが **全て同じ**
- [ ] 3つの値が **全て同じ** (`HI THIS IS BRAIN`)
- [ ] `str` を変更すると `stringREF` も変わる
- [ ] 規約違反なし

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| 参照とポインタの違いは？ | 参照は別名。NULLにできない、再代入できない、宣言時に初期化必須 |
| 参照のアドレスはなぜ元の変数と同じ？ | 参照は独立した変数ではなく、元の変数そのものだから |
| `&` の3つの意味は？ | 宣言時=参照型、式中の変数前=アドレス取得、二項演算=ビットAND |
| 参照を初期化しないとどうなる？ | コンパイルエラー。参照は宣言時に必ず初期化が必要 |
| いつ参照を使い、いつポインタを使う？ | 必ず存在+変更しない→参照。NULLの可能性あり or 変更する→ポインタ |

---

## 10. よくあるミス

!!! warning "参照の初期化忘れ"
    ```cpp
    // コンパイルエラー！
    // 参照は宣言時に初期化が必須
    std::string &ref;
    ```

!!! warning "ポインタのアドレスを間違えて出力"
    ```cpp
    // ダメ: ポインタ変数自身のアドレス
    // （str のアドレスではない）
    std::cout << &stringPTR;

    // 正しい: ポインタの中身 = str のアドレス
    std::cout << stringPTR;
    ```

---

## 11. 次の exercise へ

次の [ex03 Unnecessary violence](ex03-weapon.md) では、
**参照とポインタを実際に使い分ける** 実践的な exercise です。

「いつ参照を使い、いつポインタを使うか？」が明確になります。
