# ex01 — My Awesome PhoneBook

---

## このプログラムは何？

**対話式の電話帳プログラム**です。

コマンドを入力して、連絡先を追加したり検索したりできます。
最大8件まで保存でき、9件目以降は一番古いデータを上書きします。
プログラムを終了すると、保存したデータは全部消えます。

```
ADD    → 連絡先を登録する
SEARCH → 連絡先を表で一覧表示する
EXIT   → プログラムを終了する
```

この exercise は **C++ で初めてクラスを書く** 練習です。

---

## 1. このexerciseで学ぶこと

C の構造体 (`struct`) を卒業して、
C++ の**クラス (`class`)**を覚える exercise です。

- **`class`** -- データと関数を1つにまとめる箱
- **コンストラクタ / デストラクタ** -- 自動で呼ばれる特別な関数
- **`private` / `public`** -- 見せる・隠すの仕分け
- **getter / setter** -- private データへの窓口
- **`const`** -- 「変更しません」の約束
- **`std::getline`** -- スペースを含む1行まるごと読む
- **`<iomanip>`** -- 表示を整えるための道具

---

## 2. 新しい概念をひとつずつ解説

### クラス (`class`) って何？

**データと関数を1つの箱にまとめたもの**です。

C では「データを入れる箱 (struct)」と
「それを操作する関数」が**別々の場所**に散らばります。
C++ の `class` は両方を **1つにまとめて** 管理します。

**C の場合（3つの作業が必要）**

1. `struct` でデータの形を定義する
2. データを操作する関数を**別途**書く
3. アクセス制御がないので誰でも中身を書き換え可能

**C++ の場合（1つの class で完結）**

- `class` の中に「データ」と「関数」を両方書く
- `private` / `public` でアクセス制御までできる

=== "C の書き方"

    ```c
    /* printf 用ヘッダ */
    #include <stdio.h>

    /* ── ステップ1: データの形を定義 ── */
    /* typedef struct s_xxx { ... } t_xxx; */
    /* は C でよく使う命名規約 */
    typedef struct s_contact
    {
        /* 名前 (50文字固定の配列) */
        char first_name[50];
        /* 苗字 (50文字固定の配列) */
        char last_name[50];
    } t_contact;

    /* ── ステップ2: 関数を別途定義 ── */
    /* 構造体とは別の場所に書く必要がある */
    /* (struct の中には関数を書けない) */
    void print_contact(t_contact *c)
    {
        /* c->first_name でメンバにアクセス */
        /* (ポインタ経由なので -> を使う) */
        printf("%s %s\n",
            c->first_name,
            c->last_name);
    }

    /* ── 問題点: アクセス制御がない ── */
    /* 誰でも c->first_name を */
    /* 直接書き換えられてしまう */
    ```

=== "C++ の書き方"

    ```cpp
    // cout 用ヘッダ
    #include <iostream>
    // std::string 用ヘッダ
    #include <string>

    // ── データと関数を1つの箱にまとめる ──
    // class: データ+関数+アクセス制御を1つに
    class Contact
    {
    // ── private: 外から触れない ──
    private:
        // std::string = 可変長の文字列
        // (char[50] の進化版、長さ自動調整)
        std::string _firstName;
        std::string _lastName;

    // ── public: 外から使える ──
    public:
        // クラスの中に関数を書ける!
        // const = この関数はメンバを変更しない約束
        void print() const
        {
            // _firstName に直接アクセス可能
            // (同じクラス内なので private でもOK)
            std::cout << _firstName
                      << " "
                      << _lastName
                      << std::endl;
        }
    };

    // ── 利点: アクセス制御で安全性アップ ──
    // 外から c._firstName はコンパイルエラー
    ```

### コンストラクタって何？

**オブジェクトが作られたとき、
自動で呼ばれる特別な関数**です。

C の「メモリ確保 + 初期化関数を自分で呼ぶ」という
**2ステップ**を、C++ では**1ステップ**にまとめます。

**C の場合（2ステップ必要）**

1. `malloc` や変数宣言でメモリを用意
2. `init_xxx()` のような初期化関数を**自分で呼ぶ**

**C++ の場合（1ステップ）**

- 変数宣言 or `new` した瞬間にコンストラクタが**自動で呼ばれる**
- 呼び忘れが物理的に不可能

