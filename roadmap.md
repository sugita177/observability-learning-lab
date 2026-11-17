# Observability Learning Lab - 学習ロードマップ (`roadmap.md`)

ローカル Docker 環境（OrbStack）と Go 言語を用いて、オブザーバビリティの基礎から実務レベルのパイプライン設計・カオスエンジニアリング・SRE（SLI/SLO）実践までを段階的に習得するカリキュラムです。

---

## Phase 1: オブザーバビリティの基礎とオールインワン環境構築
**目標**: オブザーバビリティの概念を理解し、最小構成の Go マイクロサービス群から OpenTelemetry でテレメトリを送信・可視化する一連の流れを体験する。

- [ ] **Step 1-1: オブザーバビリティの概念理解**
  - メトリクス・ログ・トレース（3本柱）の役割と特性の整理
  - 従来の「モニタリング（既知の障害の監視）」と「オブザーバビリティ（未知の事象の探索）」の本質的な違い
  - OpenTelemetry (OTel) が解決する課題（ベンダーロックイン回避と標準化）
- [ ] **Step 1-2: `grafana/otel-lgtm` を用いたローカル基盤の立ち上げ**
  - `docker-compose.yml` の作成とポート・ボリューム設計
  - OTel Collector + Prometheus + Loki + Tempo + Grafana オールインワンコンテナの起動確認
- [ ] **Step 1-3: Go サンプルWebアプリケーションの構築（2サービス + DB構成）**
  - 標準ライブラリ `net/http` ベースの `frontend-api` と `backend-api` の構築
  - 軽量DB（SQLite または最小構成の PostgreSQL）の組み込みとマルチステージビルド `Dockerfile` の作成
- [ ] **Step 1-4: OpenTelemetry SDK の導入と分散トレーシングの可視化**
  - TracerProvider / OTLP Exporter / Resource（`service.name` 等）の実装
  - `frontend-api` → `backend-api` → `DB` 間の Context Propagation（W3C Trace Context 伝播）の実装
  - Grafana Tempo 上でのエンドツーエンドのトレース波形確認

---

## Phase 2: テレメトリの相関（Correlation）の実装
**目標**: メトリクス・ログ・トレースを孤立させず、相互にドリルダウンできる「テレメトリの相関」を実装し、基本ダッシュボードを構築する。

- [ ] **Step 2-1: Go `log/slog` による構造化ログとトレースコンテキスト注入**
  - 標準ライブラリ `log/slog` を用いた JSON 構造化ログの出力
  - `context.Context` から `trace_id` / `span_id` を抽出し、自動的にログフィールドへ注入するカスタム `slog.Handler` の実装
- [ ] **Step 2-2: メトリクス計装と Exemplars（エグゼンプラー）の連携**
  - OTel HTTP メトリクス（リクエスト数、レイテンシヒストグラム）の収集確認
  - メトリクスのデータポイントに `trace_id` を紐づける Exemplars の仕組み理解と確認
- [ ] **Step 2-3: Grafana での「Metrics → Logs → Traces」シームレス連携**
  - Loki の Derived Fields / Tempo データソース連携の確認と設定
  - メトリクスのスパイク → 該当時間帯・該当リクエストのログ → 該当トレースへの相互ジャンプ検証
- [ ] **Step 2-4: RED メソッドに基づく基本ダッシュボードの構築**
  - **R**ate（秒間リクエスト数）、**E**rrors（エラー率）、**D**uration（レイテンシのパーセンタイル: p50, p95, p99）を PromQL で記述し、Grafana ダッシュボード化

---

## Phase 3: OpenTelemetry Collector の独立とパイプライン設計
**目標**: オールインワン環境から OTel Collector を独立させ、本番運用で必須となるデータ保護・リソース保護・コスト最適化のパイプラインを設計する。

- [ ] **Step 3-1: OTel Collector コンテナの分離とアーキテクチャ理解**
  - Deployment パターン（Agent パターン vs Gateway パターン）の理解
  - 独立した `otel-collector-contrib` コンテナの立ち上げと `otel-collector-config.yaml` の基本構造（Receivers, Processors, Exporters, Service Pipelines）の実装
- [ ] **Step 3-2: メモリ保護とバッチ処理のパイプライン設計**
  - `memory_limiter` プロセッサによる OOM（Out Of Memory）回避の仕組みと設定
  - `batch` プロセッサによるネットワーク効率向上とメモリ・遅延のトレードオフ設計
- [ ] **Step 3-3: 機密情報（PII）のマスキングと属性加工**
  - `attributes` / `transform` (OTTL) プロセッサを用いた、メールアドレス・トークン・クエリパラメータ等の機密情報マスキング実装
