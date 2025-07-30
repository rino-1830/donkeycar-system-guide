# DonkeyCar v4.5.1 補足資料

![license](https://img.shields.io/badge/license-MIT-green)

このリポジトリは DonkeyCar v4.5.1 を対象とした、公式ドキュメントでは不足している情報をまとめたメモです。  
このドキュメントに示された [当アカウント](https://github.com/rino-1830) が著作権を有する全ての著作物は [都立産業技術高等専門学校 吉田研究室](https://github.com/kazysdlab) に帰属します。

## 目的

公式の英語ドキュメントを参照していて詰まりやすかった箇所や、実際に手を動かして得られた知見をまとめています。

## 対象バージョン

DonkeyCar v4.5.1 のみを扱います。  
他バージョンではディレクトリ構造や API が異なるため注意してください。

## 補足ドキュメント

| ファイル | 説明 |
| --- | --- |
| [docs/repo-structure.md](docs/repo-structure.md) | v4.5.1 時点のリポジトリ構成解説 |
| [docs/driving-script.md](docs/driving-script.md) | 走行スクリプトの基本構造とコマンド例 |
| [docs/donkey-cli.md](docs/donkey-cli.md) | Donkey CLI コマンド一覧 |
| [docs/teaching-data.md](docs/teaching-data.md) | 教師データの構造など |
| [docs/model-type.md](docs/model-type.md) | 生成できるモデルの種類とその特性 |
| [docs/transfer-learning.md](docs/transfer-learning.md) | 転移学習の実装状況（使用不可） |
| [docs/fine-tuning.md](docs/fine-tuning.md) | ファインチューニングの実装状況 |
| [docs/how-to-setup-pius.md](docs/how-to-setup-pius.md) | PIUS への各部品の組付けやシステムのセットアップ |
| [docs/how-to-setup-donkeycar-on-windows.md](docs/how-to-setup-donkeycar-on-windows.md) | 非 WSL 環境への DonkeyCar v4.5.1 の導入 |

## 関連リポジトリ

* **[donkeydocs-jp](https://github.com/rino-1830/donkeydocs-jp)**  
  公式 donkeydocs と公式サイトソースの日本語訳。公式サイトの全情報に加え、通常はリンク切れとなる資料も含まれています。

* **[donkeycar-for-pius](https://github.com/rino-1830/donkeycar-for-pius)**  
  DonkeyCar の PIUS 自律走行への転用（開発用）。  
  `translate/release_4_5` ブランチで日本語訳と追加 Docstring を閲覧できます。

* **[DonkeyCar_for_PIUS](https://github.com/kazysdlab/DonkeyCar_for_PIUS)**  
  PIUS を扱うためのパッチとカスタムスクリプト。  
  実際に PIUS を自律走行させる場合はこちらを使用してください。

* **[donkey-teaching-data-merger](https://github.com/rino-1830/donkey-teaching-data-merger)**  
  複数端末から取得した教師データをまとめ、一度の学習で使用可能にするスクリプト。  
  改善の余地が多いため、コード内容を理解してからの利用を推奨します。

## その他のドキュメント

> **NOTE**  
> このセクションの情報は直接役立たない場合が多い想定です。

* **[misc/my-research-records](misc/my-research-records.md)**  
  卒業研究のログ（以下リンク先はいずれも同内容）  
  * [Notion 共有リンク](https://www.notion.so/17354b326cae80009a49e3ae585028cf?source=copy_link)  
  * [Notion 公開リンク](https://boiled-helenium-092.notion.site/17354b326cae80009a49e3ae585028cf)

* **[misc/trouble-shooting.md](misc/trouble-shooting.md)**  
  雑多なトラブルシューティング集

* **[misc/terminal-logs.md](misc/terminal-logs.md)**  
  実行コマンドのログ・解説・分析

## 主要コマンド

パスについては適宜変更してください。

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

本リポジトリは MIT License で配布されます。詳細は LICENSE ファイルを参照してください。
