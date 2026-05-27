# CodexMind 開発日報（全37日）

---

## Day 1

**作業内容**
プロジェクト全体の設計を開始。FastAPI + Qdrant + Neo4j + Ollama の技術スタックを確定し、ディレクトリ構成を定義した。`pyproject.toml` を作成し、主要依存ライブラリ（fastapi, qdrant-client, neo4j, ollama, tree-sitter）を整理した。

**成果物**
- プロジェクト骨格 `backend/app/` ディレクトリ構成
- `pyproject.toml` 初版

**翌日の予定**
FastAPI エントリポイントと基本ミドルウェアの実装

---

## Day 2

**作業内容**
`app/main.py` を実装。FastAPI アプリ初期化、CORS ミドルウェア設定、lifespan によるスタートアップ処理（Qdrant 接続確認・Neo4j スキーマ初期化）を組み込んだ。`/api/status` エンドポイントも追加。

**成果物**
- `app/main.py` 実装完了
- `/api/status` レスポンス確認

**翌日の予定**
グローバル設定モジュール（config.py）の実装

---

## Day 3

**作業内容**
`app/core/config.py` を実装。pydantic-settings を使い、`.env` から `OLLAMA_MODEL`・`NEO4J_URI`・`EMBEDDING_MODEL`・`SQLITE_PATH` などを読み込む構成を整えた。`.env.example` も同時に作成。

**成果物**
- `config.py` 実装完了
- `.env.example` 作成

**翌日の予定**
SQLite 初期化（database.py）とリポジトリモデルの定義

---

## Day 4

**作業内容**
`app/db/database.py` を実装。aiosqlite を使用し、`repos`・`chunks`・`symbols`・`search_history` テーブルの DDL を定義。WAL モード・busy_timeout 設定も組み込んだ。`app/models/repo.py` の Pydantic モデルも定義。

**成果物**
- `database.py` テーブル定義
- `models/repo.py` 初版

**翌日の予定**
リポジトリ管理 API（repo.py）の実装開始

---

## Day 5

**作業内容**
`app/api/repo.py` の実装を開始。リポジトリ登録（POST /api/repo）・一覧取得・詳細取得・削除の CRUD エンドポイントを実装した。root_path のバリデーションと12桁 hash による repo_id 生成ロジックを追加。

**成果物**
- `api/repo.py` CRUD 部分完成

**翌日の予定**
ファイルツリー取得・ファイル内容読み込み API の実装

---

## Day 6

**作業内容**
`repo_service.py` を実装。ファイルシステムを走査し、`SKIP_DIRS`・`SKIP_FILES`・`SKIP_SUFFIXES` によるベンダー・ビルド成果物の除外ロジックを整備。`detect_encoding` による文字コード自動判定も追加した。

**成果物**
- `repo_service.py` 実装完了
- `/api/repo/{id}/tree`・`/api/repo/{id}/file` エンドポイント動作確認

**翌日の予定**
tree-sitter パーサーモジュールの設計

---

## Day 7

**作業内容**
`parser_service.py` の設計を開始。tree-sitter を使った Java・JavaScript・TypeScript のメソッド/クラス抽出の方針を確定。tree-sitter が使えない言語向けの正規表現フォールバック（`_JAVA_METHOD_RE`・`_PYTHON_DEF_RE`・`_JS_FUNC_RE`）も定義した。

**成果物**
- パーサー設計ドキュメント（コメントとして実装内に記述）
- `NODE_TYPES` マッピング定義

**翌日の予定**
Java パーサーの tree-sitter 実装

---

## Day 8

**作業内容**
Java の tree-sitter パーサーを実装。`method_declaration`・`constructor_declaration`・`class_declaration` ノードを抽出し、メソッドシグネチャ・アノテーション・Javadoc を構造化チャンクに変換するロジックを作成した。

**成果物**
- Java パーサー実装完了
- テスト用 Java ファイルでの抽出結果確認

**翌日の予定**
JS/TS パーサーおよび正規表現フォールバックの実装

---

## Day 9

**作業内容**
JavaScript・TypeScript のパーサーを実装。arrow_function・method_definition など複数ノードタイプに対応。Python 向けの正規表現フォールバック（`_PYTHON_DEF_RE`）も実装し、Go はスライディングウィンドウ方式にフォールバックする設計を確定した。

**成果物**
- JS/TS/Python/Go パーサー実装完了

**翌日の予定**
チャンク構造化フォーマットの設計と token カウンターの実装

