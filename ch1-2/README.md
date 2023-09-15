# 環境構築と開発、config etc
## やったこと
* vm立ててkernelのbuildをする準備
  * Ubuntu22.04をvirtualboxで立てる
  * スペックはガタがきたらまた上げる
  * 現状4GBRAM,2Processors,25GBdisk
  * 結局1時間はかかる(vm立てるだけ)
  * ターミナル立ち上がらない謎現象
    * [LOCALEがおかしいだけ](https://www.demandosigno.study/entry/2023/07/02/000000)
    * 治ったけどなんで最初から`en_US.UTF-8`にしないのか謎
  * 一からだからsudoersとかからやらないと
  * aptでいろいろインストール
* ソースコードの取得
  * 前はgitからtreeごと取ってきたけれどすごい時間がかかった
  * 本は`5.4.0`で進めているのでひとまずこれを用意する
  * と思っていたけどリンクが死んでた
    * Tipsにだめなら`5.4.1`にすると良いみたいなことが書いてあったのでそれに従う
```
wget --https-only -O ~/Downloads/linux-5.4.1.tar.xz https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.4.1.tar.xz
```
```
tar xf [ダウンロードしたtar]
```
解凍したら適宜移動しとく．本に従って`LLKD_KSRC`は`export LLKD_KSRC=${HOME}/kernels/linux-5.4.1/`としている．

config,devなどは必要に応じて後で参照する．

### menuconfigでやったこと
* `CONFIG_LOCALVERSION` = `-11kd01`
  * 5.4.0をビルドするとしたら5.4.0-11kd01になるかも(ローカルな識別子)
* `CONFIG_IKCONFIG` = y
  * `.config`が見れるようになる
* `CONFIG_IKCONFIG_PROC` = y
  * `.config`をkernel実行時に内部から`/proc/config.gz`として見れるようにする
  * デバッグとかに後々使えるのかなー
* `CONFIG_PROFILING` = n
  * 性能データの収集、モニタリングに使えるがリソースを食うのでオフにしている
* `CONFIG_HAMRADIO` = n
  * 無線通信に関する機能を無効にしてドライバーを削除
* `CONFIG_VBOXGUEST` = m
  * ゲストアディションのモジュールの追加
    * 共有フォルダとかがわかりやすい
  * ちなみにこれはデフォルトでmになっていた
* `CONFIG_UIO` = m
  * デフォルトでm
  * ユーザスペースのアプリケーションがデバイスドライバにアクセスすることができる
  * デバイスへのアクセス制御うんぬんかんぬん
* `CONFIG_UIO_PDRV_GENIRQ` = m
  * デフォルトでm
  * PDRV = Providor DRiVer
  * GETIRQ = Generic IRQ(Interrupt Request = 割り込み)
  * ユーザスペースアプリケーションがデバイスからの割り込み処理を定義するのを助ける
  * カーネルのデバイスとの連携がしやすくなる
* `CONFIG_MSDOS_FS` = m
  * デフォルトでm
  * MSDOS = Microsoft Disk Operationg System
    * 古いファイルシステム
    * FATの前身
  * FAT = File Allocation  Table
    * 汎用性の高いファイルシステム
    * リムーバブルメディアのファイルシステムがこれなのかな
    * Win, Mac, LinuxでOK
  * FATフォーマットのデバイスを扱えるようになる
* `CONFIG_SECURITY` = n
  * Kernel LSMs(Linux Security Modules)をオフにする
  * SELinuxとかもこれらしい
  * デベロップかつ手元で遊ぶだけなので今回は無効にする
  * プロダクトでは有効にしよう
* `CONFIG_DEBUG_STACK_USAGE` = y
  * スタックのデバッグとかに使える
    * SBOFとか