=== "C の書き方（2ステップ）"

    ```c
    /* ── ステップ1: 構造体の変数を宣言 ── */
    /* この時点では中身は未初期化（ゴミ値） */
    t_phonebook book;

    /* ── ステップ2: 初期化関数を自分で呼ぶ ── */
    /* init_phonebook は「自分で作る関数」 */
    /* 中身は例えば: */
    /*   void init_phonebook(t_phonebook *b) { */
    /*       b->count = 0;                    */
    /*       b->oldest = 0;                   */
    /*       memset(b->contacts, 0,           */
    /*              sizeof(b->contacts));     */
    /*   }                                    */
    /* この呼び出しを忘れるとゴミ値のまま */
    init_phonebook(&book);
    ```

=== "C++ の書き方（1ステップ）"

    ```cpp
    class PhoneBook
    {
    public:
        // ── コンストラクタ ──
        // クラス名と同じ名前、戻り値なし
        // new / 変数宣言時に自動で呼ばれる
        PhoneBook()
        {
            // メンバを初期化
            _count = 0;
            _oldest = 0;
            // _contacts[8] は Contact() が
            // 8回自動で呼ばれる (入れ子)
        }
    };

    int main()
    {
        // この1行で:
        //   ①スタック上にメモリ確保
        //   ②コンストラクタが自動で呼ばれる
        //   ③_count, _oldest が 0 に初期化
        PhoneBook book;
        return 0;
    }
    ```

| | C（2ステップ） | C++（1ステップ） |
|---|-------------|-----------------|
| 宣言 | `t_phonebook book;` | `PhoneBook book;` |
| 初期化 | `init_phonebook(&book);` 手動 | **自動で実行** |
| 呼び忘れ | あり得る（バグの元） | 物理的に不可能 |

### デストラクタって何？

**オブジェクトが消えるとき、
自動で呼ばれる特別な関数**です。

C の「片付け関数を呼ぶ + `free`」という
**2ステップ**を、C++ では**1ステップ**にまとめます。

**C の場合（2ステップ必要）**

1. `cleanup_xxx()` のような後片付け関数を自分で呼ぶ
2. `free()` でメモリ解放（ヒープの場合）

**C++ の場合（1ステップ）**

- スコープを抜けた or `delete` された瞬間にデストラクタが**自動で呼ばれる**
- 呼び忘れが物理的に不可能

=== "C の書き方（2ステップ）"

    ```c
    /* ── ステップ1: 後片付け関数を呼ぶ ── */
    /* cleanup_phonebook は「自分で作る関数」 */
    /* 中身は例えば:                        */
    /*   void cleanup_phonebook(            */
    /*           t_phonebook *b) {          */
    /*       if (b->log_file)               */
    /*           fclose(b->log_file);       */
    /*       free(b->contacts_dynamic);     */
    /*   }                                  */
    cleanup_phonebook(&book);

    /* ── ステップ2: メモリを解放 ── */
    /* スタック変数なら不要、ヒープなら必要 */
    /* free(book_ptr); */
    ```

=== "C++ の書き方（1ステップ）"

    ```cpp
    class PhoneBook
    {
    public:
        // ── デストラクタ ──
        // ~ (チルダ) + クラス名
        // スコープ終了 / delete 時に
        // 自動で呼ばれる
        ~PhoneBook()
        {
            // 終了処理を書く場所
            // (今回は空でOK)
            // ファイルを閉じたり free したり
        }
    };

    int main()
    {
        PhoneBook book;
        // ↓ ここで } を抜けるとき、
        //   ~PhoneBook() が自動で呼ばれる
        //   呼び忘れる心配なし!
        return 0;
    }
    ```

| | C（2ステップ） | C++（1ステップ） |
|---|-------------|-----------------|
| 後片付け | `cleanup_xxx(&b);` 手動 | **デストラクタで自動** |
| メモリ解放 | `free(b)` | **スコープ終了で自動** |
| 呼び忘れ | あり得る（リークの元） | 物理的に不可能 |

### `private` と `public` って何？

**`private` は「外から見えない」**、
**`public` は「外から使える」** という意味です。

`private` は鍵のかかった日記帳のようなもの。
中身を見たいなら、決められた方法（getter）を
使わないといけません。

```cpp
class Contact
{
private:
    // 外からは直接触れない
    std::string _firstName;

public:
    // 外から使える窓口
    std::string getFirstName() const
    {
        return _firstName;
    }
};

int main()
{
    Contact c;
    // c._firstName;       // NG! private!
    // c.getFirstName();   // OK! public!
    return 0;
}
```

