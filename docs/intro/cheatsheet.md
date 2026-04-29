# C ↔ C++ 対応表チートシート

印刷 / ブックマーク推奨。困ったら戻ってくる用。

## ヘッダ

| C | C++ | 備考 |
|---|---|---|
| `<stdio.h>` | `<iostream>` | cout/cin |
| `<string.h>` | `<string>` + `<cstring>` | `std::string` は `<string>` |
| `<ctype.h>` | `<cctype>` | `std::toupper` |
| `<stdlib.h>` | `<cstdlib>` | |
| `<time.h>` | `<ctime>` | ex02 で使用 |
| — | `<iomanip>` | `setw`, `right` |

!!! tip
    C の `<xxx.h>` は C++ では `<cxxx>` にするのが公式 (例: `stdio.h` → `cstdio`)。
    関数は `std::` 名前空間に入る。

## I/O

| やりたいこと | C | C++ |
|---|---|---|
| 文字列出力 | `printf("%s", s)` | `std::cout << s` |
| 整数出力 | `printf("%d", n)` | `std::cout << n` |
| 改行 | `printf("\n")` | `std::cout << std::endl` |
| 1行入力 | `fgets(buf, N, stdin)` | `std::getline(std::cin, s)` |
| エラー出力 | `fprintf(stderr, ...)` | `std::cerr << ...` |
| 幅10で右寄せ | `printf("%10d", n)` | `std::cout << std::setw(10) << std::right << n` |

## 文字列

| やりたいこと | C (`char*`) | C++ (`std::string`) |
|---|---|---|
| 長さ取得 | `strlen(s)` | `s.length()` または `s.size()` |
| 連結 | `strcat(a, b)` | `a + b` or `a += b` |
| 比較 | `strcmp(a, b) == 0` | `a == b` |
| コピー | `strcpy(d, s)` | `d = s` |
| 空判定 | `s[0] == '\0'` | `s.empty()` |
| 部分文字列 | 自前ループ | `s.substr(start, len)` |
| 入力 | `scanf` / `fgets` | `std::cin >>` / `std::getline` |
| 解放 | `free(s)` (自分で) | 自動 |

## メモリ

| 用途 | C | C++ |
|---|---|---|
| 単一オブジェクト確保 | `malloc(sizeof(T))` | `new T` |
| 配列確保 | `malloc(sizeof(T)*N)` | `new T[N]` |
| 解放 | `free(p)` | `delete p` / `delete[] p` |
| スタック確保 | 変数宣言 | 変数宣言 (コンストラクタ自動) |

!!! info "cpp00 では `new` / `delete` も不要"
    cpp00 は全部スタックで OK。`new` は cpp01 から登場。

## OOP の基本

| 概念 | C 風 | C++ |
|---|---|---|
| データ+関数 | `struct` + 関数群 | `class` |
| 可視性 | 命名規則で暗黙 | `public` / `private` 明示 |
| 初期化 | `init()` 関数を呼ぶ | コンストラクタが自動 |
| 後片付け | `destroy()` 関数を呼ぶ | デストラクタが自動 |
| 第一引数のself | `void foo(t_obj *self, ...)` | `void foo(...)` (`this` 暗黙) |

## スコープとアクセス

| 演算子 | 用途 | 例 |
|---|---|---|
| `::` | 名前空間 / クラス名のスコープ解決 | `std::cout`, `Account::getNbAccounts()` |
| `.` | オブジェクト.メンバ | `contact.getFirstName()` |
| `->` | ポインタ->メンバ | `contactPtr->getFirstName()` |

## const の三態

```cpp
const std::string& s       // (1) s を変更しない (引数)
std::string getName() const // (2) メンバ変数を変更しない (関数末尾)
const Contact c            // (3) オブジェクト自体が const
```

## 42 禁止リスト (cpp00)

- ❌ `printf`, `scanf`, `fprintf`, `fgets`, `fputs`
- ❌ `malloc`, `calloc`, `realloc`, `free`
- ❌ `using namespace foo;`
- ❌ `friend` キーワード
- ❌ ヘッダ内に関数実装 (テンプレート以外)
- ❌ STL (vector, map など) — ただし **ex02 の `tests.cpp` は例外** (42 提供コードのため)

## コンパイル定石

```bash
c++ -Wall -Wextra -Werror -std=c++98 src.cpp -o prog
```

すべての Makefile に上記フラグを記載。
