---
doc: 自社SaaS UI/UXレビューガイドライン（共通版）
version: rev2
status: draft
updated: 2026-09-26
primary_reference: SmartHR Design System
secondary_references:
  accessibility: デジタル庁デザインシステム / JIS X 8341-3（WCAG 2.2 AA相当）
  screen_patterns: Atlassian Design System / IBM Carbon
  ai_ux: Microsoft HAX Guidelines / Google PAIR Guidebook
---

# 自社SaaS UI/UXレビューガイドライン（共通版）rev2

このファイルが**正本**です。閲覧用の xlsx は `gen_guideline_xlsx.py` で本ファイルから生成します。xlsx を直接編集しないでください。

---

## 0. 使い方

1. レビュー対象サービスの **L3プロファイル**（§5）を用意する（初回のみ）。
2. 対象フェーズ（§3.3）に該当する **L2項目**（§4）を抽出する。
3. `method` に従って判定する：`auto` → ツール実行、`llm` → LLM一次レビュー後に人が確認、`human` → 人が判定。
4. 指摘には項目IDと深刻度（§3.1）を付けて記録する（§6の書式）。
5. 深刻度から対応優先度が自動で決まる（§3.2）。
6. 修正後は**再レビュー**を行う（§3.5）。修正した指摘の解消確認に加え、修正によって新しい問題（退行）が生じていないかを確認する。
7. 実施結果は、サービスごとに `findings/<service>/` に記録する（レビュー報告 md と指摘記録 yaml）。

---

## 1. 構成（3層）

| 層 | 内容 | 使い方 |
|---|---|---|
| L1 原則 | 判断の拠り所となる10原則（§2） | L2に該当項目がないとき・判断に迷ったときに参照 |
| L2 共通チェックリスト | 全サービス共通の具体項目（§4） | 評価の本体。初版は**コア項目のみ**（Must相当） |
| L3 サービス別プロファイル | ペルソナ・主要タスク・用語・重み付け・除外/追加項目（§5） | サービスごとの差分だけを持つ |

### 改訂ルール

- L2に該当項目がない指摘は、L1の原則IDで記録し、**§4.1 L2追加候補**に登録する。

- L2への項目追加は「実際のレビューで**2サービス以上**で指摘が出た」ことを条件にする（網羅目的での追加はしない）。
- 1サービス固有の指摘は、そのサービスのL3 `additional_items` に入れる。
- 1年間一度も指摘に使われなかったL2項目は、削除または統合を検討する。
- L3プロファイルは**リリースごと**に見直す（担当：サービスオーナー）。

---

## 2. L1 原則

Nielsen 10 Heuristics をベースに、ISO 9241-110（対話の原則）の観点を統合しています。

| ID | 原則 | 要旨 | 主な出典 |
|---|---|---|---|
| P01 | 状態が見える | システムが今何をしているか、操作の結果どうなったかが常に分かる | Nielsen H1 / ISO 自己記述性 |
| P02 | ユーザーの言葉で話す | 業務の用語・順序で表現し、システム内部の都合を出さない | Nielsen H2 / ISO 期待への一致 |
| P03 | 戻れる・やめられる | 取り消し・中断・やり直しが常にできる | Nielsen H3 / ISO 制御可能性 |
| P04 | 一貫している | 同じものは同じ見た目・言葉・振る舞いにする。プラットフォームの慣習に従う | Nielsen H4 / ISO 期待への一致 |
| P05 | 間違いを起こさせない | 誤操作が起きにくい設計を、エラー表示より優先する | Nielsen H5 / ISO 使用エラーへの耐性 |
| P06 | 覚えさせない | 必要な情報・選択肢を画面上に示し、記憶に頼らせない | Nielsen H6 |
| P07 | 慣れた人は速く | 熟練者向けの近道（一括操作・ショートカット・既定値）を用意する | Nielsen H7 / ISO 個別化への適合性 |
| P08 | 必要なものだけ | 主要タスクに関係ない情報・装飾を削る | Nielsen H8 |
| P09 | エラーから回復できる | 何が起き、どうすれば直るかを平易な言葉で示す | Nielsen H9 / ISO 使用エラーへの耐性 |
| P10 | 迷ったら助けがある | 文脈に沿ったヘルプ・問い合わせ導線がある | Nielsen H10 / ISO 学習性 |