---

## Day 10

**作業内容**
チャンクの構造化フォーマット（`[FILE]`・`[CLASS]`・`[METHOD]`・`[SIGNATURE]`・`[ANNOTATIONS]`・`[CALLS]`・`[JAVADOC]`）を確定。bge-m3 の XLM-RoBERTa tokenizer を使用したトークンカウンター `_count_tokens` を実装し、切り窓判定に利用する設計とした。

**成果物**
- チャンクフォーマット確定・実装
- `_get_tokenizer()` lru_cache 実装

**翌日の予定**
call_refs（呼び出し関係）抽出ロジックの実装

---

## Day 11

**作業内容**
`parser_service.py` に呼び出し関係（call_refs）の抽出ロジックを追加。Java の `method_invocation` ノードから呼び出し先のクラス・メソッド名を抽出し、`[CALLS]` フィールドに格納する処理を実装した。

**成果物**
- call_refs 抽出完了
- `parse_chunks` 関数が chunks + call_refs を返す形に整理

**翌日の予定**
Neo4j クライアントの実装

---

## Day 12

**作業内容**
`app/core/neo4j_client.py` を実装。AsyncDriver シングルトン、`init_neo4j`（制約・インデックス作成）、`run_query`・`run_write`・`run_write_batch` ユーティリティを整備。`NEO4J_WRITE_CHUNK=100` によるバッチ分割書き込みも対応した。

**成果物**
- `neo4j_client.py` 実装完了
- Method / Class / File ノードの制約・インデックス確認

**翌日の予定**
インデクサー（indexer_service.py）Pass 1 の実装

---

## Day 13

**作業内容**
`indexer_service.py` の Pass 1 を実装。ファイルを走査し、parser_service でチャンク・symbol_map を生成して SQLite の `chunks`・`symbols` テーブルに保存。Neo4j に Method / Class / File ノードを upsert する処理を実装した。

**成果物**
- Pass 1 `_pass1_parse` 実装完了

**翌日の予定**
インデクサー Pass 2（Call Graph 構築）の実装

---

## Day 14

**作業内容**
Pass 2 を実装。call_refs と symbol_map を照合し、Neo4j に `[:CALLS]` エッジを作成。バッチ書き込みで大規模リポジトリでも処理が詰まらないよう `run_write_batch` を活用した。

**成果物**
- `_pass2_call_graph` 実装完了
- Neo4j Browser でエッジ確認

**翌日の予定**
Qdrant クライアントと Embedder の実装

---

## Day 15

**作業内容**
`app/core/qdrant_client.py` を実装。コレクション作成（dense + sparse のハイブリッドベクター）、`ensure_collection`・`delete_collection` を整備。`app/core/embedder.py` では bge-m3 の dense embedding と `lexical_weights`（sparse）の両方を生成する `embed_with_sparse` 関数を実装した。

**成果物**
- `qdrant_client.py`・`embedder.py` 実装完了

**翌日の予定**
インデクサー Pass 3（Qdrant への embed 投入）

---

## Day 16

**作業内容**
Pass 3 を実装。チャンクを `BATCH_SIZE=32` 単位で bge-m3 に投入し、dense vector + sparse vector（lexical_weights）を Qdrant に書き込む処理を完成させた。`run_index` のメイン関数でステータス更新も組み込み、インデックス進捗が確認できるようにした。

**成果物**
- `_pass3_embed` 実装完了
- インデックス完了後の `chunk_count`・`symbol_count`・`edge_count` 記録確認

**翌日の予定**
PageRank 計算の追加

---

## Day 17

**作業内容**
インデックス完了後に Neo4j の GDS（Graph Data Science）で PageRank を計算し、各 Method ノードの `pagerank` プロパティに書き込む処理を追加。コア度の高いメソッドを検索スコアリングで優先するための基盤を整えた。

**成果物**
- PageRank 計算・書き込み実装完了

**翌日の予定**
インデックス API（/api/repo/{id}/index）の実装

---

## Day 18

**作業内容**
`api/repo.py` にインデックス関連エンドポイントを追加。`POST /api/repo/{id}/index`（バックグラウンドタスク起動）・`GET /api/repo/{id}/index/status`・`GET /api/repo/{id}/index/logs` を実装した。

**成果物**
- インデックス API 3本実装完了
- ステータス（待機中・実行中・完了・エラー）の遷移確認

**翌日の予定**
検索サービスの設計（5層パイプライン）

