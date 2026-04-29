# ex04 — Sed is for losers

---

## このプログラムは何？

**ファイルの中の文字を置き換えるプログラム** です。

Linux の `sed` コマンドみたいに、ファイルを読んで
「この単語を別の単語に変えて」を実行します。
結果は新しいファイル `<元の名前>.replace` に保存されます。

```
入力ファイル: "Hello World Hello"
実行: ./sed test.txt Hello Goodbye
出力ファイル: "Goodbye World Goodbye"
```

C の `fopen`/`fgets`/`fputs` 相当を C++ の
**ストリーム** で書くのがこの exercise の目的です。

---

## 1. このexerciseで学ぶこと

- **`std::ifstream`**: ファイルを読む道具（C の `fopen(..., "r")`）
- **`std::ofstream`**: ファイルに書く道具（C の `fopen(..., "w")`）
- **`std::getline()`**: 1 行ずつ読み込む関数
- **`std::string::find()`**: 文字列の中を探す
- **`std::string::substr()`**: 文字列の一部を切り取る
- **禁止事項**: `std::string::replace()` は使ってはいけない

---

## 2. 新しい概念の解説

### `std::ifstream` って何？

**ファイルを「読むための」道具** です。
C の `fopen(name, "r")` + `fclose` をひとまとめに自動化したもの。

C でファイルを読むと3ステップ必要でした。
C++ の `ifstream` はそれを1行 + 自動クローズに変えます。

#### C でファイルを読む場合（3 ステップ + 手動 close）

```c
/* ── ステップ1: fopen で開く ── */
/* "r" は読み込みモード */
FILE *fp = fopen("test.txt", "r");

/* ── ステップ2: NULL チェック ── */
/* fopen は失敗すると NULL を返す */
if (fp == NULL) {
    perror("fopen");
    return 1;
}

/* ── ステップ3: 使い終わったら fclose ── */
/* 忘れるとリソースリーク */
fclose(fp);
```

#### C++ で `std::ifstream` を使う場合（1 行 + 自動クローズ）

```cpp
// fstream ヘッダ (ifstream/ofstream 用)
#include <fstream>

// ── この1行で以下が全部起きる ──
//   ①ファイルを読み込みモードで開く
//   ②ifstream オブジェクトが作られる
std::ifstream infile("test.txt");

// is_open(): ちゃんと開けたかを true/false
// (C の NULL チェックの代わり)
if (!infile.is_open()) {
    // エラー処理
}

// ── RAII (自動クローズ) ──
// スコープを抜けると infile のデストラクタが
// 自動で close を呼んでくれる
// → fclose を書き忘れるバグが起きない
```

特徴:

- **開けたか確認** → `is_open()` で `true`/`false` が返る
- **自動で閉じる** → スコープを抜けるとデストラクタが close する (RAII)
- **`fclose` 不要**（でも明示的に `close()` してもOK）

### `std::ofstream` って何？

**ファイルを「書くための」道具** です。
C の `fopen(name, "w")` + `fprintf` に相当します。

```cpp
std::ofstream outfile("result.txt");

outfile << "Hello" << std::endl;
outfile << "数字も出せる: " << 42 << std::endl;
```

`std::cout` と同じように `<<` で書き込めます。

### `std::getline()` って何？

**ファイルから 1 行ずつ読む関数** です。
C の `fgets` に相当しますが、改行文字は自動で除去されます。

C で1行読むと3つ手間がありました。
`std::getline` はそれを1行にまとめます。

#### C で `fgets` を使う場合（3 つの手間）

```c
/* ── 手間1: バッファサイズを先に決める ── */
/* 1024 を超える行は読めない問題あり */
char buf[1024];

/* ── 手間2: fgets で1行読む ── */
/* 引数: バッファ, サイズ, ファイル */
/* 戻り値: NULL なら EOF */
if (fgets(buf, sizeof(buf), fp)) {
    /* ── 手間3: 末尾の \n を自分で消す ── */
    /* fgets は改行を残したまま返す */
    size_t len = strlen(buf);
    if (len > 0 && buf[len-1] == '\n')
        buf[len-1] = '\0';
    printf("読んだ: %s\n", buf);
}
```

#### C++ で `std::getline` を使う場合（1 行）

```cpp
// 1行を格納する string 変数
// (サイズは自動で伸びるので制限なし)
std::string line;

// ── この1行で以下が全部起きる ──
//   ①1行読み込む (サイズ制限なし)
//   ②末尾の \n を自動で削除
//   ③EOF なら false を返す
// → while の条件に書ける
while (std::getline(infile, line)) {
    // line には \n は入っていない
    std::cout << "読んだ: "
              << line << std::endl;
}
```

