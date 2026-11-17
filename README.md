# Observability Learning Lab

ローカルの Docker 環境（OrbStack）上で、**OpenTelemetry** と **Grafana LGTM スタック（Loki, Grafana, Tempo, Mimir/Prometheus）+ Pyroscope** を用いた実践的なオブザーバビリティ基盤の構築・計装・運用・SREプラクティスをゼロから体系的に学ぶためのハンズオン・ラボです。

---

## リポジトリの目的

1. **三本柱＋プロファイリングの統合理解**: メトリクス・ログ・トレースの相関（Correlation）と継続的プロファイリング（Continuous Profiling）を組み合わせた迅速な原因特定フローを習得する。
2. **OpenTelemetry によるベンダーニュートラルな計装**: Go 標準ライブラリ（`net/http`, `log/slog`）ベースのマイクロサービスに対して、自動計装と手動計装を適切に組み合わせる実装力を身につける。
3. **本番を見据えた OTel Collector パイプライン設計**: バッチ処理、メモリ保護、PII（機密情報）マスキング、Tail-based Sampling などの実践的なテレメトリパイプラインを構築する。
4. **カオスエンジニアリングと SRE 実践**: 負荷試験（`k6`）や障害シミュレーションを通じてボトルネックを特定し、SLI / SLO およびエラーバジェットのバーンレートに基づくアラート設計を行う。

---

## 技術スタック

| カテゴリ | 技術 / ツール | 用途 |
| :--- | :--- | :--- |
| **Application** | Go (`net/http`, `log/slog`) | サンプルマイクロサービス（Frontend API / Backend API） |
| **Database** | SQLite / PostgreSQL | バックエンドのデータ永続化・クエリ計測対象 |
| **Instrumentation** | OpenTelemetry Go SDK | トレース・メトリクス・コンテキスト伝播の計装 |
| **Telemetry Pipeline** | OpenTelemetry Collector (`contrib`) | テレメトリの受信・加工・マスキング・サンプリング・転送 |
| **Metrics** | Prometheus | メトリクスの時系列保存・PromQL による集計と SLI 算出 |
| **Logs** | Grafana Loki | 構造化ログの集約・LogQL による検索 |
| **Traces** | Grafana Tempo | 分散トレースの保存・TraceQL による検索 |
| **Profiling** | Grafana Pyroscope | CPU / メモリ使用量の継続的プロファイリング（Flame Graph） |
| **Visualization & Alert** | Grafana | ダッシュボード可視化、シグナル間ドリルダウン、バーンレートアラート |
| **Load Testing** | Grafana k6 | 負荷試験・トラフィック生成・カオスシナリオのトリガー |
| **Runtime** | Docker / Docker Compose (OrbStack) | ローカルコンテナ実行環境 |

---

## ディレクトリ構成（予定）

学習フェーズの進行に合わせて、段階的に以下の構成へと拡張していきます。

```text
.
├── README.md                 # リポジトリ概要・セットアップ手順
├── roadmap.md                # 学習カリキュラムと進捗管理表
├── docker-compose.yml        # オブザーバビリティ基盤＆アプリ起動定義
├── config/                   # 各種ミドルウェアの設定ファイル群
│   ├── otel-collector/       # OpenTelemetry Collector 設定
│   ├── prometheus/           # Prometheus 設定
│   ├── loki/                 # Loki 設定
│   ├── tempo/                # Tempo 設定
│   └── grafana/              # Grafana データソース・ダッシュボード定義
├── services/                 # 計装対象の Go マイクロサービス群
│   ├── frontend-api/         # エントリーポイントとなるフロントエンド API
│   └── backend-api/          # DB アクセスやビジネスロジックを持つバックエンド API
├── k6/                       # k6 負荷試験スクリプト群
└── docs/                     # フェーズごとの学習記録・アーキテクチャメモ・検証ログ
    └── 00_template.md        # 学習記録用テンプレート
```

---

## セットアップと進め方

### 前提条件
- Docker および Docker Compose が利用可能であること（macOS の場合は [OrbStack](https://orbstack.dev/) 推奨）
- Go 1.22 以上（ローカルでコード補完やビルド確認を行う場合）
- [k6](https://k6.io/)（Phase 4 以降の負荷試験で使用。Docker 経由での実行も可能）

### クイックスタート（Phase 1 以降で構築）

```bash
# 1. リポジトリのクローン
git clone <repository-url>
cd observability-learning-lab

# 2. コンテナ群の起動（Phase 1 Step 1-2 以降）
docker compose up -d --build

# 3. 動作確認
# Grafana UI: http://localhost:3000
```

---

## 学習カリキュラムと記録

詳細なカリキュラムと現在の進捗状況は [roadmap.md](./roadmap.md) を参照してください。
また、各ステップでの設計判断や検証結果は [docs/](./docs/) ディレクトリ配下に順次記録していきます。

- **Phase 1**: オブザーバビリティの基礎とオールインワン環境構築
- **Phase 2**: テレメトリの相関（Correlation）の実装
- **Phase 3**: OpenTelemetry Collector の独立とパイプライン設計
- **Phase 4**: カスタム計装と負荷試験（k6）
- **Phase 5**: 継続的プロファイリング（Continuous Profiling）の導入
- **Phase 6**: カオスエンジニアリング・障害シミュレーション演習
- **Phase 7**: SRE実践（SLI / SLO / アラート設計）
