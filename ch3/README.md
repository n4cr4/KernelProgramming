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

### うまくいかない1
**ビルドが通らない！！**

![what](img/buildfaild.png)

#### 原因
VirtualBoxのGuestOSであるUbuntuと手元でビルドするために用意しているKernelのソースのバージョンが違うから．
`make menuconfig`する前に
```
lsmod > /tmp/lsmod.now
make LSMOD=/tmp/lsmod.now localmodconfig
```
を走らせているが、`make localmodconfig`はそのコマンドを走らせているローカルの環境を見て`.config`に反映していく．正確に説明するとローカルマシンでロードされているモジュールはビルドされるカーネルにもロードされるべき(使い勝手的な観点で)なので設定を揃えてくれる．

#### 解決策
簡単にいえばKernelのバージョンをVMのOSと手元のソースコードとで揃えられれば良い
* Ubuntuの18.04を使用する(本で使用しているバージョンにちゃんと揃える)
* Linux Kernel v6.2.16を使用する

後者を選んだ．なんでこのバージョンが出てくるのかについても軽く説明する．
```
suke@klmbox:~$ uname -r
6.2.0-32-generic
```
なので、6.2.0かなーと思うとこれは間違い．Ubuntuは独自で改造されたKernelなので本流でそれにあたるバージョンを得るためには以下を見る．
```
suke@klmbox:~$ cat /proc/version_signature
Ubuntu 6.2.0-32.32~22.04.1-generic 6.2.16
```
ここから上のバージョンを得る．
ch1-2でやったこととほぼ同様にしてソースコードを取得する．
`.config`の構成についてはローカルのマシンと同様の設定をベースにするので
```
cp /boot/config-$(uname -r) .config
```
で手元にコピーする．その上で`make menuconfig`して本の通りの設定を加えてビルドする．
```
make clean
make -j4
```

### うまくいかない2
**vmの容量がパンクした**

![what](img/nospace.png)

![what](img/nospace1.png)

流石に終わったと思った．でもまだ入れる保険がある．vmのディスク拡張はVirtualBoxとGuestOS(Ubuntu)両方で作業が必要．

#### 解決策
##### VirtualBox側での操作
使用しているマシンのストレージ名を把握しておく．共通のツールからメディアを選択．ハードディスクからさっきの名前と同じものを選択．下のバーを動かして適当なストレージサイズに拡張．

##### GuestOS側での操作
`sudo apt install gparted`をすることが紹介されている記事もあるが、今さっきパンクしたばかりなのでインストールなんかできない!(この世の終わり)

* パーティションの拡張
`sudo cfdisk /dev/sda`をして、溢れている`/dev/sda3`にResizeからFreeSpaceを割り当てる．Writeを選択して設定を書き込む．

* ファイルシステム上の拡張
`sudo resize2fs /dev/sda3`

![what](img/nospace2.png)

使用率が変わってるのでうまくいってる．足掻くもんだね．

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