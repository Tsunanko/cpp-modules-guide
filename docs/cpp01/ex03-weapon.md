# ex03 — Unnecessary violence

---

## このプログラムは何？

**2人のキャラクターが武器で攻撃するプログラム** です。

ポイントは、2人が **同じ武器オブジェクトを共有** していること。
武器の名前を変えると、2人とも新しい武器名で攻撃します。

- **HumanA（ボブ）**: 必ず武器を持つ → **参照** で武器を保持
- **HumanB（ジム）**: 武器なしでも存在できる → **ポインタ** で武器を保持

「いつ参照を使い、いつポインタを使うか」を学ぶ exercise です。

---

## 1. このexerciseで学ぶこと

- **参照メンバ**: 「必ず存在する」関係の表現
- **ポインタメンバ**: 「存在しないかもしれない」関係の表現
- **`const` 参照戻り値** でコピーを避けつつ変更を防ぐ
- **初期化子リスト** が参照メンバに必須な理由

---

## 2. 新しい概念の解説

### 参照メンバって何？

クラスのメンバ変数として **参照** を持つことです。
「このオブジェクトは必ず何かを持っている」ことを保証します。

C でクラスに「必ず何かを持たせたい」時は、
ポインタ + NULLチェックを毎回書く必要がありました。
C++ の参照メンバはそれを型レベルで保証します。

#### C で「必ず武器を持つ」を表現する場合（3 ステップ）

```c
/* ── ステップ1: ポインタメンバとして持つ ── */
/* C には参照がないのでポインタしかない */
typedef struct s_humanA {
    char name[50];
    /* 武器のポインタ */
    t_weapon *weapon;
} t_humanA;

/* ── ステップ2: 使う時に毎回 NULL チェック ── */
/* コンパイラは「必ず持つ」を保証できない */
void attack(t_humanA *h) {
    if (h->weapon == NULL) return;  /* 毎回！ */
    printf("%s\n", h->weapon->type);
}

/* ── ステップ3: 後から NULL にされないよう */
/* 規約で徹底するしかない（型では守れない） */
```

#### C++ で「必ず武器を持つ」を表現する場合（1 行）

```cpp
class HumanA {
private:
    std::string _name;
    // ── 参照メンバ ──
    // この1行で以下が全部保証される:
    //   ①NULL にならない
    //   ②後から別の武器に変えられない
    //   ③使う時のデリファレンス不要
    //   ④コンストラクタで必ず初期化される
    Weapon &_weapon;

public:
    // コンストラクタで参照を束縛
    // : _weapon(w) は初期化子リスト
    // (参照メンバはここでしか初期化できない)
    HumanA(std::string name, Weapon &w)
        : _name(name), _weapon(w) {}
};
```

参照メンバの特徴:

- **NULLにならない** → 武器がない状態がありえない
- **後から変更できない** → 別の武器に付け替えられない
- **初期化子リストで初期化が必須** → コンストラクタ本体では間に合わない
- **NULLチェック不要** → 型が「必ず存在する」を保証している

!!! info "なぜ参照メンバは初期化子リストでしか初期化できないの？"
    これは「効率」ではなく **言語仕様レベルの制約** です。

    ```cpp
    class HumanA {
        Weapon &_weapon;
    public:
        HumanA(Weapon &w) {
            _weapon = w;   // ← コンパイルエラー!
        }
    };
    ```

    本体の `_weapon = w;` は「**参照 `_weapon` が指している先 (= まだ誰でもない)** に `w` を代入」という意味になりますが、参照は宣言時に必ず束縛先を確定する必要があるので、**「まだ誰でもない参照」というものが言語的に存在できません**。

    つまり

    1. メンバ領域がメモリ上に確保されるタイミング = **コンストラクタ初期化リストが評価されるタイミング**
    2. 本体 `{ ... }` に入った時点では、すべてのメンバが**もう初期化済み**でなければならない
    3. でも参照は「初期化されていない状態」を作れない型 → 初期化リスト以外で初期化することが原理的に不可能

    なので

    ```cpp
    HumanA::HumanA(Weapon &w) : _weapon(w) {}
    //                          ^^^^^^^^^^^^
    //                          ここでしか初期化できない
    ```

    が**唯一**の書き方です。同じ理由で `const` メンバも初期化リスト必須です (cpp00 ex01 で触れました)。