---

## 3. 評価基準

### 3.1 深刻度（NN/g 0〜4 に準拠）

| 深刻度 | 定義 | 判断の目安 |
|---|---|---|
| 4 致命的 | 主要タスクが完了できない／データ損失・誤処理・情報漏えいにつながる | 回避策なし |
| 3 重大 | 主要タスクの完了に大きな支障。多くのユーザーが詰まる | 回避策はあるが気づきにくい |
| 2 中 | 手間・迷いが生じるが完了できる | 頻度が高いなら3に上げる |
| 1 軽微 | 見た目・表現の問題。業務への影響はほぼない | — |
| 0 問題なし | ユーザビリティ上の問題ではない（好みの範囲） | 記録のみ |

深刻度は「影響の大きさ × 発生頻度 × 回避の難しさ」で判断します。L2項目の `severity_default` は違反時の標準値で、状況に応じて±1まで調整できます（調整理由を記録すること）。

### 3.2 深刻度 → 対応優先度（自動決定）

| 深刻度 | 対応 |
|---|---|
| 4 | リリース不可。即時修正 |
| 3 | リリース前に修正（例外はサービスオーナー承認＋次リリースで必須） |
| 2 | 次回以降のリリースで計画的に修正 |
| 1 | バックログに登録。まとめて対応 |
| 0 | 対応不要 |

### 3.3 フェーズ

| コード | フェーズ | 主な評価対象 |
|---|---|---|
| WF | ワイヤー | 情報設計・画面遷移・主要タスクの流れ |
| DS | デザイン | 見た目・文言・状態パターン・コントラスト |
| IMP | 実装 | 実際の挙動・キーボード操作・性能・権限 |
| REL | リリース後 | 実利用データ・問い合わせ・簡易ユーザビリティテスト |

### 3.4 判定手段（method）

| 値 | 意味 |
|---|---|
| auto | 自動ツールで判定（`tool` 欄に記載）。CIへの組み込みを推奨 |
| llm | LLMによる一次判定が可能（画面キャプチャ＋仕様）。**最終判定は人** |
| human | 人のみ判定可能（実操作・文脈判断が必要） |

LLM一次判定は、項目ごとに「LLM判定」と「人の最終判定」を記録し、食い違い率を見て `method` を見直します。

### 3.5 再レビュー

修正後は、**初回と同じ方法・同じ条件**（画面幅、自動計測ツール、対象画面）で再レビューを行います。修正担当（Claude Code など）の自己報告やテストの合格だけで完了にしないでください。

| 確認すること | 内容 |
|---|---|
| 解消確認 | 修正対象の指摘がすべて解消されているか |
| 退行確認 | 修正した画面・近い画面で、**修正前にできていたことが失われていないか**（表示が消えた、文言が崩れた、別の操作が効かなくなった、など）。修正前後で同じ画面を比較する |
| 周辺確認 | 共通CSS・共通コンポーネント・用語を変えた場合は、全画面を自動計測し、主要タスクの代表画面を目視する |
| 操作確認 | 絞り込み・選択・入力など、操作して初めて表示が変わる箇所は、実際に操作して確認する |

- 退行は**新しい指摘**として記録し、`description` に「◯◯の修正で生じた退行」と明記します。深刻度は通常どおり判定します。
- 解消を確認した指摘は `status: fixed` とし、`fixed_in`（修正が入ったブランチ・コミット）と `verified`（確認日と方法）を記入します。
- 深刻度3以上の指摘と退行は、再レビューで解消を確認するまでマージしません。深刻度2以下の小さな修正で、退行を検出する自動テストが追加されている場合は、テストの合格をもって確認とすることができます（その旨を `verified` に書く）。

---

## 4. L2 共通チェックリスト（コア）

### 4.1 L2追加候補

L1の原則IDで記録した指摘のうち、他サービスでも出る可能性があるものを登録します。**2サービス以上**で指摘が出た時点で、L2項目に昇格させます（改訂ルール）。

| 候補 | 内容 | 初出 | 出現サービス数 | 昇格時の想定ID |
|---|---|---|:-:|---|
| 最小文字サイズ | 本文16px、補足でも12px以上を下限とする（スマホで9〜11pxの文字を使わない） | schmatz SCH-20260926-05（P01） | 1 | A11Y-07 |

