# uiux-review-guidelines

自社SaaSのUI/UXレビュー標準です。共通ガイドライン（正本）、サービス別プロファイル、指摘の記録を管理します。

## 構成

```
guideline/   共通ガイドライン正本（uiux_review_guideline_revN.md）
profiles/    サービス別プロファイル（uiux_profile_<service>_revN.md）
findings/    指摘の記録（<service>/ 配下に yaml）
tools/       補助スクリプト
```

## 運用ルール

- **正本は md です。** xlsx は閲覧・記入用の生成物で、Git では管理しません（`.gitignore` 対象）。
- ガイドラインを改訂するときは、ファイル名の版番号（`_revN`）を上げ、本文末尾の変更履歴を更新します。
- L2（共通チェックリスト）への項目追加は、2サービス以上で指摘が出たものに限ります。1サービス固有の項目は、そのサービスのプロファイルの `additional_items` に入れます。

## xlsx の生成

```bash
pip install openpyxl pyyaml
python3 tools/gen_guideline_xlsx.py guideline/uiux_review_guideline_rev1.md
# -> guideline/uiux_review_guideline_rev1.xlsx
```

## 主な参照先

- SmartHR Design System（主な拠り所）
- デジタル庁デザインシステム / JIS X 8341-3（WCAG 2.2 AA相当）
- Atlassian Design System / IBM Carbon
- Microsoft HAX Guidelines / Google PAIR Guidebook
