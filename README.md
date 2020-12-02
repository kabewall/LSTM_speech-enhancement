# LSTM_speech-enhancement

音声強調モデル作成のためのPytorchプログラムです。

# 動作環境
Windows/Mac/Linuxいずれの環境でも動作確認済みです。

```
Python 3.7.9

torch ver 1.7.0
tqdm ver 5.0.5
librosa ver 0.8.0

GPU NVIDIA RTX2070 super
CUDA ver11.0
```
# プログラムについて

## parameter.py

データセットのパス指定やモデルのパラメーターまで、学習関連で人間が触っていいものは基本ここだけです。

強調したい音声のみのwavファイルを格納したディレクトリパスを`clean_speech_dir`に、
除去したい環境音や音楽などのwavファイルを格納したディレクトリパスを`clean_speech_dir`に指定してください。
wavファイルは再帰的に探索されるので、多階層なディレクトリ構造になってても大丈夫です。

この音声ファイルから学習に必要なデータに変換され、作成された`/datasets`ディレクトリに格納されます。
そこそこ大きなデータサイズになります。CMU Arctic CorpusとUrbanSound8k全データ使って合計14GBくらいのデータが作られます。

`num_layer`はLSTMモデルの階層サイズを設定します。値を大きくするほどモデルサイズと学習に使用されるGPUメモリサイズが大きくなります。その代わり精度が上がります。


## step0.py
学習に先立ちデータセットの前処理を行います。

## step1.py
学習を行います。
学習はGPUが利用できる環境であれば自動でGPUが選択されます。（`CUDA is available:, True`と表示されます）
そうでなければCPUで学習を実行します。詳しくはPytorchのドキュメントを参照してください。
GPUの使用状況はコマンドプロンプト及びターミナルから`nvidia-smi -l`で確認します。
GPU利用時、15000データによる学習でバッチサイズ50、Epoch数100でおよそ4時間ほどで計算は終了します。
実際にはEpoch数30以内でだいたい収束します。

- 保存されるモデルについて

モデルは自動で作られる`/model`ディレクトリ下にPTファイルで保存されます。モデルサイズはだいたい６MB程度が予想されます。
保存されるモデルは検証時にもっとも高精度なモデルであると予測される、`model_layer3_2020~日付~.pt`モデルと、
10エポックごとに自動保存される`model_layer3_2020~日付~_Epoch20.pt`があります。
これは過学習を防止するための途中離脱モードを積んでいないためこのような保存方法をとっています。
使用しないモデルは削除しても構いません。

- 学習終了後について

学習終了後、GPUはカーネルの再起動などによって各自で解放してください。

## step2.ipynb
実際に学習済みモデルを使用して音声強調を行います。
クイックな音声の確認を行うためJupyter notebookで書かれています。

`model_path`で使用したいモデルを指定します。
何度もデータセットのローディングを行うことを避けるため、作成したいろんなモデルの聞き比べを行いたい場合
後半のみを実行してください。


# 性能についてのメモ
## 実験１
音声データCMU Arctic コーパス
雑音データUrban Sound 8K

- 学習率0.002、Epoch=20程度でそこそこの精度が出ます。
- ですが、さすがに背景ノイズが大きすぎると強調しきれないようです。