### 4.2 コア項目

カテゴリ：NAV ナビ・情報設計 / FRM フォーム入力 / FBK 状態・フィードバック / ERR エラー防止と回復 / DAT データ表示 / CON 一貫性 / TXT UXライティング / A11Y アクセシビリティ / RSP レスポンシブ / PRF 体感性能 / ONB オンボーディング・空状態 / AUTH 権限・マルチテナント / NTF 通知 / AI AI機能

```yaml
items:
  # ---------- NAV ナビ・情報設計 ----------
  - id: NAV-01
    category: NAV
    principle: [P01, P06]
    title: 現在地が分かる
    check: グローバルナビの選択状態・ページタイトル・パンくず等で、今どの画面にいるかが分かる
    ok: 左ナビの該当メニューがハイライトされ、ページ見出しがメニュー名と一致している
    ng: 詳細画面に遷移するとナビの選択状態が消え、どの機能配下か分からない
    severity_default: 2
    phase: [WF, DS]
    method: llm
    refs: [Nielsen H1, SmartHR 6-2, SmartHR 6-3]

  - id: NAV-02
    category: NAV
    principle: [P02]
    title: 主要タスクへ最短で到達できる
    check: L3で定義した主要タスクの起点に、ログイン後3クリック以内で到達できる
    ok: ダッシュボードに主要タスクの開始ボタンがある
    ng: 利用頻度の高い操作が設定メニューの深い階層にある
    severity_default: 3
    phase: [WF]
    method: human
    refs: [Nielsen H7]

  - id: NAV-03
    category: NAV
    principle: [P10]
    title: ヘルプ・問い合わせ導線が全画面共通の位置にある
    check: ヘルプ・問い合わせへの導線が全画面で同じ位置にある
    ok: ヘッダー右上に常にヘルプアイコンがある
    ng: 一部の画面にしかヘルプリンクがない
    severity_default: 2
    phase: [WF, DS]
    method: llm
    refs: [WCAG 2.2 SC 3.2.6, SmartHR 6-5, Nielsen H10]

  # ---------- FRM フォーム入力 ----------
  - id: FRM-01
    category: FRM
    principle: [P06]
    title: 入力欄に常時表示のラベルがある
    check: すべての入力欄に、入力中も消えない可視ラベルがある（プレースホルダーをラベル代わりにしない）
    ok: 入力欄の上にラベルが常に表示されている
    ng: プレースホルダーだけで項目名を示しており、入力すると何の欄か分からなくなる
    severity_default: 3
    phase: [DS, IMP]
    method: llm
    refs: [WCAG 2.2 SC 3.3.2, SmartHR 7-1]

  - id: FRM-02
    category: FRM
    principle: [P05]
    title: 必須・形式・制約が入力前に分かる
    check: 必須項目、入力形式（例：半角・桁数・日付形式）、文字数上限が入力前に示されている
    ok: "「必須」表示と「例：2026-09-26」の補足がある"
    ng: 送信後に初めて「形式が不正です」と表示される
    severity_default: 2
    phase: [DS]
    method: llm
    refs: [Nielsen H5, WCAG 2.2 SC 3.3.2]

  - id: FRM-03
    category: FRM
    principle: [P06]
    title: 同じ情報を再入力させない
    check: 同一プロセス内で既に入力・保持している情報を再度入力させない
    ok: 前のステップで入力した住所が初期値として入っている
    ng: 確認画面から戻るとすべての入力が消えている
    severity_default: 3
    phase: [WF, IMP]
    method: human
    refs: [WCAG 2.2 SC 3.3.7, SmartHR 7-5]

  - id: FRM-04
    category: FRM
    principle: [P03]
    title: 入力途中のデータが失われない
    check: 画面離脱・タイムアウト・通信エラー時に入力途中のデータが失われない（警告・一時保存・復元のいずれか）
    ok: 未保存のまま離脱しようとすると確認ダイアログが出る
    ng: セッション切れでログイン画面に飛び、長文入力が消える
    severity_default: 3
    phase: [IMP]
    method: human
    refs: [WCAG 2.2 SC 2.2.1, SmartHR 5-3, Nielsen H3]

  # ---------- FBK 状態・フィードバック ----------
  - id: FBK-01
    category: FBK
    principle: [P01]
    title: 処理中であることが分かる
    check: 1秒を超える処理ではローディング表示、10秒を超える処理では進捗またはバックグラウンド処理であることを示す
    ok: 一括処理で「3/120件 処理中」と表示され、画面を離れても継続する旨が示される
    ng: ボタン押下後に何も変化せず、二重送信される
    severity_default: 3
    phase: [DS, IMP]
    method: human
    refs: [Nielsen H1, NN/g Response Times]

  - id: FBK-02
    category: FBK
    principle: [P01]
    title: 操作結果が明示される
    check: 保存・送信・削除などの操作後に、成功／失敗と対象が明示される
    ok: "「請求書 #1024 を保存しました」とトースト表示される"
    ng: 保存後に画面が再読込されるだけで、成功したか分からない
    severity_default: 2
    phase: [DS, IMP]
    method: llm
    refs: [Nielsen H1, WCAG 2.2 SC 4.1.3]

  # ---------- ERR エラー防止と回復 ----------
  - id: ERR-01
    category: ERR
    principle: [P05, P03]
    title: 破壊的操作に確認か取り消しがある
    check: 削除・一括更新・送信など取り消しにくい操作には、対象と影響を示す確認、または取り消し（Undo）がある
    ok: "「12件の取引先を削除します。この操作は取り消せません」と対象件数を示して確認する"
    ng: ゴミ箱アイコン1クリックで即時削除される
    severity_default: 4
    phase: [WF, DS, IMP]
    method: llm
    refs: [Nielsen H3, Nielsen H5, WCAG 2.2 SC 3.3.4]

  - id: ERR-02
    category: ERR
    principle: [P09]
    title: エラーの箇所・原因・直し方が分かる
    check: エラー時に「どこが」「なぜ」「どうすれば直るか」を平易な言葉で示す。フォームでは該当欄の近くに表示する
    ok: "該当欄の下に「メールアドレスに@が含まれていません」と表示される"
    ng: "画面上部に「エラーが発生しました（E-5021）」とだけ表示される"
    severity_default: 3
    phase: [DS, IMP]
    method: llm
    refs: [Nielsen H9, WCAG 2.2 SC 3.3.1, WCAG 2.2 SC 3.3.3, SmartHR 7-3]

  - id: ERR-03
    category: ERR
    principle: [P09]
    title: システムエラー時にも次の行動が示される
    check: 通信エラー・権限エラー・404/500等で、再試行・戻る・問い合わせなどの次の行動が示される
    ok: 再試行ボタンと、問い合わせ時に伝えるべきIDが表示される
    ng: スタックトレースや英語の技術的メッセージがそのまま表示される
    severity_default: 3
    phase: [IMP]
    method: human
    refs: [Nielsen H9]

  # ---------- DAT データ表示 ----------
  - id: DAT-01
    category: DAT
    principle: [P06, P07]
    title: 一覧で探す・絞る・並べ替えができる
    check: 件数が多くなりうる一覧に、検索・フィルタ・並べ替えがあり、適用中の条件と件数が表示される
    ok: "「絞り込み中：ステータス=未処理（24件）」と表示され、解除できる"
    ng: 絞り込みが適用されていることに気づけず、データが消えたと誤解する
    severity_default: 3
    phase: [WF, DS]
    method: llm
    refs: [Atlassian Design System, Carbon Data table]

  - id: DAT-02
    category: DAT
    principle: [P02]
    title: 数値・日付・単位の表記が業務に合っている
    check: 金額の桁区切り・通貨、日付形式、単位、右揃え（数値）など、業務で使う表記に統一されている
    ok: 金額は右揃え・3桁区切り・「円」表記で統一されている
    ng: 同じ画面で「2026/9/26」と「09-26-2026」が混在している
    severity_default: 2
    phase: [DS]
    method: llm
    refs: [Nielsen H2, Nielsen H4]

  - id: DAT-03
    category: DAT
    principle: [P07]
    title: 一括操作の対象が明確
    check: 複数選択・一括操作で、選択件数と対象範囲（表示中のみか／全件か）が明示される
    ok: "「このページの20件を選択中｜全120件を選択」と表示される"
    ng: 全選択がページ内のみか全件か分からないまま一括削除できる
    severity_default: 3
    phase: [DS, IMP]
    method: llm
    refs: [Nielsen H1, Nielsen H5]

  # ---------- CON 一貫性 ----------
  - id: CON-01
    category: CON
    principle: [P04]
    title: デザイントークン・共通コンポーネントを使っている
    check: 色・文字サイズ・余白・角丸・ボタン等が、デザイントークンと共通コンポーネントから作られている（独自の値を直書きしていない）
    ok: ボタンはすべて共通 Button コンポーネントの variant で表現されている
    ng: 画面ごとにプライマリボタンの色・高さが微妙に違う
    severity_default: 1
    phase: [DS, IMP]
    method: auto
    tool: stylelint / ESLint（トークン外の値・直書きの検出）
    refs: [Nielsen H4, SmartHR Design System]

  - id: CON-02
    category: CON
    principle: [P04]
    title: 同じ操作は同じ位置・同じ名前
    check: 保存・キャンセル・戻る等の同じ操作が、全画面で同じ位置・同じラベル・同じ見た目になっている
    ok: ダイアログの主ボタンは常に右、「保存」「キャンセル」の表記で統一されている
    ng: 画面によって「登録」「保存」「確定」が同じ意味で混在している
    severity_default: 2
    phase: [DS]
    method: llm
    refs: [Nielsen H4, WCAG 2.2 SC 3.2.4]

  # ---------- TXT UXライティング ----------
  - id: TXT-01
    category: TXT
    principle: [P02]
    title: ユーザーの業務用語で書かれている
    check: 画面文言がL3の用語集に沿っており、内部用語・英語の技術用語・略語が出ていない
    ok: "「取引先」で統一（L3用語集どおり）"
    ng: "「エンティティ」「レコード」「null」などが画面に出ている"
    severity_default: 2
    phase: [DS]
    method: llm
    refs: [Nielsen H2]

  - id: TXT-02
    category: TXT
    principle: [P02, P04]
    title: ボタン・リンクのラベルから結果が分かる
    check: ボタンは動詞で結果を示し、リンクテキストだけで遷移先が分かる（「こちら」「OK」だけにしない）
    ok: "「請求書を送信」「CSVをダウンロード」"
    ng: "「詳しくはこちら」「OK」だけでは何が起きるか分からない"
    severity_default: 2
    phase: [DS]
    method: llm
    refs: [WCAG 2.2 SC 2.4.4, SmartHR 6-4]

  # ---------- A11Y アクセシビリティ ----------
  - id: A11Y-01
    category: A11Y
    principle: [P01]
    title: コントラスト比を満たしている
    check: テキストは4.5:1以上（大きな文字は3:1以上）、UI部品・フォーカス表示は3:1以上
    ok: 本文・補足テキスト・無効状態以外のボタンが基準を満たす
    ng: 薄いグレーの補足テキストが背景に埋もれている
    severity_default: 3
    phase: [DS, IMP]
    method: auto
    tool: axe-core / Lighthouse
    refs: [WCAG 2.2 SC 1.4.3, WCAG 2.2 SC 1.4.11, SmartHR 4-2]

  - id: A11Y-02
    category: A11Y
    principle: [P01]
    title: 色だけで情報を伝えていない
    check: ステータス・エラー・グラフ系列などを、色に加えてテキスト・アイコン・形でも区別している
    ok: "エラー欄は赤枠＋アイコン＋メッセージで示される"
    ng: ステータスが色付きの丸だけで表されている
    severity_default: 3
    phase: [DS]
    method: llm
    refs: [WCAG 2.2 SC 1.4.1, SmartHR 4-3]

  - id: A11Y-03
    category: A11Y
    principle: [P07]
    title: キーボードだけで操作でき、フォーカスが見える
    check: すべての操作がキーボードで可能で、フォーカス位置が常に見え、順序が見た目と一致し、固定ヘッダー等に隠れない
    ok: Tabで上から順に移動し、フォーカスリングが明確に見える
    ng: カスタムのドロップダウンがキーボードで開けない
    severity_default: 3
    phase: [IMP]
    method: human
    refs: [WCAG 2.2 SC 2.1.1, WCAG 2.2 SC 2.4.3, WCAG 2.2 SC 2.4.7, WCAG 2.2 SC 2.4.11, SmartHR 5-1, SmartHR 5-2]

  - id: A11Y-04
    category: A11Y
    principle: [P01]
    title: 支援技術が名前・役割・状態を取得できる
    check: 入力欄・ボタン・アイコンボタンにアクセシブルネームがあり、見出し・表・リストが正しくマークアップされている
    ok: アイコンのみのボタンに aria-label がある
    ng: div にクリックイベントを付けただけのボタンがある
    severity_default: 3
    phase: [IMP]
    method: auto
    tool: axe-core
    refs: [WCAG 2.2 SC 1.3.1, WCAG 2.2 SC 4.1.2, SmartHR 3-1, SmartHR 3-2, SmartHR 7-2]

  - id: A11Y-05
    category: A11Y
    principle: [P01]
    title: 200%拡大でも情報が欠けない
    check: ブラウザで200%拡大、または文字サイズを2倍にしても、情報や操作が欠けたり重なったりしない
    ok: 拡大時にレイアウトが1カラムに切り替わる
    ng: 拡大するとボタンが画面外に出て押せない
    severity_default: 2
    phase: [IMP]
    method: human
    refs: [WCAG 2.2 SC 1.4.4, WCAG 2.2 SC 1.4.10, SmartHR 4-1]

  - id: A11Y-06
    category: A11Y
    principle: [P05]
    title: クリック対象が小さすぎない
    check: クリック・タップ対象が24×24px以上、または隣接する対象と十分な間隔がある
    ok: 表の行内アイコンボタンが24px以上で並んでいる
    ng: 小さな編集・削除アイコンが密着していて押し間違える
    severity_default: 2
    phase: [DS, IMP]
    method: llm
    refs: [WCAG 2.2 SC 2.5.8]

  # ---------- RSP レスポンシブ ----------
  - id: RSP-01
    category: RSP
    principle: [P01]
    title: 対象画面幅で崩れない
    check: L3で定義した対象デバイス・画面幅で、横スクロール（表を除く）や要素の重なりが発生しない
    ok: 1280px〜1920pxで崩れない（L3の対象がPCのみの場合）
    ng: ノートPC（1366px）でモーダルの下部ボタンが見切れる
    severity_default: 2
    phase: [IMP]
    method: human
    refs: [WCAG 2.2 SC 1.4.10]

  # ---------- PRF 体感性能 ----------
  - id: PRF-01
    category: PRF
    principle: [P01]
    title: 主要画面の表示と応答が遅くない
    check: 主要画面の初期表示（LCP）が2.5秒以内、操作への応答（INP）が200ms以内を目安とする
    ok: 一覧画面のLCPが2秒前後で安定している
    ng: 一覧画面を開くたびに5秒以上白画面になる
    severity_default: 2
    phase: [IMP, REL]
    method: auto
    tool: Lighthouse / Web Vitals計測
    refs: [Core Web Vitals, NN/g Response Times]

  # ---------- ONB オンボーディング・空状態 ----------
  - id: ONB-01
    category: ONB
    principle: [P10, P06]
    title: 空状態で次にやることが分かる
    check: データが0件の画面で、状況の説明と最初の行動（作成・インポート等）が示される
    ok: "「まだ取引先がありません」＋「取引先を追加」「CSVで一括登録」ボタン"
    ng: 空の表とヘッダーだけが表示される
    severity_default: 2
    phase: [WF, DS]
    method: llm
    refs: [Nielsen H10, Atlassian Design System Empty state]

  # ---------- AUTH 権限・マルチテナント ----------
  - id: AUTH-01
    category: AUTH
    principle: [P01, P09]
    title: 権限がない操作の扱いが明確
    check: 権限がない操作は、非表示・無効化（理由付き）のどちらにするかが方針どおりで、実行時に権限エラーになる導線がない
    ok: 無効化ボタンにカーソルを当てると「管理者のみ実行できます」と表示される
    ng: ボタンを押してから「権限がありません」エラーになる
    severity_default: 2
    phase: [WF, IMP]
    method: human
    refs: [Nielsen H5]

  - id: AUTH-02
    category: AUTH
    principle: [P01]
    title: どのテナント・アカウントで操作しているかが分かる
    check: 複数テナント・組織・環境を切り替えられる場合、現在のテナント／環境が常に表示される
    ok: ヘッダーに組織名が常時表示され、検証環境は色帯で区別される
    ng: 別組織に切り替わったことに気づかず、誤ってデータを登録する
    severity_default: 4
    phase: [WF, DS]
    method: llm
    refs: [Nielsen H1]

  # ---------- NTF 通知 ----------
  - id: NTF-01
    category: NTF
    principle: [P08, P03]
    title: 通知が制御でき、必要な情報が残る
    check: 通知の種類・経路（メール/アプリ内等）をユーザーが設定でき、重要な通知は自動で消えず後から確認できる
    ok: 通知センターに履歴が残り、種類ごとにメール通知をオフにできる
    ng: エラー通知が3秒で消え、内容を確認できない
    severity_default: 2
    phase: [WF, IMP]
    method: human
    refs: [HAX G17, WCAG 2.2 SC 2.2.1]

  # ---------- AI AI機能 ----------
  - id: AI-01
    category: AI
    principle: [P02, P01]
    title: AIにできること・できないことが示されている
    check: AI機能の入口で、対応範囲・苦手なこと・精度の目安が示され、AIが生成した内容であることが明示される
    ok: "「社内規程集をもとに回答します。規程外の質問には回答できません」と表示される"
    ng: 何でも答えられるように見え、根拠のない回答をユーザーが信じてしまう
    severity_default: 3
    phase: [WF, DS]
    method: llm
    refs: [HAX G1, HAX G2, Google PAIR]

  - id: AI-02
    category: AI
    principle: [P01]
    title: 出典・根拠を確認できる
    check: AIの回答・生成結果に出典（参照文書・該当箇所）へのリンクがあり、ユーザーが根拠を検証できる
    ok: 回答内の各主張に参照文書の該当ページへのリンクが付いている
    ng: 回答のみ表示され、どの資料に基づくか分からない
    severity_default: 3
    phase: [DS, IMP]
    method: llm
    refs: [HAX G11, Google PAIR]

  - id: AI-03
    category: AI
    principle: [P03]
    title: AIの結果を修正・破棄でき、自動実行されない
    check: AIの提案は、ユーザーが確認・編集・破棄してから反映される。データ更新・送信などを確認なしで実行しない
    ok: AI下書きは編集可能な状態で表示され、「反映」を押すまで保存されない
    ng: AIが抽出した値が確認なしでマスタに登録される
    severity_default: 4
    phase: [WF, IMP]
    method: human
    refs: [HAX G8, HAX G9, HAX G16]

  - id: AI-04
    category: AI
    principle: [P01, P03]
    title: 待ち時間・失敗時の扱いが分かる
    check: 生成中の進捗表示（ストリーミング等）と中断手段があり、失敗・タイムアウト時に再試行や代替手段が示される
    ok: 生成中に「停止」ボタンがあり、失敗時は「再生成」と「担当者に問い合わせ」が出る
    ng: 30秒以上スピナーが回り続け、止める方法がない
    severity_default: 3
    phase: [DS, IMP]
    method: human
    refs: [HAX G10, Nielsen H1, Nielsen H3]

  - id: AI-05
    category: AI
    principle: [P10]
    title: 誤りを報告でき、人に引き継げる
    check: 回答への評価・誤り報告ができ、解決しない場合に人（担当者・問い合わせ窓口）へ引き継ぐ導線がある
    ok: 各回答に「役に立った／誤りを報告」があり、「担当者に質問する」で会話履歴ごと起票できる
    ng: 誤回答があっても報告手段がなく、同じ誤りが繰り返される
    severity_default: 2
    phase: [WF, REL]
    method: human
    refs: [HAX G15, Google PAIR]
```