- [ ] **Step 3-4: Tail-based Sampling（テールベースサンプリング）の実装**
  - Head-based Sampling と Tail-based Sampling の違いとトレードオフの理解
  - `tail_sampling` プロセッサを用い、「エラー発生トレース（100%保存）」「高レイテンシトレース（100%保存）」「正常トレース（低確率サンプリング）」のポリシー実装と検証

---

## Phase 4: カスタム計装と負荷試験（k6）
**目標**: 自動計装だけでは見えないビジネスロジック内部の挙動を可視化し、高負荷時におけるテレメトリとシステムの挙動を観察する。

- [ ] **Step 4-1: 手動計装（Manual Instrumentation）によるカスタムスパンと属性**
  - ビジネスロジック内の特定処理に対する子スパン（Child Span）の作成
  - 検索・フィルタリングに有用なカスタム属性（Span Attributes）と Span Events / Status の付与
- [ ] **Step 4-2: カスタムメトリクス（Counter / UpDownCounter / Histogram）の実装**
  - ビジネスメトリクスや内部キュー長などを計測する OTel Meter API の実装
  - カーディナリティ爆発（High Cardinality）の危険性とラベル設計のベストプラクティス学習
- [ ] **Step 4-3: `k6` を用いたローカル負荷試験シナリオの作成と実行**
  - 段階的に負荷を上げる（Ramp-up / Steady / Ramp-down）`k6` スクリプトの作成
  - 負荷試験中のアプリケーションおよび OTel Collector のスループット・レイテンシ・サンプリング挙動の観察

---

## Phase 5: 継続的プロファイリング（Continuous Profiling）の導入
**目標**: 「どの関数・どのコード行が CPU やメモリを消費しているか」を特定する第4のシグナル（Profiling）を導入する。

- [ ] **Step 5-1: Grafana Pyroscope の導入と Go アプリケーション連携**
  - `docker-compose.yml` への Pyroscope コンテナ追加
  - Go の `runtime/pprof` と Pyroscope SDK（または Pull スクレイピング）を用いたプロファイル収集の実装
- [ ] **Step 5-2: フレームグラフ（Flame Graph）の読解とボトルネック特定**
  - CPU Profile / Alloc Space（メモリ割り当て） / Goroutine Profile のフレームグラフ読解
  - トレースの特定スパンとプロファイルを紐づける Span Profiles の概念とボトルネックコードの特定・改善

---

## Phase 6: カオスエンジニアリング・障害シミュレーション演習
**目標**: 意図的に仕込まれた複数のパフォーマンス劣化・障害シナリオに対し、ソースコードを最初に見るのではなく、ダッシュボード・トレース・ログ・プロファイルのみを頼りに根本原因（Root Cause）を特定・改修する。

- [ ] **Step 6-1: シナリオ1 - N+1 クエリ問題の検知と解消**
  - トレースのスパン数・波形の特徴から N+1 問題を特定し、クエリ改修後のレイテンシ改善を定量確認する
- [ ] **Step 6-2: シナリオ2 - DB スロークエリとインデックス欠如の特定**
  - DB スパンの属性・実行時間からボトルネッククエリを特定し、改善する
- [ ] **Step 6-3: シナリオ3 - トランザクションのロック競合 / コネクションプール枯渇**
  - 並行リクエスト下での待機時間増加・タイムアウト発生原因をメトリクスとトレースから切り分ける
- [ ] **Step 6-4: シナリオ4 - 外部/下流 API の間欠的タイムアウトとリトライ嵐**
  - 下流サービスの遅延・エラーが上流サービスへ波及（Cascading Failure）する様子を可視化し、タイムアウト・回復設計を検証する

---

## Phase 7: SRE実践（SLI / SLO / アラート設計）
**目標**: 単なるリソース監視（CPU使用率など）から脱却し、ユーザー体験に基づく SLI/SLO とエラーバジェットのバーンレートアラートを設計・実装する。

- [ ] **Step 7-1: SLI（サービスレベル指標）と SLO（サービスレベル目標）の策定**
  - Availability（可用性）と Latency（応答時間）の SLI を PromQL で定義する
  - ターゲット SLO（例: 99.9%）とエラーバジェット（Error Budget）の算出
- [ ] **Step 7-2: エラーバジェット・バーンレート（Burn Rate）の理解と計算**
  - なぜ単純な閾値アラートではなく Multi-window, Multi-burn-rate アラートが必要なのか（検知速度と誤報率のトレードオフ）の理解
- [ ] **Step 7-3: Grafana Alerting による実践的アラートルールの設定と発火テスト**
  - Fast Burn（ページャー相当：急激なバジェット消費）と Slow Burn（チケット相当：緩やかなバジェット消費）のアラートルール実装
  - `k6` で意図的にエラー/遅延を発生させ、アラート発火から原因調査（Runbook的フロー）までを一気通貫で検証する
