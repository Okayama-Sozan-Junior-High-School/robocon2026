# robocon2026

ロボコン向けのロボット制御プログラムです。走行は Raspberry Pi、サーボの一部は Arduino で動かします。

## どのファイルを使うか

| やりたいこと | 使うファイル |
| --- | --- |
| いまの機体をキーボードで動かす | [`mecanum/mecanamWithKeyboard6.py`](mecanum/mecanamWithKeyboard6.py) |
| 走行だけの動作確認（キー入力なし） | [`mecanum/mecanam.py`](mecanum/mecanam.py) |
| Raspberry Pi から Arduino のサーボを動かす | [`pi-arduino-servo/`](pi-arduino-servo/) |
| サーボを Pi の GPIO に直結して動かす | [`servoWithKeyboard/servoWithKeyboard.py`](servoWithKeyboard/servoWithKeyboard.py) |
| ボタン1つでサーボ2個を開閉する（Arduino 単体） | [`twoServoMotor/`](twoServoMotor/) |

`mecanum/` の番号付きファイルは試作の履歴です。番号が大きいほど機構が増えていて、ピン配置も途中で変わっています。実機で使うのは **6** です。

## ディレクトリ一覧

```
.
├── mecanum/             Raspberry Pi でメカナムホイールと機構を動かす
├── pi-arduino-servo/    Pi から USB シリアルで Arduino のサーボを動かす（現行）
├── serialServo/         上と同じ構成の、より単純な前の版
├── servoWithKeyboard/   Pi の GPIO にサーボを直結し、矢印キーで動かす
└── twoServoMotor/       Arduino だけでサーボを開閉する
```

## mecanum/

Raspberry Pi の GPIO（`gpiozero`）で4輪のメカナムを動かします。キーボード操作の版は `pygame` で小さなウィンドウを出し、押しているキーを画面に表示します。`mecanamWithKeyboard.py` だけは `keyboard` ライブラリを使い、ウィンドウはありません。

| ファイル | 内容 |
| --- | --- |
| [`mecanam.py`](mecanum/mecanam.py) | キー入力なし。前進してから右へ横移動し、停止する動作確認 |
| [`mecanamWithKeyboard.py`](mecanum/mecanamWithKeyboard.py) | `keyboard` で走行のみ（W A S D / Q E） |
| [`mecanamWithKeyboard2.py`](mecanum/mecanamWithKeyboard2.py) | 同じ走行を `pygame` のウィンドウで操作する |
| [`mecanamWithKeyboard3.py`](mecanum/mecanamWithKeyboard3.py) | 走行にローラーを追加（`1` 正転 / `2` 逆転） |
| [`mecanumWithKeyboard4.py`](mecanum/mecanumWithKeyboard4.py) | ローラーに加え、プッシャーを2系統追加（`3`〜`6`） |
| [`mecanamWithKeyboard5.py`](mecanum/mecanamWithKeyboard5.py) | 4 の機構に、Pi 直結サーボ（`7` / `8`）と Arduino サーボ（`9`）を追加。ピンは旧配置 |
| [`mecanamWithKeyboard6.py`](mecanum/mecanamWithKeyboard6.py) | いまのピン配置。ローラー、プッシャー、スライド、Arduino サーボ |

ファイル名は `mecanum`（ホイールの綴り）と `mecanam` が混在しています。4 だけ `mecanumWithKeyboard4.py` です。

### キー（`mecanamWithKeyboard6.py`）

押しているあいだ動き、離すと停止します。`9` だけは押すたびに角度が切り替わります。終了は `Esc` かウィンドウを閉じます。

| キー | 動き |
| --- | --- |
| `W` / `S` | 前進 / 後退 |
| `A` / `D` | 左へ横移動 / 右へ横移動 |
| `Q` / `E` | 左回転 / 右回転 |
| `1` / `2` | ローラー正転 / 逆転 |
| `3` / `4` | プッシャー前進 / 後退 |
| `5` / `6` | スライド前進 / 後退 |
| `9` | Arduino のサーボを 0° と 90° で切り替え |

`5` 以前のキーボード版も、走行の W A S D / Q E は同じです。`5` では `5` / `6` が2本目のプッシャー、`7` / `8` が Pi 上のサーボ開閉です。

### ピン（`mecanamWithKeyboard6.py`）

各モーターは `gpiozero.Motor` で、前進用と後退用の2ピンを使います。`5` 以前は別の割り当てなので、動かすファイル先頭の `Motor(...)` を見てください。

| 役割 | 変数 | forward | backward |
| --- | --- | --- | --- |
| 左前輪 | `fl` | 4 | 17 |
| 右前輪 | `fr` | 18 | 23 |
| 左後輪 | `bl` | 27 | 22 |
| 右後輪 | `br` | 24 | 25 |
| ローラー | `ru` | 19 | 26 |
| プッシャー | `ps` | 13 | 12 |
| スライド | `sl` | 5 | 6 |