!!! warning "改行は自動で消える"
    `fgets` は末尾に `\n` が残りますが、
    `getline` は改行を取り除いて `string` に入れます。
    出力するときは自分で改行を付け直す必要があります。

### `std::string::find()` って何？

**文字列の中から別の文字列を探す関数** です。
C の `strstr` に相当します。

#### C で `strstr` を使う場合

```c
/* haystack から needle を探す */
char *s = "Hello World";
/* strstr: 見つかったらポインタを返す */
/*        見つからないと NULL */
char *p = strstr(s, "World");
if (p != NULL) {
    /* 位置 = p - s (ポインタ引き算) */
    printf("位置: %ld\n", p - s);
}
```

#### C++ で `std::string::find` を使う場合

```cpp
std::string s = "Hello World";
// find: 見つかったら位置(数字)を返す
//       見つからないと npos を返す
// npos は size_t の最大値 (=見つからない印)
std::size_t pos = s.find("World");

// std::string::npos と比較して判定
// (C のポインタ NULL 比較の代わり)
if (pos != std::string::npos) {
    std::cout << "位置: "
              << pos << std::endl;
}
```

`npos` は **「見つからなかった」** を意味する特別な値です。
`(size_t)-1` とほぼ同じ扱い（とても大きな数）。

### `std::string::substr()` って何？

**文字列の一部を切り取る関数** です。
C の `strncpy` を安全にしたものと考えると分かりやすい。

```cpp
std::string s = "Hello World";

// substr(開始位置, 長さ)
s.substr(0, 5);    // "Hello"
s.substr(6, 5);    // "World"
s.substr(6);       // "World"（末尾まで）
```

### `.c_str()` って何？

**`std::string` を `const char*` に変換するメソッド** です。

```cpp
std::string filename = "test.txt";

// ❌ C++98 ではこれはコンパイルエラー
std::ifstream f(filename);

// ✅ .c_str() で char* に変換する
std::ifstream f(filename.c_str());
```

!!! info "なぜ必要？"
    C++98 の `ifstream`/`ofstream` のコンストラクタは
    `const char*` しか受け取れません。
    C++11 以降は `std::string` も直接 OK ですが、
    **42 は C++98 縛りなので `.c_str()` が必要** です。

### なぜ `std::string::replace()` は禁止？

subject で明示的に禁止されています。
このexerciseの目的は **「置換アルゴリズムを自分で書くこと」** です。

```cpp
// ❌ 禁止
line.replace(pos, len, s2);

// ✅ 自分で書く
result += line.substr(0, pos);  // 前の部分
result += s2;                    // 置換文字列
line = line.substr(pos + s1.length());  // 残り
```

`find` + `substr` で位置を特定し、
新しい文字列を組み立てるのが正解です。

---

## 3. 課題仕様

| 項目 | 内容 |
|------|------|
| プログラム名 | `sed`（または `replace`） |
| 引数 | `<filename> <s1> <s2>` |
| 動作 | `<filename>` を開き、`s1` を全て `s2` に置換 |
| 出力先 | `<filename>.replace` という新しいファイル |
| 禁止 | `std::string::replace()` の使用 |
| エラー処理 | 引数不足、ファイル読めない |

---

## 4. 実行例

```console
$ echo "Hello World Hello World" > test.txt
$ ./sed test.txt Hello Goodbye
$ cat test.txt.replace
Goodbye World Goodbye World

$ ./sed
Usage: ./sed <file> <s1> <s2>

$ ./sed nothere.txt a b
Error: cannot open nothere.txt
```

---

## 5. C と C++ の比較

=== "C の書き方"

    ```c
    /* printf, fprintf, FILE 用のヘッダ */
    #include <stdio.h>
    /* strlen などの文字列関数用 */
    #include <string.h>

    int main(int argc, char **argv) {
        /* 引数チェック: 4個必要 */
        if (argc != 4) {
            /* stderr: 標準エラー出力 */
            fprintf(stderr, "Usage...\n");
            return 1;
        }
        /* fopen: ファイルを開く (読み込み) */
        /* 失敗すると NULL を返す */
        FILE *in = fopen(argv[1], "r");
        /* NULL チェック */
        if (!in) {
            fprintf(stderr, "cannot open\n");
            return 1;
        }
        /* 1行を溜めるバッファ (サイズ固定) */
        char buf[1024];
        /* fopen: 書き込みモード "w" で開く */
        FILE *out = fopen("out.replace", "w");
        /* fgets: 1行読む (EOF で NULL) */
        while (fgets(buf, sizeof(buf), in)) {
            /* 自分で strstr + 置換 */
            /* fputs: 文字列を書き込む */
            fputs(buf, out);
        }
        /* fclose: 必ず閉じる */
        /* (忘れるとリークするので手動必須) */
        fclose(in);
        fclose(out);
        return 0;
    }
    ```