### ポインタメンバって何？

クラスのメンバ変数として **ポインタ** を持つことです。
「持っていないかもしれない」状態を表現できます。

```cpp
class HumanB {
private:
    // ポインタメンバ: 武器なしでもOK
    Weapon *_weapon;

public:
    // 最初は武器なし（NULL）
    HumanB(std::string name)
        : _name(name), _weapon(NULL) {}

    // 後から武器を設定できる
    void setWeapon(Weapon &w) {
        _weapon = &w;
    }
};
```

ポインタメンバの特徴:

- **NULLにできる** → 武器なし状態を表現できる
- **後から変更できる** → `setWeapon()` で別の武器に変えられる
- **NULLチェックが必要** → 使う前に「武器ある？」を確認

### 判断フローチャート

```
対象が NULL（不在）になる可能性がある？
  |
  +--- YES → ポインタを使う
  |          (Weapon *_weapon)
  |          後から変更する？
  |          +--- YES → setter も作る
  |          +--- NO  → 初期化後は変更しない
  |
  +--- NO  → 参照を使う
             (Weapon &_weapon)
             初期化子リストで初期化する
```

### `const std::string&` 戻り値って何？

「**コピーせずに値を返す、でも変更はさせない**」書き方です。

```cpp
// パターン1: 値を返す（コピーが発生）
std::string getType() {
    return _type;  // コピーが作られる
}

// パターン2: const参照を返す（コピーなし）
const std::string &getType() const {
    return _type;  // コピーなし、変更も不可
}
```

| | 値返し | const参照返し |
|---|--------|-------------|
| コピー | 発生する | なし |
| 速さ | 遅い | 速い |
| 呼び出し側で変更 | できる(コピーだから) | **できない** |

---

## 3. 課題仕様

| クラス | メンバ | 説明 |
|------|------|------|
| **Weapon** | `_type` (private) | 武器の種類 |
| | `getType()` | const参照を返す |
| | `setType()` | 武器の種類を変更 |
| **HumanA** | `_weapon` (参照) | 常に武器を持つ |
| | `attack()` | 攻撃メッセージ表示 |
| **HumanB** | `_weapon` (ポインタ) | 武器なしでもOK |
| | `setWeapon()` | 後から武器を設定 |
| | `attack()` | 攻撃メッセージ表示 |

---

## 4. 実行例

```console
$ make
$ ./violence
Bob attacks with their crude spiked club
Bob attacks with their some other type of club
Jim attacks with their crude spiked club
Jim attacks with their some other type of club
```

**注目**: 武器の `setType()` を呼ぶと、
**ボブもジムも新しい武器名で攻撃する**。
2人とも同じ Weapon を **コピーではなく参照/ポインタで共有** しているからです。

---

## 5. コード解説

### プログラムの流れ

```
スタート
  |
  v
Weapon を作る ("crude spiked club")
  |
  v
HumanA Bob を Weapon の参照で初期化
  |  → Bob は武器を参照で持つ
  v
Bob が attack → "crude spiked club"
  |
  v
Weapon の setType() で名前を変更
  |  → "some other type of club"
  v
Bob が attack → "some other type of club"
  |  ※ 同じ武器を参照しているので変更が反映
  |
  v
HumanB Jim を武器なしで作る
  |  → _weapon は NULL
  v
Jim に setWeapon() で武器を設定
  |
  v
Jim が attack → 武器名で攻撃
  |
  v
Weapon を変更 → Jim も新しい名前で攻撃
  v
終了
```

### Weapon.hpp

```cpp title="Weapon.hpp" linenums="1"
// ── インクルードガード ──
// 同じヘッダが2回読まれるのを防ぐ
#ifndef WEAPON_HPP
#define WEAPON_HPP

// std::string 用のヘッダ
#include <string>

class Weapon {
private:
    // 武器の種類（外から直接触れない）
    std::string _type;

public:
    // コンストラクタ: 武器名を設定
    Weapon(std::string type);
    // デストラクタ: 破棄時に自動で呼ばれる
    ~Weapon(void);

    // ── ゲッター (値を取り出す関数) ──
    // const 参照を返す:
    //   コピーなし + 呼び出し側で変更不可
    // 末尾の const:
    //   この関数はオブジェクトを変更しない
    //   と約束する印
    const std::string &getType(void) const;

    // ── セッター (値を設定する関数) ──
    // 武器の種類を変更
    void setType(std::string type);
};

#endif
```