!!! tip "なぜ private にするの？"
    外から勝手にデータを書き換えられないように
    するためです。

    例えば電話番号を空っぽにされたら困りますよね。
    setter（書き込み窓口）の中で
    「空っぽはダメ！」とチェックできます。

    これを**カプセル化**と呼びます。
    ディフェンスで確実に聞かれるポイントです。

### getter / setter って何？

**getter は「private データを読む窓口」**、
**setter は「private データを書く窓口」** です。

```cpp
class Contact
{
private:
    std::string _firstName;  // 隠れている

public:
    // getter: 読み取り窓口
    std::string getFirstName() const
    {
        return _firstName;
    }

    // setter: 書き込み窓口
    void setFirstName(
        const std::string &value)
    {
        _firstName = value;
    }
};
```

getter は「覗き窓」、setter は「投入口」
というイメージです。

### `const` って何？

**「この関数はデータを変えません」という約束**です。

2つの使い方があります。

```cpp
// (1) const メンバ関数
// 「この関数はメンバ変数を変更しない」
std::string getFirstName() const
{                        // ^^^^^ ここ
    return _firstName;
    // _firstName = "Bob"; // NG! const違反
}

// (2) const 参照引数
// 「この引数は読むだけ、変更しない」
void setFirstName(
    const std::string &value)
{// ^^^^^               ^ 参照（コピーしない）
    _firstName = value;
}
```

!!! info "なぜ `const std::string &` と書くの？"
    - `const` = 変更しないよという約束
    - `&` = コピーせず元のデータを直接見る（速い）

    文字列をまるごとコピーすると遅いので、
    「見るだけだからコピーしないで」と伝えています。

### `std::getline` って何？

**1行まるごと読み込む関数**です。

C の `fgets` は「バッファサイズ指定 + 改行も残る」ので、
**改行除去**と**バッファサイズ管理**を自分でやる必要があります。
`std::getline` は**1発で**全部やってくれます。

**C の場合（3ステップ必要）**

1. `char buf[256]` 等で**固定バッファ**を用意（長いと切れる）
2. `fgets` で読み込み（末尾に改行 `\n` が残る）
3. 末尾の改行を自分で消す

**C++ の場合（1ステップ）**

- `std::getline(std::cin, input)` の1行で完了
- バッファサイズも改行除去も**自動**

=== "C の書き方（3ステップ）"

    ```c
    /* strlen を使うためのヘッダ */
    #include <string.h>
    /* fgets を使うためのヘッダ */
    #include <stdio.h>

    /* ── ステップ1: 固定バッファを用意 ── */
    /* 256 文字を超える入力は切られる */
    char buf[256];

    /* ── ステップ2: fgets で読む ── */
    /* fgets: 標準入力から1行読む C 関数 */
    /* 引数: 読み込み先、最大サイズ、入力元 */
    /* 改行文字 '\n' も buf に残る */
    fgets(buf, 256, stdin);

    /* ── ステップ3: 改行を手動で消す ── */
    /* strlen: 文字列の長さを返す C 関数 */
    /* buf[長さ-1] が '\n' なのでそこを '\0' に */
    buf[strlen(buf) - 1] = '\0';
    ```

=== "C++ の書き方（1ステップ）"

    ```cpp
    // std::string の空文字列で初期化
    std::string input;

    // ── この1行で全部やってくれる ──
    //   ①標準入力から1行読む (fgets 相当)
    //   ②改行を自動で除去
    //   ③入力長に応じて std::string が自動拡張
    //     (バッファオーバーフロー不可能)
    std::getline(std::cin, input);
    ```

```cpp
// cin >> だとスペースで切れる
std::cin >> input;
// 入力: "Alice Liddell"
// input: "Alice" だけ！

// getline ならスペースも読める
std::getline(std::cin, input);
// 入力: "Alice Liddell"
// input: "Alice Liddell" 全部！
```

!!! danger "`>>` と `getline` を混ぜないこと"
    `>>` は改行文字を残すので、
    次の `getline` が空行を読んでしまいます。
    対話プログラムでは `getline` だけ使いましょう。

---

## 3. 課題仕様

| 項目 | 内容 |
|------|------|
| プログラム名 | `phonebook` |
| クラス | `Contact` と `PhoneBook` の2つ |
| コマンド | `ADD`, `SEARCH`, `EXIT` |
| 最大件数 | 8件（固定配列、動的確保禁止） |
| 9件目以降 | 最古の連絡先を上書き |
| Makefile | `all`, `clean`, `fclean`, `re` |

