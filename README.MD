# DonkeyCar v4.5.1 補足資料

このリポジトリは DonkeyCar v4.5.1 を対象とした、公式ドキュメントでは不足している情報をまとめたメモです。  
当アカウントがここに示した当アカウントが著作権を有する全著作物は都立産業技術高等専門学校 吉田研究室に帰属します。

## 目的

公式の英語ドキュメントを参照していて詰まりやすかった箇所や、実際に手を動かして得られた知見をまとめています。

## 対象バージョン

* DonkeyCar v4.5.1 のみを扱います
  他バージョンではディレクトリ構造や API が異なるため注意してください。

## 補足ドキュメント

* [docs/repo-structure.md](docs/repo-structure.md): v4.5.1 時点のリポジトリ構成解説
* [docs/driving-script.md](docs/driving-script.md): 走行スクリプトの基本構造とコマンド例
* [docs/donkey-cli.md](docs/donkey-cli.md): Donkey CLI コマンド一覧
* [docs/teaching-data.md](docs/teaching-data.md): 教師データの構造など
* [docs/model-type.md](docs/model-type.md): 生成できるモデルの種類とその特性
* [docs/transfer-learning.md](docs/transfer-learning.md): 転移学習の実装状況（使用不可）
* [docs/fine-tuning.md](docs/fine-tuning.md): ファインチューニングの実装状況

---

* [donkeydocs-jp](https://github.com/rino-1830/donkeydocs-jp): donkeydocs の日本語訳版
* [donkeycar-for-pius](https://github.com/rino-1830/donkeycar-for-pius): DonkeyCar の PIUS の自律走行への転用など（開発用）
    * translate/release_4_5 で各ファイルの日本語訳と追加 Docstring が閲覧できます
* [DonkeyCar_for_PIUS](https://github.com/kazysdlab/DonkeyCar_for_PIUS): DonkeyCar で PIUS を扱うためのパッチとカスタムスクリプト
    * 実際に PIUS を自律走行させる場合はこちらを使用してください
* [donkey-teaching-data-merger](https://github.com/rino-1830/donkey-teaching-data-merger): 複数の端末から取得した教師データをまとめ、一度の学習に使えるようにするスクリプト

## その他のドキュメント

> **注意**: 基本的にこのセクションの情報が役立つことはないと思います。

* [misc/my-research-records](misc/my-research-records.md): 卒業研究中のログ（以下二つのリンクも中身は同一です）
    * [Notion 共有リンク](https://www.notion.so/17354b326cae80009a49e3ae585028cf?source=copy_link)
    * [Notion 公開リンク](https://boiled-helenium-092.notion.site/17354b326cae80009a49e3ae585028cf)

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

Released under the MIT license  
https://opensource.org/licenses/mit-license.php
