# リポジトリ構造概要

このドキュメントでは、`donkeycar-for-pius` リポジトリに含まれる主要なディレクトリとファイルの役割を説明します。

## ルートディレクトリ直下

| パス | 役割 |
| --- | --- |
| `donkeycar/` | Donkeycar の主要な Python パッケージ。車両制御や学習用モジュールを提供。 |
| `arduino/` | モーターエンコーダなど Arduino 用スケッチを格納。 |
| `scripts/` | 学習用補助スクリプトやツール群。 |
| `install/` | Raspberry Pi 等へのインストールスクリプトや conda 環境定義。 |
| `Dockerfile` | Docker 環境構築用設定ファイル。 |
| `Makefile` | `pytest` やパッケージ作成などの簡易コマンド。 |
| `.github/` | GitHub Actions 用のワークフロー設定。 |
| `setup.py` / `setup.cfg` | パッケージインストール用の設定ファイル。 |
| `MANIFEST.in` | パッケージング時に含めるファイルの指定。 |
| `CONTRIBUTING.md` | コントリビュートの方法をまとめたドキュメント。 |
| `LICENSE` | ライセンス表記。MIT ライセンスを採用。 |
| `README.md` | プロジェクト概要と基本的な利用方法。 |
| `.coveragerc` | テストカバレッジ計測用の設定。 |
| `README.md` | プロジェクト概要と基本的な利用方法。 |

## `donkeycar/` ディレクトリ

Donkeycar の本体ライブラリで、以下のサブディレクトリ・ファイルを持ちます。

| パス | 役割 |
| --- | --- |
| `parts/` | センサー、アクチュエータ、カメラ、学習モデルなど車を構成する各種パーツ実装。 |
| `templates/` | 走行・学習用の設定スクリプト例。カスタム構成の雛形として利用。 |
| `pipeline/` | データセット管理や学習処理を行うモジュール群。 |
| `tests/` | `pytest` による単体テスト。 |
| `management/` | `donkey` コマンドの実装。UI やジョイスティック設定などを含む。 |
| `utilities/` | 補助的なユーティリティ関数・クラス。 |
| `contrib/` | 外部コントリビューターによる追加モジュール。例: `robohat`。 |
| `gym/` | 強化学習向けに OpenAI Gym 形式で車両を制御する環境。 |
| `benchmarks/` | Tub ファイルの処理速度計測用スクリプト。 |
| `config.py` | 外部ファイルから設定を読み込む仕組みを提供。 |
| `vehicle.py` | パーツを組み合わせた車両クラスを定義。 |
| `memory.py` | キー・バリュー形式の簡易メモリ。 |
| `la.py` | 2次元ベクトル演算など線形代数処理。 |
| `utils.py` | 画像変換やファイル操作などの汎用関数。 |
| `geom.py` | 幾何処理用の補助関数。 |

## `scripts/` ディレクトリ

学習やデバッグを補助するスクリプトを配置しています。主なものは次の通りです。

- `tflite_convert.py` : TensorFlow Lite 形式へのモデル変換ツール。
- `multi_train.py` : 複数データセットを使った学習を実行。
- `profile.py` / `profile_coral.py` : 推論速度計測用のプロファイラ。
- `pigpio_donkey.py` : pigpio ライブラリを用いた車両制御サンプル。

## `install/` ディレクトリ

各種プラットフォームへのインストールを支援するファイルを含みます。

- `pi/` : Raspberry Pi 用のインストールスクリプト。
- `nano/` : Jetson Nano 用セットアップスクリプト。
- `envs/` : conda 環境定義 (`mac.yml`, `ubuntu.yml` など)。

## `arduino/` ディレクトリ

エンコーダ読み取り用の Arduino スケッチを格納しています。

- `mono_encoder/` : 単相エンコーダ用ファームウェア。
- `quadrature_encoder/` : クアドラチャエンコーダ用ファームウェア。

## テスト

`donkeycar/tests/` 以下に `pytest` を用いたテストコードが多数存在します。環境が整っていれば `pytest` コマンドで実行できます。