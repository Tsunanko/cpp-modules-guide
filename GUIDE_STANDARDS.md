# ガイドサイト作成基準

このドキュメントは、42 Tokyo 課題の解説サイトを作るときの**統一ルール**です。
CPP Modules で確立した方法論を、今後の cub3D / Exam / NetPractice などにも横展開できるように整理しました。

---

## 1. サイトの目的と読者像

### 想定読者

- **C 言語は書ける** が、対象の技術（C++、レイキャスティング、TCP/IP 等）は初めて
- **ディフェンス（ピアレビュー）対策** をしたい
- **短時間で要点だけ** 押さえたい

### サイトの目的

1. 課題で学ぶべき**概念**を超初心者向けに解説する
2. **C との違い** を明確にする（C 経験者が最短で理解できるように）
3. **ディフェンスで聞かれること** と **模範回答** を整理する
4. **コードの一行一行** を読み解けるようにする

---

## 2. 技術スタック

| 項目 | 採用 |
|------|------|
| ドキュメント生成 | **MkDocs** |
| テーマ | **Material for MkDocs** |
| 表記言語 | **日本語** (`language: ja`) |
| ホスティング | **GitHub Pages**（`gh-pages` ブランチ） |
| デプロイ | **GitHub Actions**（`push` → 自動ビルド） |

### 必須 Markdown 拡張

```yaml
markdown_extensions:
  - admonition           # !!! info / warning / danger ボックス
  - pymdownx.details     # 折りたたみ
  - pymdownx.superfences # コードブロックの拡張
  - pymdownx.highlight   # コードハイライト + 行番号
  - pymdownx.tabbed      # C vs C++ の並列比較タブ
  - pymdownx.tasklist    # チェックリスト
  - tables, toc, attr_list, footnotes 等
```

---

## 3. ディレクトリ構成

```
guide/
├── mkdocs.yml              # 設定ファイル
├── requirements.txt        # Python 依存
├── .gitignore              # site/ を除外
└── docs/
    ├── index.md            # トップページ
    ├── intro/              # 共通の入門セクション
    │   ├── c-vs-cpp.md     # C との比較
    │   ├── setup.md        # 開発環境
    │   ├── cheatsheet.md   # 対応表
    │   └── defense-flow.md # ディフェンスの流れ
    ├── <moduleXX>/         # 各モジュール
    │   ├── index.md        # モジュール概要
    │   ├── exNN-xxx.md     # 各 exercise
    │   └── eval.md         # 評価対策まとめ
    └── reference/
        ├── glossary.md     # 用語集
        └── submit-check.md # 提出前チェック
```

---

## 4. ページテンプレート

### 4.1 Exercise ページ（12 セクション統一）

```markdown
# exNN — タイトル

---

## このプログラムは何？
2〜3 行で平易に。小学生でも分かる言葉で。

---

## 1. このexerciseで学ぶこと
箇条書きで3〜7 項目。

---

## 2. 新しい概念の解説
### 〇〇って何？
登場する概念を1つずつ、独立したサブセクションで解説。

---

## 3. 課題仕様
表形式でコンパクトに。

---

## 4. 実行例
```console
$ make
$ ./binary
（実機の出力）
```

---

## 5. C と C++ の比較
=== "C の書き方"
    ```c
    /* 1行1行コメント */
    ```
=== "C++ の書き方"
    ```cpp
    // 1行1行コメント
    ```

| C | C++ | 一言で言うと |
|---|-----|------------|

---

## 6. コード解説
### プログラムの流れ
ASCII フローチャート。

### ファイル別解説
1行1行コメント付きのソースコード。

---

## 7. 評価シートの確認項目
!!! note "評価シート原文"
    > 英語原文をそのまま引用

チェックリスト。

---

## 8. テストチェックリスト
基本動作・エッジケース・禁止事項。

---

## 9. ディフェンスで聞かれること
| 質問 | 答え方 |
|---|---|

---

## 10. よくあるミス
!!! warning "..."

---

## 11. 次の exercise へ
リンク。
```

### 4.2 モジュール index.md