### Weapon.cpp

```cpp title="Weapon.cpp" linenums="1"
// 自分のヘッダを読む
#include "Weapon.hpp"

// ── コンストラクタ ──
// : _type(type) は初期化子リスト
// コンストラクタ本体に入る前に
// _type を type で初期化する書き方
Weapon::Weapon(std::string type)
    : _type(type) {
}

// ── デストラクタ ──
// ここでは特に後片付けすることがないので
// 空のままで OK
Weapon::~Weapon(void) {
}

// ── getType (ゲッター) ──
// 戻り値が const std::string & なので:
//   ①コピーが発生しない (速い)
//   ②呼び出し側で変更できない (安全)
// 末尾の const でこの関数は
// メンバを変更しないと宣言
const std::string &Weapon::getType(
    void
) const {
    // _type (std::string) を返す
    // 参照返しなのでコピーされない
    return _type;
}

// ── setType (セッター) ──
// 引数 type で _type を上書きする
void Weapon::setType(std::string type) {
    // std::string は = で代入可能
    // (C なら strcpy が必要だった)
    _type = type;
}
```

### HumanA.hpp（参照メンバ）

```cpp title="HumanA.hpp" linenums="1"
// インクルードガード
#ifndef HUMANA_HPP
#define HUMANA_HPP

// Weapon を使うので読み込む
#include "Weapon.hpp"
// cout 用のヘッダ
#include <iostream>

class HumanA {
private:
    // 名前 (人間の名前)
    std::string _name;
    // ── 参照メンバ ──
    // Weapon の「別名」としてメンバに持つ
    // 「必ず武器を持つ」ことを型で保証
    // NULL にできない + 再代入できない
    Weapon &_weapon;

public:
    // ── コンストラクタ ──
    // 名前と武器の両方を受け取る
    // (武器なしでは作れない設計)
    HumanA(
        std::string name,
        Weapon &weapon
    );
    // デストラクタ
    ~HumanA(void);
    // ── attack ──
    // 末尾の const: このオブジェクトを
    // 変更しないことを約束
    void attack(void) const;
};

#endif
```

### HumanA.cpp

```cpp title="HumanA.cpp" linenums="1"
#include "HumanA.hpp"

// ── コンストラクタ ──
// : _name(name), _weapon(weapon)
// が初期化子リスト
// 参照メンバは本体では初期化できないので
// 必ずこの書き方にする！
HumanA::HumanA(
    std::string name,
    Weapon &weapon
) : _name(name), _weapon(weapon) {
}

// デストラクタ: 後片付け特になし
HumanA::~HumanA(void) {
}

// ── attack メンバ関数 ──
// _weapon は参照メンバなので
// 普通の変数と同じく . でアクセス
// (ポインタの -> とは違う)
void HumanA::attack(void) const {
    // cout で攻撃メッセージを出力
    std::cout << _name
              // 連結: 固定文字列
              << " attacks with their "
              // _weapon.getType() で
              // 武器名を取得
              << _weapon.getType()
              // 改行 + flush
              << std::endl;
}
```

### HumanB.hpp（ポインタメンバ）

```cpp title="HumanB.hpp" linenums="1"
// インクルードガード
#ifndef HUMANB_HPP
#define HUMANB_HPP

// Weapon クラスの定義を読む
#include "Weapon.hpp"
// cout 用のヘッダ
#include <iostream>

class HumanB {
private:
    // 名前
    std::string _name;
    // ── ポインタメンバ ──
    // NULL が許される (=武器なしOK)
    // 後から別の武器に変更もできる
    Weapon *_weapon;

public:
    // 名前だけで作れる (武器なしOK)
    HumanB(std::string name);
    // デストラクタ
    ~HumanB(void);
    // 後から武器を設定するセッター
    void setWeapon(Weapon &weapon);
    // 攻撃メッセージを表示
    // const: オブジェクトを変更しない
    void attack(void) const;
};

#endif
```

### HumanB.cpp