---

## 4. 実行例

```console
$ make
$ ./phonebook
Enter command (ADD, SEARCH, EXIT): ADD
First name: Alice
Last name: Liddell
Nickname: alice
Phone number: 0901234567
Darkest secret: rabbit hole
Contact added.
Enter command (ADD, SEARCH, EXIT): ADD
First name: Bob
Last name: TheVeryLongLastName
Nickname: bob
Phone number: 0907654321
Darkest secret: -
Contact added.
Enter command (ADD, SEARCH, EXIT): SEARCH
     index|first name| last name|  nickname
         0|     Alice|   Liddell|     alice
         1|       Bob|TheVeryLo.|       bob
Enter index: 0
First name: Alice
Last name: Liddell
Nickname: alice
Phone number: 0901234567
Darkest secret: rabbit hole
Enter command (ADD, SEARCH, EXIT): EXIT
$
```

`TheVeryLongLastName` は19文字なので、
表では `TheVeryLo.`（9文字 + `.`）に切り詰められます。
詳細表示では全文字が表示されます。

---

## 5. コード解説

### プログラムの流れ

```
スタート
  |
  v
PhoneBook オブジェクトを作る
（コンストラクタで _count=0, _oldest=0 に初期化）
  |
  v
コマンド入力を待つ ←──────────────────┐
  |                                    |
  v                                    |
何のコマンド？                          |
  |                                    |
  |── "ADD" ──→ 5項目入力 → 保存 ──→──┘
  |                                    |
  |── "SEARCH" → 表で一覧 → 詳細 ──→──┘
  |                                    |
  |── "EXIT" ──→ 終了                  |
  |                                    |
  └── その他 ──→ 無視して戻る ──→──────┘
```

### ファイル構成

```
ex01/
 +-- Contact.hpp    # Contact クラス宣言
 +-- Contact.cpp    # Contact クラス実装
 +-- PhoneBook.hpp  # PhoneBook クラス宣言
 +-- PhoneBook.cpp  # PhoneBook クラス実装
 +-- main.cpp       # 対話ループ
 +-- Makefile
```

### 5.1 `Contact.hpp` -- 連絡先1件のクラス

```cpp title="Contact.hpp" linenums="1"
// ── インクルードガード ──
// 同じヘッダを2回読み込まないようにする
#ifndef CONTACT_HPP
#define CONTACT_HPP

#include <string>

class Contact
{
// ── private: 外から直接触れないデータ ──
private:
    // 先頭の _ は「private だよ」の印
    // 42 の慣習（ルールではない）
    std::string _firstName;
    std::string _lastName;
    std::string _nickname;
    std::string _phoneNumber;
    std::string _darkestSecret;

// ── public: 外から使える関数 ──
public:
    // コンストラクタ（作るとき自動で呼ばれる）
    Contact();
    // デストラクタ（消えるとき自動で呼ばれる）
    ~Contact();

    // getter: 読み取り窓口
    // 末尾の const = 「変更しません」の約束
    std::string getFirstName() const;
    std::string getLastName() const;
    std::string getNickname() const;
    std::string getPhoneNumber() const;
    std::string getDarkestSecret() const;

    // setter: 書き込み窓口
    // const std::string & = コピーしない参照渡し
    void setFirstName(
        const std::string &value);
    void setLastName(
        const std::string &value);
    void setNickname(
        const std::string &value);
    void setPhoneNumber(
        const std::string &value);
    void setDarkestSecret(
        const std::string &value);

    // 空かどうか判定する関数
    bool isEmpty() const;
};

#endif
```

### 5.2 `Contact.cpp` -- Contact の実装

