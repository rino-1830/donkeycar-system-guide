# 教師データのフォーマットと扱い方

このドキュメントでは、Donkeycarで使用される教師データ(学習用データ)の構造と、データの取り扱いに関する手順を説明します。

## データ構造

Donkeycarでは、学習用データを`Tub`と呼ばれるデータストアに保存します。`Tub`は以下の2つの要素から構成されます。

1. **カタログファイル**: JSON 形式の1行テキストで、各レコードのメタデータを保持します。
2. **images ディレクトリ**: 画像ファイル (JPEG または PNG) を格納します。カタログファイルから参照されます。

カタログ内の各レコードには、以下のようなフィールドが含まれます。

- `cam/image_array`: 画像ファイル名
- `user/angle`: ステアリング角度 (float)
- `user/throttle`: スロットル値 (float)
- `user/mode`: 手動/自動などのモード
- `imu/acl_x` 等の IMU センサ値 (オプション)
- `behavior/one_hot_state_array` や `localizer/location` など、追加の入力情報 (オプション)
- `_timestamp_ms`, `_index`, `_session_id`: 内部管理用フィールド

レコードは辞書形式で保存され、必要に応じて画像へのパスや数値データが格納されます。

## ディレクトリ構造例

`TubWriter` でデータを収集すると、次のようなディレクトリ構造になります。

```
mycar/data/
├── manifest.json
├── catalog_0.catalog
├── catalog_1.catalog
└── images/
    ├── 0_cam_image_array_.jpg
    ├── 1_cam_image_array_.jpg
    └── ...
```

`manifest.json` は入力名や型、カタログファイル一覧を保持します。`catalog_x.catalog` はレコードを1行JSON形式で記録するファイルで、一定数ごとに新しいファイルが生成されます。画像入力は `images/` 以下に `<index>_<key>_.jpg` の形式で保存され、レコードから参照されます。

## レコードキーの扱い

各レコードには自動的に `_index` が振られ、0 からの連番として保存されます。この値が欠番や重複を起こすと `Collator` が連続データとして認識できなくなるため注意が必要です。`TubWriter` を介さずに編集する場合は、`_index` の整合性を保ってください。`_timestamp_ms` は記録時刻をミリ秒単位で示し、`_session_id` は録画セッションを識別するためのIDです。

## データの保存

データ記録は `TubWriter` パーツを用いて行います。以下は記録処理の簡単な流れです。

```python
from donkeycar.parts.tub_v2 import TubWriter

inputs = ['cam/image_array', 'user/angle', 'user/throttle']
types = ['image_array', 'float', 'float']

tub = TubWriter('~/mycar/data', inputs=inputs, types=types)

# ループ中でデータを書き込む
while running:
    image = camera.run()
    angle = controller.angle
    throttle = controller.throttle
    tub.run(image, angle, throttle)
```

`run()`メソッドに渡された値が`Tub`内に保存され、画像ファイルは`images`ディレクトリに書き出されます。

## データの読み込み

学習時には `TubDataset` クラスを利用してデータを読み込みます。以下のように使用します。

```python
from donkeycar.config import Config
from donkeycar.pipeline.types import TubDataset

cfg = Config()
dataset = TubDataset(cfg, ['/path/to/tub'])
records = dataset.get_records()
```

`get_records()` を呼び出すと、`TubRecord` オブジェクトのリストが得られます。各 `TubRecord` は必要に応じて画像を読み込み、学習用に変換するためのメソッドを提供します。

## 画像データの扱い

画像はデフォルトで JPEG 形式で保存されます。IMU や深度画像など、16bit グレースケールを使用する場合は PNG 形式が用いられます。画像の前処理や拡張は `ImageAugmentation` や `ImageTransformations` の各パーツで行われます。

学習時のパイプラインでは、画像を`np.uint8`形式で読み込み、必要に応じて正規化 (0〜1 の範囲にスケーリング) や回転・トリミングが行われます。

## データ整理のポイント

- **削除・復元**: `TubWiper` パーツにより、不要なレコードを指定数だけ削除できます。誤操作で削除した場合も `restore_records()` を使って復元可能です。
- **連番管理**: レコードには`_index`が付与され、連続したデータ列として扱われます。RNN など連続データが必要な場合は `Collator` を利用してシーケンスを生成します。
- **フィルタ**: `TRAIN_FILTER` を設定すると、特定の条件を満たすレコードだけを学習に使用できます。

## まとめ

Donkeycarの教師データは `Tub` 形式で保存され、カタログと画像ディレクトリから構成されます。`TubWriter` で収集したデータを `TubDataset` で読み込み、`TubRecord` を通して画像やセンサ値を取得します。前処理や拡張を適切に設定することで、安定したモデル学習を行うことが可能です。
