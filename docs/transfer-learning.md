# 転移学習の実装

> [!CAUTION]
> 現在、`donkeycar/pipeline/training.py` の `train` に `FREEZE_LAYERS` を利用する機能が実装されていないため、転移学習は利用できません。
> 後述の内容は公式のドキュメントやソースコードからの推測を含みます。
> 今後実装する際のメモとして捉えてください。

## 1. トレーニングスクリプト

`donkeycar/templates/train.py` では、学習処理を実行する `train` 関数が呼び出されます。ここで `--transfer` オプションを指定することで、既存モデルの重みを読み込んで学習を開始できます。

```bash
python train.py --tubs=~/mycar/data --model=models/mypilot.h5 --transfer=models/old.h5
```

## 2. 転移学習処理

学習を行う `donkeycar/pipeline/training.py` の `train` 関数では、以下のように指定されたモデルを読み込みます。

```python
kl = get_model_by_type(model_type, cfg)
if transfer:
    kl.load(transfer)
```

この `kl.load` メソッドは `KerasPilot` クラスに実装されており、`KerasInterpreter` を通してモデルを読み込みます。【F:donkeycar/pipeline/training.py†L101-L115】【F:donkeycar/parts/keras.py†L45-L62】【F:donkeycar/parts/interpreter.py†L150-L167】

## 3. 設定オプション

設定ファイル (`cfg_basic.py` など) では転移学習に関するオプションが定義されています。主な項目は以下の通りです。

- `FREEZE_LAYERS`：転移元モデルの一部層を固定するかどうか
- `NUM_LAST_LAYERS_TO_TRAIN`：学習させる最後の層数

これらの値を調整することで、学習範囲を細かく指定できます。【F:donkeycar/templates/cfg_basic.py†L96-L104】

## 4. コマンドライン

`donkeycar/management/base.py` では学習コマンドに `--transfer` オプションが定義されています。これにより、学習開始時に転移学習を有効化できます。【F:donkeycar/management/base.py†L545-L566】

```bash
python manage.py train --tub data --model models/new.h5 --transfer models/base.h5
```

## 5. モデルデータベース

学習結果は `donkeycar/pipeline/database.py` によって記録されます。転移元モデル名も `Transfer` フィールドとして保存されるため、後から確認可能です。【F:donkeycar/pipeline/database.py†L118-L127】【F:donkeycar/pipeline/database.py†L126-L135】
