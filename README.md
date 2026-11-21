# takokurage-skills: 自作スキル集(Claude / ChatGPT 共用)

ここにあるスキルは「Agent Skills」形式(フォルダ + `SKILL.md`)。
Claude も ChatGPT(Codex含む)も同じ形式を読めるので、**1か所で管理して両方で使う**。

```
skills/
  <スキル名>/
    SKILL.md        ← 先頭に name / description を書いた手順書(必須)
    references/     ← 詳しい仕様など(任意)
    assets/         ← テンプレートのpptxなど(任意)

.claude/skills/     ← skills/ へのリンク。Claude Code がリポジトリ内で自動検出
.agents/skills/     ← skills/ へのリンク。Codex / ChatGPT Desktop が自動検出
```

## 使い方

### ChatGPT(チャッピー)
1. GitHub の Releases → **skills-latest** から使いたいスキルの zip をダウンロード
   (`main` に入ると自動で作り直される。手元なら `scripts/package-skills.sh` でも作れる)
2. ChatGPT のスキル画面からその zip をアップロード

### Claude.ai / Claude デスクトップ
同じ zip を、設定 → 機能(Capabilities)→ スキル からアップロード。

### Claude Code
このリポジトリを開くだけで `.claude/skills/` 以下の全スキルを自動検出する。
別の作業場所でも使う場合は、マーケットプレイスからまとめてインストールする。

```
/plugin marketplace add qrbathbomb-sketch/takokurage-skills
/plugin install takokurage@takokurage-skills
```
スキルを追加・更新したら `/plugin marketplace update takokurage-skills`。

### Codex(ChatGPTのコーディング版)
このリポジトリを開くだけで `.agents/skills/` 以下の全スキルを自動検出する。
別の作業場所でも使う場合は、`skills/<スキル名>` フォルダを `~/.codex/skills/` にコピーする。

### 明示的な呼び出し
- Claude Code: `/<スキル名>`
- ChatGPT: 入力欄で `@` を入力してスキルを選択
- Codex: 入力欄で `$` を入力してスキルを選択（または `/skills`）

説明文に作業内容が一致すれば、Claude / ChatGPT / Codex が自動的に選ぶこともある。

## スキルを追加するとき

1. `skills/<スキル名>/` に置く(ChatGPT からダウンロードしたスキルもここに入れる)
2. `.claude-plugin/marketplace.json` の `skills` 配列にパスを1行追加
3. 両方で動くように書くコツ:
   - frontmatter は `name` と `description` だけにする(片方にしかない項目は使わない)
   - 片方専用の道具名は並べて書く
     例: 「PowerPoint スキル(Claude: `pptx` / ChatGPT: Presentations)」
   - ファイルは `references/` `assets/` からの相対パスで参照する

## 収録スキル

| スキル | 内容 |
| --- | --- |
| build-two-event-case-study-slides | 1枚に2イベントを並べる実績スライド(pptx)を参照デッキの体裁どおりに作る |
| natural-making-slide | パワポ資料（提案書・報告資料など）を作る・直すときの作成ルール |
| event-time-schedule | イベントの施工・本番・撤去スケジュールをExcelの時間軸表で作り、ズレを自動チェック |

## おすすめ外部スキル

他者・他組織が公開しているスキルは、このリポジトリへコピーせず配布元から導入する。

| スキル | 用途 | 配布元 |
| --- | --- | --- |
| pdf | PDFの読取・生成・レイアウト確認 | [OpenAI公式](https://github.com/openai/skills/tree/main/skills/.curated/pdf) |
| slides | PDF・画像・既存資料から編集可能なPPTXを再構成 | [OpenAI公式（固定リビジョン）](https://github.com/openai/skills/tree/e6afb0d74cc75d220df2faf3dd6c635c2dc6a108/skills/.curated/slides) |

Codexへの導入:

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo openai/skills --path skills/.curated/pdf

python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo openai/skills --ref e6afb0d74cc75d220df2faf3dd6c635c2dc6a108 \
  --path skills/.curated/slides
```

導入後は新しいCodexセッションで `$pdf` または `$slides` を指定できる。PDFからPowerPointへ変換するときは、`$slides` にPDFを渡して「各ページを編集可能なPPTXとして再構成」と依頼する。
