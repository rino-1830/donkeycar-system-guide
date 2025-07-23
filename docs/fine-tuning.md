# ファインチューニングの実装

Donkeycar で既存モデルを使って学習を継続する方法をまとめます。

## 1. 前提条件

- 学習済みモデル (`*.h5`) が存在すること。
- 追加で学習させたい走行データ（Tub ディレクトリ）が準備されていること。
- `manage.py` がある車両ディレクトリで作業を行うこと。

## 2. コマンドラインからの実行

以下のように `manage.py train` コマンドを実行します。

```bash
python manage.py train --tub <新しいTubパス> --model <出力モデル名> \
    --type <モデルタイプ> --transfer <既存モデルのパス>
```

主な引数の説明は次の通りです。

- `--tub`: ファインチューニングに使う Tub ディレクトリをカンマ区切りで指定します。
- `--model`: 学習後に保存するモデルのファイル名を指定します。
- `--type`: 使用するモデルタイプを指定します。省略時は `config.py` の `DEFAULT_MODEL_TYPE` が利用されます。
- `--transfer`: 既存の学習済みモデルを指定すると、その重みを読み込んで学習を再開します。

## 3. プログラムからの実行

`donkeycar.pipeline.training` の `train` 関数を直接呼び出してファインチューニングを行うこともできます。

```python
from donkeycar.pipeline.training import train
import donkeycar as dk

cfg = dk.load_config()
tubs = 'data/new_run'
model = 'models/mypilot.h5'
train(cfg, tubs, model, transfer='models/previous.h5')
```

## 4. 設定ファイルのポイント

`config.py` にはモデル転移に関する設定項目として `FREEZE_LAYERS` と
`NUM_LAST_LAYERS_TO_TRAIN` が用意されています。現状これらの値はコード内で
積極的に使用されていませんが、今後の拡張で特定レイヤの固定や
後半レイヤのみ学習する処理を組み込む際に役立ちます。

## 5. 注意点

- ファインチューニングで過学習を避けるため、`EARLY_STOP_PATIENCE` などの設定を調整すると
  効果的です。
- 学習結果は `tflite` 形式への変換も自動で行われます (`config.py` の `CREATE_TF_LITE` が有効な場合)。
