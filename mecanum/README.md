# mecanum

Raspberry Pi でメカナムホイールと、ローラー・プッシャー・スライドをキーボード操作するプログラムです。リポジトリ全体のファイル説明は [../README.md](../README.md) にあります。

実機で使うのは [`mecanamWithKeyboard6.py`](mecanamWithKeyboard6.py) です。それより番号が小さいファイルは、機構を足していった途中の版です。

| ファイル | 内容 |
| --- | --- |
| [`mecanam.py`](mecanam.py) | キー入力なし。前進してから右へ横移動し、停止する |
| [`mecanamWithKeyboard.py`](mecanamWithKeyboard.py) | `keyboard` ライブラリで走行だけ操作する。ウィンドウは出ない |
| [`mecanamWithKeyboard2.py`](mecanamWithKeyboard2.py) | 同じ走行を `pygame` のウィンドウで操作する |
| [`mecanamWithKeyboard3.py`](mecanamWithKeyboard3.py) | 走行にローラーを追加（`1` / `2`） |
| [`mecanumWithKeyboard4.py`](mecanumWithKeyboard4.py) | ローラーに加え、プッシャー2系統（`3`〜`6`） |
| [`mecanamWithKeyboard5.py`](mecanamWithKeyboard5.py) | 4 に Pi 直結サーボ（`7` / `8`）と Arduino サーボ（`9`）を追加。ピンは旧配置 |
| [`mecanamWithKeyboard6.py`](mecanamWithKeyboard6.py) | いまのピン配置。ローラー、プッシャー、スライド、Arduino サーボ |

`gpiozero` でモーターを動かし、キーボード版は `pygame` を使います（最初のキーボード版だけ `keyboard`）。Arduino サーボを使う `5` と `6` は [../pi-arduino-servo](../pi-arduino-servo/) のスケッチを Arduino に書き込み、`pyserial` が必要です。

## 動かし方

```bash
python mecanamWithKeyboard6.py
```

`5` 以前はピン割り当てが違うので、動かすファイルの先頭にある `Motor(...)` を確認してください。`6` のピンとキー一覧は [../README.md](../README.md) の mecanum の節にまとめてあります。
