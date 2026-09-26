---
doc: UI/UXレビュー L3プロファイル — Janus
version: rev1
status: draft
updated: 2026-09-26
guideline: guideline/uiux_review_guideline_rev1.md
---

# L3プロファイル：Janus（社内問い合わせAIエージェント）rev1

`# 要確認` の付いた値は仮置きです。satoru の確認後に確定させます。

## 1. サービス概要

- 社内の問い合わせ窓口となるチャットAIエージェント。社内文書（規程・図面関連資料など）をナレッジとして取り込み、根拠付きで回答する。
- 想定業種は製造業・建設業。利用者は全員社内ユーザー。
- 1企業1テナントで物理的に分離。LLM はローカル（自社管理GPU）で運用。
- レビュー対象画面：**チャットUI**（一般利用者）と **Admin UI**（ナレッジ管理者）。

## 2. プロファイル

```yaml
service: Janus
code: JNS
guideline_version: rev1
owner: satoru
reviewed: 2026-09-26

personas:
  - name: 一般社員（製造業の設計・品質・購買／建設業の設計部門など）
    it_literacy: 中          # 要確認
    context: 業務の合間にPCブラウザから質問する。回答の根拠を確認してから業務に使う。AIの誤回答をそのまま信じると業務事故につながる
  - name: ナレッジ管理者（顧客企業の情報システム・総務担当）
    it_literacy: 中〜高      # 要確認
    context: Admin UIから文書をアップロードし、取り込み状態（検索可能か・OCR待ちか）を確認する。版の差し替えも担う
  - name: 導入検討中の見込み客（営業デモ）
    it_literacy: 中
    context: 営業同行デモまたはセルフ体験環境で数分触り、自社への導入効果を判断する。初見で価値が伝わるかが重要

primary_tasks:
  - id: T1
    name: 質問して、根拠付きの回答を得る
    frequency: 日次
    success_criteria: 質問から回答表示まで待ち状態が分かり、根拠（ファイル名・ページ）を1クリックで確認できる
  - id: T2
    name: 回答できない（確度が低い）ときに次の行動が分かる
    frequency: 随時
    success_criteria: 回答ではないことが一目で分かり、参考候補の扱いと問い合わせ先が理解できる
  - id: T3
    name: 文書をアップロードし、検索可能になったことを確認する
    frequency: 週次〜月次
    success_criteria: 取り込み結果（検索可能／OCR待ち／失敗）と理由が一覧で分かる
  - id: T4
    name: 規程などの新版を登録し、旧版と差し替える
    frequency: 月次〜年次
    success_criteria: どれが現行版かが分かり、誤って旧版で回答されない
  - id: T5
    name: 見込み客がデモで価値を理解する
    frequency: 商談ごと
    success_criteria: 初回操作から3分以内に「自社文書で使える」イメージが持てる   # 要確認

glossary:
  - term: 根拠
    avoid: [ソース, citation, 出典チャンク]
  - term: 参考（確度が低い候補）
    avoid: [根拠, 検索結果]
  - term: 担当者に問い合わせる        # 要確認（エスカレーション時の画面上の表記）
    avoid: [エスカレーション, escalation]
  - term: 文書
    avoid: [ドキュメント, ファイル, チャンク]
  - term: 検索できる／OCR待ち        # 要確認（取り込み状態の表記）
    avoid: [searchable, pending, ingested]
  - term: 現行版／旧版
    avoid: [superseded, series]

target_devices:
  - PC ブラウザ 1280px以上（Chrome / Edge 最新）   # 要確認（スマホ対応の要否）

weights:
  - id: AI-01
    adjust: +1
    reason: 規程に基づく回答を業務判断に使うため、対応範囲外の質問に答えたように見えると実害が大きい
  - id: AI-02
    adjust: +1
    reason: 根拠を確認できることがJanusの価値の中核（営業上の訴求点でもある）
  - id: AI-04
    adjust: 0
    reason: 回答まで数秒〜20秒程度かかる前提。経過秒数付きのスピナーは実装済みで、既定値のまま評価する
  - id: ONB-01
    adjust: +1
    reason: 導入直後や営業デモでは文書0件の状態から始まる。空状態で価値が伝わらないと商談に響く
  - id: DAT-01
    adjust: +1
    reason: Admin UIの文書一覧は、取り込み状態で絞り込めないと管理者がOCR待ちを見落とす

excluded_items:
  - id: AUTH-02
    reason: 1企業1テナントの物理分離で、画面上のテナント切替がない
  - id: AI-03
    reason: Janusは回答を表示するだけで、データ更新や送信を自動実行しない（機能追加時に再適用する）
  - id: RSP-01
    reason: 対象はPCブラウザのみ（要確認。スマホ対応が必要なら除外を外す）

additional_items:
  - id: JNS-01
    category: AI
    principle: [P01, P05]
    title: 回答と「参考（確度が低い候補）」が一目で区別できる
    check: 確度が低い場合の表示が、回答とは別の見た目（警告色ラベル・注記「回答の根拠ではありません」・控えめなリスト）になっており、根拠付き回答と誤認されない
    ok: 参考候補は警告色ラベル＋注記付きで、小さく淡い番号なしリストで表示される
    ng: 参考候補が根拠と同じ体裁で並び、回答したように見える
    severity_default: 4
    phase: [DS, IMP]
    method: llm
    refs: [HAX G2, HAX G11, Nielsen H1]
    note: 2026-08 に実際に発生し修正済み（c5989c1）。回帰防止のため毎回確認する

  - id: JNS-02
    category: DAT
    principle: [P01, P09]
    title: 文書の取り込み状態と理由が分かる
    check: Admin UIの文書一覧で、検索できる／OCR待ち／失敗の状態と、検索できない理由が表示され、再取り込みなどの次の行動が示される
    ok: "「OCR待ち：スキャン文書のため文字認識が必要です」と表示される"
    ng: 一覧に表示されているのに検索にヒットせず、理由が分からない
    severity_default: 3
    phase: [DS, IMP]
    method: llm
    refs: [Nielsen H1, Nielsen H9]

  - id: JNS-03
    category: FBK
    principle: [P01]
    title: 重複ファイルをスキップしたことが理由付きで伝わる
    check: 同一内容のファイルを別名でアップロードしたとき、取り込まなかったことと理由（同じ内容の既存文書名）が表示される
    ok: "「就業規則_v2.docx は既存の『就業規則.docx』と同じ内容のため取り込みませんでした」"
    ng: アップロード成功のように見えるが、一覧に増えていない
    severity_default: 2
    phase: [DS, IMP]
    method: human
    refs: [Nielsen H1]

  - id: JNS-04
    category: AI
    principle: [P01, P05]
    title: 根拠が現行版かどうかが分かる
    check: 回答の根拠に版の情報（現行版か、版番号・施行日）が表示され、Admin UIでは文書の現行版／旧版の関係が分かる
    ok: "根拠に「就業規則（第3版・2026-04-01施行・現行版）p.12」と表示される"
    ng: 旧版の規程を根拠に回答しても、利用者が気づけない
    severity_default: 3
    phase: [WF, DS, IMP]
    method: llm
    refs: [HAX G11]
    note: 版管理の段階2以降で本格適用。段階1の範囲ではAdmin UIの差し替え操作のみ評価する

  - id: JNS-05
    category: ONB
    principle: [P10]
    title: 何を質問できるかが初見で分かる
    check: チャット画面の初期状態で、取り込み済みの文書の範囲と質問例が示され、初めての利用者やデモ参加者が最初の質問を迷わない
    ok: "「就業規則・賃金規程について質問できます」＋質問例3件のボタン"
    ng: 空の入力欄だけが表示され、何を聞けばよいか分からない
    severity_default: 2
    phase: [WF, DS]
    method: llm
    refs: [HAX G1, Nielsen H6]
```

## 3. 初回レビューの進め方（案）

1. 実装済みの画面（チャットUI・Admin UI）に対して、IMP フェーズの項目で評価する。
2. 主要タスク T1〜T3 のシナリオに沿って操作し、スクリーンショットを取得する。
3. `method: llm` の項目は、スクリーンショットとこのプロファイルを渡して LLM が一次判定し、人が最終判定する。
4. 指摘は `findings/janus/` 配下に、ガイドライン §6 の書式で記録する。

## 変更履歴

| 版 | 日付 | 内容 |
|---|---|---|
| rev1 | 2026-09-26 | 初版（要確認項目あり） |
