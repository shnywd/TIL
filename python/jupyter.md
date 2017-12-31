# Jupyter notebook

### インストール

```
curl -O https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh
bash Anaconda3-5.0.1-Linux-x86_64.sh
```

ユーザーホーム以下にインストールされるので、パスを通します。

```
export PATH=/home/centos/anaconda3/bin:$PATH
```

### 仮想環境の作成

仮想環境を作成する

```
conda create -n jupyter-env python=3.6
source activate jupyter-env
```

作成した仮想環境にサードパーティ製パッケージをインストールする

```
$ conda install -y jupyter==1.0.0
$ conda install -y notebook==5.0.0
$ conda install -y pandas==0.19.2
$ conda install -y bokeh==0.12.5
$ conda install -y matplotlib==2.01
```

### リモートから接続できるように設定する

jupyter_notebook_config.py ファイルを生成する

```
jupyter notebook --generate-config
```

接続用パスワードのハッシュ値を生成する
```
In [1]: from IPython.lib import passwd

In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'sha1:a1c .... 1cc:c4ef9ef6   ..... 6d1e750'
```

```
vi ~/.jupyter/jupyter_notebook_config.py

c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 9999
c.NotebookApp.password = u'sha1:a1c .... 1cc:c4ef9ef6   ..... 6d1e750'
```

### 起動する

```
jupyter notebook
```