---

## 5. L3 サービス別プロファイル（テンプレート）

サービスごとに `uiux_profile_<service>_revN.md` として複製し、以下を埋めます。

```yaml
service: <サービス名>
guideline_version: rev1          # 準拠する共通版
owner: <サービスオーナー>
reviewed: <yyyy-mm-dd>

personas:                         # 1〜3名に絞る
  - name: <例：経理担当（週次で請求処理）>
    it_literacy: <低/中/高>
    context: <利用環境・頻度・制約>

primary_tasks:                    # シナリオベース評価の対象。3〜5個
  - id: T1
    name: <例：請求書を作成して送信する>
    frequency: <日次/週次/月次>
    success_criteria: <例：5分以内・ヘルプなしで完了>

glossary:                         # TXT-01 の判定基準になる
  - term: <正しい表記>
    avoid: [<使わない表記>]

target_devices:                   # RSP-01 の判定基準になる
  - <例：PC 1280px以上（Chrome/Edge最新）>

weights:                          # 共通項目の深刻度調整（±1まで）
  - id: <例：DAT-01>
    adjust: +1
    reason: <例：1テナントあたり数万件の一覧を扱うため>

excluded_items:                   # 適用しない項目
  - id: <例：AI-01〜AI-05>
    reason: <例：AI機能なし>

additional_items: []              # このサービス固有の項目。書式は §4 と同じ（IDは <サービス略号>-NN）
```

