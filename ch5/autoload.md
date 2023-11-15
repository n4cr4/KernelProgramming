## Auto-start execution and kernel module extentions
[MITRE ATT&CK](https://attack.mitre.org/techniques/T1547/006/)のなかでテクニックとして紹介されている．永続化や権限昇格に関わってくる．
LKMを悪意を持って利用する場合`Rootkit`が大きなトピックとなる．

### Rootkitとは
遠隔操作のためのソフトウェアをまとめたパッケージツールのこと．攻撃者がPCに侵入した痕跡を消したり、プロセスの隠蔽ができる．C&Cサーバからの命令を受けてさらに活動することも考えられる．

### LKMとの関連性は
Linux kernelの関数をフックしてパケットをモニタリングしている．[`ip_rcv()`](https://elixir.bootlin.com/linux/latest/source/net/ipv4/ip_input.c#L560)が例として取り上げられていた．また、ファイルやディレクトリの隠蔽のために[`fillonedir()`](https://elixir.bootlin.com/linux/latest/source/fs/readdir.c#L179)をフックし、`ENOENT`(Error No ENTry)を返すことをしていた．

プロセス、通信、ファイルの隠蔽にはKernelの特権を利用する必要があるので関連性が非常にある．

### Autoloadの実現
LKMの集合体だから教科書で勉強したやり方が利用されているのかと思ったら`/lib/udev/rules.d/`に自動起動のためのルールが生成されるのみだった．(systemdのとりまき)