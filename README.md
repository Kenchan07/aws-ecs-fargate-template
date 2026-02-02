# aws-ecs-fargate-template

## 概要
本リポジトリは、AWS 上で **ECS（Fargate）+ ALB + CloudWatch Logs** を中心に、目的別の「再現可能な最小構成（Case）」を整理したポートフォリオです。

主な方針は次のとおりです。
- **Case 定義 → 手順 → 検証 → つまずき → コスト → 削除**までを一貫して記録します。
- まずは **手動構築で成立条件を固定**し、その後 IaC（CloudFormation 等）へ展開します。
- 「動いた」だけで終わらせず、再現性・切り分け・後片付けまで含めて記載します。

---

## 対象ドキュメント（現行）
- **Case A：NAT なし / Public のみ（最小成立）**
  - [case-a-no-nat-public.md](./docs/case-a-no-nat-public.md)
- **Case B：NAT あり / Private タスク（Private Subnet + NAT）**
  - [case-b-nat-private.md](./docs/case-b-nat-private.md)
- **Case B（Cost）：NAT / EIP 等のコスト検証・収束**
  - [case-b-cost-portfolio.md](./docs/case-b-cost-portfolio.md)

> 各ドキュメントは共通の構成（目的・構成図・再現可能手順・つまずき・コスト・削除手順）で揃えます。

---

## 推奨の読む順番
初見の方は、以下の順で読むのが最短です。

1. [case-a-no-nat-public.md](./docs/case-a-no-nat-public.md)
   - NAT なし / Public のみで成立条件を固定します。
2. [case-b-nat-private.md](./docs/case-b-nat-private.md) 
   - Task を Private に移し、NAT 経由の egress を成立させます。
3. [case-b-cost-portfolio.md](./docs/case-b-cost-portfolio.md)
   - NAT / EIP の残存や請求の見え方を検証し、収束までの手順を整理します。

---

## アーキテクチャ（共通コンセプト）
- VPC（Public/Private Subnet）
- ALB（Internet-facing）
- ECS（Fargate）で nginx 等のシンプルな Web コンテナを稼働
- CloudWatch Logs（アプリログ／ヘルスチェック観測）

詳細は各 Case ドキュメントの「構成」および「構成図」を参照してください。

---

## 前提（共通）
- 個人 AWS アカウント（学習用途）
- 予算アラート（Budgets）設定（放置コスト対策）
- リージョン：ap-northeast-1（Tokyo）を前提（各 Case 側にも明記します）

---

## リポジトリ構成
- README.md
- docs/
  - case-a-no-nat-public.md
  - case-b-nat-private.md
  - case-b-cost-portfolio.md

画像は `docs/images/` 配下に配置する想定です（各ドキュメント内で `./images/...` を参照します）。

---

## 運用・トラブルシュート（共通の見方）
各 Case の「つまずき」では、切り分け順を固定します。

- ECS
  - Service events / Task stopped reason
- ALB
  - Target Group の Health（典型原因：SG / Port / Health check）
- HTTP 確認
  - curl で 200 OK を確認してからブラウザ確認へ進みます。
- Logs
  - CloudWatch Logs でヘルスチェック（例：`ELB-HealthChecker/2.0`）を観測します。

---

## コスト方針
- 学習用途のため、最小構成・短時間稼働を前提とします。
- 放置を避けるため、各 Case に **削除手順**と **削除完了チェック**を必ず記載します。
- Case B のコスト検証は `docs/case-b-cost-portfolio.md` に集約しています。

---

## 今後のロードマップ（任意）
- [ ] IaC 化（CloudFormation / Terraform 等）
- [ ] 監視（CloudWatch Alarm）追加（最小限の運用シグナル）
- [ ] 代表的な障害シナリオの Runbook 追加（2〜3本）

---

## 注意
- 本リポジトリは学習・検証目的であり、各 Case は「成立条件の固定」を優先します。
- 本番相当のセキュリティ（HTTPS、WAF、アクセスログ、最小権限、脆弱性対策等）は段階的に扱います。