=== "C++ の書き方"

    ```cpp
    // cerr 用のヘッダ
    #include <iostream>
    // ifstream / ofstream 用のヘッダ
    #include <fstream>
    // std::string 用のヘッダ
    #include <string>

    int main(int argc, char **argv) {
        // 引数チェック
        if (argc != 4) {
            // cerr: 標準エラー出力
            // (C の stderr 相当)
            std::cerr << "Usage..."
                      << std::endl;
            return 1;
        }
        // ifstream: 読み用ファイルを開く
        // fopen(..., "r") + fclose 自動化
        std::ifstream in(argv[1]);
        // is_open: ちゃんと開けたか確認
        if (!in.is_open()) {
            std::cerr << "cannot open"
                      << std::endl;
            return 1;
        }
        // ofstream: 書き用ファイルを開く
        std::ofstream out("out.replace");
        // 1行を格納する string (可変長)
        std::string line;
        // getline: 1行読む + 改行除去
        // (C の fgets + 改行削除を合体)
        while (std::getline(in, line)) {
            // find + substr で自作置換
            // << で書き込み (printf の代わり)
            out << line << "\n";
        }
        // close 不要 (スコープ抜ける時に
        // デストラクタが自動で呼ぶ = RAII)
        return 0;
    }
    ```

**何が変わった？**

| C | C++ | 一言で言うと |
|---|-----|-------------|
| `FILE *f = fopen(..., "r")` | `std::ifstream f(...)` | 読み用ストリーム |
| `FILE *f = fopen(..., "w")` | `std::ofstream f(...)` | 書き用ストリーム |
| `fgets(buf, N, f)` | `std::getline(f, line)` | 1 行読み込み |
| `fprintf(f, "%s", s)` | `f << s` | `<<` で書き込み |
| `fclose(f)` | 自動 close | RAII |
| `strstr(s, sub)` | `s.find(sub)` | 見つからないと `npos` |

---

## 6. コード解説

### プログラムの流れ

```
スタート
  |
  v
引数は 4 つ？ --NO--> エラーメッセージ → 終了
  |
 YES
  v
入力ファイルを開く --失敗--> エラー → 終了
  |
  v
出力ファイル <name>.replace を開く
  |
  v
1 行読む (getline)
  |
  v
行の中の s1 を全部 s2 に置換 (find + substr)
  |
  v
出力ファイルに書き出し
  |
  v
EOF じゃなければ戻る ↑
  |
 EOF
  v
自動で close → 終了
```

### ファイル別の解説

#### main.cpp — ファイルを開いて処理する

```cpp title="main.cpp" linenums="1"
#include <iostream>  // cout, cerr 用
#include <fstream>   // ifstream, ofstream 用
#include <string>    // string 用

// ─── 置換関数（下で定義）───
static std::string replaceAll(
    std::string line,
    const std::string &s1,
    const std::string &s2
);

int main(int argc, char **argv)
{
    // ── 引数チェック ──
    // プログラム名 + 3 つの引数 = 4
    if (argc != 4)
    {
        // cerr は標準エラー出力（stderr 相当）
        std::cerr
            << "Usage: ./sed <file> <s1> <s2>"
            << std::endl;
        return (1);
    }

    // 引数を string に入れる（扱いやすい）
    std::string filename = argv[1];
    std::string s1 = argv[2];
    std::string s2 = argv[3];

    // ── 入力ファイルを開く ──
    // .c_str() は C++98 で必須
    std::ifstream infile(filename.c_str());
    if (!infile.is_open())
    {
        std::cerr
            << "Error: cannot open "
            << filename << std::endl;
        return (1);
    }

    // ── 出力ファイルを開く ──
    // 名前は <元の名前>.replace
    std::string outname = filename + ".replace";
    std::ofstream outfile(outname.c_str());
    if (!outfile.is_open())
    {
        std::cerr
            << "Error: cannot create output"
            << std::endl;
        return (1);
    }

    // ── 1 行ずつ読んで、置換して、書き出す ──
    std::string line;
    bool first = true;
    while (std::getline(infile, line))
    {
        // 2 行目以降は先頭に改行を付ける
        // （末尾改行でなく行頭改行にすると
        //   最終行に余計な改行が付かない）
        if (!first)
            outfile << "\n";
        outfile << replaceAll(line, s1, s2);
        first = false;
    }

    // close は不要（デストラクタが自動で閉じる）
    // でも明示的に書いてもOK
    return (0);
}
```

#### replaceAll 関数 — 置換の核心

