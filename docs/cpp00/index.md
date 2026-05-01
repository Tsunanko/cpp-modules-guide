# cpp00 モジュール概要

## このモジュールの位置づけ

cpp00 は C++ モジュール全9回 (cpp00〜cpp09) の**入門編**です。
**「C の書き方から C++ の OOP 思考にスイッチする」**ことが最大の目標。

## 学ぶ概念マップ

```
                    cpp00
                      │
        ┌─────────────┼─────────────┐
        │             │             │
       ex00          ex01          ex02
    Megaphone     PhoneBook       Account
        │             │             │
  • iostream    • class         • static メンバ
  • namespace   • public/private • ログ出力
  • cout        • コンストラクタ • const const const
  • cctype      • デストラクタ   • typedef
                • const 参照
                • std::string
                • iomanip
                • std::getline
```

## このマップに出てくる用語の最低限の定義

上のマップで初めて見る単語が多いはずなので、まず **1 行ずつの最小定義** を載せます。詳細は ex01/ex02 で各概念を初登場した時に「〇〇って何？」形式で丁寧に解説します。

| 用語 | 1 行で言うと | 詳しく学ぶ場所 |
|---|---|---|
| **`namespace`** | 関数や変数がぶつからないように区切る「住所」 | [ex00 Megaphone](ex00-megaphone.md) |
| **`std::cout`** | `printf` の代わりの出力装置 (`std` という住所にいる) | [ex00 Megaphone](ex00-megaphone.md) |
| **`class`** | データと関数を 1 つの箱にまとめる仕組み (C の `struct` の進化版) | [ex01 PhoneBook](ex01-phonebook.md) |
| **`public` / `private`** | クラスの中で「外から触れる/触れない」を分けるラベル | [ex01 PhoneBook](ex01-phonebook.md) |
| **コンストラクタ** | オブジェクトが作られた瞬間に**自動で呼ばれる関数**。初期化担当 | [ex01 PhoneBook](ex01-phonebook.md) |
| **デストラクタ** | オブジェクトが消える瞬間に**自動で呼ばれる関数**。後片付け担当 | [ex01 PhoneBook](ex01-phonebook.md) |
| **const 参照** | 「変更しない」と約束した変数の別名。コピーせずに渡せる | [ex01 PhoneBook](ex01-phonebook.md) |
| **`std::string`** | 可変長の文字列クラス (`char[]` の上位互換) | [ex01 PhoneBook](ex01-phonebook.md) |
| **static メンバ** | 1 つのクラスに 1 つだけ存在する**全インスタンス共有の変数**。インスタンス A も B も同じ値を見る | [ex02 Account](ex02-account.md) |
| **`typedef`** | 型に**別名**を付ける機能 (`typedef int t_count;` なら `t_count` は `int` の別名) | [ex02 Account](ex02-account.md) |

!!! tip "読み進める順番"
    各 exercise には「2. 新しい概念の解説」セクションがあります。**そこを最初に読む**と、コード解説に入る前に概念が腹落ちします。

## 3 つの exercise と所要時間の目安

| # | 名前 | 難度 | 所要時間 | 主要トピック |
|---|---|:---:|:---:|---|
| [ex00](ex00-megaphone.md) | Megaphone | ★ | 15分 | 基本 I/O、namespace |
| [ex01](ex01-phonebook.md) | My Awesome PhoneBook | ★★★ | 3〜5時間 | class, コンストラクタ, `std::string`, `iomanip` |
| [ex02](ex02-account.md) | The Job Of Your Dreams (任意) | ★★★★ | 4〜8時間 | static メンバ、ログ出力の逆算 |

!!! note "ex02 は任意だが推奨"
    ex02 は必須ではないが、cpp01 以降で頻繁に使う **static メンバ** の唯一の練習機会。飛ばさずやったほうがよい。

## 共通ルール

- コンパイル: `c++ -Wall -Wextra -Werror -std=c++98`
- `using namespace std;` は**禁止**
- `printf` / `malloc` / `free` は**使用禁止**
- ヘッダに関数実装を書かない (テンプレートを除く)
- STL コンテナ (vector, map 等) は ex02 の `tests.cpp` 以外**使用禁止**

## 評価 (defense) の大まかな流れ

1. コードを `make` してクリーンビルドできるか
2. サンプル実行を行い、期待通りの出力になるか
3. ソースコードを評価者が読み、規則違反がないかチェック
4. 評価者からの口頭質問 (「なぜここは `const` ?」「`std::endl` と `\n` の違いは？」等) に答える

!!! tip "このサイトの「テストチェックリスト」は評価時に試される項目をまとめたもの"
    各 exercise のチェックリストを全部パスすれば defense で詰まることは少ない。

次は [ex00 Megaphone](ex00-megaphone.md) から始めましょう。
