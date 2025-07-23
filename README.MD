# DonkeyCar v4.5.1 補足資料

このリポジトリは DonkeyCar v4.5.1 を対象とした、公式ドキュメントでは不足している情報をまとめたメモです。

## 目的

公式の英語ドキュメントを参照していて詰まりやすかった箇所や、実際に手を動かして得られた知見をまとめています。

## 対象バージョン

* DonkeyCar v4.5.1 のみを扱います
  他バージョンではディレクトリ構造や API が異なるため注意してください。

## 補足ドキュメント

* [docs/repo-structure.md](docs/repo-structure.md): v4.5.1 時点のリポジトリ構成解説
* [docs/driving-script.md](docs/driving-script.md): 走行スクリプトの基本構造とコマンド例
* [docs/donkey-cli.md](docs/donkey-cli.md): Donkey CLI コマンド一覧

## 主要コマンド

```bash
# 環境構築
conda activate donkey
donkey createcar --path ~/mycar

# モデル学習と走行
python manage.py train --tub ./data --model ./models/model.h5
python manage.py drive --model ./models/model.h5
```

実験で不要になった tub データは容量が大きいので適宜削除してください。

## 参考リンク

* 公式ドキュメント: [https://docs.donkeycar.com](https://docs.donkeycar.com)
* フォーラム: [https://www.donkeycar.com/community](https://www.donkeycar.com/community)

## ライセンス

MIT License