```markdown
# moduleXX モジュール概要

---

## このモジュールは何？
何を学ぶかを平易に。

## このモジュールで初めて出てくる道具
新概念を「〇〇って何？」形式で導入。

## 学ぶ概念マップ
ASCII 図で全体像。

## exercise 一覧
| # | 名前 | 難度 | 時間 | 何を学ぶ？ |

## 共通ルール
コンパイルフラグ、禁止事項。

## C と C++ の比較表
このモジュールで特に重要な対応。

## 評価の流れ
ディフェンスの概要 + eval.md へのリンク。
```

### 4.3 eval.md（評価対策）

```markdown
# moduleXX 評価対策

## このモジュールの評価テーマ
1 行で要約。

## 評価前チェック
即不合格を避けるための確認。

## Exercise 別 Q&A
| 質問 | 模範回答 | なぜ聞かれるか |

## 概念の深掘り
難しい質問への詳細解説。

## 即不合格フラグ一覧
```

---

## 5. 執筆基準（最重要）

### 基準1: 超初心者向けに書く

- **専門用語を使う前に必ず説明**する
- 「クラスとは」「コンストラクタとは」といった基本概念から丁寧に
- 小学生でも分かる比喩を使う（例: `private` = 鍵のかかった日記帳）

### 基準2: C との多ステップ比較

新しい C++ 機能を紹介する時は、**「C なら N ステップ、C++ なら 1 ステップ」** と明示する。

```markdown
**C の場合（2ステップ）**
1. `malloc` でメモリ確保
2. `init_xxx()` を自作して呼ぶ

**C++ の場合（1ステップ）**
- `new Zombie("name")` の1行で両方実行
```

### 基準3: コードは1行1行コメント

比較セクションのコードは、**C 側も C++ 側も1行1行に日本語コメント**を付ける。

```c
/* printf を使うためのヘッダ */
#include <stdio.h>
/* toupper を使うためのヘッダ */
#include <ctype.h>

/* argc: 引数の個数 */
/* argv: 引数文字列の配列 */
int main(int argc, char **argv)
{
    /* argc == 1 = プログラム名のみ */
    if (argc == 1) { ... }
}
```

### 基準4: 仮想ヘルパー関数は中身を見せる

`init_zombie(z, name)` のような自作関数を C 例に書く時は、**中身がどういう実装かを必ずコメントで示す**。

```c
/* init_zombie は「自分で作る関数」 */
/* 中身は例えば:                   */
/*   void init_zombie(t_zombie *z, */
/*                    char *name) {*/
/*       strcpy(z->name, name);    */
/*   }                             */
init_zombie(z, "name");
```

「標準関数ではなく自作関数」であることを読者に明示するため。

### 基準5: コード横幅は 55 文字以内

ブラウザで**横スクロールなし**で読めるように、コードの1行は最大 55 文字程度に抑える。

```cpp
// ❌ 長い (横スクロール発生)
std::cout << "* LOUD AND UNBEARABLE FEEDBACK NOISE *" << std::endl;

// ✅ 改行して収める
std::cout
    << "* LOUD AND UNBEARABLE FEEDBACK NOISE *"
    << std::endl;
```

### 基準6: 標準ライブラリ関数にも一言コメント

`malloc`, `free`, `strcpy`, `strcat`, `fopen`, `fgets`, `strftime` など、**登場する標準関数すべて**にコメントを付ける。読者が知っている前提にしない。

### 基準7: 評価シート原文を引用

各 exercise ページの「評価シートの確認項目」には、**英語原文をそのまま `!!! note` で引用**する。

```markdown
!!! note "評価シート原文"
    > "The goal is to develop a to_upper with specific behavior
    > if launched without parameters."
```

チェックリストは原文に基づいて作る（勝手に追加しない）。

### 基準8: ディフェンス Q&A は各ページに

`eval.md` に集約するだけでなく、**各 exercise ページの 9 節にも Q&A 表**を置く。読者が「今見ているページ」で必要な情報を完結させるため。

### 基準9: `=== ` タブで C/C++ 並列比較

Material for MkDocs の Tabbed 拡張を使って、**左右に並べて比較**する。

```markdown
=== "C の書き方"
    ```c
    printf("hello\n");
    ```

=== "C++ の書き方"
    ```cpp
    std::cout << "hello" << std::endl;
    ```
```

### 基準10: 文字化けチェック必須