```cpp title="main.cpp (replaceAll)" linenums="1"
// line の中の s1 を全て s2 に置換する
static std::string replaceAll(
    std::string line,
    const std::string &s1,
    const std::string &s2
) {
    std::string result;
    std::size_t pos;

    // ── 空文字列チェック ──
    // s1 が "" だと find が常に 0 を返して
    // 無限ループになるので先に弾く
    if (s1.empty())
        return (line);

    // ── s1 が見つかる間、繰り返し処理 ──
    // find は見つからないと npos を返す
    while ((pos = line.find(s1))
           != std::string::npos)
    {
        // s1 より前の部分を結果に追加
        result += line.substr(0, pos);
        // s1 の代わりに s2 を入れる
        result += s2;
        // 残りの部分を次のループで処理
        line = line.substr(pos + s1.length());
    }

    // もう s1 がない残り部分を追加
    result += line;
    return (result);
}
```

**ループの動きを図にすると:**

```
line = "Hello World Hello"
s1   = "Hello"
s2   = "Hi"

1回目: pos = 0
       result = "" + "Hi" = "Hi"
       line   = " World Hello"

2回目: pos = 7
       result = "Hi" + " World" + "Hi" = "Hi WorldHi"
       line   = ""

3回目: find は npos → ループ終了
       result += "" → "Hi WorldHi"
```

---

## 7. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex04/"
    > "Files to turn in: Makefile, main.cpp,
    > and any extra files you need."

    禁止事項: `std::string::replace()` の使用。

- [ ] 引数 3 つ（ファイル名、s1、s2）を受け取る
- [ ] 引数不足でエラーメッセージを出す
- [ ] 開けないファイルでエラーメッセージを出す
- [ ] `<filename>.replace` が生成される
- [ ] 中身の s1 が全て s2 に置換されている
- [ ] `std::string::replace()` を使っていない

---

## 8. テストチェックリスト

### 基本動作

- [ ] `make` が警告なく通る
- [ ] 1 回の置換が正しく動く
- [ ] 1 行に複数出現する s1 が全て置換される
- [ ] 複数行のファイルで全行に置換が適用される
- [ ] 出力ファイル名が `<input>.replace` になる

### エッジケース

- [ ] `s1` が空文字列 → 無限ループにならない（そのまま返す）
- [ ] `s1` と `s2` が同じ → 変化なし（でもファイル生成）
- [ ] 存在しないファイル → エラーメッセージ
- [ ] 引数が足りない → エラーメッセージ
- [ ] 空のファイル → 空の `.replace` が生成される
- [ ] s1 が行をまたがる → 置換されない（行単位処理のため）

### 禁止事項

- [ ] `std::string::replace()` 不使用
- [ ] `printf` / `malloc` / `free` 不使用
- [ ] `using namespace std;` なし

---

## 9. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| `ifstream` と `ofstream` の違いは？ | `ifstream` が読み用、`ofstream` が書き用 |
| なぜ `.c_str()` が必要？ | C++98 の ifstream は `const char*` しか受け取れない |
| `getline` と `fgets` の違いは？ | `getline` は改行を除去する、`fgets` は残す |
| `find` は見つからないと何を返す？ | `std::string::npos`（とても大きな数） |
| なぜ `replace()` を使わない？ | subject で禁止。置換ロジックを自作するのが目的 |
| ファイルの close はいつ？ | デストラクタで自動。スコープを抜けると閉じる（RAII） |
| `s1` が空だとどうなる？ | `find` が常に 0 を返し無限ループ。先に弾く必要あり |

---

## 10. よくあるミス

!!! warning "`std::string::replace()` を使ってしまう"
    ```cpp
    line.replace(pos, len, s2);  // ❌ 禁止
    ```
    `find` + `substr` で自作するのが正解。

!!! warning "`.c_str()` を忘れる"
    ```cpp
    // ❌ C++98 ではコンパイルエラー
    std::ifstream infile(filename);

    // ✅
    std::ifstream infile(filename.c_str());
    ```

!!! warning "改行の扱いを間違える"
    `std::getline` は改行を取り除きます。
    そのまま書き出すと全行が連結されて
    1 行になってしまいます。
    行間に改行を入れる処理を忘れないこと。

!!! warning "空の s1 で無限ループ"
    ```cpp
    "".find("")  // 常に 0 を返す！
    ```
    空文字列チェックを忘れると無限ループに。

!!! warning "ファイルの開けたか確認しない"
    ```cpp
    std::ifstream f(name.c_str());
    // is_open() チェックなしだと
    // 存在しないファイルでも気づかない
    ```

---

## 11. 次の exercise へ

次の [ex05 Harl 2.0](ex05-harl.md) では、
**メンバ関数ポインタ** を学びます。

if/else の代わりに「関数のアドレスを配列に入れて
インデックスで呼び出す」というテクニックです。
