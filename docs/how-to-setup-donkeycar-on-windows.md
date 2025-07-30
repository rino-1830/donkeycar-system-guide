# 非 WSL 環境での DonkeyCar v4.5.1 環境構築

このドキュメントでは、非 WSL 環境で DonkeyCar v4.5.1 を扱える環境を構築するための方法について説明します。  
非 WSL 環境をターゲットに記述していますが、必要箇所を読み替えれば WSL 環境でも同様に v4.5.1 環境を構築可能です。  
WSL 環境での構築については、別途 [こちら](https://github.com/rino-1830/donkeydocs-jp/blob/main/docs/guide/host_pc/setup_ubuntu.md) の内容について理解する必要があります。

## 目的

公式ドキュメントでは WSL を利用した環境構築を推奨していますが、[DonkeySim](https://github.com/tawnkramer/gym-donkeycar) を利用する場合や使用するイメージに対応した `apt` リポジトリによっては公式ドキュメント通りでは不具合が発生することが確認されています。  
そのため、下記手段に則り、DonkeyCar v4.5.1 をネイティブな Windows 上に構築することでそれらの問題を回避することができます。

> [!CAUTION]
> 本ドキュメントでは、ホスト OS として Windows 11 x64 を想定しています。32bit 版やWindows 10 以下の場合は適宜読み替えてください。

## 環境構築

> [!NOTE]
> Miniconda とは、Anaconda と呼ばれる Python 仮想環境ソフトウェアのミニマルバージョンです。  
> Python 3.6 が実行できる環境であれば Miniconda ではなく venv でも構いませんが、ここでは Python とパッケージの安定性の観点から Miniconda を推奨します。

### Miniconda の導入

[このリンク](https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86.exe) から最新の Windows 用 Miniconda のインストーラーを入手します。  
入手したプログラムを実行し、任意のディレクトリにインストールします。

### Miniconda による仮想環境構築

Miniconda がインストール出来たら、スタートメニューから Anaconda Prompt を起動します。  
起動すると、次のようなプロンプトが表示されます。

```bash
(base) C:\User\user>
```

まずは DonkeyCar 用の仮想環境を作成します。  
下記コマンドを実行することで、Python 3.6.13（ドキュメント執筆当時）がインストールされた仮想環境が作成・起動されます。  
このとき、既に同名の仮想環境が存在し共存させたい場合、`donkey` に当たる箇所を任意の仮想環境名に読み替えてください。

```bash
(base) C:\User\user>conda create -n donkey python=3.6
(base) C:\User\user>conda activate donkey
(donkey) C:User\user>
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
(donkey) C:\User\user> mkdir projects
(donkey) C:\User\user> cd projects
(donkey) C:\User\user\projects> git clone https://github.com/autorope/donkeycar -b release_4_5
(donkey) C:\User\user\projects> cd donkeycar
(donkey) C:\User\user\projects\donkeycar>
```

クローンしたリポジトリに移動し、`setup.py` を使った依存関係のインストールを行いますが、<span style="color: red">**GPU を使用する場合とそうでない場合で導入手順が変わります**</span>。

#### GPU を使用する場合

GPU を使用する場合、次のコマンドで `tensorflow==2.2.0` を導入します。  

```bash
(donkey) C:\User\user\projects\donkeycar> pip install -e .[tf]
```

> [!NOTE]
> Python 3.6 で利用可能な TensorFlow の最新バージョンはドキュメント執筆当時 2.6.2 ですが、依存関係の `six~=1.15.0`、`typing-extensions~=3.7.4` が `donkeycar` の依存関係と競合します。  
> そのため、`setup.py` に明記されたバージョンを使用する必要があります。

次に、下記リンクを参照し使用する GPU に合わせた CUDA Toolkit をインストールします。

- [CUDA GPU Compute Capability](https://developer.nvidia.com/cuda-gpus): GPU と CUDA の対応表
- [CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive): CUDA Toolkit のダウンロードリンクのリスト
  - ローカルインストーラーはかなり容量が大きいので、必要なくなったら削除することを推奨します。  

ドライバに対応している CUDA のバージョンは PowerShell の `nvidia-smi` コマンドでも確認できます。

```shell
> nvidia-smi
Thu Jul 31 00:04:18 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 577.00                 Driver Version: 577.00         CUDA Version: 12.9     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                  Driver-Model | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3060 Ti   WDDM  |   00000000:01:00.0  On |                  N/A |
| 30%   42C    P8             22W /  200W |    2138MiB /   8192MiB |      3%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

（後略）
```

CUDA Toolkit インストールが完了した後、PwerShell で `nvcc --version` コマンドを実行することで CUDA が正常にインストールできたか確認します。

```shell
> nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Tue_May_27_02:24:01_Pacific_Daylight_Time_2025
Cuda compilation tools, release 12.9, V12.9.86
Build cuda_12.9.r12.9/compiler.36037853_0
```

---

このとき、CUDA 11 以上を使用して居る場合は次の修正を行う必要があります。

参考文献：[cudart64_101.dllが見つからない問題](https://qiita.com/Radley/items/985454b4df7f86e743d9)

> [!CAUTION]
>この修正は TensorFlow 2.2.0 をどうにか最新の CUDA で利用するための処置なので、どのような不具合を誘発するか不明です。
>必ず自己責任で行ってください。

エクスプローラーなどで CUDA Toolkit のインストールディレクトリを開きます。  
デフォルトでは `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\<cuda-version>\bin` です。

このディレクトリ内の `cudart64_*.dll` を同じディレクトリ内にコピーし、コピーしたものを `cudart64_101.dll` にリネームします。

`<cuda-version>` や `*` などのプレースホルダーは適宜読み替えてください。

![`cudart64_*.dll` のリネーム](/assets/rename-cudart.png)

これにより、下記警告が発生せずに CUDA を利用することができます。

```bash
> (donkey) C:\User\user\mycar> python manage.py train --tubs=./data --model=./models/model.h5
cv2: 4.6.0
sklearn: 0.24.2
skimage: 0.17.2
________             ______                   _________
___  __ \_______________  /___________  __    __  ____/_____ ________
__  / / /  __ \_  __ \_  //_/  _ \_  / / /    _  /    _  __ `/_  ___/
_  /_/ // /_/ /  / / /  ,<  /  __/  /_/ /     / /___  / /_/ /_  /
/_____/ \____//_/ /_//_/|_| \___/_\__, /      \____/  \__,_/ /_/
                                 /____/

using donkey v4.5.1 ...
2025-07-30 23:07:05.146062: W tensorflow/stream_executor/platform/default/dso_loader.cc:55] Could not load dynamic library 'cudart64_101.dll'; dlerror: cudart64_101.dll not found
2025-07-30 23:07:05.146842: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
Usage:
    manage.py drive [--model=<model>]
    manage.py train [--tubs=<tub_paths>] (--model=<model>)
    [--transfer=<transfer_model>]
    [--comment=<comment>]

（後略）
```

完了後は次のセクション **GPU を使用しない場合** に進みます。

#### GPU を使用しない場合

前述の **GPU を使用する場合** を行った場合は、このセクションで先ほど導入したいくつかのパッケージを `donkeycar` の依存関係で上書きすることになりますが、正常な動作です。

以下のコマンドでホスト PC に必要な依存環境を導入します。

```bash
(donkey) C:\User\user\projects\donkeycar> pip install -e .[pc]
```

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
