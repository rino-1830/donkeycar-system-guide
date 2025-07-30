# 非 WSL 環境での DonkeyCar v4.5.1 環境構築

このドキュメントでは、非 WSL 環境で DonkeyCar v4.5.1 を扱える環境を構築するための方法について説明します。  
非 WSL 環境をターゲットに記述していますが、必要箇所を読み替えれば WSL 環境でも同様に v4.5.1 環境を構築可能です。  
WSL 環境での構築については、別途 [こちら](https://github.com/rino-1830/donkeydocs-jp/blob/main/docs/guide/host_pc/setup_ubuntu.md) の内容について理解する必要があります。

## 目的

公式ドキュメントでは WSL を利用した環境構築を推奨していますが、[DonkeySim](https://github.com/tawnkramer/gym-donkeycar) を利用する場合や使用するイメージに対応した `apt` リポジトリによっては公式ドキュメント通りでは不具合が発生することが確認されています。  
そのため、下記手段に則り、DonkeyCar v4.5.1 をネイティブな Windows 上に構築することでそれらの問題を回避することができます。

> [!CAUTION]
> 本ドキュメントでは、ホスト OS として Windows 11 x64 を想定しています。32bit 版や Windows 10 以下の場合は適宜読み替えてください。

## 環境構築

> [!NOTE]
> Miniconda とは、Anaconda と呼ばれる Python 仮想環境ソフトウェアのミニマルバージョンです。  
> Python 3.6 が実行できる環境であれば Miniconda ではなく venv でも構いませんが、ここでは Python とパッケージの安定性の観点から Miniconda を推奨します。

### Miniconda の導入

[このリンク](https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86.exe) から最新の Windows 用 Miniconda のインストーラーを入手します。  
入手したプログラムを実行し、任意のディレクトリにインストールします。

### Miniconda による仮想環境構築

Miniconda がインストール出来たら、スタートメニューから **Anaconda Prompt** を起動します。  
起動すると、次のようなプロンプトが表示されます。

```bash
(base) C:\Users\user>
```

まずは DonkeyCar 用の仮想環境を作成します。  
下記コマンドを実行することで、Python 3.6.13（ドキュメント執筆当時）がインストールされた仮想環境が作成・起動されます。  
このとき、既に同名の仮想環境が存在し共存させたい場合、`donkey` に当たる箇所を任意の仮想環境名に読み替えてください。

```bash
(base) C:\Users\user> conda create -n donkey python=3.6
(base) C:\Users\user> conda activate donkey
(donkey) C:\Users\user>
```

公式ドキュメントには既に v5.1.0 未満のサポートについて情報が存在しないため Python 3.11 を推奨していますが、その通りに環境を構築すると v4.5.1 の環境は作成できません。必ず Python 3.6 を使用してください。

> [!NOTE]
> リポジトリの `setup.py` には Python 3.7 も依存バージョンに登録されていますが、`donkeycar` の依存パッケージが Python 3.7 に対応していないためこれは無意味です。

### リポジトリのクローンと `donkeycar` ライブラリのインストール

DonkeyCar は Python ライブラリとして構築され、プロンプトコマンドとして実行可能です。  
ライブラリは直接 Python 環境と紐づくため、Miniconda のような何らかの手段で仮想環境を作る必要があります。

このプロンプトでは最初から `git` コマンドが使用できるため、`git` コマンドを利用して DonkeyCar v4.5.1 のリポジトリを任意のディレクトリにクローンします。  
下記コマンドでは、`projects` ディレクトリを作成し、そこにリポジトリをクローンしています。  
既に同名のクローンが存在し共存させたい場合、コマンド末尾にスペースを空けて任意のローカルリポジトリ名を設定することができます。

```bash
(donkey) C:\Users\user> mkdir projects
(donkey) C:\Users\user> cd projects
(donkey) C:\Users\user\projects> git clone https://github.com/autorope/donkeycar -b release_4_5
(donkey) C:\Users\user\projects> cd donkeycar
(donkey) C:\Users\user\projects\donkeycar>
```

クローンしたリポジトリに移動し、`setup.py` を使った依存関係のインストールを行いますが、<span style="color: red">**GPU を使用する場合とそうでない場合で導入手順が変わります**</span>。

#### GPU を使用する場合

GPU を使用する場合、次のコマンドで `tensorflow==2.2.0` を導入します。

```bash
(donkey) C:\Users\user\projects\donkeycar> pip install -e .[tf]
```

> [!NOTE]
> Python 3.6 で利用可能な TensorFlow の最新バージョンはドキュメント執筆当時 2.6.2 ですが、依存関係の `six~=1.15.0`、`typing-extensions~=3.7.4` が `donkeycar` の依存関係と競合します。  
> そのため、`setup.py` に明記されたバージョンを使用する必要があります。

次に、下記リンクを参照し CUDA Toolkit 10.1 と cuDNN をインストールします。

- [CUDA Toolkit 10.1 Update 2 Archive](https://developer.nvidia.com/cuda-10.1-download-archive-update2)
- [cuDNN 9.11.0 Downloads](https://developer.nvidia.com/rdp/cudnn-download)

CUDA のインストールが完了した後、PowerShell で `nvcc --version` コマンドを実行することで CUDA が正常にインストールできたか確認します。

```powershell
> nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Sun_Jul_28_19:12:52_Pacific_Daylight_Time_2019
Cuda compilation tools, release 10.1, V10.1.243
```

> CUDA 11 以降を強引に使用する方法についての記述を削除しました。  
> 必要な方はコミット履歴を参照してください。

完了後は次のセクション **GPU を使用しない場合** に進みます。

#### GPU を使用しない場合

前述の **GPU を使用する場合** を実施した場合は、このセクションで先ほど導入したいくつかのパッケージを `donkeycar` の依存関係で上書きすることになりますが、正常な動作です。

以下のコマンドで、`opencv-python-headless==4.6.0.66` をインストールします。  
対応バージョン自体はより高いものがあるはずですが、不明な理由で wheel のビルドに失敗するためこのバージョン以外は使用できません。  
これはホスト PC に限らず、NVIDIA Jetson Nano

```bash
(donkey) C:\Users\user\projects\donkeycar> pip install opencv-python-headless==4.6.0.66
```

次に、以下のコマンドでホスト PC に必要な依存環境を導入します。

```bash
(donkey) C:\Users\user\projects\donkeycar> pip install -e .[pc]
```

念のため、`pip check` などで依存関係の破綻を確認しておきます。

正常に完了すると、ターミナルで `donkeycar` コマンドが利用可能になります。

## 終わりに

ここまでの手段により、非 WSL 環境および Miniconda を利用したホスト PC での DonkeyCar v4.5.1 の環境構築が完了しました。

当ドキュメントはあくまで筆者の環境で上手く動いた（動いている）例を再現するための実装なので、この環境でのみ発生する問題を `autorope/donkeycar` に報告しないでください。

ここに書かれた内容は DonkeyCar v4.5.1 についてのみ言及していますが、環境構築の骨組み自体は変わらず維持できるはずです。

### DonkeySim について

別のドキュメントで解説予定ですが、WSL で実行するとフレームレートが著しく低下することが判明しています。  
これは、WSL の描画ツールチェーンが余りに多段経由となり大量のフレームバッファのコピー／RDP 圧縮が狭まることに起因していると考えられるため、おそらく解決不可能な問題だと考えられます。  
そのため、Windows で DonkeySim を利用するには非 WSL 環境での構築が必要となるのです。

参考文献：

- [WSL2 glxgears performance (extremely slow)](https://forums.developer.nvidia.com/t/wsl2-glxgears-performance-extremely-slow/187906)
- [Extremely low rendering fps in wsl2 #1008](https://github.com/google-deepmind/mujoco/issues/1008)