---

## Day 19

**作業内容**
`search_service.py` の設計を開始。5 層パイプライン（クエリ理解 → dual-channel 検索 → Call Graph 拡張 → reranker → 総合スコアリング）の方針を確定。Layer 1 のクエリ理解（言語検出・意図分類・識別子抽出）を実装した。

**成果物**
- Layer 1 クエリ理解モジュール実装

**翌日の予定**
HyDE（仮想コード生成）の実装

---

## Day 20

**作業内容**
Layer 1 に HyDE（Hypothetical Document Embedding）を追加。自然言語クエリを Ollama に渡して仮想コードを生成し、その embed を dense 検索に使う設計を実装した。LLM が応答しない場合は元クエリにフォールバックする処理も組み込んだ。

**成果物**
- HyDE 実装完了・フォールバック動作確認

**翌日の予定**
Layer 2（Qdrant dual-channel 検索 + RRF 融合）の実装

---

## Day 21

**作業内容**
Layer 2 を実装。bge-m3 の dense ベクターと sparse（lexical_weights）でそれぞれ Qdrant を検索し、RRF（Reciprocal Rank Fusion、k=60）で融合するロジックを完成させた。

**成果物**
- Layer 2 RRF 融合実装完了

**翌日の予定**
Layer 3（Call Graph 1-hop 拡張）の実装

---

## Day 22

**作業内容**
Layer 3 を実装。RRF 上位候補のシンボルに対して Neo4j から 1-hop の呼び出し元・呼び出し先を取得し、PageRank 加重でスコアを補正するロジックを追加した。

**成果物**
- Layer 3 Call Graph 拡張実装完了

**翌日の予定**
Layer 4（bge-reranker-v2-m3 による cross-encoder 重排）の実装

---

## Day 23

**作業内容**
`app/core/reranker.py` を実装。bge-reranker-v2-m3 を使い、top-50 候補をクエリと pair-wise で重排するロジックを作成。reranker が利用不可の場合のフォールバック重み（`W_RRF_FALLBACK` 等）も定義した。

**成果物**
- `reranker.py` 実装完了
- Layer 4 統合テスト完了

**翌日の予定**
Layer 5（総合スコアリング + symbol 去重）の実装

---

## Day 24

**作業内容**
Layer 5 を実装。`W_RERANKER=0.60`・`W_RRF=0.20`・`W_GRAPH=0.10`・`W_SYMBOL=0.08`・`W_STRUCT=0.02` の重み付けで最終スコアを計算し、symbol レベルの重複除去を行う処理を完成させた。検索 API（`/api/search`）と検索履歴の保存も実装。

**成果物**
- `search_service.py` 全 5 層実装完了
- 検索 API 動作確認

**翌日の予定**
MyBatis mapper XML パーサーの実装

---

## Day 25

**作業内容**
`mapper_xml_service.py` を実装。MyBatis の `<select>`・`<insert>`・`<update>`・`<delete>` タグから SQL ID・SQL 本文を抽出し、チャンク化するロジックを作成した。Java サービスクラスとの紐付けも `[CALLS]` フィールドで表現できるよう設計した。

**成果物**
- `mapper_xml_service.py` 実装完了

**翌日の予定**
Graph Service の実装

---

## Day 26

**作業内容**
`graph_service.py` を実装。メソッドレベルの N-hop 呼び出しグラフ取得（`get_method_graph`）、影響分析（`get_impact_graph`）、最短経路（`get_path`）の 3 機能を Cypher クエリで実装。`/api/graph` エンドポイントも追加した。

**成果物**
- `graph_service.py` 実装完了
- graph API 3本確認

**翌日の予定**
分析サービス（analysis_service.py）の設計

---

## Day 27

**作業内容**
`analysis_service.py` の設計を開始。summary・bug・deps・custom の 4 モードを定義し、全モードで LLM 呼び出し前に Neo4j から呼び出しチェーンコンテキストを取得・プロンプトに注入する設計を確定した。

**成果物**
- 分析サービス設計完了
- `_build_call_context` 関数実装

**翌日の予定**
summary・bug モードの LLM ストリーミング実装

---

## Day 28

**作業内容**
summary・bug モードの実装。Ollama の async ストリーミング API を使い、SSE（Server-Sent Events）でフロントエンドにチャンクを逐次送信する処理を完成させた。トークン推定・コード切り詰め（`_truncate_code`）も実装。

**成果物**
- summary・bug モード SSE ストリーミング実装完了

