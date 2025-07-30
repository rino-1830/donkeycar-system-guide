# 変更履歴・メモ

### donkey:userinstall

- WSL2操作

    ```powershell
    # Powershellで実行
    
    # WSL2を更新
    wsl --update
    
    # WSL2にUbuntu24.04を導入
    wsl --install ubuntu:24.04
    
    # WSL2のデフォルトに設定されたUbuntu24.04を起動
    wsl
    ```

    ユーザ名は`usr`、パスは`pass`

- docker操作

    ```bash
    # WSL2(Ubuntu)で実行
    
    # イメージ取得
    docker pull ubuntu:24.04
    
    # コンテナ作成
    docker run -it --name donkey ubuntu:24.04 /bin/bash
    
    # コンテナ一覧
    docker ps -a
    
    # コンテナ再開
    docker start donkey
    
    # コンテナ再起動
    docker restart donkey
    ```

    コンテナ・イメージの削除はDocker Desktopから行うと安心。

    作業ディレクトリはWSL2(Ubuntu)にプロジェクトフォルダを作成してそこに保存すること。

    コンテナの停止は`exit`で大丈夫。

- ユーザ変更

    デフォルトユーザである`ubuntu` を以下のように変更。

    [Ubuntu 24.04 docker images now includes user ubuntu with UID/GID 1000?](https://askubuntu.com/questions/1513927/ubuntu-24-04-docker-images-now-includes-user-ubuntu-with-uid-gid-1000)

    **ユーザ名**

    ubuntu → donkey

    ```bash
    # rootで実行
    
    # 現在のユーザー名とUIDを確認
    id ubuntu
    
    # ユーザー名を変更
    usermod -l donkey ubuntu
    
    # ホームディレクトリの名前を変更
    usermod -d /home/donkey -m donkey
    
    # グループ名を変更
    groupmod -n donkey ubuntu
    
    # 設定ファイルを確認
    # /etc/passwd, /etc/group, /etc/shadow などのファイルが正しく更新されたことを確認します。
    cat /etc/passwd | grep newuser
    cat /etc/group | grep newuser
    ```

    ```bash
    # WSL2(Ubuntu)で実行
    docker restart donkey
    ```

    **パスワード**

    パスワード：pass

    ```bash
    # rootで実行
    
    # パスワードを変更
    passwd donkey
    ```

    **sudoを利用可能に**

    あまりガイドにないことはやりたくなかった。

    せめてこれ以降必要なパッケージはガイドに倣って`apt-get` で導入することにする。

    [Docker上に構築したLINUX(Debian)で「bash: sudo: command not found」エラーが出る - Qiita](https://qiita.com/TakuyaMiyashita/items/46542a2b94f86a7bc740)

    ```bash
    # rootで実行
    
    # sudoを導入
    apt update
    apt install sudo -y
    ```

- ユーザログイン

    ```bash
    # rootで実行
    
    login donkey
    ```

- コンテナOS更新（無意味だった）

    ```bash
    # donkeyで実行
    
    sudo apt update
    sudo apt dist-upgrade
    sudo apt autoremove
    ```

- 追加パッケージ関連

    ```bash
    # donkeyで実行
    
    sudo apt-get install wget
    
    # 必要な奴
    echo "LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6" >> ~/.bashrc
    source ~/.bashrc
    ```

- miniconda関係

    謎選択肢はデフォの[no]を選択（Enter押しただけ）。

    パスは以下を`bashrc`に追記。

    ```bash
    echo "export PATH=~/miniconda3/bin:$PATH" >> ~/.bashrc
    echo "source ~/miniconda3/etc/profile.d/conda.sh" >> ~/.bashrc
    source ~/.bashrc
    ```

    仮想環境はUser Installの方で実行。

    chmod 777 ~/projects/DonkeySimLinux/donkey_sim.x86_64

- bashrc

    ```bash
    # from donkeycar installation guide
    LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
    
    # miniconda path
    export PATH=~/miniconda3/bin:$PATH
    source ~/miniconda3/etc/profile.d/conda.sh
    ```

```bash
# WSL

docker commit donkey donkey:userinstall
```

- 構成の変更について

    **現状の課題**

  - JetsonNanoによる安定した制御が不可能

    **新規構成**

    1. 専用モデルを用いたシミュレーションでモデルを作成

        別案：TT02のマニュアル操作でデータを作成
        ⇒出力層を削除する必要性

    2. PIUSのマニュアル操作データを使って転移学習
    3. JetsonNanoで入力画像から制御量を推論
    4. 推論結果をシリアル通信でRaspberryPiに転送し、PIUSを制御

    **新規の課題**

  - シミュレーションの利用
    - `donkey:userinstall`が正常動作するか
    - 専用モデルの使用可否
  - 転移学習用のプログラム改変
  - RaspberryPiとのシリアル通信
    - 送信側GUIで操作したデータをリスト型として送信
    - カメラの画像取得に合わせ情報を伝達
      - 精度要検証

    **ToDo**

  - [x]  シリアル通信可否の確認
    - [ ]  コードの作成
    - [x]  オシロスコープ
  - [ ]  シミュレータの検証
    - [ ]  モデル
    - [x]  バージョン
  - [ ]  転移学習用プログラムの改変
    - [ ]  インターフェース
    - [ ]  学習層
    - [ ]  出力層
  - [x]  操作機能の変更
    - [x]  manage.py
    - [x]  train.py
    - [ ]  meta.json

    **代替案**

  - 学習・推論ソフトを作り直し、シミュ⇒PIUSで行う
  - DonkeySimで得たモデルを転移学習させるソフトを作成
  - TT02のマニュアル走行で得たモデルを転移学習させるソフトを作成
- シミュレータ用環境構築

  ### Python3.6.9

    ```bash
    sudo apt-get update
    sudo apt install -y python3-pip
    sudo apt-get -y install libmtdev1 libgl1 xclip
    
    echo "LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6" >> ~/.bashrc
    source ~/.bashrc
    
    mkdir -p ~/miniconda3
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
    bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
    rm ~/miniconda3/miniconda.sh
    
    echo "export PATH=~/miniconda3/bin:\$PATH" >> ~/.bashrc
    echo "source ~/miniconda3/etc/profile.d/conda.sh" >> ~/.bashrc
    source ~/.bashrc
    
    conda create -n donkey python=3.6.9
    conda activate donkey
    
    mkdir projects
    cd ~/projects
    git clone https://github.com/autorope/donkeycar
    cd ~/projects/donkeycar
    git checkout release_4_5
    
    pip install opencv-python-headless==4.6.0.66
    pip install -e .[pc]
    
    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb -O ~/cuda.deb
    sudo dpkg -i ~/cuda.deb
    rm ~/cuda.deb
    sudo apt-get update
    sudo apt-get -y install cuda-toolkit-12-6
    
    sudo apt-get install -y nvidia-open
    
    cd ~/projects
    sudo apt-get install -y unzip
    wget https://github.com/tawnkramer/gym-donkeycar/releases/download/v22.03.24/DonkeySimLinux.zip
    unzip DonkeySimLinux.zip
    rm DonkeySimLinux.zip
    chmod +x ~/projects/DonkeySimLinux/donkey_sim.x86_64
    
    cd ~/projects
    git clone https://github.com/tawnkramer/gym-donkeycar
    cd ~/projects/gym-donkeycar
    git checkout refs/tags/v22.03.24
    pip install -e .[gym-donkeycar]
    
    donkey createcar --path ~/mysim
    cd ~/mysim
    nano myconfig.py
    ```

    ```bash
    DONKEY_GYM = True
    DONKEY_SIM_PATH = "~/projects/DonkeySimLinux/donkey_sim.x86_64"
    DONKEY_GYM_ENV_NAME = "donkey-generated-track-v0"
    ```

  ### Python3.7

    ```bash
    sudo apt-get update
    sudo apt install -y python3-pip
    sudo apt-get -y install libmtdev1 libgl1 xclip
    
    echo "LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6" >> ~/.bashrc
    source ~/.bashrc
    
    mkdir -p ~/miniconda3
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
    bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
    rm ~/miniconda3/miniconda.sh
    
    echo "export PATH=~/miniconda3/bin:\$PATH" >> ~/.bashrc
    echo "source ~/miniconda3/etc/profile.d/conda.sh" >> ~/.bashrc
    source ~/.bashrc
    
    conda create -n donkey-py37 python=3.7
    conda activate donkey-py37
    
    mkdir projects
    cd ~/projects
    git clone https://github.com/autorope/donkeycar
    cd ~/projects/donkeycar
    git checkout release_4_5
    pip install -e .[pc]
    
    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb -O ~/cuda.deb
    sudo dpkg -i ~/cuda.deb
    rm ~/cuda.deb
    sudo apt-get update
    sudo apt-get -y install cuda-toolkit-12-6
    
    sudo apt-get install -y nvidia-open
    
    cd ~/projects
    sudo apt-get install -y unzip
    wget https://github.com/tawnkramer/gym-donkeycar/releases/download/v22.11.06/DonkeySimLinux.zip
    unzip DonkeySimLinux.zip
    rm DonkeySimLinux.zip
    chmod +x ~/projects/DonkeySimLinux/donkey_sim.x86_64
    
    cd ~/projects
    pip install git+https://github.com/tawnkramer/gym-donkeycar
    
    donkey createcar --path ~/mysim-py37
    cd ~/mysim-py37
    nano myconfig.py
    ```

  ### Ubuntu24.04 Python3.7

    ```bash
    # WSL用設定
    sudo apt-get update
    sudo apt-get -y upgrade
    sudo apt-get -y install libmtdev1 libgl1 xclip
    sudo apt-get clean
    sudo apt-get autoremove
    
    echo "LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6" >> ~/.bashrc
    source ~/.bashrc
    ```

    ```bash
    # minicondaのインストール
    mkdir -p ~/miniconda3
    wget https://repo.anaconda.com/miniconda/Miniconda3-py37_23.1.0-1-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
    bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
    rm ~/miniconda3/miniconda.sh
    
    echo "export PATH=~/miniconda3/bin:\$PATH" >> ~/.bashrc
    echo "source ~/miniconda3/etc/profile.d/conda.sh" >> ~/.bashrc
    source ~/.bashrc
    
    conda create -n donkey python=3.7 -y
    conda activate donkey
    ```

    ```bash
    # donkeycar4.5.1のインストール
    mkdir projects
    cd ~/projects
    git clone https://github.com/autorope/donkeycar
    cd ~/projects/donkeycar
    git checkout release_4_5
    pip install -e .[pc]
    ```

  - 実行結果一部抜粋

        ```bash
        Successfully installed Kivy-Garden-0.1.5 PrettyTable-3.7.0 PyWavelets-1.3.0 albumentations-1.3.1 charset-normalizer-3.4.1 cycler-0.11.0 docopt-0.6.2 docutils-0.20.1 donkeycar-4.5.1 fonttools-4.38.0 h5py-3.8.0 idna-3.10 imageio-2.31.2 importlib-metadata-6.7.0 joblib-1.3.2 kivy-2.3.0 kiwisolver-1.4.5 matplotlib-3.5.3 networkx-2.6.3 numpy-1.19.0 opencv-python-headless-4.11.0.86 packaging-24.0 paho-mqtt-2.1.0 pandas-1.3.5 pillow-9.5.0 plotly-5.18.0 progress-1.6 protobuf-4.24.4 psutil-6.1.1 pyfiglet-0.8.post1 pygments-2.17.2 pynmea2-1.19.0 pyparsing-3.1.4 pyserial-3.5 python-dateutil-2.9.0.post0 pytz-2024.2 pyyaml-6.0.1 qudida-0.0.4 requests-2.31.0 scikit-image-0.19.3 scikit-learn-1.0.2 scipy-1.7.3 simple_pid-2.0.1 six-1.17.0 tenacity-8.2.3 threadpoolctl-3.1.0 tifffile-2021.11.2 tornado-6.2 typing_extensions-4.7.1 urllib3-2.0.7 utm-0.7.0 wcwidth-0.2.13 zipp-3.15.0
        ```

    ```bash
    # WSL用cudaのインストール（https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network）
    wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb -O ~/cuda.deb
    sudo dpkg -i ~/cuda.deb
    rm ~/cuda.deb
    sudo apt-get update
    sudo apt-get -y install cuda-toolkit-12-6
    ```

    ```bash
    # donkeygymの実行バイナリを取得
    cd ~/projects
    sudo apt-get install -y unzip
    wget https://github.com/tawnkramer/gym-donkeycar/releases/download/v22.11.06/DonkeySimLinux.zip
    unzip DonkeySimLinux.zip
    rm DonkeySimLinux.zip
    chmod +x ~/projects/DonkeySimLinux/donkey_sim.x86_64
    ```

    ```bash
    # donkeygymをインストール
    cd ~/projects
    pip install git+https://github.com/tawnkramer/gym-donkeycar
    ```

    ```bash
    # donkeycarのインスタンスを作成
    donkey createcar --path ~/mysim
    cd ~/mysim
    nano myconfig.py
    ```

    ```bash
    ONKEY_GYM = True
    DONKEY_SIM_PATH = "~/projects/DonkeySimLinux/donkey_sim.x86_64"
    DONKEY_GYM_ENV_NAME = "donkey-generated-track-v0"
    ```
