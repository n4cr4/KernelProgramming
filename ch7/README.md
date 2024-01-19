# Understanding the VM split
VASは0x0から`high address`までといっていた．これは単にOSが32bitプロセッサ用なら$2^{32}$だし、64bitなら$2^{64}$というだけの話．簡単のために32bit前提で進む．
>1978年にIntelが開発したMPUの型番が「8086」であり、1989年に発売された「80486」まで、「○○86」という型番が使用されていました。これらの製品に使われている命令セットアーキテクチャ(ISA)の名称がx86であり、また、製品シリーズの名称でもあるのです。

厳密にはx86=32bitではない．

## Hello, world internal
VASはサンドボックスのように各々が独立している．Cの`printf()`のようなライブラリ関数はどのようにアクセスできるのだろうか．

elfファイルのなかに埋め込まれているローダーが必要なライブライを検出し、.text,.dataをプロセスのVASにmmap(system call)することで、ライブラリ関数が呼び出せる．

`ldd`したら実際の依存関係がわかる．

ch6でユーザースレッドはユーザーVASとカーネルVASを持ち、内部でシステムコールがあったらカーネルVASの出番のような書き方があった．実際にVASを**VM split**することで2者を共存させつつ利用することができる．

![textimg](image.png)

2:2 = u:kで分割された32bitのVASは上図の通り．~~kernel space addrっぽいなぁってのが`0xffff`なのもうなずける~~
分割されたVASについて、スレッドの実行レベルに応じてコンテキストスイッチが走る．

VM splitは32bitの場合、`CONFIG_VMSPLIT_[1|2|3]G`, `CONFIG_PAGE_OFFSET`によって調整できる(カーネルビルド時の設定)．しかし、64bitでは直接の調整ができない．

## VM split on 64bit
### 前提
- 64bitシステムは下位48bitをアドレッシングに利用している(64bitをフルに使いこなせるのは16,384ペタバイトのRAMを持ったPC、そんなものはない．．)
- `printf("%x")`で表示できる仮想アドレスは
  - ユーザースペースで実行するならUVA
  - カーネル内、カーネルモジュールとかで実行するならKVA
  - Virtual Address

![Alt text](image-1.png)
<!-- 48bit内のエントリについて適当に調べる？-->

MSB16bitに関しては符号拡張のようなノリで使われる．
- K-VASならbitを立てて
- U-VASなら落とす

つまり、
- KVAは`0xffff...`
- UVAは`0x0000...`

~~kernel space addrっぽいなぁってのがry~~

### 結局
64bitでのVM splitは高位、下位から128TBのセグメントで考える．
![Alt text](image-2.png)

真ん中は使われていない．

![Alt text](image-3.png)

アーキ別のVM splitは図の通り．
仮想アドレスなのでどうでもいいが、すべて真ん中が使われない形になる．

### おまけ
- KVAは非スワップメモリ
- UVAはスワップメモリ
- スワップメモリ = RAMが不足してるときにHDD等に移動してよいメモリ

![Alt text](image-4.png)

![a](image-5.png)

2つの図を並べてみると脳みそが若干混乱するのだが、、
- プロセスは(ユーザーモードなら)、Uスタックを1つ、Kスタックを1つ持つ
- UVASはプロセスごとに独立している．が、KVAS自体は共有している．
  - つまり、UVASの上に仮想的にくっついているKVASは実のとこは同一セグメントを見てる


# Examining the process VAS
# Examining the kernel segment
