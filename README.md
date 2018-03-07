# FreeRTOS Ver10.0.0 DemoProject for Renesas RX65N (CS+ CC-RX)

　このデモはFreeRTOSv10.0.0をRenesas RX65Nマイコン用に移植したものです。評価ボード上の2つのLEDを2タスクでLチカするだけの簡単なサンプルです。
 
　最近ルネサスから発売された[Target Board for RX family](https://www.renesas.com/ja-jp/products/software-tools/boards-and-kits/cpu-mpu-boards/rx-family-target-board.html)(RX65N)を手に入れたのでFreeRTOSを動かしてみました。
秋葉原の[マルツ](https://www.marutsu.co.jp/pc/i/953239/)で2,980円（税抜き）で売られています。デバッガのE2Liteがボード上に搭載されてこの値段は安いと思います。

　なお、ド素人の学生が趣味で作成したものですので、もしご利用の際はくれぐれも自己責任でお願いします。

文字コードは __UTF-8__ を使用しています。

## 動作環境
FreeRTOS:v10.0.0 (RX600 RXv2)  
開発環境：CS+forCC V6.01.00  
コンパイラ：CC-RX V2.08.00 (C99)  
CPUボード：TARGET BOARD for RX65N (RTK5RX65N0C00000BR)  
CPU(ボード上)：R5F565NEDDFP (100-pin LFQFP,120MHz,RAM 640KB,ROM 2MB+32KB)  
E2エミュレータLite(ボード上)：この環境ではデバッグに使用できるようです。ただ、RFPによるプログラム書き込みは今はできません。


## サンプルコード内容
CPUボード上のLED0(PD6)を1Hz、LED1(PD7)を5Hzで点滅させる2つのタスクを動かします。

　クロック発生回路とポート初期化をスマートコンフィグレータで設定しています。 
FreeRTOSではカーネルタイマにCMT0,コンテキストスイッチにソフトウェア割込み(SWINT)を使用しているので、その周辺機能は使用しないでください。

## ファイル構成
自力でプロジェクトを作るためのメモ。サンプルコードの`main.c`と`ApplicationHook.c`は自分が用意しています。

  1. CS+でプロジェクト新規作成する。ここで自分はビルド設定(CC-RXのプロパティ)で文字コードをUTF-8に変更してしまうが、SHIFT-JISのままでいけるかどうかは未検証。ちなみにUTF-8の変更箇所はコンパイル・オプションで2か所、アセンブル・オプションで1か所である。
  1. スマートコンフィグレータで周辺機能の設定を必要があれば設定する。ただし、コンペアマッチタイマ(CMT0)、ソフトウェア割込み(SWINT)はFreeRTOS側が使うので何もしないこと。  
  1. サンプルコードのFreeRTOSフォルダ以下をそのままプロジェクトフォルダにコピーしてプロジェクトに登録。
  1. サンプルコードの`ApplicationHook.c`の中身はフック関数類を定義してあります。これらはユーザー側で定義しないとビルドが通らないので、最小限のものだけ定義してあります。取りあえずこれもファイルごとコピーしてプロジェクトに登録。必要に応じて自力で関数を追加すること。
  1. サンプルコードのmain.cにある`vApplicationSetupTimerInterrupt()`はカーネルタイマの初期設定で必ず必要なので関数を新規の`main.c`にコピーするなり新たにソースファイル作るなりする。適当にどっかにあればたぶん大丈夫。
  1. `/FreeRTOS/FreeRTOSConfig.h`を目的に合わせて設定する。
  1. 各ソースファイルで`FreeRTOS.h, task.h, queue.h`など適切なヘッダをインクルードする。
  1. ビルドする。
  1. 任意でデバッグなどなど。

## 注意点
* ソースコードの文字エンコードに __UTF-8__ を使用するため、ビルド設定をShift-JISからUTF-8に変更してあります。
* FreeRTOSのメモリ管理ファイルは「heap_1.c」を使用しています。目的に応じて変更してください。`/FreeRTOS/portable/heap_1.c`に置いてあります。
* iodefine.hの不具合について  
プロジェクト生成直後はルートにRX65N用レジスタ定義ファイルが作成されます。しかし、スマートコンフィグレータを使用すると 
`/src/smc_gen/r_bsp/mcu/rx65n/register_access/iodefine.h`   
に新たに作られ、ルートにある方のファイルは登録が解除されます（ファイル削除はされない）。しかし、新たに作られた方はなぜかRX651用なので（バグ？）他のソースでiodefine.hをインクルードすると多重インクルードエラーが出ることがあります。対策として、
    1. インクルードパスをフルパスで記述する。
    1. ルートにあるRX65N用のファイルをカットし、RX651用のファイルにペーストする。ただし、スマートコンフィグレータでコード生成するたび元に戻ってしまう。
    1. RX651の機能のみしか使わないのであれば、ルートにあるRX65Nのファイルを最初に削除するだけ。

あたりを試してください。
また、`/no_used/iodefine_rx65n/iodefine.h`にRX65N用のレジスタ定義ファイルを置いてあります（最新とは限りません）。