```cpp title="HumanB.cpp" linenums="1"
#include "HumanB.hpp"

// ── コンストラクタ ──
// _weapon(NULL) で最初は武器なし状態に
// (参照と違ってポインタは NULL OK)
HumanB::HumanB(std::string name)
    : _name(name), _weapon(NULL) {
}

// デストラクタ: 特に片付けなし
HumanB::~HumanB(void) {
}

// ── setWeapon ──
// 引数は参照で受け取る (NULL 不可)
// &weapon でアドレスを取得して
// ポインタメンバに格納する
void HumanB::setWeapon(Weapon &weapon) {
    _weapon = &weapon;
}

// ── attack ──
// ポインタなので使う前にチェック
void HumanB::attack(void) const {
    // if (_weapon) で NULL でないか確認
    // (C と同じ: 非NULLで true)
    if (_weapon)
        // _weapon はポインタなので
        // -> でメンバにアクセス
        std::cout << _name
                  << " attacks with their "
                  << _weapon->getType()
                  << std::endl;
    else
        // 武器なしの場合のメッセージ
        std::cout << _name
                  << " has no weapon"
                  << std::endl;
}
```

!!! info "参照は `.` でアクセス、ポインタは `->` でアクセス"
    ```cpp
    // HumanA: 参照メンバ → . を使う
    _weapon.getType()

    // HumanB: ポインタメンバ → -> を使う
    _weapon->getType()
    ```

---

## 6. 評価シートの確認項目

!!! note "評価シート原文"
    > "Turn-in directory: ex03/"
    > "Files to turn in: Makefile, main.cpp,
    > Weapon.{h, hpp}, Weapon.cpp,
    > HumanA.{h, hpp}, HumanA.cpp,
    > HumanB.{h, hpp}, HumanB.cpp"

    参照とポインタの使い分けが評価のポイント。

- [ ] HumanA が **参照** で Weapon を持っている
- [ ] HumanB が **ポインタ** で Weapon を持っている
- [ ] `getType()` が `const std::string&` を返している
- [ ] Weapon を変更すると両者の attack メッセージも変わる

---

## 7. テストチェックリスト

- [ ] `make` が警告なく通る
- [ ] HumanA が参照で Weapon を持っている
- [ ] HumanB がポインタで Weapon を持っている
- [ ] `getType()` が `const std::string&` を返す
- [ ] Weapon の `setType()` 後に attack メッセージが変わる
- [ ] HumanB を武器なしで作ってもクラッシュしない
- [ ] 規約違反なし

---

## 8. ディフェンスで聞かれること

| 質問 | 答え方 |
|------|--------|
| なぜ HumanA は参照で HumanB はポインタ？ | HumanA は必ず武器を持つ設計→参照。HumanB は武器なしOK→ポインタ |
| 参照メンバの初期化方法は？ | 初期化子リスト `: _weapon(weapon)` が必須 |
| 参照メンバを本体で代入するとどうなる？ | コンパイルエラー。参照は再代入できない |
| `const std::string&` 戻り値の利点は？ | コピーコストゼロ + 変更不可で安全 |
| Weapon を変えると attack が変わる理由は？ | コピーではなく参照/ポインタで同じ Weapon を共有しているから |
| NULLチェックはなぜ必要？ | ポインタは NULL の可能性があるから。チェックしないと segfault |

---

## 9. よくあるミス

!!! warning "参照メンバを初期化子リストで初期化しない"
    ```cpp
    // コンパイルエラー！
    HumanA::HumanA(std::string name,
                   Weapon &weapon) {
        _weapon = weapon;
        // 参照は再代入できない
    }
    ```

!!! warning "ポインタのNULLチェック忘れ"
    ```cpp
    void HumanB::attack() {
        // _weapon が NULL だと segfault！
        std::cout << _weapon->getType();
    }
    ```

!!! warning "Weapon のコピーを持ってしまう"
    ```cpp
    // ダメ: コピーだから元の Weapon を
    // 変更しても反映されない
    Weapon _weapon;
    ```

---

## 10. 次の exercise へ

次の [ex04 Sed is for losers](ex04-sed.md) では、
**ファイルの読み書き** を学びます。

C の `fopen` / `fgets` の代わりに、
C++ の `std::ifstream` / `std::ofstream` を使います。