```cpp title="Contact.cpp" linenums="1"
#include "Contact.hpp"

// コンストラクタ（中身は空でOK）
// std::string は自動で "" に初期化される
Contact::Contact() {}

// デストラクタ（中身は空でOK）
Contact::~Contact() {}

// ── getter（5つとも同じパターン）──
std::string Contact::getFirstName() const
{
    return _firstName;
}

std::string Contact::getLastName() const
{
    return _lastName;
}

std::string Contact::getNickname() const
{
    return _nickname;
}

std::string Contact::getPhoneNumber() const
{
    return _phoneNumber;
}

std::string Contact::getDarkestSecret() const
{
    return _darkestSecret;
}

// ── setter（5つとも同じパターン）──
void Contact::setFirstName(
    const std::string &value)
{
    _firstName = value;
}

void Contact::setLastName(
    const std::string &value)
{
    _lastName = value;
}

void Contact::setNickname(
    const std::string &value)
{
    _nickname = value;
}

void Contact::setPhoneNumber(
    const std::string &value)
{
    _phoneNumber = value;
}

void Contact::setDarkestSecret(
    const std::string &value)
{
    _darkestSecret = value;
}

// 名前が空なら「空の連絡先」と判定
bool Contact::isEmpty() const
{
    return _firstName.empty();
}
```

!!! note "`Contact::getFirstName()` の `Contact::` って何？"
    「Contact クラスに属する getFirstName 関数」
    という意味です。

    `.hpp` で宣言した関数を `.cpp` で定義するとき、
    **どのクラスの関数か** を `クラス名::` で示します。

### 5.3 `PhoneBook.hpp` -- 電話帳クラス

```cpp title="PhoneBook.hpp" linenums="1"
#ifndef PHONEBOOK_HPP
#define PHONEBOOK_HPP

#include "Contact.hpp"

class PhoneBook
{
private:
    // Contact が8個入る固定配列
    // 各 Contact のコンストラクタが
    // 自動で呼ばれる
    Contact _contacts[8];
    // 今何件入っているか
    int     _count;
    // リングバッファの上書き位置
    int     _oldest;

    // private ヘルパー関数
    // （外から呼ぶ必要がないので隠す）
    std::string _truncate(
        const std::string &str) const;
    void        _displayRow(
        int index,
        const Contact &contact) const;

public:
    // コンストラクタ / デストラクタ
    PhoneBook();
    ~PhoneBook();

    // 外から使う機能
    void addContact(
        const Contact &contact);
    void searchContacts() const;
    void displayContact(int index) const;
    int  getCount() const;
};

#endif
```

### 5.4 `PhoneBook.cpp` -- 実装のポイント

#### (a) メンバ初期化子リスト

コンストラクタで `:` の後ろに書く初期化方法です。

```cpp
// : の後ろで初期化する
PhoneBook::PhoneBook()
    : _count(0), _oldest(0)
{
    // 本体に入る前に初期化が終わっている
    // _contacts[8] は Contact() が
    // 8回自動で呼ばれるので書かなくてOK
}
```

!!! info "なぜ本体で `_count = 0` と書かないの？"
    どちらでも動きますが、
    初期化子リストの方が**効率が良い**です。

    本体の `=` は「一度作ってから上書き」ですが、
    初期化子リストは「最初からその値で作る」です。

    `const` メンバや参照メンバは
    初期化子リストでしか設定できません。

#### (b) 切り詰め関数

10文字を超える文字列を「9文字 + `.`」にする関数です。

```cpp
std::string PhoneBook::_truncate(
    const std::string &str) const
{
    // 10文字「超」なら切り詰め
    // ちょうど10文字はそのまま
    if (str.length() > 10)
        return str.substr(0, 9) + ".";
    return str;
}
```

- `.substr(0, 9)` -- 先頭9文字を取り出す
- `+ "."` -- 文字列の連結
    （C の `strcat` と違い、あふれる心配なし）

!!! warning "`> 10` であって `>= 10` ではない"
    subject は "longer than 10" と書いてあるので、
    ちょうど10文字はそのまま表示します。

#### (c) `<iomanip>` による表の整形

=== "C の書き方"

    ```c
    /* snprintf 用のヘッダ */
    #include <stdio.h>

    /* ── バッファを用意 ── */
    /* 256文字固定、溢れたら切り捨て */
    char buf[256];

    /* ── snprintf で整形 ── */
    /* snprintf: 文字列を整形してバッファに入れる */
    /* 引数: 出力先、最大サイズ、書式、値 */
    /* %10s = 幅10、右寄せで文字列を入れる */
    snprintf(buf, 256,
        "%10s|%10s|", name1, name2);

    /* ── 画面に出力 ── */
    /* printf: C の標準出力関数 */
    printf("%s\n", buf);
    ```

