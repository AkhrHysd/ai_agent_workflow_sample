---
adr: 002
title: GraphQL Gateway を社内 BFF として導入
status: Accepted         # Proposed / Accepted / Superseded / Deprecated
date: 2025-05-05
superseded_by:           # 置き換えた ADR ID (あれば)
tags: [architecture, api, graphql]
authors: [@haya-frontend, @backend_taro]
reviewers: [@pm_yamada]
---

## TL;DR

- 社内サービス間 API 統合のため **Apollo Gateway** を採用。
- 外部向け REST は現状維持、BFF 位置づけで内部の集約を実現。

---

## 1. Context

1. マイクロサービス化に伴い **10 以上** の REST エンドポイント乱立
2. クライアント（Next.js & mobile）で **集約ロジック** が肥大化
3. チーム間で **型共有・スキーマ管理** が煩雑

## 2. Decision

- フロントエンドと BE チームで協議し、**GraphQL Gateway** を導入する
  - 実装：**Apollo GraphQL Federation 2**
  - デプロイ：社内 Kubernetes on EKS（Istio Ingress 経由）
- スキーマ管理を **GraphOS Cloud** で一元化
- 既存 REST 直アクセスは **6 ヶ月間段階的に廃止**

## 3. Consequences

| 項目     | 正の影響                          | 負の影響                        |
| -------- | --------------------------------- | ------------------------------- |
| コード量 | 集約レイヤ削減で UI 側 30% 軽量化 | Gateway 設計・保守コスト        |
| 型整合   | GraphQL SDL に一本化              | サービス間の **N+1** クエリ注意 |
| デプロイ | 既存 CI/CD にジョブ追加のみ       | K8s Pod 数 +1 → CPU コスト微増  |

### 未解決 / Follow‑up

- **権限リゾルバ** の実運用ガイドライン作成（担当: @haya-frontend、期限: 6/15）
- 既存 Postman コレクション → GraphQL Playground へ移行

## 4. Alternatives Considered

1. **REST Aggregator (nestjs/bff)**
   - Pros: 学習コスト低
   - Cons: 型共有・スキーマ分割の問題解決できず
2. **gRPC Mesh**
   - Pros: バイナリ高速・IDL あり
   - Cons: Mobile チームの gRPC 対応が困難

## 5. Links & References

- 実装 PoC PR: https://github.com/org/repo/pull/123
- GraphQL Federation 2 Docs: https://www.apollographql.com/docs/federation/
