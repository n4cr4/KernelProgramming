# Kernel Build
ch2までを終わらせていないと進められないので注意．
- [ ] kernelのソースを落としている
- [ ] 設定を終わらせている
- [ ] `.config`はすでに用意できている

## 確認・実行
1. `make all`(こっちは使わない)
   `make help`したときにでてくる項目の内、先頭に`*`がついているものがターゲットになってビルドされる．
   `*`がついている項目としては、
   * `vmlinux`
     * カーネルイメージとかシンボル情報とかデバッグ情報とか．配布されるとPwnerがニッコリするやつ
   * modules
     * menuconfigでmにした項目がこれに該当する．`.ko`みたいなファイルとしてビルドされる．
   * `bzImage`
     * `vmlinux`より圧縮されて小さくなったカーネルイメージ．ブートローダーによって起動時にRAMに展開されるやつがこれ．

   がある．
2. `time make -j4`
   `time`はコマンドの実行時間を測ってるだけ．
   Kernelのビルドは負荷のかかる作業なので、並列処理をするためにタスクをspawnするが、vmでやっているので好き放題やられると色々こまる．そこで`-jn`(nには任意の数字)を入れて増やせるタスクに上限を設ける．
   `nproc`でコアの数を確認して2倍すると並列処理できるタスクの目安の値になるらしい．vmの設定でcpuのコアは2にしているので`-j4`を与える．

   **ビルドが通らない！！**
    ![what](img/buildfaild.png)

# Installing the kernel modules
ビルドが通ってないが一応書いてあることに目を通す．
`make menuconfig`でmに設定したやつは`.ko`ファイルとして存在することは上で触れた通り．なので、`vmlinux`や`bzImage`のなかに埋め込まれていない．
本ではVirtualBox supportなどの項目をモジュールにしたよねということで、それをさがしていた．
```
find . -name "*.ko" -ls | egrep -i "vbox|msdos|uio" | awk '{printf "%-40s %9d\n", $11, $7}'
```
`make`で`INSTALL_MOD_PATH`を指定しておくと任意のパスにモジュルールを置けるみたい．

# What's GRUB bootloader
VM上のGuestOSのマシンでBootloaderと戯れる章？

## おまけ
### 単語集
* Device Tree Blob
  * Blob = Binary Large OBject
    * データーベース管理とかでバイナリデータを格納するときにつかう(音声、圧縮ファイル、実行ファイル)
  * 組み込みシステム、BSPにおいてハードウェアの設定やリソース割当の表現に使用される
    * BSP = Board Support Package
      * ハードウェアの初期化と制御
      * ブートローダー
      * デバイスドライバ
      * ハードウェアごとに違う．組み込みボードに対するソフトウェアサポートパッケージ
      * だから、BSPはKernelを含む(もっと広範囲をさす言葉)