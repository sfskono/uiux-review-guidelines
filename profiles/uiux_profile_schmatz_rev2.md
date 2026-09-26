---
doc: UI/UXレビュー L3プロファイル — schmatz
version: rev2
status: draft
updated: 2026-09-26
guideline: guideline/uiux_review_guideline_rev1.md
source: sfskono/schmatz（README.md / docs/SCREENS.md / docs/TESTING.md）
---

# L3プロファイル：schmatz（研修出席管理）rev2

## 1. サービス概要

- 新入社員研修の出席連絡・申請管理・出席実績の確定・出力を、社員・講師・人事担当・コースマネージャの4つの立場で扱うWebサービス。
- 設計の軸：「連絡はすぐに。状況は明確に。確認は確実に。」
- 現段階は **UI/UX検討用のフロントエンドモック**（50画面＋index）。認証・サーバー側のアクセス制御・自動通知・外部連携は未接続。
- レスポンシブ対応は初期リリースの受入条件（Must）。社員はモバイルファースト、人事・コースマネージャもスマホで業務を完結できること。

## 2. プロファイル

```yaml
service: schmatz
code: SCH
guideline_version: rev1
owner: satoru
reviewed: 2026-09-26

personas:
  - name: 社員（新入社員・受講者）
    it_literacy: 中
    device: スマホ中心（PCも可）
    context: 通勤中や研修直前に片手で出席・遅刻・欠席を連絡する。急いでいて誤操作しやすい。社会人になったばかりで、連絡の作法や期限に不安がある
  - name: 人事担当（受講企業側）
    it_literacy: 中
    device: スマホ・PCの両方
    context: 自社社員の出席状況を確認し、補助金申請の準備資料として出席実績を出力する。出力した数字は外部への申請資料の根拠になる
  - name: コースマネージャ（研修会社側。モック上の表記は「ベンダーPM」）
    it_literacy: 中〜高
    device: スマホ・PCの両方
    context: 複数の受講企業を横断して出席状況を把握し、ユーザーを管理する。会社を取り違えると他社の情報を扱う事故になる
  - name: 講師
    it_literacy: 中
    device: PC
    context: 研修の合間に参加・退出時刻を入力し、出席実績を個別・一括で確定する。確定後の訂正も行う

primary_tasks:
  - id: T1
    name: 社員が当日の出席を連絡する
    frequency: 日次
    success_criteria: スマホでホームから1タップで連絡でき、受付状態が分かる。二重連絡にならない
  - id: T2
    name: 社員が遅刻・欠席・早退を連絡する
    frequency: 随時
    success_criteria: 例外連絡は1操作挟んでから選べ、送信前に対象日・研修・内容を確認できる。到着見込みが未定でも送信できる
  - id: T3
    name: 社員が申請を変更・取消する／確定済み実績の訂正を依頼する
    frequency: 随時
    success_criteria: 変更・取消できる条件と、できない場合の代替（訂正依頼）が分かる
  - id: T4
    name: 講師が出席実績を入力し、個別・一括で確定する
    frequency: 日次
    success_criteria: 確定対象と件数を確認してから確定でき、未確定と確定済みを取り違えない
  - id: T5
    name: 人事担当が出席実績を出力する（補助金申請の準備）
    frequency: 月次〜研修終了時
    success_criteria: 確定済みのみの出力と確認用出力を取り違えず、未確定の除外件数を把握したうえで出力できる
  - id: T6
    name: コースマネージャがユーザーを追加・変更・利用停止する
    frequency: 研修開始前・随時
    success_criteria: 対象会社・対象ユーザーを確認してから登録・停止できる

glossary:
  - term: 出席連絡
    avoid: [出勤, 打刻, チェックイン]
  - term: 連絡・申請
    avoid: [承認申請, 届出書]          # 承認フローはないため「承認」を連想させない
  - term: 出席連絡済み
    avoid: [出席, 出席済み]            # 出席実績と混同させない
  - term: 出席実績
    avoid: [勤怠, 出席記録]
  - term: 未確定／確定済み
    avoid: [未承認／承認済み, 仮／本]
  - term: 訂正依頼
    avoid: [修正申請, 変更]            # 確定済み実績は申請の書き換えではない
  - term: 取消
    avoid: [削除]                      # 履歴を残す操作のため
  - term: コースマネージャ
    avoid: [ベンダーPM, 管理者, 運営]   # モックは「ベンダーPM」表記のため、本番化時に置き換える

target_devices:
  - スマホ 幅320〜430px（主要業務は320pxで完結すること）
  - タブレット 幅768px
  - PC 幅1440px前後
  - 確認対象ブラウザ：iOS Safari / Android Chrome / PC Chrome・Edge（実機・Safariはモックでは未検証）
  - 講師画面はPCを主対象とする（スマホでも崩れないことは RSP-01 で確認）

weights:
  - id: RSP-01
    adjust: +1
    reason: レスポンシブ対応は初期リリースの受入条件（Must）。社員の主要端末はスマホ
  - id: A11Y-06
    adjust: +1
    reason: スマホ片手操作が前提。基準も共通版の24pxより厳しい48pxを採用（SCH-03）
  - id: DAT-03
    adjust: +1
    reason: 講師の一括確定は対象を誤ると出席実績がまとめて誤る
  - id: FRM-04
    adjust: +1
    reason: スマホでは通知や画面ロックで離脱しやすく、入力途中の連絡が失われると連絡漏れになる
  - id: TXT-01
    adjust: +1
    reason: 「出席連絡済み」と「出席実績」など、似た用語の取り違えが業務事故に直結する
  - id: AUTH-02
    adjust: 0
    reason: テナント切替はないが、コースマネージャは会社横断で閲覧・操作する。どの会社を見ているかの表示として適用する

excluded_items:
  - id: AI-01〜AI-05
    reason: AI機能なし
  - id: PRF-01
    reason: 現段階はサーバー処理のないモックのため、本番実装時に適用する

additional_items:
  - id: SCH-01
    category: FBK
    principle: [P01, P02]
    title: 申請・出席連絡済み・出席実績・確定状態を混同させない
    check: 「本人の届出（申請・連絡）」「出席連絡済み」「出席実績」「未確定／確定済み」が、ラベル・配置・見た目で区別され、未連絡を欠席、出席連絡済みを出席確定として表示していない
    ok: 社員ホームで「出席連絡済み（09:12受付）」と「出席実績：未確定」が別の行・別のラベルで表示される
    ng: 出席連絡をしただけで「出席」と表示され、確定済みと誤解される
    severity_default: 4
    phase: [WF, DS, IMP]
    method: llm
    refs: [Nielsen H1, Nielsen H2, README「状態を混同させない」]

  - id: SCH-02
    category: RSP
    principle: [P01]
    title: 幅320pxで主要業務が完結する
    check: 幅320pxで、社員・人事・コースマネージャの主要タスク（T1〜T6。講師の T4 はPC主対象）が横スクロールなし（表を除く）で最後まで操作できる。スマホは縦型配置・短いフロー、PCは一覧性を重視している
    ok: 人事の出力条件〜最終確認までスマホで完了できる
    ng: スマホでは閲覧のみで、出力や確定はPCでないとできない
    severity_default: 3
    phase: [DS, IMP]
    method: human
    refs: [WCAG 2.2 SC 1.4.10, README「レスポンシブ対応」]

  - id: SCH-03
    category: A11Y
    principle: [P05]
    title: 主要なタップ領域が48×48px以上
    check: 主要な操作ボタン・リンク・選択肢のタップ領域が48×48 CSSピクセル以上ある（共通版A11Y-06の24pxより厳しい自社基準）
    ok: 「出席を連絡する」ボタンが幅いっぱい・高さ48px以上
    ng: 一覧行内の「詳細」リンクが文字サイズ分しかタップできない
    severity_default: 2
    phase: [DS, IMP]
    method: auto
    tool: Playwright（要素サイズ計測）
    refs: [WCAG 2.2 SC 2.5.5（AAA・44px）, Material Design 48dp, README「レスポンシブ対応」]

  - id: SCH-04
    category: ERR
    principle: [P05]
    title: スマホでの誤操作・二重連絡を防いでいる
    check: 遅刻・欠席・早退の連絡は1操作挟んでから表示され、メニューや種別の選択だけでは登録されない。通常の出席連絡は受付後に二重操作が抑止される
    ok: ホームには「出席を連絡する」「その他の連絡・申請」だけがあり、欠席連絡は種別選択→入力→確認を経て送信される
    ng: ホームの「欠席」ボタンを誤タップすると即時に欠席連絡が送られる
    severity_default: 3
    phase: [WF, DS, IMP]
    method: human
    refs: [Nielsen H5, README「スマホでの誤操作防止」]

  - id: SCH-05
    category: DAT
    principle: [P01, P05]
    title: 出力で確定済みと確認用を取り違えない
    check: 出席実績の出力で、既定が「確定済みの実績のみ」であり、未確定記録の件数と除外されることを確認してから進む。確認用（未確定を含む）出力は、画面上もファイル上も確定実績と区別される。確定済みが0件なら出力できない
    ok: プレビューで「未確定 3件は出力されません」と表示され、確認用出力のファイルには確認用であることが明記される
    ng: 確認用の出力ファイルが確定版と同じ見た目で、補助金申請の資料に誤って使われる
    severity_default: 4
    phase: [WF, DS, IMP]
    method: human
    refs: [Nielsen H1, Nielsen H5, README「確定済みと確認用を区別する」]

  - id: SCH-06
    category: FBK
    principle: [P01]
    title: 出力完了を実際以上に表示しない
    check: 出力後の表示が「ダウンロードを開始しました」「印刷画面を開きました」など実際に起きたことに留まり、保存完了やPDF生成完了を検知したかのように表示しない
    ok: "「ダウンロードを開始しました。保存先はブラウザの設定によります」"
    ng: "「出力が完了しました」と表示されるが、実際にはダウンロードがブロックされている"
    severity_default: 2
    phase: [DS, IMP]
    method: llm
    refs: [Nielsen H1, README「確定済みと確認用を区別する」]

  - id: SCH-07
    category: ERR
    principle: [P03, P09]
    title: 変更・取消できない理由と代わりの手段が分かる
    check: 確定済み実績や過去日の申請など、変更・取消できない場合に理由が示され、訂正依頼などの代わりの手段へ案内される
    ok: "「この日の出席実績は確定済みのため変更できません。訂正依頼を送る」"
    ng: 変更ボタンが理由なく無効化され、どうすればよいか分からない
    severity_default: 3
    phase: [WF, DS]
    method: llm
    refs: [Nielsen H9, README「社員：連絡・申請、変更・取消」]

  - id: SCH-08
    category: AUTH
    principle: [P01]
    title: 見ている会社と権限の範囲が分かる
    check: 人事担当には自社のみであること、コースマネージャには全社表示か特定の会社表示かが常に分かる。人事の申請詳細が閲覧専用であることが明示される
    ok: 人事の一覧見出しに「株式会社みどりテック（自社）」、コースマネージャの一覧に「表示中：全社／会社で絞り込み」が常に表示される
    ng: コースマネージャが会社で絞り込んだまま別画面に移り、全社の数字だと誤解する
    severity_default: 3
    phase: [WF, DS]
    method: llm
    refs: [Nielsen H1, README「人事担当」「ベンダーPM」（本番呼称はコースマネージャ）]
```

## 3. 初回レビューの進め方（案）

現段階はモックなので、**DS フェーズ（デザイン）を中心に評価**します。本番の挙動に依存する項目（性能・実際の保存・端末間の同期など）は、IMP フェーズで改めて評価します。

1. 役割ごとの主要タスク（T1〜T6）に沿って、幅390px（スマホ）と1440px（PC）で画面を操作し、スクリーンショットを取得する。
2. `method: llm` の項目は、スクリーンショットとこのプロファイルを渡して LLM が一次判定し、人が最終判定する。
3. SCH-03 はリポジトリの Playwright テストに、タップ領域の計測を追加して自動判定する。
4. 指摘は `findings/schmatz/` 配下に、ガイドライン §6 の書式で記録する。

## 変更履歴

| 版 | 日付 | 内容 |
|---|---|---|
| rev1 | 2026-09-26 | 初版（README・画面定義から作成。要確認項目あり） |
| rev2 | 2026-09-26 | 要確認項目を確定（講師の端末＝PC、呼称＝コースマネージャ、ITリテラシー・対象ブラウザを確定） |
