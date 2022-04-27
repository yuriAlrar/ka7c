# Globally and Locally Consistent Image Completion

このリポジトリはneka-nat氏のリポジトリ(https://github.com/neka-nat/image_completion_tf2)
を改修したものです。

> This is a Tensorflow2 Keras implementation of ["Globally and Locally Consistent Image Completion"](http://hi.cs.waseda.ac.jp/%7Eiizuka/projects/completion/data/completion_sig2017.pdf).

## 動作環境
- Hardware
  - CPU: AMD Ryzen 1700
  - Men: 64GB(DDR4)
  - GPU: AMD Radeon RX480
- Ubuntu 20.04.3 LTS (kernel: 5.4.0-53-generic)
- ROCm version: 3.5.030
- python: 3.6.15
  - Tensorflow: 2.2.0

※ROCm...CUDAのAMD版

## pythonライブラリのインストール

```
pip install pipenv
```

## Prepare dataset

各ディレクトリの役割は以下です。
### data
学習に使用するデータセットです。
```
data
└── place365
    ├── Places365_00000001.jpg
    ├── Places365_00000002.jpg
    ├── Places365_00000003.jpg
    ├── Places365_00000004.jpg
    ├── Places365_00000005.jpg
    ├── Places365_00000006.jpg
    ├── Places365_00000007.jpg
    ├── Places365_00000008.jpg
    ...
```
### test
trainにおいては使用しません。

### output
各ステップでの計算終了時点において、画像とマスクを引数として
補完を実施した結果の画像が出力されます。

### checkpoint
各ステップでの計算終了時点にけるkerasモデル(.h5)をダンプします。

このフォルダにモデルデータがある状態でtrain.pyを実行した場合、
最新の.h5ファイルをロードして計算を再開します。

## 学習
dataフォルダに学習に用いる画像を入れた状態で以下を実行します。
```
python train.py
```

## 基本的な動き
重要そうな箇所、特にアルゴリズムについて説明します。
### GANの大雑把なアルゴリズム
偽物かを判定するモデル(D)と、偽物を生成するモデル(G)を双方学習させて最終的に高精度な偽物を生成するモデルを作る手法をGANといいます。

GLCICもGANの一種なので、DとGのネットワークを学習します。
学習は以下の流れで実施します。
1. Gモデルを学習(Dモデルは学習しない) -> line:138
2. Dモデルを学習(この間Gモデルは学習しない) -> line:141-143
3. G,Dモデルを学習 -> line:145

GLCICの特徴として画像をlocal_sizeに分割して…なんかする…忘れた。

line:164-179で2ステップごとにモデルを保存しています。

## 既知の不具合、改良すべき点
- 計算をしているとたまに止まる
  現状はチェックポイント機能の実装+再起動で対応
  ハードウェア/ROCm側の問題の可能性はありえる。
- 画像を隠すマスクサイズと形状が固定、かつランダム
  長方形の小さいマスクにした時どの程度精度が出るかによって追加学習の程度を考えなければならない。
  -> マスクサイズを可変にして出力できるようプログラムを改変する必要ある(line:38 DataGenerator.flow?)。

## Test

```
pipenv shell
python test.py
```

## Result

こんなところには表示できない。