---

## 6. 指摘の記録書式

```yaml
- finding_id: <サービス略号>-<yyyymmdd>-NN
  item_id: <例：ERR-02>           # L2/L3の項目ID。該当なしは L1 の原則ID
  screen: <画面名／URL>
  phase: <WF/DS/IMP/REL>
  severity: <0-4>
  severity_note: <既定値から変えた場合の理由>
  description: <何が問題か（事実）>
  suggestion: <改善案>
  judged_by: <llm / human / auto>
  llm_verdict: <LLM一次判定の結果（llm項目のみ）>
  status: <open / fixed / wontfix>
  fixed_in: <修正が入ったブランチ・コミット（fixed のとき）>
  verified: <解消を確認した日付と方法（例：2026-09-26 再レビューで確認）>
```

- 再レビューで見つかった退行は、新しい `finding_id` で追加し、`description` に「◯◯の修正で生じた退行」と書く。

---

## 7. 参照先

| 区分 | 名称 | URL |
|---|---|---|
| 主な拠り所 | SmartHR Design System | https://smarthr.design/ |
| 主な拠り所 | SmartHR ウェブアクセシビリティ簡易チェックリスト | https://smarthr.design/accessibility/check-list/ |
| アクセシビリティ | デジタル庁デザインシステム | https://design.digital.go.jp/ |
| アクセシビリティ | ウェブアクセシビリティ導入ガイドブック（デジタル庁） | https://www.digital.go.jp/resources/introduction-to-web-accessibility-guidebook |
| アクセシビリティ | WAIC（JIS X 8341-3 関連） | https://waic.jp/guideline/ |
| アクセシビリティ | WCAG 2.2 | https://www.w3.org/TR/WCAG22/ |
| 画面パターン | Atlassian Design System | https://atlassian.design/ |
| 画面パターン | IBM Carbon Design System | https://carbondesignsystem.com/ |
| 原則 | NN/g 10 Usability Heuristics | https://www.nngroup.com/articles/ten-usability-heuristics/ |
| 原則 | NN/g Severity Ratings | https://www.nngroup.com/articles/how-to-rate-the-severity-of-usability-problems/ |
| 原則 | NN/g Response Times | https://www.nngroup.com/articles/response-times-3-important-limits/ |
| AI | Microsoft HAX Guidelines for Human-AI Interaction | https://www.microsoft.com/en-us/haxtoolkit/ai-guidelines/ |
| AI | Google PAIR People + AI Guidebook | https://pair.withgoogle.com/guidebook/ |
| 性能 | Core Web Vitals | https://web.dev/articles/vitals |
| 補助 | Apple HIG / Material Design 3 / Fluent 2 | https://developer.apple.com/design/human-interface-guidelines/ ・ https://m3.material.io/ ・ https://fluent2.design/ |

---

## 変更履歴

| 版 | 日付 | 内容 |
|---|---|---|
| rev1 | 2026-09-26 | 初版（方針確定：主な拠り所＝SmartHR DS、正本＝md、コア項目のみ） |
| rev2 | 2026-09-26 | 再レビュー（§3.5）と記録項目（fixed_in / verified）を追加。L2追加候補（§4.1）を新設し「最小文字サイズ」を登録（schmatz 初回レビューの結果を反映） |