**翌日の予定**
deps モードと custom（チャット）モードの実装

---

## Day 29

**作業内容**
deps モードを実装。LLM に Mermaid を生成させるのではなく、Neo4j の call_graph データを直接 JSON で返す設計に変更し、レスポンスの安定性を高めた。custom モード（チャット Q&A）も実装し、呼び出しチェーン要約をシステムプロンプトに注入する処理を追加。

**成果物**
- deps・custom モード実装完了
- `api/analysis.py` SSE エンドポイント完成

**翌日の予定**
i18n モジュールの実装

---

## Day 30

**作業内容**
`app/core/i18n/` を実装。中国語・日本語・英語の 3 ロケールに対応した `locales/*.json` を作成し、分析プロンプトの多言語切り替えロジックを `_CALL_CONTEXT_LABELS` として実装した。

**成果物**
- バックエンド i18n 実装完了（zh / ja / en）

**翌日の予定**
Vue 3 フロントエンドの骨格実装

---

## Day 31

**作業内容**
フロントエンドの開発を開始。Vite + Vue 3 + TypeScript + Tailwind CSS の構成を整備し、`App.vue`・`main.ts`・Pinia ストアの骨格を作成した。`api/client.ts`（axios インスタンス）と `api/repo.ts`・`api/status.ts` を実装。

**成果物**
- フロントエンド骨格完成
- `/api/status` との疎通確認

**翌日の予定**
リポジトリセレクターとファイルツリーの実装

---

## Day 32

**作業内容**
`components/sidebar/RepoSelector.vue` と `FileTree.vue`、`stores/repoStore.ts` を実装。リポジトリ登録・選択・インデックス進捗表示の UI を完成させた。`TopBar.vue`・`StatusBar.vue`・`SidebarNav.vue` のレイアウト部品も実装。

**成果物**
- サイドバー UI 一式完成

**翌日の予定**
Monaco Editor 統合と検索パネルの実装

---

## Day 33

**作業内容**
`composables/useMonaco.ts` を実装し、Monaco Editor をコードビューアーとして組み込んだ。`SearchPanel.vue`・`SearchResults.vue` を実装し、検索結果クリックでエディタへのジャンプ機能を追加。`stores/searchStore.ts` で検索履歴管理も実装。

**成果物**
- Monaco 統合・検索パネル完成

**翌日の予定**
分析パネルと Tab 群の実装開始

---

## Day 34

**作業内容**
`AnalysisPanel.vue` と `stores/analysisStore.ts` を実装。SSE ストリームの受信・パースロジックを `api/analysis.ts` に実装し、逐次テキストをストアに積む処理を完成させた。`TabSummary.vue`・`TabHistory.vue`・`TabBug.vue` を実装。

**成果物**
- 分析パネル基盤・Summary / Bug / History タブ完成

**翌日の予定**
TabDeps（D3 呼び出しグラフ）の実装

---

## Day 35

**作業内容**
`TabDeps.vue`（572行）を実装。D3.js で force-directed graph を描画し、Controller / Service / DAO / SQL / Util ごとのノード色分け・ホバーツールチップ・深度調節・フィットビューリセット機能を実装した。`api/graph.ts` も整備。

**成果物**
- TabDeps D3 グラフ完成
- 深度 1〜5 の切り替え動作確認

**翌日の予定**
TabDocs・TabChat の実装と i18n 対応

---

## Day 36

**作業内容**
`TabDocs.vue`（ドキュメント生成）・`TabChat.vue`（インタラクティブ Q&A、237行）を実装。フロントエンド i18n（`src/i18n/locales/`）を中国語・日本語・英語の 3 言語で整備し、`LanguageSwitcher.vue` による切り替え UI を追加した。

**成果物**
- TabDocs・TabChat 完成
- フロントエンド i18n 完成

**翌日の予定**
全体統合テスト・バグ修正・ドキュメント整備

---

## Day 37

**作業内容**
全体の統合テストを実施。Qdrant・Neo4j・Ollama を起動した状態で Java リポジトリをインデックスし、検索・分析・グラフ表示の E2E 動作を確認した。いくつかの SSE パース不具合と D3 レイアウト崩れを修正。`README.md` を更新し、セットアップ手順・API 一覧を整備してプロジェクトを完成させた。

**成果物**
- バグ修正完了
- README 最終版
- CodexMind v2.0.0 リリース

---

*以上、CodexMind 開発日報（全37日）*