=== "C++ の書き方"

    ```cpp
    // setw / right を使うためのヘッダ
    #include <iomanip>

    // setw(10) = 次の出力を幅10で表示
    //           (printf の %10s と同じ効果)
    // right    = 右寄せ指定
    std::cout
        << std::setw(10)  // 幅10を指定
        << std::right     // 右寄せ
        << name1          // 1つ目の値
        << "|"            // 区切り文字
        << std::setw(10)  // 再び幅10
        << std::right     // 再び右寄せ
        << name2          // 2つ目の値
        << "|"            // 区切り文字
        << std::endl;     // 改行+フラッシュ
    ```

| マニピュレータ | 効果 |
|---|---|
| `std::setw(N)` | 次の出力を幅Nで表示 |
| `std::right` | 右寄せ |
| `std::left` | 左寄せ |

!!! warning "`setw` は1回しか効かない"
    `setw(10)` は **次の `<<` 1回だけ** に適用されます。
    列ごとに毎回書き直す必要があります。

    ```cpp
    // NG: a だけ幅10、b と c は幅なし
    std::cout << std::setw(10)
              << a << b << c;

    // OK: 全部に幅10
    std::cout << std::setw(10) << a
              << std::setw(10) << b
              << std::setw(10) << c;
    ```

### 5.5 `main.cpp` -- 対話ループ

```cpp title="main.cpp" linenums="1"
#include "PhoneBook.hpp"
#include <iostream>

// 空入力を許さず再入力させるヘルパー
// static = このファイル内でだけ使える
static std::string readField(
    const std::string &prompt)
{
    std::string input;

    while (true)
    {
        std::cout << prompt;
        // getline で1行読む
        // EOF(Ctrl+D) なら空文字を返す
        if (!std::getline(std::cin, input))
            return "";
        // 空でなければ OK
        if (!input.empty())
            return input;
        std::cout
            << "Field cannot be empty."
            << std::endl;
    }
}

int main(void)
{
    // PhoneBook オブジェクトをスタック上に生成
    // → コンストラクタが自動で走る
    PhoneBook   book;
    std::string command;

    while (true)
    {
        std::cout
            << "Enter command "
            << "(ADD, SEARCH, EXIT): ";

        // 1行読む（EOF なら抜ける）
        if (!std::getline(
                std::cin, command))
            break;

        // std::string は == で比較できる
        // strcmp は不要！
        if (command == "ADD")
            doAdd(book);
        else if (command == "SEARCH")
            doSearch(book);
        else if (command == "EXIT")
            break;
        // それ以外は無視してループに戻る
    }
    // } を抜けると book のデストラクタが
    // 自動で呼ばれる
    return 0;
}
```

!!! info "なぜ動的メモリ確保を使わないの？"
    subject で「最大8件」と決まっているので、
    `Contact _contacts[8]` の固定長配列で十分です。

    `new` / `delete` を使う必要がなく、
    メモリリークの心配もありません。

    9件目以降はリングバッファ方式で対応します。

    ```cpp
    // リングバッファ: 周回するインデックス
    // 0 → 1 → 2 ... → 7 → 0 に戻る
    _oldest = (_oldest + 1) % 8;
    ```

---

## 6. テストチェックリスト

### 評価シートの確認項目

!!! note "評価シート原文"
    > "Beginners tend to make everything public.
    > That's what you need to check here."

    「なぜ private にしたのか」は**確実に聞かれます**。
    カプセル化を説明できるようにしておきましょう。

!!! note "評価シート原文"
    > "A slight deviation from the expected format
    > is not important. This part is about
    > using iomanips in C++."

    フォーマットの多少のずれは減点されにくいです。
    **iomanip を使っていることが重要**です。

### 基本動作

- [ ] `make` が警告なく通る
- [ ] `ADD` → 5フィールド入力 → 登録される
- [ ] `SEARCH` で登録済みの連絡先が表で表示される
- [ ] 各列が幅10、右寄せ、`|` 区切り
- [ ] `EXIT` で終了
- [ ] 未知のコマンド (`foo`) は無視される

### 空入力のハンドリング

- [ ] `First name:` で Enter → 再入力を要求
- [ ] すべてのフィールドで同様
- [ ] 空の状態で `SEARCH` → エラーメッセージ
- [ ] `Ctrl+D`（EOF）で安全に終了、segfault しない

### 表示フォーマット

- [ ] 10文字ぴったり → そのまま表示
- [ ] 10文字未満 → 右寄せ（左に空白）
- [ ] 11文字以上 → 9文字 + `.` に切り詰め
- [ ] `SEARCH` 後に `Enter index:` プロンプト
- [ ] 範囲外の数字 → `Invalid index.`

