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
