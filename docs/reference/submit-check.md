# 提出前チェックリスト

defense (評価) で**必ず**チェックされる項目。提出前に一通り見直すこと。

## 全 exercise 共通

### コンパイル

- [ ] `make` がエラーも**警告も**なく通る
- [ ] `make re` で完全リビルドできる
- [ ] `make clean` / `make fclean` で中間ファイルが消える
- [ ] `-Wall -Wextra -Werror -std=c++98` のフラグが Makefile にある
- [ ] `CXX = c++` を使用 (`g++` や `clang++` を直書きしない)

### ソースコード

- [ ] `printf` / `scanf` / `fprintf` / `fputs` / `fgets` を使っていない
- [ ] `malloc` / `calloc` / `realloc` / `free` を使っていない (cpp00では `new`/`delete` も不要)
- [ ] `using namespace std;` を書いていない
- [ ] `friend` キーワードを使っていない
- [ ] STL コンテナ (`vector`, `map` 等) を使っていない (ex02 の `tests.cpp` は例外)
- [ ] ヘッダに関数実装を書いていない (テンプレートを除く)
- [ ] すべての `.hpp` にインクルードガード (`#ifndef ... #define ... #endif`)
- [ ] `.cpp` ごとに対応する `.hpp` を `#include`

### ネーミング / スタイル

- [ ] private メンバ変数に `_` プレフィックス (42 慣習)
- [ ] クラス名は `CamelCase` (例: `PhoneBook`)
- [ ] ファイル名がクラス名に対応 (`Contact.hpp` / `Contact.cpp`)
- [ ] インデントはタブ (42 Norm的には)

## ex00 Megaphone

- [ ] 引数なし → `* LOUD AND UNBEARABLE FEEDBACK NOISE *`
- [ ] 引数あり → 全部大文字で連結、末尾に改行
- [ ] `(char)` キャストを使って `toupper` の戻り値を文字として出力

## ex01 PhoneBook

### クラス設計

- [ ] `Contact` と `PhoneBook` の2クラス
- [ ] すべてのメンバ変数が `private`
- [ ] すべての getter に末尾 `const`
- [ ] setter の引数が `const std::string &`
- [ ] `std::string` を使っている (`char[]` ではない)
- [ ] 動的確保 (`new`) を使っていない — 固定長配列 `Contact _contacts[8]`

### 挙動

- [ ] `ADD` → 5 フィールド必須 (空なら再入力)
- [ ] `SEARCH` で空時は `PhoneBook is empty.` 表示
- [ ] 各列が幅10・右寄せ・`|` 区切り
- [ ] 11文字以上のフィールドは `先頭9文字+.` に切り詰め (10文字はそのまま)
- [ ] Invalid な index は `Invalid index.`
- [ ] 8件以降は最古から上書き
- [ ] EOF (`Ctrl+D`) で正しく終了

## ex02 Account

### 実装

- [ ] `Account.hpp` と `tests.cpp` を改変していない
- [ ] `static` メンバ変数 4 つの実体を `.cpp` で定義
- [ ] `_displayTimestamp()` が `[YYYYMMDD_HHMMSS] ` 形式 (末尾半角スペース1個)
- [ ] `const` メンバ関数 `checkAmount`, `displayStatus` が末尾 `const`
- [ ] private なデフォルトコンストラクタを書いている

### 出力一致

- [ ] 行数が `19920104_091532.log` と一致
- [ ] タイムスタンプ以降の文字列が**完全一致** (スペース混入・句読点ミスなし)
- [ ] `refused` が 2 回 (index 0, 5)
- [ ] 最終 `accounts:8;total:12442;deposits:8;withdrawals:6`
- [ ] デストラクタの `;closed` が 8 行

### 比較コマンド例

```bash
# タイムスタンプを除外して diff
./account 2>&1 | cut -c20- > got.log
cut -c20- 19920104_091532.log > expect.log
diff got.log expect.log
# → 差分ゼロなら OK
```

## defense で聞かれがちな質問

| 質問 | 答え方の方針 |
|---|---|
| なぜ `std::endl` ? | 改行 + flush。フラッシュが要らないなら `"\n"` でも可 |
| なぜ getter に `const` ? | `const Foo&` からも呼び出せるようにするため |
| `std::string` と `char*` の違いは？ | 自動メモリ管理、`+`/`==` が使える、バッファあふれしない |
| なぜメンバ変数は `private` ? | カプセル化。外部から勝手に書き換えられない設計 |
| メンバ初期化子リストと代入の違いは？ | 初期化子リストは**初期化**、本体は**代入**。const/参照は前者でしか初期化できない |
| static メンバ変数はなぜ `.cpp` で定義？ | ヘッダは宣言。実体はリンク時に1つだけ必要 |
| `#pragma once` と include guard の違い | 前者はコンパイラ依存、後者は標準。42 は後者を好む |

## モジュール別の評価対策

各モジュールの詳しい Q&A・模範回答・即不合格フラグは以下を参照:

- [cpp00 評価対策](../cpp00/eval.md) — 基本 I/O、クラス設計、static メンバ
- [cpp01 評価対策](../cpp01/eval.md) — new/delete、参照 vs ポインタ、ファイル I/O
- [cpp02 評価対策](../cpp02/eval.md) — OCF、演算子オーバーロード、固定小数点
- [cpp03 評価対策](../cpp03/eval.md) — 継承、構築/破棄順序、ダイヤモンド問題
- [cpp04 評価対策](../cpp04/eval.md) — ポリモーフィズム、ディープコピー、抽象クラス

## 最後に

- [ ] `git status` で余計なファイル (`.o`, バイナリ, ログ) を含めていない
- [ ] `.gitignore` 設定済み
- [ ] 提出先は vogsphere — `cpp-modules` repo へは push せず、vogsphere に exercise ごとに push する