### 8件上限とリングバッファ

- [ ] 8件登録 → `SEARCH` で8件表示
- [ ] 9件目追加 → 最古（index 0）が上書きされる
- [ ] 10件目追加 → index 1 が上書きされる

### 詳細表示

- [ ] 10文字超の名前も詳細では全文字表示
- [ ] 全5フィールドが正しく表示される

### 規約

- [ ] クラスは `Contact` と `PhoneBook` の2つ
- [ ] 動的メモリ確保 (`new`, `malloc`) 不使用
- [ ] `printf` / `scanf` 等 C 関数を使っていない
- [ ] `using namespace std;` なし
- [ ] getter に `const` が付いている
- [ ] メンバ変数がすべて `private`
- [ ] 各 `.hpp` にインクルードガード
- [ ] ヘッダに関数の実装を書いていない

---

## 7. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| なぜメンバ変数を private にした？ | **カプセル化**のため。外部から直接データを書き換えられると、バリデーション（空文字チェック等）をバイパスされる。setter を経由させることで入力制御ができる |
| `private` と `public` の違いは？ | `private` はクラスの外からアクセス不可。`public` は外から使える。データは `private` にして getter/setter を `public` にするのが基本 |
| getter に `const` を付ける理由は？ | 「この関数はメンバ変数を変更しない」という約束。`const` オブジェクトからでも呼べるようにするために必要 |
| `const std::string&` の `&` は何？ | 参照渡し。文字列をコピーせず元のデータを直接見る。`const` で「変更しない」と約束している |
| `std::getline` と `std::cin >>` の違いは？ | `getline` はスペースを含む1行全部を読む。`>>` はスペースで区切られ、改行が残る |
| `setw` の注意点は？ | **次の `<<` 1回だけ** に効く。列ごとに毎回 `setw` を書き直す |
| 動的メモリを使わない理由は？ | 最大8件と決まっているので固定配列で十分。`new`/`delete` 不要でメモリリークの心配もない |
| 9件目以降はどうなる？ | リングバッファ方式で最古のエントリを上書き。`_oldest = (_oldest + 1) % 8` で周回 |
| コンストラクタとは何か？ | オブジェクト生成時に自動で呼ばれる初期化関数。クラス名と同じ名前で、戻り値なし |
| デストラクタとは何か？ | オブジェクト破棄時に自動で呼ばれる関数。`~クラス名` で定義する。今回は空だが、リソース解放に使う |

---

## 8. よくあるミス

!!! warning "`std::cin >>` で読むとハマる"
    ```cpp
    // NG: スペースで切れる、改行が残る
    std::cin >> command;

    // OK: 1行まるごと読む
    std::getline(std::cin, command);
    ```
    対話プログラムでは `>>` と `getline` を
    混ぜないこと。改行が残って謎の動作になります。

!!! warning "`setw` は1回しか効かない"
    ```cpp
    // NG: a だけ幅10
    std::cout << std::setw(10)
              << a << b << c;

    // OK: 全部に幅10
    std::cout << std::setw(10) << a
              << std::setw(10) << b
              << std::setw(10) << c;
    ```

!!! warning "切り詰め判定は10文字「超」"
    subject は "longer than 10" なので、
    **ちょうど10文字はそのまま**表示します。

    ```cpp
    // OK: > 10（11文字以上で切り詰め）
    if (str.length() > 10)

    // NG: >= 10（10文字も切り詰めてしまう）
    if (str.length() >= 10)
    ```

!!! warning "リングバッファのインデックス計算ミス"
    ```cpp
    // OK: 0→1→...→7→0 と周回
    _oldest = (_oldest + 1) % 8;

    // NG: 8を超えると配列外アクセス
    _oldest++;
    ```

!!! warning "getter の戻り値とコピー"
    ```cpp
    // 参照を返す（コピーなしで速い）
    const std::string& getFirstName() const;

    // 値を返す（コピーする）
    std::string getFirstName() const;
    ```
    42 的にはどちらでも OK ですが、
    参照を返す方が効率的です。

---

## 9. 次の exercise へ

次の [ex02 Account](ex02-account.md) では、
**`static` メンバ**を学びます。

`static` とは「全オブジェクトで共有されるデータ」
のこと。銀行口座の「合計残高」「総口座数」のように、
クラス全体で1つだけ存在する値を管理します。