ファイル編集後は必ず `grep` で **U+FFFD（`�` 置換文字）** をスキャンする。

```bash
grep -rn $'\xef\xbf\xbd' guide/docs guide/mkdocs.yml
```

日本語を扱うとき、編集過程で不正バイトが混入することがある。ビルドは通るがブラウザで黒い菱形のクエスチョンマークになる事故を防ぐ。

---

## 6. 視覚的要素の使い分け

| 要素 | 用途 |
|------|------|
| `!!! info` | 補足情報、「なぜ〇〇するのか」 |
| `!!! tip` | 便利なコツ、ディフェンス対策 |
| `!!! warning` | よくあるミス、注意点 |
| `!!! danger` | 即不合格フラグ、重大な禁止事項 |
| `!!! note` | 評価シート原文引用 |
| `!!! quote` | subject からの引用 |
| タブ (`=== `) | C と C++ の並列比較 |
| 表 (`\|`) | 対応関係、判定基準 |
| ASCII 図 | 概念図、フロー、メモリ構造 |

---

## 7. デプロイ（GitHub Pages）

### GitHub Actions ワークフロー

`.github/workflows/deploy-guide.yml`:

```yaml
name: Deploy Guide to GitHub Pages

on:
  push:
    branches: [main]
    paths: ['guide/**']

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - run: pip install -r guide/requirements.txt
      - run: |
          cd guide
          mkdocs gh-deploy --force --config-file mkdocs.yml
```

### GitHub Pages の有効化

1. `main` に push すると Actions が走る
2. `gh-pages` ブランチが自動生成される
3. リポジトリ設定の **Settings → Pages** で:
   - Source: **Deploy from a branch**
   - Branch: **`gh-pages`** / root
4. 数分後 `https://<user>.github.io/<repo>/` で公開される

---

## 8. 他のマイルストーンへの横展開

### cub3D 用ガイド（予定）

```
guide/docs/cub3d/
├── index.md              # レイキャスティング概要
├── 01-parser.md          # .cub ファイルのパース
├── 02-raycasting.md      # 距離計算アルゴリズム
├── 03-rendering.md       # テクスチャ描画
├── 04-player.md          # 移動・回転
└── eval.md               # 評価対策
```

### Exam 用ガイド（予定）

```
guide/docs/exam/
├── index.md              # 試験概要
├── rush00.md             # Level 0 問題
├── rush01.md             # Level 1 問題
└── strategy.md           # 試験戦略
```

### NetPractice 用ガイド（予定）

```
guide/docs/netpractice/
├── index.md              # TCP/IP 基礎
├── subnet-basics.md      # サブネットマスク
├── routing.md            # ルーティング
├── level-01.md 〜 10.md  # 各レベル解説
└── eval.md               # 評価対策
```

### サイト全体の位置づけ

```
42 Tokyo Milestone 4 Guide（仮）
├── CPP Modules (cpp00-04)  ← 現在の範囲
├── cub3D                    ← 次のフェーズ
├── Exam (Exam05 等)         ← 次のフェーズ
└── NetPractice              ← 次のフェーズ
```

全モジュールで **同じ 12 セクションテンプレート** と **10 個の執筆基準** を守る。

---

## 9. チェックリスト（公開前の最終確認）

- [ ] すべてのページが 12 セクションテンプレートに従っている
- [ ] コード横幅 ≤ 55 文字
- [ ] C/C++ 比較タブが全 exercise にある
- [ ] 1行1行の日本語コメントがコードに付いている
- [ ] 評価シート原文の引用がある
- [ ] 各 exercise にディフェンス Q&A 表がある
- [ ] `grep -rn $'\xef\xbf\xbd'` で文字化けゼロ
- [ ] `mkdocs build --strict` がエラーなく通る
- [ ] トップページから全ページへのリンクが辿れる
- [ ] GitHub Actions のデプロイが成功している

---

## 参考: CPP Modules ガイドの成果物

- **ページ数**: 42 ページ
- **対象**: CPP Modules cpp00〜cpp04（全 22 exercise）
- **構成**: 入門 4ページ + モジュール別（5 モジュール × 約 7 ページ）+ リファレンス 2ページ
- **完成日**: 2026-04-19
