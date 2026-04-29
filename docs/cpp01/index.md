# cpp01 モジュール概要

---

## このモジュールは何？

cpp01 は **「メモリの使い方」と「参照」を覚える** モジュールです。

C言語では `malloc` / `free` でメモリを管理していましたが、
C++ では **`new` / `delete`** を使います。
さらに、ポインタの代わりに使える **参照 (`&`)** という便利な道具が登場します。

このモジュールを終えると、
**「ヒープとスタックの違い」「参照とポインタの使い分け」** が
自信を持って説明できるようになります。

---

## このモジュールで初めて出てくる道具

### `new` / `delete` って何？

C の `malloc` / `free` の代わりになるものです。
**最大の違いは、`new` はコンストラクタ（初期化処理）を自動で呼ぶ**こと。

```cpp
// C の書き方
t_zombie *z = malloc(sizeof(t_zombie));
// → メモリを確保するだけ。初期化は自分でやる

// C++ の書き方
Zombie *z = new Zombie("name");
// → メモリ確保 + コンストラクタ呼び出し
//    を同時にやってくれる
```

### 参照 (`&`) って何？

**変数の「別名」** です。
ポインタと似ていますが、もっとシンプルに使えます。

```cpp
int x = 42;
int &ref = x;  // ref は x の別名

ref = 100;     // x も 100 になる！
// ref と x は同じものを指している
```

---

## 学ぶ概念マップ

```
                       cpp01
                         |
    +--------+-------+---+---+-------+--------+--------+
    |        |       |       |       |        |        |
   ex00     ex01    ex02    ex03    ex04     ex05     ex06
  Zombie   Horde   Brain  Weapon    Sed     Harl    Filter
    |        |       |       |       |        |        |
  new/     new[]/  ポインタ 参照 vs  ファイル  メンバ   switch
  delete   delete[] vs 参照 ポインタ  I/O    関数ptr  fall-through
    |        |       |    の使い分け   |        |        |
  スタック   動的   アドレス  const   std::    並列     意図的な
  vs ヒープ  配列   の一致  参照戻り ifstream  配列    break省略
```

---

## 7つの exercise と所要時間の目安

| # | 名前 | 難度 | 時間 | 何を学ぶ？ |
|---|---|:---:|:---:|---|
| [ex00](ex00-zombie.md) | BraiiiiiiinnnzzzZ | ★★ | 30分 | `new`/`delete`、スタック vs ヒープ |
| [ex01](ex01-zombie-horde.md) | Moar brainz! | ★★ | 30分 | `new[]`/`delete[]`、配列の動的確保 |
| [ex02](ex02-brain.md) | HI THIS IS BRAIN | ★ | 15分 | ポインタ vs 参照のアドレス比較 |
| [ex03](ex03-weapon.md) | Unnecessary violence | ★★★ | 1-2時間 | 参照メンバ vs ポインタメンバ |
| [ex04](ex04-sed.md) | Sed is for losers | ★★★ | 2-3時間 | ファイル I/O、文字列置換 |
| [ex05](ex05-harl.md) | Harl 2.0 | ★★ | 1時間 | メンバ関数ポインタ |
| [ex06](ex06-harl-filter.md) | Harl filter | ★★ | 30分 | switch 文の fall-through |

---

## 共通ルール（全 exercise で守ること）

- コンパイル: `c++ -Wall -Wextra -Werror -std=c++98`
- `using namespace std;` は **禁止**
- `printf` / `malloc` / `free` は **使用禁止**
- ヘッダに関数実装を書かない（テンプレートを除く）
- `friend` キーワードは禁止

---

## C と C++ の比較表

| やりたいこと | C の書き方 | C++ の書き方 |
|------|---|-----|
| メモリ確保（1個） | `malloc(sizeof(T))` | `new T(args)` |
| メモリ解放（1個） | `free(ptr)` | `delete ptr` |
| メモリ確保（配列） | `malloc(N * sizeof(T))` | `new T[N]` |
| メモリ解放（配列） | `free(ptr)` | `delete[] ptr` |
| 変数の別名 | ポインタしかない | **参照 (`&`)** が使える |
| ファイル読み込み | `fopen`/`fgets`/`fclose` | `std::ifstream` + `std::getline` |

!!! tip "`new` と `malloc` の一番大きな違い"
    `new` は **コンストラクタを自動で呼ぶ**。
    `malloc` はメモリを確保するだけで、中身は初期化されない。

    同じように `delete` は **デストラクタを呼んでから** メモリを解放する。
    `free` はいきなりメモリを解放するだけ。

---

## 評価（ディフェンス）の流れ

1. `make` してエラーなくビルドできるか
2. プログラムを実行して、正しい出力が出るか
3. メモリリークがないか確認（特に ex00, ex01）
4. 口頭質問に答える

よく聞かれる質問の例：

- 「なぜ参照にした？ポインタじゃダメ？」
- 「`new[]` と `delete[]` を対にする理由は？」
- 「スタックとヒープの違いは？」

!!! tip "評価対策ページ"
    詳しい Q&A と模範回答は [評価対策](eval.md) ページを参照。

---

次は [ex00 BraiiiiiiinnnzzzZ](ex00-zombie.md) から始めましょう。
