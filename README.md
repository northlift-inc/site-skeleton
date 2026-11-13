# 医院サイトのデータ（AIスイセン）

このリポジトリは、AIスイセンで作る医院ホームページの **データ** を置く場所です。医院の基本情報・診療時間・休診日・診療メニュー・ページの本文・画像を、決まった形のファイルで持ちます。サイトの見た目（テンプレート）やビルドの仕組みはここには入りません。AIスイセンがこのリポジトリの `main` ブランチを読んで、サイトを組み立てて公開します。

## 所有権

- このリポジトリとその中身（文章、事実の情報、画像）は **医院のもの** です。リポジトリは医院の GitHub Organization に置き、管理者も医院です。
- AIスイセンは、医院がインストールした GitHub App として、このリポジトリだけを読み書きします。変更は必ずブランチとプルリクエストを通し、`main` に直接書き込むことはしません。変更の公開は、AIスイセンの画面で医院が承認したものだけです。
- コミットには `Aisuisen-Change-Id:` などの行（トレーラ）が付きます。いつ・どの操作で・誰の承認で変わったかを、Git の履歴だけで追えるようにするためです。

## 解約したとき

- AIスイセンを解約しても、このリポジトリは **そのまま医院の手元に残ります**。AIスイセンの GitHub App をアンインストールするだけで、AIスイセンからの読み書きは止まります。ファイルの削除や移動は行いません。
- 残ったファイルは YAML と Markdown なので、別の制作会社や別の仕組みでもそのまま読めます。画像も元のファイルのまま `assets/images/` にあります。

## ファイルの構成

| パス | 中身 |
| --- | --- |
| `site.yaml` | サイトの設定（使うテンプレートと版、業種、問い合わせの方式など）。`site_id` は AIスイセンが最初のコミットで書き込みます |
| `.aisuisen/skeleton` | このひな形の版（AIスイセンが読むだけ） |
| `facts/organization.yaml` | 医院の基本情報（名称、住所、電話番号、標榜科目など） |
| `facts/hours.yaml` | 診療時間 |
| `facts/closures.yaml` | 休診日・診療時間の変更 |
| `facts/services/<id>.yaml` | 診療メニュー（1 メニュー 1 ファイル） |
| `facts/staff/<id>.yaml` | スタッフ紹介（1 人 1 ファイル） |
| `content/pages/<id>.md` | 固定ページの本文 |
| `content/news/<YYYY-MM-DD>-<id>.md` | お知らせ |
| `assets/images/` | 画像と、その説明のファイル（`<id>.yaml`） |
| `overrides/` | 見た目の部分的な調整（AIスイセンの担当者が使います） |
| `redirects.yaml` | 古い URL から新しい URL への転送 |

この表にないファイルはサイトに使われず、検査で「規約外のファイル」として警告になります。次のものは **置かないでください**。

- `package.json`、ビルドのスクリプト、テンプレートのコード
- `.github/workflows/`（GitHub Actions）。置くと検査でエラーになり、公開できません
- 問い合わせの転送先メールアドレス、患者さんやスタッフの個人の連絡先

## YAML の書き方

`facts/*.yaml`、`site.yaml`、`redirects.yaml` と、Markdown の先頭の `---` で囲んだ部分（フロントマター）は YAML で書きます。AIスイセンは YAML の機能のうち基本的なものだけを受け付けます。

**時刻・日付・電話番号・郵便番号は `"` で囲んで文字列にします。** 囲まないと、書き方によっては数値などに読まれてしまいます（例: `0900` は数値の 900 になります）。

```yaml
# よい例
sessions:
  - open: "09:00"
    close: "13:00"
date_from: "2026-12-29"
phone: "03-0000-0000"
postal_code: "100-0001"

# よくない例（検査で止まります）
sessions:
  - open: 0900        # 数値の 900 と読まれる
date_from: 2026/12/29 # 日付の形式が違う
```

- 真偽値は `true` か `false` だけです（`yes`／`no`／`on`／`off` は文字列として読まれ、検査で止まります）。
- 金額は税込の円を整数で書きます（`amount_jpy_incl_tax: 3300`。`3,300` や `3300円` は不可）。
- 文字列の中に `: `（コロンと空白）や `#` で始まる部分があるときは `"` で囲みます（例 `note: "受付: 午前のみ"`）。
- 次の書き方は **使えません**（読み込みの段階で拒否します）。
  - アンカーとエイリアス（`&名前`、`*名前`）、マージキー（`<<:`）
  - タグ（`!!str`、`!foo` など）
  - 1 つのファイルに `---` で区切った複数のドキュメント（Markdown のフロントマターの `---` は別です）
  - 同じキーの重複
  - 制御文字と、文字の向きを変える制御文字（見た目では分からない文字です。他の文書から貼り付けたときに紛れ込むことがあります）
- インデントは半角空白 2 つです。タブは使えません。

AIスイセンがファイルを書き換えるときは、キーの順序と引用符を決まった形に整えます。そのため、手で書いたファイルを AIスイセンの画面から変更すると、値が同じでも引用符の有無などの差分が出ることがあります。

## ブランチの保護（ルールセット）の設定手順

AIスイセンは `main` を GitHub のルールセットで保護する前提で動きます（承認していない変更が `main` に入らないようにするため）。リポジトリを作ったら、医院の管理者が次のとおり設定してください。画面の名前は 2026 年 11 月時点の GitHub のものです。画面の写真付きの手順は後日用意します。

1. リポジトリの **Settings** → 左の **Rules** → **Rulesets** → **New ruleset** → **New branch ruleset** を開く。
2. **Ruleset Name** に `aisuisen-main` と入力する。
3. **Enforcement status** を **Active** にする。
4. **Bypass list** は **空のまま** にする（誰も、AIスイセンの App も、ルールを迂回できないようにする）。
5. **Target branches** の **Add target** → **Include by pattern** で `main` を追加する（対象は `refs/heads/main` だけ。除外は無し）。
6. **Branch rules** で次の 4 つに印を付け、ほかは付けない。
   1. **Restrict deletions**（`main` の削除を禁止）
   2. **Block force pushes**（強制 push を禁止）
   3. **Require a pull request before merging**（プルリクエストを必須に）。開いた中の設定は次のとおり。
      - **Required approvals**: `0`（承認は AIスイセンの画面で行います。GitHub 側でレビュー承認を必須にしないでください）
      - **Dismiss stale pull request approvals when new commits are pushed**: 印を付けない
      - **Require review from Code Owners**: 印を付けない
      - **Require approval of the most recent reviewable push**: 印を付けない
      - **Require conversation resolution before merging**: 印を付けない
      - **Allowed merge methods**: **Merge** だけ（Squash と Rebase は外す。コミットのトレーラを残すため）
   4. **Require status checks to pass**（チェックの成功を必須に）。開いた中の設定は次のとおり。
      - **Require branches to be up to date before merging**: 印を付けない
      - **Do not require status checks on creation**: 印を付けない
      - **Add checks** で次の 3 つを追加し、それぞれの提供元（source）に AIスイセンの GitHub App を選ぶ: `aisuisen/validate`、`aisuisen/build`、`aisuisen/approval`
7. **Create** を押して保存する。

古い形式のブランチ保護（Settings → Branches の **Branch protection rules**）が `main` に設定されている場合は、削除してルールセット 1 本にしてください。AIスイセンは保護が設定どおりかを定期的に確かめます。

GitHub の無料プランでは、private リポジトリのルールセットは設定できても強制されません。その場合も AIスイセンの側で `main` への想定外の変更を検知しますが、変更そのものは止められません。扱いは AIスイセンの担当者にご相談ください。
