# donkeycar/templates/complete.py の構造

**complete.py** は DonkeyCar プロジェクトで *フル機能* の車両制御を行うスクリプトです。カメラ・コントローラ・AI モデルなど複数の「パーツ」を組み合わせ、実車を走行させたり学習用データを収集したりできます。

---

## 目次

1. [コマンドラインの使い方](#コマンドラインの使い方)
2. [主な機能とフロー](#主な機能とフロー)
3. [内部クラス一覧](#内部クラス一覧)
4. [各クラスの詳細](#各クラスの詳細)
   1. [DriveMode](#drivemode)
   2. [LedConditionLogic](#ledconditionlogic)
   3. [RecordTracker](#recordtracker)
   4. [ToggleRecording](#togglerecording)
   5. [UserPilotCondition](#userpilotcondition)
   6. [Vectorizer](#vectorizer)
5. [参考](#参考)

---

## コマンドラインの使い方

`manage.py` から **drive** と **train** の 2 つのサブコマンドとして呼び出します。

```bash
# 走行（データ収集・推論）
python manage.py drive \
  --model=myPilot.h5 \
  --js \
  --type=categorical \
  --camera=single \
  --myconfig=myconfig.py

# 学習
python manage.py train \
  --tubs=data/* \
  --model=myPilot.h5 \
  --type=linear
```

> **補足**: スクリプト本体は `donkeycar/templates/complete.py` にありますが、エントリーポイントはプロジェクトルートの `manage.py` です。

---

## 主な機能とフロー

1. `Vehicle` パイプラインを生成し、カメラ・コントローラ・AI モデルなどのパーツを追加。モードに応じて「手動」「自動」「ハイブリッド」を切り替え、走行ログを `TubWriter` で保存します。
2. カメラ・オドメトリ・IMU・モーターなど個別ハードウェアを初期化し `Vehicle` に登録します。
3. `docopt` で CLI 引数を解析し、`drive()` または `train()` を実行します。

---

## 内部クラス一覧

| クラス                  | 役割                         |
| -------------------- | -------------------------- |
| `DriveMode`          | 手動／自動モードに応じたステアリング・スロットル計算 |
| `LedConditionLogic`  | 走行状態に応じた RGB LED 制御        |
| `RecordTracker`      | 録画数を監視しアラートを生成             |
| `ToggleRecording`    | 記録開始・停止を管理                 |
| `UserPilotCondition` | 手動／自動モード切替と UI 表示画像の選択     |
| `Vectorizer`         | IMU データを 6 次元ベクトルに変換       |

---

## 各クラスの詳細

### DriveMode

手動／自動／ハイブリッド（ステアリングのみ自動）各モードでステアリング・スロットル出力を切り替えるパーツ。

```python
class DriveMode:
    def __init__(self, ai_throttle_mult: float = 1.0): ...

    def run(self,
            mode: str,
            user_steering: float,
            user_throttle: float,
            pilot_steering: float,
            pilot_throttle: float):
        ...
```

| `mode` 値               | ステアリング | スロットル                   |
| ---------------------- | ------ | ----------------------- |
| `user`                 | ユーザー入力 | ユーザー入力                  |
| `local_angle`          | AI     | ユーザー入力                  |
| その他 (`pilot`, `local`) | AI     | AI × `ai_throttle_mult` |

---

### LedConditionLogic

走行モード・記録状態・行動モデル状態など複数の情報をもとに LED の色と点滅周期を決定します。

```python
# 戻り値: 0=消灯, -1=常時点灯, >0=点滅周期[s]
blink_rate = LedConditionLogic(cfg).run(
    mode,
    recording,
    recording_alert,
    behavior_state,
    model_file_changed,
    track_loc
)
```

- 軌道追従 (`track_loc`) が有効なら位置に応じた色を点灯。
- モデル更新時や記録アラート時は短時間ブリンク。
- **戻り値** は点滅周期（秒）。`0` → 消灯、`-1` → 常時点灯。

---

### RecordTracker

Tub Writer の記録件数を監視し、節目ごとにコンソール出力や LED アラートを行います。

- 10 件ごとに進捗を `print()`。
- `cfg.REC_COUNT_ALERT` の倍数ごとに `cfg.REC_COUNT_ALERT_CYC` 秒間点滅。
- `force_alert` が立つと次回 `run` で即座にアラート発動。

---

### ToggleRecording

自動／手動モードやユーザー操作に応じて録画フラグをオン／オフします。

```python
rec_part = ToggleRecording(
    auto_record_on_throttle=True,
    record_in_autopilot=False
)
```

- `set_recording(state)` … 次フレームで強制設定。
- `toggle_recording()` … 次フレームで反転。
- `run(mode, recording)` … 内部状態を更新し返します。

---

### UserPilotCondition

Web UI に送る画像を「ユーザー」「パイロット(AI)」のどちらにするか選択し、各パーツの実行可否を返します。

```python
run_user, run_pilot, ui_img = UserPilotCondition(
    show_pilot_image=True
).run(
    mode,
    user_image,
    pilot_image
)
```

---

### Vectorizer

```python
class Vectorizer:
    def run(self, *components):
        return components
```

IMU センサー 6 軸出力（加速度 3 + 角速度 3）を 1 つのタプルにまとめ、AI モデルへ渡します。

---

## 参考

- **ソース**: [`donkeycar/templates/complete.py`](../../donkeycar/templates/complete.py)
- **利用ライブラリ**: `docopt`, `numpy`, `keras`, `pytorch` ほか
- 詳細なハードウェア設定については [公式ドキュメント](https://docs.donkeycar.com) を参照してください。
