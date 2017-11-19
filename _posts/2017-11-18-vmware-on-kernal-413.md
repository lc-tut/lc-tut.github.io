---
title: VMware Workstation Pro 14がLinux Kernel4.13で動くようにする
tags: Linux_Tips
categories: poem
---


# VMware Workstation Pro 14がLinux Kernel4.13で動くようにする

## Linux Kernal 4.13がやっと動いた
私のマシンでは長らくLinux kernel 4.4系しか動かなかったのですが、Oracle VirtualBoxの関連コンポーネントを根こそぎ削除したところ4.13系が動作しました。ですが...

**VMware Workstation Pro 14上の仮想マシンが起動しなくなりました**

具体的には次のようなエラーメッセージが表示されます。

![unable to reserve memory](/images/vmware/img01.png)

![could not lock memory](/images/vmware/img02.png)

![failed to switch to 64bit mode](/images/vmware/img03.png)

VMware Workstation がkernel4.13に対応していないことが原因のようです。Kernelに対する命令である`global_page_state`が4.13以降で`global_zone_page_state`に変わったことが原因です。有志の方がこれに対応するパッチを作成していたので、これを用いて対処していきます。

次のコマンドでパッチを当てることができます。

        sudo -i
        cd /tmp
        cp /usr/lib/vmware/modules/source/vmmon.tar .
        tar xf vmmon.tar
        rm vmmon.tar
        wget https://raw.githubusercontent.com/mkubecek/vmware-host-modules/fadedd9c8a4dd23f74da2b448572df95666dfe12/vmmon-only/linux/hostif.c
        mv -f hostif.c vmmon-only/linux/hostif.c 
        tar cf vmmon.tar vmmon-only
        rm -fr vmmon-only
        mv -f vmmon.tar /usr/lib/vmware/modules/source/vmmon.tar 
        vmware-modconfig --console --install-all

コマンド実行後に再起動を行うことで、VMware Workstationが正常に動作します。

---
参考文献

[Arch Linux User Forums VMWare Workstation 14.0 - not enough physical memory](https://bbs.archlinux.org/viewtopic.php?id=230487)
