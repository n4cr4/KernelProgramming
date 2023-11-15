<!-- # Passing parameters to a kernel

# Floating point not allowed in the kernel -->

# Auto-loading modules on system boot
```
install:
  make -C $(KDIR) M=$(PWD) modules_install
```
このセクションはMakefile内に記述されるので、`make install`に対応するものである．(rootでないとうまくいかないかも)
デモ用のMakefileでは`depmod`が内部で呼ばれていて、モジュールの依存関係を解決してくれる．

LKMのオートロードのために`/etc/modules-load.d/<module_name>.conf`を作成する方法がある．また、オートロードの際にLKMに引数を渡したい場合に`/etc/modprobe.d/<module_name>.conf`に以下のような記述をすればよい．
```
options <module_name> <param_key>=<val>
```
実際に`/etc/modprobe.d/alsa-base.conf`を確認してみると例えば以下のような記述が確認できる．
```
options snd-usb-audio index=-2
options snd-usb-caiaq index=-2
options snd-usb-ua101 index=-2
```

LKMがinstallされると`/lib/modules/$(uname -r)/extra/`のような場所にオブジェクトファイルが配置される．

逆にオートロードされたくないLKMが存在する場合、ブラックリストを定義することで実現できる．

## insmod vs modprobe
modprobeはモジュールの依存関係を考慮して必要とあれば事前に該当のものを含めてロードしてくれる．depファイル等の話とつながるはず．一方insmodは純粋に指定されたモジュールのみをロードする．

## Auto-loading and security ??
[悪いこと](./autoload.md)ができるか検証する．周辺知識や分析をできるとこまで．

# Kernel modules and security
## Proc fs tunables affecting the system log
* [dmesg_restrict](https://man7.org/linux/man-pages/man5/proc.5.html#/proc/sys/kernel/dmesg_restrict)
  * kernelのsyslogを見れるユーザを定める．`dmesg_restrict=1`だと特権ユーザのみ見ることができる．
  * デフォルトは0でなにも制限しない
  * `CAP_SYSLOG`がいる
* [kptr_restrict](https://man7.org/linux/man-pages/man5/proc.5.html#/proc/sys/kernel/kptr_restrict)
  * `CAP_SYSLOG`を持たないユーザにはカーネルポインタを見せない(単に0を表示するのみ)
  *  `%pK`というフォーマット文字列を使用する．
  *  `kptr_restrict=0`は制限なし
  *  `kptr_restrict=1`は`CAP_SYSLOG`を持っているuserのみ
  *  `kptr_restrict=2`は誰にもアドレスを表示しない
  *  アドレスリークを防ぎたい．

アドレスリークを防ぐ取り組みとしてこんな[スクリプト](https://elixir.bootlin.com/linux/v6.2/source/scripts/leaking_addresses.pl)が存在する．

## The cryptographic signing of kernel modules
なんらかの形でroot権限を取得できたとき、攻撃者はrootkitと呼ばれるバックドア作成やキーロギングに使用するカーネルモジュールのパッケージをインストールし、さらなる攻撃を試みるはずである．
これに抗うため、暗号署名していないカーネルモジュールのインストールを許可しないことが考えられる．この仕組みはkernel versiong>=3.7で有効．

## Disabling kernel modules altogether
`modules_disabled`を設定することで、モジュールのロード自体を制限することができる．また、`CONFIG_MODULES`をoffにすることで、ビルド時に静的に必要なものを含んでいる前提になる．両者とも動的なkernel変更を許さないため極端であることは間違いないのだが、数々の防御機構も回避方法があるのでこれもひとつのやり方か．

<!-- # Contributing to the mainline kernel -->