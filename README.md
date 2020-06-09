# rtk_kernel_test

組み込みの勉強のためにT-Kernelをラズパイに入れたりしてみる

## 前準備

- 必要なもの
  - Raspberry Pi (versionごとに多少手順異なる。自分は3B)
  - Micro SD (4GBもあればいいとか。FATでフォーマットしておく)
  - USB-UART変換
  - PC (MacBook Pro (13-inch, 2017, Four Thunderbolt 3 Ports), macOS Catalina 10.15.5)
- 必要と思われるもの
  - モニタ: シリアル通信でPCのターミナル上に出力されるからいらない説

- ARMツールチェーンのインストール

  https://www.yokoweb.net/dokuwiki/develop/rtk_kernel/rtk_kernel-build-toolchain
  https://www.yokoweb.net/2018/05/16/macos-gcc-arm-brew-install/

  ``` sh
  $ brew update && brew upgrade
  $ brew tap ArmMbed/homebrew-formulae
  $ brew install arm-none-eabi-gcc
  ```

- 確認

  ``` sh
  $ arm-none-eabi-gcc --version
  $ which arm-none-eabi-gcc
  ```

- アンインストール

  ``` sh
  $ brew uninstall gcc-arm-none-eabi
  $ brew untap ArmMbed/homebrew-formulae
  ```

## ビルド、導入

https://www.yokoweb.net/dokuwiki/develop/rtk_kernel/rtk_kernel-rpi/execution/start

### clone

``` sh
$ cd 作業ディレクトリ
$ git clone git@github.com:jr4qpv/rtk_kernel.git
```

### 環境変数設定 fish shell 一時的設定

``` sh
set -x BD /Users/ユーザー/作業ディレクトリ/rtk_kernel/tkernel_source
set -x GNU_BD /usr/local/Cellar/arm-none-eabi-gcc/9-2019-q4-major/
```

### T-Monitor コンパイル

``` sh
cd $BD/monitor/tmmain/build/rpi_bcm283x.rpi3
make
```

生成されたtmonitor.binを利用

### Config情報 コンパイル

``` sh
cd $BD/config/build/rpi_bcm283x.rpi3
make
```

生成されたrominfo-rom.binを利用

### T-Kernel コンパイル

``` sh
cd $BD/kernel/sysmain/build/rpi_bcm283x.rpi3
make
```

生成されたkernel-rom.binを利用

https://www.yokoweb.net/2016/08/13/raspberrypi-uboot/

## 実行

https://www.yokoweb.net/dokuwiki/develop/rtk_kernel/rtk_kernel-rpi/execution/start

### u-boot関連ファイルの書き込み

> GitHub（https://github.com/jr4qpv/rpi_u-boot_jtag_bins）から、各Raspberry Piの機種用の下記4つのファイルを入手し、SDカードのルートに書き込む。詳しくは、関連記事(1.)を参照。
> - bootcode.bin
> - start.elf
> - u-boot.bin
> - config.txt

1. https://github.com/raspberrypi/firmware/tree/master/boot
   - bootcode.bin
   - start.elf

2. https://github.com/jr4qpv/rpi_u-boot_jtag_bins/blob/master/rpi_3_32b.zip
   解凍し、u_boot.binを入手

3. config.txtには以下を記入
   ```
   enable_uart=1
   kernel=u-boot.bin
   ```

### T-Kernel関連ファイルの書き込み

T-Kernel関連のファイルは、コンパイル手順で作成した下記3つのファイルを、SDカードに書き込む。

- tmonitor.bin
- rominfo-rom.bin
- kernel-rom.bin

### ここから先はUSB-UART接続(3.3V)必須

- USB-TTL GND(黒色) ⇔ ラズパイ GND   ( 6番ピン)
- USB-TTL RX (白色) ⇔ ラズパイ GPIO14( 8番ピン): UART_TXD
- USB-TTL TX (緑色) ⇔ ラズパイ GPIO15(10番ピン): UART_RXD
- 赤(3.3V)は接続不要 (microUSBで電源供給される)

### 接続

- COMポート一覧取得及び接続
  - ボーレート :115200
  - データ長 :8bit
  - パリティー :なし
  - ストップビット :1
  - フロー制御 :なし

  ``` sh
  $ ls -l /dev/tty.usb*
  $ screen /dev/tty.usbserial-______ 115200,cs8,-parenb,-cstopb
  ```

- 切断
  Control-a Control-k

- 中断
  Control-a Control-d

- 再開

  ``` sh
  $ screen -r
  ```

### U-bootでT-kernel立ち上げ

> 謎の自信により3B+で実行しようとしたが当然のようにbootで詰む。3Bを購入。uboot正常起動。

- u-boot自動化(envバイナリファイル作成)

  ``` sh
  u-boot> setenv tk_bootcmd 'fatload mmc 0 8000 tmonitor.bin; fatload mmc 0 4000 rominfo-rom.bin; fatload mmc 0 30000 kernel-rom.bin; go 8000'
  u-boot> setenv bootcmd 'run tk_bootcmd'
  u-boot> setenv bootdelay 2
  u-boot> saveenv
  ```

  コードつないで電源入れたらT-kernel立ち上がるようになった

## 実装

https://www.yokoweb.net/dokuwiki/develop/rtk_kernel/rtk_kernel-rpi/implement/start

## 参考GitHub

https://github.com/jr4qpv/rtk_kernel
