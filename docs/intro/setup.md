# 開発環境とコンパイル

## 必要なもの

| 項目 | macOS (42 校内) | Linux |
|---|---|---|
| コンパイラ | `c++` (Apple clang) | `c++` (g++) |
| Makefile | GNU make | GNU make |
| C++ 標準 | **c++98** (42 指定) | **c++98** |

バージョン確認:
```bash
c++ --version
make --version
```

## 42 指定のコンパイルフラグ

cpp00 のすべての Makefile はこのフラグで統一します。

```makefile
CXX      = c++
CXXFLAGS = -Wall -Wextra -Werror -std=c++98
```

| フラグ | 意味 | なぜ必要か |
|---|---|---|
| `-Wall` | よくある警告を有効 | バグの温床を早期発見 |
| `-Wextra` | 追加の警告 | `-Wall` が見逃す警告もカバー |
| `-Werror` | 警告をエラー扱い | **警告を残したら即コンパイル失敗** |
| `-std=c++98` | C++98 規格で解釈 | 42 モジュール全体の縛り。STL は限定的にのみ使える |

!!! danger "なぜ C++98 なのか？"
    42 の教育方針で、**まずは OOP の基礎を体得させるため** に古い規格を指定。
    `auto`、ラムダ式、`nullptr`、`std::unique_ptr` 等は cpp00〜cpp09 では **使えない**。

## Makefile テンプレート (cpp00 共通)

```makefile
NAME = megaphone
CXX = c++
CXXFLAGS = -Wall -Wextra -Werror -std=c++98

SRCS = megaphone.cpp
OBJS = $(SRCS:.cpp=.o)

all: $(NAME)

$(NAME): $(OBJS)
	$(CXX) $(CXXFLAGS) -o $(NAME) $(OBJS)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS)

fclean: clean
	rm -f $(NAME)

re: fclean all

.PHONY: all clean fclean re
```

!!! note "C と変わらない部分"
    Makefile の書き方自体は C と同じ。拡張子が `.cpp` / `.hpp` になり、コンパイラ名が `c++` になるだけ。

## 42 の禁則事項 (cpp00 全体に適用)

| ❌ 使ってはダメ | 代わりに使うもの |
|---|---|
| `printf` / `scanf` / `fprintf` | `std::cout`, `std::cin`, `std::cerr` |
| `malloc` / `calloc` / `realloc` / `free` | `new` / `delete` (cpp01 から) or stack |
| `strcpy`, `strcat`, `strcmp` 等 `<cstring>` の大半 | `std::string` のメンバ関数と演算子 |
| `using namespace std;` | 常に `std::` 付き |
| ヘッダに関数実装 (テンプレート以外) | 宣言は `.hpp`、実装は `.cpp` |

## よくある環境トラブル

### 警告で落ちる → 修正必須

`-Werror` があるため、警告は必ず消す。`// unused` 的なコメントで逃げず、根本対処。

### `std::string` が `undefined reference`

→ `<string>` を `#include` 忘れ。

### コンパイル成功なのに `./program` で何も出ない

→ `std::cout << ...` の後に `std::endl` か `"\n"` を忘れている (バッファフラッシュ未実行)。
{==`std::endl` は改行＋フラッシュ==}、`"\n"` は改行のみ。

### `make` が「No such file」と言う

→ `.cpp` と `.hpp` の **大文字小文字** は厳密。macOS はデフォルトで大小無視の FS だが Linux では区別される。