`9` キーのサーボは Pi の GPIO ではなく、USB でつないだ Arduino の D9 です。スケッチは [`pi-arduino-servo`](pi-arduino-servo/) のものを書き込んでおきます。Arduino が無いときは走行と機構だけ動き、起動時に「Arduino サーボ未接続」と出ます。

### 動かし方

Raspberry Pi のターミナルで、このリポジトリのルートから実行します。

```bash
python mecanum/mecanamWithKeyboard6.py
```

`gpiozero` と `pygame` が必要です。`mecanamWithKeyboard.py` だけ追加で `keyboard` パッケージが必要です。Arduino サーボを使う `5` と `6` は、さらに `pyserial` が必要です（[`pi-arduino-servo/requirements.txt`](pi-arduino-servo/requirements.txt)）。

## pi-arduino-servo/

Raspberry Pi から USB シリアルで、Arduino のデジタルピン 9 につないだサーボを動かします。`mecanum` の `5` と `6` も、ここの `servo_control.py` を読み込んで同じ Arduino につなぎます。

| ファイル | 内容 |
| --- | --- |
| [`arduino/servo_serial/servo_serial.ino`](pi-arduino-servo/arduino/servo_serial/servo_serial.ino) | Arduino 側。改行区切りで角度を受け取り、D9 のサーボを動かす |
| [`servo_control.py`](pi-arduino-servo/servo_control.py) | Pi 側。ポートを自動検出してコマンドを送る |
| [`requirements.txt`](pi-arduino-servo/requirements.txt) | Python 依存（`pyserial`） |

通信速度は 8080 baud です。サーボの信号線は D9、GND は Arduino と電源で共通にします。大きなサーボは Arduino の 5V から給電せず、別電源を使います。

受け取れるコマンドは次のとおりです。

| 送る文字列 | 動き |
| --- | --- |
| `90` や `a:45` | その角度（0〜180）へ動く |
| `center` | 90° |
| `sweep` | 0° から 180° まで往復する |

```bash
pip install -r pi-arduino-servo/requirements.txt
python pi-arduino-servo/servo_control.py          # 対話モード
python pi-arduino-servo/servo_control.py 90       # 90°
python pi-arduino-servo/servo_control.py sweep    # 往復
```

ポートを指定するときは `--port /dev/ttyACM0` を付けます。

## serialServo/

`pi-arduino-servo/` と同じ「Pi → USB → Arduino → サーボ」の、より前の版です。新しい作業は `pi-arduino-servo/` を使ってください。

| ファイル | 内容 |
| --- | --- |
| [`aruduino.ino`](serialServo/aruduino.ino) | Arduino 側。D9 のサーボを 9600 baud で動かす。ファイル名は `aruduino` |
| [`python.py`](serialServo/python.py) | `/dev/ttyACM0` に固定接続し、角度（0〜180）を入力させる |

`center` や `sweep` は無く、角度の数値だけを送ります。

## servoWithKeyboard/

サーボを Arduino を介さず、Raspberry Pi の GPIO 17 に直結します。`pygame` の小さなウィンドウを前面にした状態で、左右の矢印キーを押すと 10° ずつ動きます（0〜180）。起動時は 90° です。ウィンドウを閉じると終了します。

| ファイル | 内容 |
| --- | --- |
| [`servoWithKeyboard.py`](servoWithKeyboard/servoWithKeyboard.py) | GPIO 17 のサーボを矢印キーで動かす |

```bash
python servoWithKeyboard/servoWithKeyboard.py
```

## twoServoMotor/

Arduino 単体のサーボ開閉です。配線・部品・書き込み手順は [`twoServoMotor/README.md`](twoServoMotor/README.md) にあります。

| ファイル | 内容 |
| --- | --- |
| [`two_servo_motor.ino`](twoServoMotor/two_servo_motor.ino) | ボタン（ピン 2）を押すたびに、サーボ2個（ピン 9 と 10）を 0° と 90° で切り替える |
| [`sugar.ino`](twoServoMotor/sugar.ino) | サーボ1個（ピン 9）を、同じボタンで 0° と 90° に切り替える |
| [`sugar793.ino`](twoServoMotor/sugar793.ino) | ボタンの代わりにシリアルへ `b` を送ると、サーボ1個を 0° と 150° で切り替える |
| [`two_servo_motor.fzz`](twoServoMotor/two_servo_motor.fzz) | Fritzing の回路データ |
| [`schematic.png`](twoServoMotor/schematic.png) | 回路図 |

`two_servo_motor.ino` がこのディレクトリの本体です。`sugar.ino` と `sugar793.ino` はサーボ1個の別案です。

## その他

- [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) — コントリビューター行動規範
